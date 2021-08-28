---
layout: post
categories: Java
title: Activity 工作流中的表，原来表示的是这些
tagline: by 懿
tags: 
  - 懿
---

前几天，阿粉给大家说了关于 Activiti 的使用，后台就有好友私信阿粉说，这些表都不知道是什么意思，不行呀，看不明白呀，于是阿粉就打算再这次给大家讲一下关于 Activiti 的这些表中的字段都是表示的什么意思。

<--more-->

#### Activiti 的表

| 表 |  | 说明 |
| :-----| ----: | :----: |
| act_ge_bytearray |  | 通用数据 |
| act_ge_property |  | 流程引擎数据 |
| act_hi_actinst |  | 历史节点表 |
| act_hi_attachment |  | 历史附件表 |
| act_hi_comment |  | 历史意见表 |
| act_hi_detail |  | 历史详情 |
| act_hi_identitylink |  | 历史流程人员 |
| act_hi_procinst |  | 历史流程实例 |
| act_hi_taskinst |  | 历史任务 |
| act_hi_varinst |  | 历史变量 |
| act_id_group |  | 用户信息组 |
| act_id_info |  | 用户信息详情 |
| act_id_membership |  | 组和对应信息关联表 |
| act_id_user |  | 用户信息表 |
| act_procdef_info |  | 流程定义信息 |
| act_re_deployment |  | 部署信息 |
| act_re_model |  | 流程设计模型 |
| act_re_procdef |  | 流程定义数据 |
| act_ru_event_subscr |  | 信息监听 |
| act_ru_execution |  | 运行时流程执行数据 |
| act_ru_identitylink |  | 运行时节点人员数据信息 |
| act_ru_job |  | 定时任务数据 |
| act_ru_task |  | 运行时任务节点 |
| act_ru_variable |  | 流程变量数据 |

阿粉把之前的表给大家都拿出来了，然后我们一一来看这些字段都是些什么意思。在说这个Activiti的表结构的意思的时候，我们肯定首先要知道 Activiti 的生命周期，生命周期要经过的步骤如下：

1.流程部署 --->  2.启动流程实例 --- > 3.执行流程对象(一个流程实例包含多执行对象) ---> 4.完成整个流程

而我们要想了解这个流程，就得先从表结构开始了，我们来看看面所有的表结构吧。

大家如果对表结构没有任何的兴趣，那就往后滑，看后面的关于 Activiti 的流程的内容。

##### 1.act_ge_bytearray 通用数据，二进制数据表

保存流程定义图片和xml、Serializable(序列化)的变量,即保存所有二进制数据，特别注意类路径部署时候，不要把svn等隐藏文件或者其他与流程无关的文件也一起部署到该表中，会造成一些错误（可能导致流程定义无法删除）。 

**字段说明**

- ID_ : 主键ID,也是主键唯一索引

- REV_: Version(版本) 乐观锁

- NAME_: 部署的文件名称，如：mail.bpmn、mail.png 、mail.bpmn20.xml

- DEPLOYMENT_ID_: 部署表ID

- BYTES_: 部署文件

- GENERATED_: 是否是引擎生成,0为用户生成 1为Activity生成

在这里我们就要注意到 REV_ 这个乐观锁了，乐观锁每次在执行数据的修改操作时，都会带上一个版本号，一旦版本号和数据的版本号一致就可以执行修改操作并对版本号执行+1操作，否则就执行失败。因为每次操作的版本号都会随之增加，所以不会出现ABA问题，因为版本号只会增加不会减少。除了version以外，还可以使用时间戳，因为时间戳天然具有顺序递增性。好像又开始偏题了，我们回归正题，继续看表。

![](http://www.justdojava.com/assets/images/2019/java/image_yi/2021/06-11/1.jpg)

##### 2.act_ge_property 流程引擎数据表

- NAME_: 属性名称，也是主键

- VALUE_: 资源

- REV_: 乐观锁

- next.dbid 当 Activiti 使用DbIdGenerator来生成主键时，用来表示Id块的起始值；Id块就是Activiti产生主键时，Id的取值范围，从next.dbid ~ next.dbid+idBlockSize-1 ，默认idBlockSize = 2500 

- schema.version 表示数据结构版本

- schema.history 表示数据表结构的更新历史

这里面的数据一般情况下是这几个内容，标识的实际上相当于是 Activiti 的版本的一些相关的信息。

##### 3.act_hi_actinst 历史节点表

这个表实际上就是表示的都是历史活动信息，流程流转过的所有节点的记录都在这个表中，但是他是记录的**所有**节点信息，而在 taskinst 只记录 usertask 内容

- ID_: 主键ID

- PROC_DEF_ID_: 流程定义ID

- PROC_INST_ID_: 流程实例ID

- EXECUTION_ID_: 执行实例ID

- ACT_ID_: 节点定义ID

- TASK_ID_: 任务实例ID 其他节点类型实例ID在这里为空

- CALL_PROC_INST_ID_: 调用外部流程的流程实例ID

- ACT_NAME_: 节点定义名称

- ACT_TYPE_: 节点类型，如startEvent、userTask

- ASSIGNEE_: 节点签收人

- START_TIME_: 开始时间

- END_TIME_: 结束时间

- DURATION_: 耗时

##### 4.act_hi_attachment 历史附件表

- ID_: 主键ID

- REV_: 乐观锁

- USER_ID_: 用户ID

- NAME_: 附件名称

- DESCRIPTION_: 描述信息

- TYPE_: 附件类型

- TASK_ID_: 节点实例ID

- PROC_INST_ID_: 流程实例ID

- URL_: 附件地址

- CONTENT_ID_: ACT_GE_BYTEARRAY的ID 二进制数据表的ID(对应关系)

##### 5.act_hi_comment 历史意见表

- ID_: 主键ID

- TYPE_: 类型：event（事件）comment（意见）

- TIME_: 填写时间

- USER_ID_: 填写人用户ID

- TASK_ID_: 节点实例ID

- PROC_INST_ID_: 流程实例ID

- ACTION_: 行为类型 AddUserLink、DeleteUserLink、AddGroupLink、DeleteGroupLink、AddComment、AddAttachment、DeleteAttachment.

- MESSAGE_: 用于存放流程产生的信息，比如审批意见

- FULL_MSG_: 附件地址

##### 6.act_hi_detail 历史详情表

- ID_: 主键ID

- TYPE_: 类型 FormProperty--表单  VariableUpdate--参数

- PROC_INST_ID_: 流程实例ID

- EXECUTION_ID_: 执行实例ID

- TASK_ID_: 任务实例ID

- ACT_INST_ID_:  节点实例ID,ACT_HI_ACTINST表的ID

- NAME_: 名称

- VAR_TYPE_: 参数类型

- REV_: 乐观锁

- TIME_: 创建时间

- BYTEARRAY_ID_: ACT_GE_BYTEARRAY表的ID

- DOUBLE_: 存储变量类型为Double

- LONG_: 存储变量类型为long

- TEXT_: 存储变量值类型为String

- TEXT2_: 此处存储的是JPA持久化对象时，才会有值。此值为对象ID

业务表单中填写的流程需要用到的变量,以及控制流程流转的变量所有的详细信息都会保存在这个历史详情表中。

##### 7.act_hi_identitylink 历史流程人员

这个表其实就比较好了，因为当我们发起各种各样流程的时候，我们需要的永远都是保证数据从哪里来，已经谁发起的，追根溯源好找人呀。

- ID_: 主键ID

- GROUP_ID_: 组ID

- TYPE_: 类型 assignee、candidate、owner、starter 、participant

- USER_ID_: 用户ID

- TASK_ID_: 节点实例ID

- PROC_INST_ID_: 流程实例ID

主要存储历史节点参与者的信息，就是把发起流程的，还有参与过这个流程的人员信息，全部都加到表中。

##### 8.act_hi_procinst 历史流程实例 (画重点的表!)

- ID_: 主键ID

- PROC_INST_ID_: 流程实例ID

- BUSINESS_KEY_: 业务主键，业务表单的ID

- PROC_DEF_ID_: 流程定义ID

- START_TIME_: 开始时间

- END_TIME_: 结束时间

- DURATION_: 耗时

- START_USER_ID_: 起草人的ID

- START_ACT_ID_: 开始节点ID

- END_ACT_ID_: 结束节点ID

- SUPER_PROCESS_INSTANCE_ID_: 父流程实例ID

- DELETE_REASON_: 删除原因

##### 9.act_hi_taskinst 历史任务信息表 (画重点的表!)

- ID_: 主键ID

- PROC_DEF_ID_: 流程定义ID

- TASK_DEF_KEY_: 节点定义ID

- PROC_INST_ID_: 流程实例ID

- EXECUTION_ID_: 执行实例ID

- NAME_: 名称

- PARENT_TASK_ID_: 父节点实例ID

- DESCRIPTION_: 描述

- OWNER_: 签收人（默认为空，只有在委托时才有值）任务的拥有者

- ASSIGNEE_: 签收人或被委托

- START_TIME_: 开始时间

- CLAIM_TIME_: 提醒时间

- END_TIME_: 结束时间

- DURATION_: 耗时

- DELETE_REASON_: 删除原因(completed,deleted)

- PRIORITY_: 优先级别

- DUE_DATE_: 过期时间，表明任务应在多长时间内完成

- FORM_KEY_: 节点定义的formkey,desinger节点定义的form_key属性

##### 10.act_hi_varinst 历史变量表

- ID_: 主键ID

- PROC_INST_ID_: 流程实例ID

- EXECUTION_ID_: 执行实例ID

- TASK_ID_: 任务实例ID

- NAME_: 名称

- VAR_TYPE_: 参数类型

- REV_: 乐观锁

- BYTEARRAY_ID_: ACT_GE_BYTEARRAY表的主键

- DOUBLE_: 存储DoubleType类型的数据

- LONG_: 存储LongType类型的数据

- TEXT_: 存储变量值类型为String，如此处存储持久化对象时，值jpa对象的class

- TEXT2_: 此处存储的是JPA持久化对象时，才会有值。此值为对象ID

##### 11.act_id_group 用户信息组

- ID_: 主键ID

- REV_: 乐观锁

- NAME_: 用户组名称

- TYPE_: 用户组类型

##### 12.act_id_info 用户信息详情表

- ID_: 主键ID

- REV_: 乐观锁

- USER_ID_: 用户ID

- TYPE_: 类型

- KEY_: formINPut名称

- VALUE_: 值

- PASSWORD_: 密码

- PARENT_ID_: 父节点

用户信息详情表，这个表好像有点鸡肋，目前说是还没有用到，

##### 13.act_id_membership 用户与分组对应信息表

总得来说，这个表是真的简单，因为只是表示用户和组之间的对应关系，和很多硬件方面的内容好像很类似，就几个字段。

- USER_ID: 用户ID

- GROUP_ID: 用户组ID

##### 14.act_id_user 用户信息表

- ID_: 主键ID

- REV_: 乐观锁

- FIRST_: 用户姓

- LAST_: 用户名

- EMAIL_: 邮箱

- PWD_: 密码

- PICTURE_ID_: 头像Id

##### 15.act_procdef_info 流程定义信息表

##### 16.act_re_deployment 部署信息表

- ID_: 主键ID

- NAME_: 部署文件名

- CATEGORY_: 分类类别

- DEPLOY_TIME_: 部署时间

这个表主要就是在部署流程定义时需要被持久化保存下来的信息。

##### 17.act_re_model 流程设计模型表

- ID_: 主键ID

- REV_: 乐观锁

- NAME_: 名称

- KEY_:模型的关键字 流程引擎用到。比如：FTOA_SWGL

- CATEGORY_: 类型，用户自己对流程模型的分类。

- CREATE_TIME_: 创建时间

- LAST_UPDATE_TIME_: 最后的修改时间

- VERSION_: 版本

- META_INFO_: 以json格式保存流程定义的信息

- DEPLOYMENT_ID_: 部署ID

- EDITOR_SOURCE_VALUE_ID_: 编辑源值ID

- EDITOR_SOURCE_EXTRA_VALUE_ID_: 编辑源额外值ID（外键ACT_GE_BYTEARRAY ）

##### 18.act_re_procdef 流程定义数据表

这个表示业务流程定义数据表，对应关系和 act_re_deployment 是多对一的关系。

- ID_: 主键ID

- REV_: 乐观锁

- CATEGORY_: 流程定义的Namespace就是类别，该编号就是流程文件targetNamespace的属性值

- NAME_: 流程名称，该编号就是流程文件process元素的name属性值

- KEY_: 流程编号，该编号就是流程文件process元素的id属性值

- VERSION_: 流程版本号，由程序控制，新增即为1，修改后依次加1来完成的

- DEPLOYMENT_ID_: 部署表ID

- RESOURCE_NAME_: 流程bpmn文件名称

- DGRM_RESOURCE_NAME_: png流程图片名称

- DESCRIPTION_: 描述信息

- HAS_START_FORM_KEY_: 是否从key启动，start节点是否存在formKey 0否  1是

- SUSPENSION_STATE_: 是否挂起，1激活 2挂起

##### 19.act_ru_event_subscr 信息监听表

- ID_: 主键ID

- REV_: 乐观锁

- EVENT_TYPE_: 事件类型

- EVENT_NAME_: 事件名称

- EXECUTION_ID_: 执行实例ID

- PROC_INST_ID_: 流程实例ID

- ACTIVITY_ID_: 活动实例ID

- CONFIGURATION_: 配置信息

- CREATED_: 创建时间

##### 20.act_ru_execution 运行时流程执行数据表

这个表实际上就是很多 OA 中会出现的比如说，待办信息的展示。

- ID_: 主键ID

- REV_: 乐观锁

- PROC_INST_ID_: 流程实例ID

- BUSINESS_KEY_: 业务主键ID

- PARENT_ID_: 父节点实例ID

- PROC_DEF_ID_: 流程定义ID

- SUPER_EXEC_:

- ACT_ID_: 节点实例ID即 ACT_HI_ACTINST 中ID

- IS_ACTIVE_: 激活状态，是否存活

- IS_CONCURRENT_: 是否为并行(true/false）

- IS_SCOPE_: 范围定义

- IS_EVENT_SCOPE_: 事件范围

- SUSPENSION_STATE_: 挂起状态 1激活 2挂起

- CACHED_ENT_STATE_: 缓存结束状态

##### 21.act_ru_identitylink 运行时节点人员数据信息表

任务参与者数据表。主要存储当前节点参与者的信息。

- ID_: 主键ID

- REV_: 乐观锁

- GROUP_ID_: 分组ID

- TYPE_: 用户组类型 主要分为以下几种：assignee、candidate、owner、starter、participant。即：受让人,候选人,所有者、起动器、参与者

- USER_ID_: 用户ID

- TASK_ID_: 任务Id

- PROC_INST_ID_: 流程实例ID

- PROC_DEF_ID_: 流程定义Id

##### 22.act_ru_job 定时任务数据表

- ID_: 主键ID

- REV_: 乐观锁

- TYPE_: 类型

- LOCK_EXP_TIME_: 锁定释放时间

- LOCK_OWNER_: 挂起者

- EXCLUSIVE_: 

- EXECUTION_ID_: 执行实例ID

- PROCESS_INSTANCE_ID_: 流程实例ID

- PROC_DEF_ID_: 流程定义ID

- RETRIES_: 

- EXCEPTION_STACK_ID_: 异常信息ID

- EXCEPTION_MSG_: 异常信息

- DUEDATE_: 到期时间

- REPEAT_: 重复

- HANDLER_TYPE_: 处理类型

- HANDLER_CFG_: 标识

##### 23.act_ru_task 运行时任务节点表

- ID_: 主键ID

- REV_: 乐观锁

- EXECUTION_ID_: 执行实例ID

- PROC_INST_ID_: 流程实例ID

- PROC_DEF_ID_: 流程定义ID

- NAME_: 节点定义名称

- PARENT_TASK_ID_: 父节点实例ID

- DESCRIPTION_: 节点定义描述

- TASK_DEF_KEY_: 任务定义的ID

- OWNER_: 拥有者（一般情况下为空，只有在委托时才有值） 实际签收人

- ASSIGNEE_: 签收人或委托人

- DELEGATION_: 委托类型

- PRIORITY_: 优先级别，默认为：50

- CREATE_TIME_: 创建时间

- DUE_DATE_: 耗时

- SUSPENSION_STATE_: 是否挂起，1代表激活 2代表挂起

##### 24.act_ru_variable 流程变量数据表

- ID_: 主键ID

- REV_: 乐观锁

- TYPE_: 编码类型

- NAME_: 变量名称

- EXECUTION_ID_: 执行实例ID

- PROC_INST_ID_: 流程实例ID

- TASK_ID_: 节点实例ID(Local)

- BYTEARRAY_ID_:ACT_GE_BYTEARRAY的ID

- DOUBLE_: 存储变量类型为Double

- LONG_: 存储变量类型为long

- TEXT_: 存储变量值类型为String 如此处存储持久化对象时，值jpa对象的class

- TEXT2_: 此处存储的是JPA持久化对象时，才会有值。此值为对象ID

####  Activiti生命周期

1.流程部署 --->  2.启动流程实例 --- > 3.执行流程对象(一个流程实例包含多执行对象) ---> 4.完成整个流程

![](http://www.justdojava.com/assets/images/2019/java/image_yi/2021/06-11/2.jpg)

上图是个请假流程图，我们按照这个来整点代码来安排一下这个工作请假审批流。

1. 画个图，发布流程,进行流程部署

```
rocessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
RepositoryService repositoryService = processEngine.getRepositoryService();
repositoryService.createDeployment()
  .addClasspathResource("activity.cfg.xml")
  .deploy();
Log.info("Number of process definitions: " + repositoryService.createProcessDefinitionQuery().count());
```

2. 启动一个流程实例

````
    Map<String, Object> variables = new HashMap<String, Object>();
    variables.put("employeeName", "Kermit");
    variables.put("numberOfDays", new Integer(4));
    variables.put("vacationMotivation", "I'm really tired!");
    
    RuntimeService runtimeService = processEngine.getRuntimeService();
    ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("vacationRequest", variables);
    
    Log.info("Number of process instances: " + runtimeService.createProcessInstanceQuery().count());
````

3. 执行流程对象

```
TaskService taskService = processEngine.getTaskService();
List<Task> tasks = taskService.createTaskQuery().taskCandidateGroup("management").list();
for (Task task : tasks) {
  Log.info("Task available: " + task.getName());
}
```

上面代码中的 tasks 实际上可以说是一个任务列表，展示了所有必须由整个用户处理的任务

4. 完成任务

```
Task task = tasks.get(0);
Map<String, Object> taskVariables = new HashMap<String, Object>();
taskVariables.put("vacationApproved", "false");
taskVariables.put("managerMotivation", "We have a tight deadline!");
taskService.complete(task.getId(), taskVariables);
```

对Activiti来说，就是需要complete任务,这个时候，实际上一个简单的请假流程就已经完成了，这时候，流程实例就会进入到下一个环节中。

而在这中间，我们可以设置一下把请假流程挂起，挂起的时候，就不能创建新流程了，不然就会出现异常，

```
repositoryService.suspendProcessDefinitionByKey("vacationRequest");
try {
  runtimeService.startProcessInstanceByKey("vacationRequest");
} catch (ActivitiException e) {
  e.printStackTrace();
}
```

我们通过 RepositoryService 挂起了流程，这个时候，流程不能继续执行，也不会执行异步操作，当我们需要激活这个流程的时候，我们就需要调用：

runtimeService.activateProcessInstanceXXX 方法来对流程进行激活了。

Activiti 的生命周期，就是这么简单，你学会了么？

##### 文章参考

《Activiti使用手册》
《Spring-Activiti代码》
