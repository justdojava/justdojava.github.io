---
layout: post
categories: springboot
title: 手把手教你搞定菜单权限设计，精确到按钮级别，建议收藏
tags: 
  - 炸鸡可乐
---

菜单权限管理，一直都是后端管理系统必不可少的一个模块，今天我们就一起来瞅瞅，如何精准的控制到按钮。

<!--more-->

### 一、介绍
在实际的项目开发过程中，**菜单权限功能**可以说是后端管理系统中必不可少的一个环节，根据业务的复杂度，**设计的时候可深可浅，但无论怎么变化，设计的思路基本都是围绕着用户、角色、菜单进行相应的扩展**。

今天小编就和大家一起来讨论一下，怎么设计一套可以**精确到按钮级别**的菜单权限功能，废话不多说，直接开撸！

### 二、数据库设计
先来看一下，用户、角色、菜单表对应的ER图，如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-menu-auth/62177d70c01093cf3d7985b637166a93.jpg)

其中，**用户和角色是多对多的关系**，**角色与菜单也是多对多的关系**，**用户通过角色来关联到菜单**，当然也有的业务系统菜单权限模型，是可以直接通过用户关联到菜单，对菜单权限可以直接控制到用户级别，不过这个都不是问题，这个也可以进行扩展。

对于用户、角色表比较简单，下面，我们重点来看看菜单表的设计，如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-menu-auth/3b4d3f17084ee42f5ea37877ec377998.jpg)

可以看到，整个菜单表就是一个树型结构，**关键字段说明**：

* menu_code：菜单编码，用于后端权限控制
* parent_id：菜单父节点ID，方便递归遍历菜单
* node_type：节点类型，可以是文件夹、页面或者按钮类型
* link_url：页面对应的地址，如果是文件夹或者按钮类型，可以为空
* level：菜单树的层次，以便于查询指定层级的菜单
* path：树id的路径，主要用于存放从根节点到当前树的父节点的路径，逗号分隔，想要找父节点会特别快

为了后面方便开发，我们先创建一个名为`menu_auth_db`的数据库，初始脚本如下：
```sql
CREATE DATABASE IF NOT EXISTS menu_auth_db default charset utf8mb4 COLLATE utf8mb4_unicode_ci;

CREATE TABLE menu_auth_db.tb_user (
  id bigint(20) unsigned NOT NULL COMMENT '消息给过来的ID',
  mobile varchar(20) NOT NULL DEFAULT '' COMMENT '手机号',
  name varchar(100) NOT NULL DEFAULT '' COMMENT '姓名',
  password varchar(128) NOT NULL DEFAULT '' COMMENT '密码',
  is_delete tinyint(4) NOT NULL DEFAULT '0' COMMENT '是否删除 1：已删除；0：未删除',
  PRIMARY KEY (id),
  KEY idx_name (name) USING BTREE,
  KEY idx_mobile (mobile) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='用户表';

CREATE TABLE menu_auth_db.tb_user_role (
  id bigint(20) unsigned NOT NULL COMMENT '主键',
  user_id bigint(20) NOT NULL COMMENT '用户ID',
  role_id bigint(20) NOT NULL COMMENT '角色ID',
  PRIMARY KEY (id),
  KEY idx_user_id (user_id) USING BTREE,
  KEY idx_role_id (role_id) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='用户角色表';

CREATE TABLE menu_auth_db.tb_role (
  id bigint(20) unsigned NOT NULL COMMENT '主键',
  code varchar(100) NOT NULL DEFAULT '' COMMENT '编码',
  name varchar(100) NOT NULL DEFAULT '' COMMENT '名称',
  is_delete tinyint(4) NOT NULL DEFAULT '0' COMMENT '是否删除 1：已删除；0：未删除',
  PRIMARY KEY (id),
  KEY idx_code (code) USING BTREE,
  KEY idx_name (name) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='角色表';


CREATE TABLE menu_auth_db.tb_role_menu (
  id bigint(20) unsigned NOT NULL COMMENT '主键',
  role_id bigint(20) NOT NULL COMMENT '角色ID',
  menu_id bigint(20) NOT NULL COMMENT '菜单ID',
  PRIMARY KEY (id),
  KEY idx_role_id (role_id) USING BTREE,
  KEY idx_menu_id (menu_id) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='角色菜单关系表';


CREATE TABLE menu_auth_db.tb_menu (
  id bigint(20) NOT NULL COMMENT '主键',
  name varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '名称',
  menu_code varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '菜单编码',
  parent_id bigint(20) DEFAULT NULL COMMENT '父节点',
  node_type tinyint(4) NOT NULL DEFAULT '1' COMMENT '节点类型，1文件夹，2页面，3按钮',
  icon_url varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT '' COMMENT '图标地址',
  sort int(11) NOT NULL DEFAULT '1' COMMENT '排序号',
  link_url varchar(500) COLLATE utf8mb4_unicode_ci DEFAULT '' COMMENT '页面对应的地址',
  level int(11) NOT NULL DEFAULT '0' COMMENT '层次',
  path varchar(2500) COLLATE utf8mb4_unicode_ci DEFAULT '' COMMENT '树id的路径 整个层次上的路径id，逗号分隔，想要找父节点特别快',
  is_delete tinyint(4) NOT NULL DEFAULT '0' COMMENT '是否删除 1：已删除；0：未删除',
  PRIMARY KEY (id) USING BTREE,
  KEY idx_parent_id (parent_id) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='菜单表';
```

### 三、后端开发
菜单权限模块的数据库设计，一般5张表就可以搞定，真正有点复杂的地方在于数据的写入和渲染，当然如果老板突然让你来开发一套菜单权限系统，我们也没必要慌张，下面，我们一起来看看后端应该如何开发。

#### 3.1、创建项目
为了方便快捷，小编我采用的是`springboot+mybatisPlus`组件来快速开发，直接利用`mybatisPlus`官方提供的快速生成代码的`demo`，一键生成所需的`dao`、`service`、`web`层的代码，结果如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-menu-auth/f53b04498a52c0477a441fd429d323f6.jpg)

#### 3.2、编写菜单添加服务
```java
@Override
public void addMenu(Menu menu) {
    //如果插入的当前节点为根节点，parentId指定为0
    if(menu.getParentId().longValue() == 0){
        menu.setLevel(1);//根节点层级为1
        menu.setPath(null);//根节点路径为空
    }else{
        Menu parentMenu = baseMapper.selectById(menu.getParentId());
        if(parentMenu == null){
            throw new CommonException("未查询到对应的父节点");
        }
        menu.setLevel(parentMenu.getLevel().intValue() + 1);
        if(StringUtils.isNotEmpty(parentMenu.getPath())){
            menu.setPath(parentMenu.getPath() + "," + parentMenu.getId());
        }else{
            menu.setPath(parentMenu.getId().toString());
        }
    }
    //可以使用雪花算法，生成ID
    menu.setId(System.currentTimeMillis());
    super.save(menu);
}
```
新增菜单比较简单，直接将数据插入即可，需要注意的地方是`parent_id`、`level`、`path`，这三个字段的写入，如果新建的是根节点，默认`parent_id`为`0`，方便后续递归遍历。
#### 3.3、编写菜单后端查询服务

* 新建一个菜单视图实体类

```java
@Data
@EqualsAndHashCode(callSuper = false)
@Accessors(chain = true)
public class MenuVo implements Serializable {

    private static final long serialVersionUID = -4559267810907997111L;

    /**
     * 主键
     */
    private Long id;

    /**
     * 名称
     */
    private String name;

    /**
     * 菜单编码
     */
    private String menuCode;

    /**
     * 父节点
     */
    private Long parentId;

    /**
     * 节点类型，1文件夹，2页面，3按钮
     */
    private Integer nodeType;

    /**
     * 图标地址
     */
    private String iconUrl;

    /**
     * 排序号
     */
    private Integer sort;

    /**
     * 页面对应的地址
     */
    private String linkUrl;

    /**
     * 层次
     */
    private Integer level;

    /**
     * 树id的路径 整个层次上的路径id，逗号分隔，想要找父节点特别快
     */
    private String path;

    /**
     * 子菜单集合
     */
    List<MenuVo> childMenu;
}
```

* 编写菜单查询服务，使用递归重新封装菜单视图

```java
@Override
public List<MenuVo> queryMenuTree() {
    Wrapper queryObj = new QueryWrapper<>().orderByAsc("level","sort");
    List<Menu> allMenu = super.list(queryObj);
	// 0L：表示根节点的父ID
    List<MenuVo> resultList = transferMenuVo(allMenu, 0L);
    return resultList;
}
```
```java
/**
 * 封装菜单视图
 * @param allMenu
 * @param parentId
 * @return
 */
private List<MenuVo> transferMenuVo(List<Menu> allMenu, Long parentId){
    List<MenuVo> resultList = new ArrayList<>();
    if(!CollectionUtils.isEmpty(allMenu)){
        for (Menu source : allMenu) {
            if(parentId.longValue() == source.getParentId().longValue()){
                MenuVo menuVo = new MenuVo();
                BeanUtils.copyProperties(source, menuVo);
                //递归查询子菜单，并封装信息
                List<MenuVo> childList = transferMenuVo(allMenu, source.getId());
                if(!CollectionUtils.isEmpty(childList)){
                    menuVo.setChildMenu(childList);
                }
                resultList.add(menuVo);
            }
        }
    }
    return resultList;
}
```

* 编写一个菜单树查询接口，如下：

```java
@RestController
@RequestMapping("/menu")
public class MenuController {

    @Autowired
    private MenuService menuService;

    @PostMapping(value = "/queryMenuTree")
    public List<MenuVo> queryTreeMenu(){
        return menuService.queryMenuTree();
    }
}
```
为了便于演示，我们先初始化7条数据，如下图：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-menu-auth/2f36c3517bb7ac024fa410e1fdd03c5b.jpg)

其中最后三条是按钮类型，等下会用于**后端权限控制**，接口查询结果如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-menu-auth/22900c18c9b4943c83fa13bd82ee4b1c.jpg)

这个服务是针对后端管理界面查询的，会将所有的菜单全部查询出来以便于进行管理，展示结果类似如下图：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-menu-auth/937881425b821f492c27639aa00ca87d.jpg)

这个图片截图于小编正在开发的一个项目，**内容可能不一致，但是数据结构基本都是一致的**。

#### 3.4、编写用户菜单权限查询服务
在上面，我们介绍到了用户通过角色来关联菜单，因此，很容易想到，流程如下：

* 第一步：先通过用户查询到对应的角色；
* 第二步：然后再通过角色查询到对应的菜单；
* 第三步：最后将菜单查询出来之后进行渲染；

实现过程相比菜单查询服务多了前2个步骤，过程如下：

```java
@Override
public List<MenuVo> queryMenus(Long userId) {
    //1、先查询当前用户对应的角色
    Wrapper queryUserRoleObj = new QueryWrapper<>().eq("user_id", userId);
    List<UserRole> userRoles = userRoleService.list(queryUserRoleObj);
    if(!CollectionUtils.isEmpty(userRoles)){
        //2、通过角色查询菜单（默认取第一个角色）
        Wrapper queryRoleMenuObj = new QueryWrapper<>().eq("role_id", userRoles.get(0).getRoleId());
        List<RoleMenu> roleMenus = roleMenuService.list(queryRoleMenuObj);
        if(!CollectionUtils.isEmpty(roleMenus)){
            Set<Long> menuIds = new HashSet<>();
            for (RoleMenu roleMenu : roleMenus) {
                menuIds.add(roleMenu.getMenuId());
            }
            //查询对应的菜单
            Wrapper queryMenuObj = new QueryWrapper<>().in("id", new ArrayList<>(menuIds));
            List<Menu> menus = super.list(queryMenuObj);
            if(!CollectionUtils.isEmpty(menus)){
                //将菜单下对应的父节点也一并全部查询出来
                Set<Long> allMenuIds = new HashSet<>();
                for (Menu menu : menus) {
                    allMenuIds.add(menu.getId());
                    if(StringUtils.isNotEmpty(menu.getPath())){
                        String[] pathIds = StringUtils.split(",", menu.getPath());
                        for (String pathId : pathIds) {
                            allMenuIds.add(Long.valueOf(pathId));
                        }
                    }
                }
                //3、查询对应的所有菜单,并进行封装展示
                List<Menu> allMenus = super.list(new QueryWrapper<Menu>().in("id", new ArrayList<>(allMenuIds)));
                List<MenuVo> resultList = transferMenuVo(allMenus, 0L);
                return resultList;
            }
        }

    }
    return null;
}
```

* 编写一个用户菜单查询接口，如下：

```java
@PostMapping(value = "/queryMenus")
public List<MenuVo> queryMenus(Long userId){
	//查询当前用户下的菜单权限
    return menuService.queryMenus(userId);
}
```

有的同学，可能觉得没必要存放`path`这个字段，的确在某些场景下不需要。

为什么要存放这个字段呢？

小编在跟前端进行对接的时候，发现这么一个问题，有些前端的树型组件，在勾选子集的时候，不会将对应的父ID传给后端，例如，我在勾选【列表查询】的时候，前端无法将父节点【菜单管理】ID也传给后端，**所有后端实际存放的是一个尾节点**，需要一个字段`path`，来存放节点对应的父节点路径。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-menu-auth/c4996abe19dc5f0b4f5a8885a1451529.jpg)

其实，前端也可以传，只不过需要修改组件的属性，前端修改完成之后，树型组件就无法全选，不满足业务需求。

所以，有些时候得根据实际得情况来进行取舍。

#### 3.5、编写后端权限控制
后端进行权限控制目标，主要是为了防止无权限的用户，进行接口请求查询。

**其中菜单编码`menuCode`就是一个前、后端联系的桥梁，细心的你会发现，所有后端的接口，与前端对应的都是按钮操作，所以我们可以以按钮为基准，实现前后端双向控制**。

以【角色管理-查询】这个为例，前端可以通过菜单编码实现是否展示这个查询按钮，后端可以通过菜单编码来判断，当前用户是否具备请求接口的权限。

以后端为例，我们只需编写一个权限注解和代理拦截器即可！

* 编写一个权限注解

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface CheckPermissions {

    String value() default "";
}
```

* 编写一个代理拦截器，拦截有`@CheckPermissions`注解的方法

```java
@Aspect
@Component
public class CheckPermissionsAspect {

    @Autowired
    private MenuMapper menuMapper;

    @Pointcut("@annotation(com.company.project.core.annotation.CheckPermissions)")
    public void checkPermissions() {}

    @Before("checkPermissions()")
    public void doBefore(JoinPoint joinPoint) throws Throwable {
        Long userId = null;
        Object[] args = joinPoint.getArgs();
        Object parobj = args[0];
        //用户请求参数实体类中的用户ID
        if(!Objects.isNull(parobj)){
            Class userCla = parobj.getClass();
            Field field = userCla.getDeclaredField("userId");
            field.setAccessible(true);
            userId = (Long) field.get(parobj);
        }
        if(!Objects.isNull(userId)){
            //获取方法上有CheckPermissions注解的参数
            Class clazz = joinPoint.getTarget().getClass();
            String methodName = joinPoint.getSignature().getName();
            Class[] parameterTypes = ((MethodSignature)joinPoint.getSignature()).getMethod().getParameterTypes();
            Method method = clazz.getMethod(methodName, parameterTypes);
            if(method.getAnnotation(CheckPermissions.class) != null){
                CheckPermissions annotation = method.getAnnotation(CheckPermissions.class);
                String menuCode = annotation.value();
                if (StringUtils.isNotBlank(menuCode)) {
                    //通过用户ID、菜单编码查询是否有关联
                    int count = menuMapper.selectAuthByUserIdAndMenuCode(userId, menuCode);
                    if(count == 0){
                        throw new CommonException("接口无访问权限");
                    }
                }
            }
        }
    }
}
```

* 我们以【角色管理-查询】为例，先新建一个请求实体类`RoleDto`，添加用户ID属性

```java
@Data
@EqualsAndHashCode(callSuper = false)
@Accessors(chain = true)
public class RoleDto extends Role {

	//添加用户ID
    private Long userId;
}
```

* 在需要的接口上，添加`@CheckPermissions`注解，增加权限控制

```java
@RestController
@RequestMapping("/role")
public class RoleController {

    private RoleService roleService;

    @CheckPermissions(value="roleMgr:list")
    @PostMapping(value = "/queryRole")
    public List<Role> queryRole(RoleDto roleDto){
        return roleService.list();
    }

    @CheckPermissions(value="roleMgr:add")
    @PostMapping(value = "/addRole")
    public void addRole(RoleDto roleDto){
        roleService.add(roleDto);
    }

    @CheckPermissions(value="roleMgr:delete")
    @PostMapping(value = "/deleteRole")
    public void deleteRole(RoleDto roleDto){
        roleService.delete(roleDto);
    }
}
```

依次类推，当我们想对某个接口进行权限控制的时候，只需要添加一个注解`@CheckPermissions`，并填写对应的菜单编码即可！
### 四、用户权限测试
我们先初始化一个用户【张三】，然后给他分配一个角色【访客人员】，同时给这个角色分配一下2个菜单权限【系统配置】、【用户管理】，等会用于权限测试。

初始内容如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-menu-auth/b0e5589380030be7fc4bc84a196076f2.jpg)

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-menu-auth/08af7a6f7fb7bbe6456ca186f47d4765.jpg)

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-menu-auth/9ca2c9f5ddda8d82ddcf0a1e1441eb73.jpg)

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-menu-auth/f2a3ee5e2ac9fcedb491b083dd31fb96.jpg)

数据初始化完成之后，我们来启动项目，传入用户【张三】的ID，查询用户具备的菜单权限，结果如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-menu-auth/98e7b0add65f6353451c590916e5402a.jpg)

查询结果，用户【张三】有两个菜单权限！

接着，我们来验证一下，用户【张三】是否有角色查询权限，请求角色查询接口如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-menu-auth/f9538f9dd14d4707ad26f2b74c6f232b.jpg)

因为没有配置角色查询接口，所以无权访问！

### 五、总结
整片内容，只介绍了后端关键的服务实现过程，可能也有遗漏的地方，欢迎网友点评、吐槽！