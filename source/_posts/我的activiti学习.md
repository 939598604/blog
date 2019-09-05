---
title: activiti学习笔记
date: 2019-08-09 20:32:59
password: 123
abstract: 欢迎来到test, 请输入密码.
message: 欢迎来到test, 请输入密码.
toc: false
mathjax: false
tags:  [activiti] 
category: javaEE

---

#  activiti学习笔记

##   一、Activiti获取ProcessEngine的三种方法

###  1.1 ProcessEngineConfiguration获取

```java
public static void config() {
        //获取config对象
        ProcessEngineConfiguration processEngineConfiguration = ProcessEngineConfiguration.createStandaloneProcessEngineConfiguration();
        //Jdbc设置
        String jdbcDriver = "com.mysql.jdbc.Driver";
        processEngineConfiguration.setJdbcDriver(jdbcDriver);
        String jdbcUrl = "jdbc:mysql://localhost:3306/activiti?useUnicode=true&characterEncoding=utf-8";
        processEngineConfiguration.setJdbcUrl(jdbcUrl);
        String jdbcUsername = "root";
        processEngineConfiguration.setJdbcUsername(jdbcUsername);
        String jdbcPassword = "root";
        processEngineConfiguration.setJdbcPassword(jdbcPassword);
        //DB_SCHEMA_UPDATE_FALSE = "false";不自动创建新表
        // DB_SCHEMA_UPDATE_CREATE_DROP = "create-drop";每次运行创建新表
        // DB_SCHEMA_UPDATE_TRUE = "true";设置自动对表结构进行改进和升级
        //设置是否自动更新
   processEngineConfiguration.setDatabaseSchemaUpdate(ProcessEngineConfiguration.DB_SCHEMA_UPDATE_TRUE);
        //获取引擎对象
        ProcessEngine processEngine = processEngineConfiguration.buildProcessEngine();
        processEngine.close();
    }

```

###  1.2 ProcessEngineConfiguration载入xml文件

xml文件： 

```xml
<beans xmlns="http://www.springframework.org/schema/beans"     xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
   http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd
   http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.0.xsd">
<!--这里的类太多别导错了 -->
<bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
    <!-- 配置流程引擎配置对象 -->
    <property name="jdbcDriver" value="com.mysql.jdbc.Driver"></property>
    <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/activiti?useUnicode=true&amp;characterEncoding=utf-8"></property>
    <property name="jdbcUsername" value="root"></property>
    <property name="jdbcPassword" value="123456"></property>
    <!-- 注入数据源信息 -->
    <property name="databaseSchemaUpdate" value="true"></property>
</bean>
<bean id="processEngine" class="org.activiti.spring.ProcessEngineFactoryBean">
    <!-- 注入自动建表设置 -->
    <property name="processEngineConfiguration" ref="processEngineConfiguration"></property>
</bean>
</beans>
```

java代码加载xml文件

```java
public void configByConf() {
   //载入资源
   ProcessEngineConfiguration processEngineConfiguration =      ProcessEngineConfiguration.createProcessEngineConfigurationFromResource("activiti.cfg.xml");
        //创建引擎
   ProcessEngine processEngine = processEngineConfiguration.buildProcessEngine();
   processEngine.getRepositoryService();
}
```

###  1.3 默认载入activiti.cfg.xml进行获取

```java
public void configByDefault() {
   //通过获取载入默认获取
   ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
   processEngine.close();
}
```

这里的xml文件名必须设置为activiti.cfg.xml

 

##  二、流程定义

`部署流程 `-->`启动流程实例 `

###  2.1 设计流程定义文档

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904182142.png)

###  2.2 bpmn文件



```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.activiti.org/test">
  <process id="helloworld" name="helloworldProcess" isExecutable="true">
    <startEvent id="startevent1" name="Start"></startEvent>
    <endEvent id="endevent1" name="End"></endEvent>
    <userTask id="usertask1" name="提交申请" activiti:assignee="张三"></userTask>
    <userTask id="usertask2" name="审批【部门经理】" activiti:assignee="李四"></userTask>
    <userTask id="usertask3" name="审批【总经理】" activiti:assignee="王五"></userTask>
    <sequenceFlow id="flow1" sourceRef="startevent1" targetRef="usertask1"></sequenceFlow>
    <sequenceFlow id="flow2" sourceRef="usertask1" targetRef="usertask2"></sequenceFlow>
    <sequenceFlow id="flow3" sourceRef="usertask2" targetRef="usertask3"></sequenceFlow>
    <sequenceFlow id="flow4" sourceRef="usertask3" targetRef="endevent1"></sequenceFlow>
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_helloworld">
    <bpmndi:BPMNPlane bpmnElement="helloworld" id="BPMNPlane_helloworld">
      <bpmndi:BPMNShape bpmnElement="startevent1" id="BPMNShape_startevent1">
        <omgdc:Bounds height="35.0" width="35.0" x="330.0" y="20.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="endevent1" id="BPMNShape_endevent1">
        <omgdc:Bounds height="35.0" width="35.0" x="330.0" y="380.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="usertask1" id="BPMNShape_usertask1">
        <omgdc:Bounds height="55.0" width="105.0" x="295.0" y="100.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="usertask2" id="BPMNShape_usertask2">
        <omgdc:Bounds height="55.0" width="105.0" x="295.0" y="200.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="usertask3" id="BPMNShape_usertask3">
        <omgdc:Bounds height="55.0" width="105.0" x="295.0" y="290.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNEdge bpmnElement="flow1" id="BPMNEdge_flow1">
        <omgdi:waypoint x="347.0" y="55.0"></omgdi:waypoint>
        <omgdi:waypoint x="347.0" y="100.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow2" id="BPMNEdge_flow2">
        <omgdi:waypoint x="347.0" y="155.0"></omgdi:waypoint>
        <omgdi:waypoint x="347.0" y="200.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow3" id="BPMNEdge_flow3">
        <omgdi:waypoint x="347.0" y="255.0"></omgdi:waypoint>
        <omgdi:waypoint x="347.0" y="290.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow4" id="BPMNEdge_flow4">
        <omgdi:waypoint x="347.0" y="345.0"></omgdi:waypoint>
        <omgdi:waypoint x="347.0" y="380.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```

###  2.3 部署流程定义

(1) 部署流程定义

```java
public void deploymentProcessDefinition_classpath(){
	Deployment deployment = processEngine.getRepositoryService()
					.createDeployment().name("流程定义")
                      //从classpath的资源中加载，一次只能加载一个文件
					.addClasspathResource("diagrams/helloworld.bpmn")
					.addClasspathResource("diagrams/helloworld.png").deploy();//完成部署
	System.out.println("部署ID："+deployment.getId());
	System.out.println("部署名称："+deployment.getName());
}
```

(2) ZipInputStream的部署方式

```
public static void main(String[] args) throws Exception{
    DeploymentBuilder deployment = rs.createDeployment();
    FileInputStream fis = new FileInputStream(new File(""));
    ZipInputStream zis = new ZipInputStream(fis);
    deployment.addZipInputStream(zis);
    deployment.deploy();
}
```

### 2.4 流程部署后数据库的变化

- act_ge_bytearray（资源文件表）        存储流程定义相关的部署信息。即流程定义文档的存放地。每部署一次就会增加两条记录，一条是关于bpmn规则文件的，一条是图片的（如果部署时只指定了bpmn一个文件，activiti会在部署时解析bpmn文件内容自动生成流程图）。两个文件不是很大，都是以二进制形式存储在数据库中。

-  act_re_procdef（流程定义表）    存放流程定义的属性信息，部署每个新的流程定义都会在这张表中增加一条记录。    注意：当流程定义的key相同的情况下，使用的是版本升级    

​       其中`act_re_deployment`的id会和`act_ge_bytearray`的deployment_id_关联

- act_re_deployment（流程部署表）       存放流程定义的显示名和部署时间，每部署一次增加一条记录

流程部署之后act_ge_bytearray插入两条记录

| ID_   | REV_ | NAME_                  | DEPLOYMENT_ID_ | BYTES_   | GENERATED_ |
| ----- | ---- | ---------------------- | -------------- | -------- | ---------- |
| ID    | 版本 | 名称                   | 部署ID         | 字节     | 不详       |
| 10002 | 1    | simple.bpmn            | 10001          | blob文件 | 1          |
| 10003 | 1    | simple.myProcess_1.png | 10001          | blob文件 | 1          |

act_re_procdef（流程定义表）       存放流程定义的显示名和部署时间，每部署一次增加一条记录

| ID_                 | REV_ | CATEGORY_ | NAME_ | KEY_        | VERSION_ | DEPLOYMENT_ID_ | RESOURCE_NAME_ | DGRM_RESOURCE_NAME_    | DESCRIPTION_ | HAS_START_FORM_KEY_ | HAS_GRAPHICAL_NOTATION_ | SUSPENSION_STATE_ | TENANT_ID_ | ENGINE_VERSION_ |
| ------------------- | ---- | --------- | ----- | ----------- | -------- | -------------- | -------------- | ---------------------- | ------------ | ------------------- | ----------------------- | ----------------- | ---------- | --------------- |
| 流程定义id          |      |           |       | 流程key     | 版本     | 流程部署id     | 流程资源名称   | 流程资源图片           | 描述         |                     |                         |                   |            |                 |
| myProcess_1:1:10004 | 1    | Examples  |       | myProcess_1 | 1        | 10001          | simple.bpmn    | simple.myProcess_1.png |              | 0                   | 1                       | 1                 |            |                 |

act_re_deployment（流程部署表）       存放流程定义的显示名和部署时间，每部署一次增加一条记录

| ID_    | NAME_    | CATEGORY_ | KEY_    | TENANT_ID_ | DEPLOY_TIME_   | ENGINE_VERSION_ |
| ------ | -------- | --------- | ------- | ---------- | -------------- | --------------- |
| 部署id | 部署名称 | 部署类型  | 部署key |            | 部署时间       |                 |
| 10001  |          |           |         |            | 2019/4/3 16:35 |                 |

###  2.5 根据名称查询流程部署

```java
public void testQueryDeploymentByName(){
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        List<Deployment> deployments = processEngine.getRepositoryService()
                .createDeploymentQuery()
                .orderByDeploymenTime()//按照部署时间排序
                .desc()//按照降序排序
                .deploymentName("请假流程")
                .list();
        for (Deployment deployment : deployments) {
            System.out.println(deployment.getId());
        }
    }
```

###  2.6 查询所有的部署流程

```java
public void queryAllDeplyoment(){
        //得到流程引擎
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        List<Deployment> lists = processEngine.getRepositoryService()
                .createDeploymentQuery()
                .orderByDeploymenTime()//按照部署时间排序
                .desc()//按照降序排序
                .list();
        for (Deployment deployment:lists) {
            System.out.println(deployment.getId() +"    部署名称" + deployment.getName());
        }
}
```

###  2.7 查看所有流程定义

```java
public void findProcessDefinition(){
	List<ProcessDefinition> list = RepositoryService.createProcessDefinitionQuery()//创建一个流程定义的查询
			/**指定查询条件,where条件*/
//			.deploymentId(deploymentId)//使用部署对象ID查询
//			.processDefinitionId(processDefinitionId)//使用流程定义ID查询
//			.processDefinitionKey(processDefinitionKey)//使用流程定义的key查询
//			.processDefinitionNameLike(processDefinitionNameLike)//使用流程定义的名称模糊查询
			.orderByProcessDefinitionVersion().asc()//按照版本的升序排列
//			.orderByProcessDefinitionName().desc()//按照流程定义的名称降序排列
			.list();//返回一个集合列表，封装流程定义
//			.singleResult();//返回惟一结果集
//			.count();//返回结果集数量
//			.listPage(firstResult, maxResults);//分页查询
	if(list!=null && list.size()>0){
		for(ProcessDefinition pd:list){
			System.out.println("流程定义ID:"+pd.getId());//流程定义的key+版本+随机生成数
			System.out.println("流程定义的名称:"+pd.getName());//对应helloworld.bpmn文件中的name属性值
			System.out.println("流程定义的key:"+pd.getKey());//对应helloworld.bpmn文件中的id属性值
			System.out.println("流程定义的版本:"+pd.getVersion());//当流程定义的key值相同的相同下，版本升级，默认1
			System.out.println("资源名称bpmn文件:"+pd.getResourceName());
			System.out.println("资源名称png文件:"+pd.getDiagramResourceName());
			System.out.println("部署对象ID："+pd.getDeploymentId());
		}
	}			
}
```

###  2.8 删除流程定义

```java
public void deleteProcessDefinition(){
	String deploymentId = "601"; //使用部署ID，完成删除
	//不带级联的删除 只能删除没有启动的流程，如果流程启动，就会抛出异常
    RepositoryService().deleteDeployment(deploymentId);
	//级联删除  不管流程是否启动，都能可以删除
	RepositoryService().deleteDeployment(deploymentId, true);
}
```

说明：    

-  因为删除的是流程定义，而流程定义的部署是属于仓库服务的，所以应该先得到RepositoryService。    

- 如果该流程定义下没有正在运行的流程，则可以用普通删除。如果是有关联的信息，用级联删除。项目开发中使用级联删除的情况比较多，删除操作一般只开放给超级管理员使用。

###  2.9 查看流程图

```java
public void viewPic() throws IOException{
	/**将生成图片放到文件夹下*/
	String deploymentId = "801";
	//获取图片资源名称
	List<String> list = repositoryService.getDeploymentResourceNames(deploymentId);
	//定义图片资源的名称
	String resourceName = "";
	if(list!=null && list.size()>0){
		for(String name:list){
			if(name.indexOf(".png")>=0){
				resourceName = name;
			}
		}
	}	
	//获取图片的输入流
	InputStream in = repositoryService.getResourceAsStream(deploymentId, resourceName);
	//将图片生成到D盘的目录下
	File file = new File("D:/"+resourceName);
	//将输入流的图片写到D盘下
	FileUtils.copyInputStreamToFile(in, file);
}
```

###  2.10 流程定义的暂停挂起

**测试暂停流程定义执行步骤如下：**

在程序中，我们需要暂停一个流程定义，停止所有的该流程定义下的流程实例，并且不允许发起这个流程定义的流程实例，那么我们就需要挂起这个流程定义

　　1，启动一个流程实例（该流程定义未挂起前）

　　2，挂起上面流程实例对应的流程定义

　　3，完成上述流程实例的下一个任务节点（观察效果，是否会和流程实例挂起一样）

（1）根据流程实例的id来挂起这个流程定义

```
public void testSuspendProcessDefinition(){        
     String processDefinitionKey ="purchasingflow";
     //根据流程定义的key暂停一个流程定义
     repositoryService.suspendProcessDefinitionByKey(processDefinitionKey );
}
```

（2）完成这个流程实例的下一个节点，通过taskService来结束下一个任务节点

　　这时候，我们发现这个流程实例居然是可以继续执行的，并且可以执行到结束，带着这个疑问，我们再启动一个流程实例看看

（3）重新启动这个流程定义下的流程实例

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904182230.png)

报错说不可以启动这个被挂起流程定义的流程实例

##  三、流程实例

###  3.1 根据流程部署id启动流程实例

```java
public void startProcess(String deploymentId){
    ProcessDefinition pd=repositoryService.createProcessDefinitionQuery()
        .deploymentId(deploymentId)
        .singleResult();
    ProcessInstance pi=runtimeService.startProcessInstanceById(pd.getId());
}
```

###  3.2 根据流程id启动流程实例,可以设置一个流程变量

```java
/**
* 流程变量
* 给<userTask id="请假申请" name="申请" activiti:assignee="#{student}"></userTask>的student赋值
*/
public void startProcess(){
    Map<String, Object> variables = new HashMap<String, Object>();
    variables.put("student", "小明");
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    runtimeService.startProcessInstanceById("shenqing1:1:1304",variables);
}
```

###  3.3 启动流程实例之后数据库的变化

首先向act_ru_execution表中插入一条记录，记录的是这个流程定义的执行实例，其中id和proc_inst_id相同都是流程执行实例id，也就是本次执行这个流程定义的id，包含流程定义的id外键(simpleProcess:1:5004)。

| ID_   | REV_ | PROC_INST_ID_ | BUSINESS_KEY_ | PARENT_ID_ | PROC_DEF_ID_        | SUPER_EXEC_ | ROOT_PROC_INST_ID_ | ACT_ID_ | IS_ACTIVE_ | IS_CONCURRENT_ | IS_SCOPE_ | IS_EVENT_SCOPE_ | IS_MI_ROOT_ | SUSPENSION_STATE_ | START_TIME_       |
| ----- | ---- | ------------- | ------------- | ---------- | ------------------- | ----------- | ------------------ | ------- | ---------- | -------------- | --------- | --------------- | ----------- | ----------------- | ----------------- |
|       |      |               |               |            |                     |             |                    |         |            |                |           |                 |             |                   |                   |
| 12501 | 1    | 12501         |               |            | myProcess_1:1:10004 |             | 12501              |         | 1          | 0              | 1         | 0               | 0           | 1                 | 2019-4-3 17:25:30 |
| 12503 | 1    | 12501         |               | 12501      | myProcess_1:1:10004 |             | 12501              | _3      | 1          | 0              | 0         | 0               | 0           | 1                 | 2019-4-3 17:25:30 |

然后向act_ru_task插入一条记录，记录的是第一个任务的信息，也就是开始执行第一个任务。包括act_ru_execution表中的execution_id外键和proc_inst_id外键，也就是本次执行实例id。

| ID_   | REV_ | EXECUTION_ID_ | PROC_INST_ID_ | PROC_DEF_ID_        | NAME_    | PARENT_TASK_ID_ | DESCRIPTION_ | TASK_DEF_KEY_ | OWNER_ | ASSIGNEE_ | DELEGATION_ | PRIORITY_ | CREATE_TIME_   | DUE_DATE_ | CATEGORY_ | SUSPENSION_STATE_ |
| ----- | ---- | ------------- | ------------- | ------------------- | -------- | --------------- | ------------ | ------------- | ------ | --------- | ----------- | --------- | -------------- | --------- | --------- | ----------------- |
|       |      |               |               |                     |          |                 |              |               |        |           |             |           |                |           |           |                   |
| 12506 | 1    | 12503         | 12501         | myProcess_1:1:10004 | UserTask |                 |              | _3            |        | a         |             | 50        | 2019/4/3 17:07 |           |           | 1                 |

然后向act_hi_procinst表插入一条记录，记录的是本次执行实例：

| ID_   | PROC_INST_ID_ | BUSINESS_KEY_ | PROC_DEF_ID_        | START_TIME_    | END_TIME_ | DURATION_ | START_USER_ID_ | START_ACT_ID_ | END_ACT_ID_ | SUPER_PROCESS_INSTANCE_ID_ | DELETE_REASON_ | TENANT_ID_ | NAME_ |
| ----- | ------------- | ------------- | ------------------- | -------------- | --------- | --------- | -------------- | ------------- | ----------- | -------------------------- | -------------- | ---------- | ----- |
|       |               |               |                     |                |           |           |                |               |             |                            |                |            |       |
| 12501 | 12501         |               | myProcess_1:1:10004 | 2019/4/3 17:07 |           |           |                | _2            |             |                            |                |            |       |

然后向act_hi_taskinst表中插入一条记录，记录的是任务的历史记录：

| ID_   | PROC_DEF_ID_        | TASK_DEF_KEY_ | PROC_INST_ID_ | EXECUTION_ID_ | NAME_    | PARENT_TASK_ID_ | DESCRIPTION_ | OWNER_ | ASSIGNEE_ | START_TIME_    | CLAIM_TIME_ | END_TIME_ | DURATION_ | DELETE_REASON_ | PRIORITY_ | DUE_DATE_ | FORM_KEY_ | CATEGORY_ | TENANT_ID_ |
| ----- | ------------------- | ------------- | ------------- | ------------- | -------- | --------------- | ------------ | ------ | --------- | -------------- | ----------- | --------- | --------- | -------------- | --------- | --------- | --------- | --------- | ---------- |
|       |                     |               |               |               |          |                 |              |        |           |                |             |           |           |                |           |           |           |           |            |
| 12506 | myProcess_1:1:10004 | _3            | 12501         | 12503         | UserTask |                 |              |        | a         | 2019/4/3 17:07 |             |           |           |                | 50        |           |           |           |            |

###  3.4 流程实例的暂停挂起

**测试暂停流程实例执行步骤如下：**

　　1，通过流程定义的key或者id启动一个流程实例

　　2，根据流程实例的id来挂起这个流程实例

　　3，得到下一个节点的对应的任务的id，调用taskService来完成这个任务观察效果

 　   4，重新激活这个流程实例

　　5，继续完成这个流程实例

（1）通过上面发起的流程实例的id挂起这个流程实例 

```java
public void testSuspendProcessInstance(){
    String processInstanceId="1801";
    //根据一个流程实例的id挂起该流程实例
    runtimeService.suspendProcessInstanceById(processInstanceId);
}
```

（2）任务的下一处理人来完成这个实例 

```java
public void completeProcessInstance(){
    //任务的id，后期整合后会通过当前登录人身份查询到该用户的任务，然后获取到该id
    String taskId="1804";
    //根据任务id完成该任务
    taskService.complete(taskId);
}
```

执行完报错：

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904183119.png)

　　上面的信息说明无法完成一个已经被挂起的任务

（3）激活这个流程实例 

```
public void testActivateProcessInstance(){
    String processInstanceId="1801";
    runtimeService.activateProcessInstanceById(processInstanceId);
}
```

5，重新完成这个任务，执行ok 



##  四、任务管理

###  4.1 根据办理人查询任务

`根据任务的执行人查询正在执行任务(通过act_ru_task数据表) `

```java
public void testQueryTaskByAssignee(){
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        //当前班主任小毛人这个人当前正在执行的所有的任务
        List<Task> tasks = processEngine.getTaskService()
                .createTaskQuery()
                .orderByTaskCreateTime()
                .desc()
                .taskAssignee("小毛")
                .list();
        for (Task task : tasks) {
            System.out.println(task.getName());
            System.out.println(task.getAssignee());
        }
    }
```

###  4.2 个人任务的三种指派方式

**（1）方式一**

定义流程图时直接指定完成任务人（项目开发中任务的办理人不要放置XML文件中，不够灵活，较少使用） 

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904183158.png)

定义的bpmn文件中定义任务办理人的名称

```
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.activiti.org/test">
  <process id="personalAssignee1" name="PersonalAssignee1" isExecutable="true">
    <startEvent id="startevent1" name="Start"></startEvent>
    <userTask id="usertask1" name="审批" activiti:assignee="crystal"></userTask>
    <sequenceFlow id="flow1" sourceRef="startevent1" targetRef="usertask1"></sequenceFlow>
    <endEvent id="endevent1" name="End"></endEvent>
    <sequenceFlow id="flow2" sourceRef="usertask1" targetRef="endevent1"></sequenceFlow>
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_personalAssignee1">
    <bpmndi:BPMNPlane bpmnElement="personalAssignee1" id="BPMNPlane_personalAssignee1">
      <bpmndi:BPMNShape bpmnElement="startevent1" id="BPMNShape_startevent1">
        <omgdc:Bounds height="35.0" width="35.0" x="360.0" y="20.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="usertask1" id="BPMNShape_usertask1">
        <omgdc:Bounds height="55.0" width="105.0" x="325.0" y="100.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="endevent1" id="BPMNShape_endevent1">
        <omgdc:Bounds height="35.0" width="35.0" x="360.0" y="200.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNEdge bpmnElement="flow1" id="BPMNEdge_flow1"></bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow2" id="BPMNEdge_flow2"></bpmndi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>

```

重点代码  **activiti:assignee="crystal"**

启动流程实例

```java
public void start() {
    ProcessInstance pi = runtimeService().startProcessInstanceByKey("task");
}
```

**（2）方式二**

定义流程图时配置任务节点变量，完成任务之前由流程变量指定任务办理人。在开发中，可以在页面中指定下一个任务的办理人，通过流程变量设置下一个任务的办理人。

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904183224.png)

定义的bpmn文件中定义任务办理人的名称

```
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.activiti.org/test">
  <process id="personalTask2" name="PersonalTask2" isExecutable="true">
    <startEvent id="startevent1" name="Start"></startEvent>
    <userTask id="usertask1" name="审批" activiti:assignee="#{userId}"></userTask>
    <sequenceFlow id="flow1" sourceRef="startevent1" targetRef="usertask1"></sequenceFlow>
    <endEvent id="endevent1" name="End"></endEvent>
    <sequenceFlow id="flow2" sourceRef="usertask1" targetRef="endevent1"></sequenceFlow>
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_personalTask2">
    <bpmndi:BPMNPlane bpmnElement="personalTask2" id="BPMNPlane_personalTask2">
      <bpmndi:BPMNShape bpmnElement="startevent1" id="BPMNShape_startevent1">
        <omgdc:Bounds height="35.0" width="35.0" x="380.0" y="40.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="usertask1" id="BPMNShape_usertask1">
        <omgdc:Bounds height="55.0" width="105.0" x="345.0" y="110.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="endevent1" id="BPMNShape_endevent1">
        <omgdc:Bounds height="35.0" width="35.0" x="380.0" y="210.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNEdge bpmnElement="flow1" id="BPMNEdge_flow1"></bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow2" id="BPMNEdge_flow2"></bpmndi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```

重点代码  **activiti:assignee="#{userId}"**

启动流程实例，需要设置流程变量

```java
public void start() {
    Map<String, Object> variables = new HashMap<String, Object>();
    variables.put("userId", "crystal");
    ProcessInstance pi = runtimeService().startProcessInstanceByKey("task",variables);
}
```

**（3）方式三**

流程图中不指定任务办理人，添加监听类，需要实现TaskListener接口

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904183040.png)

定义的bpmn文件中定义任务办理人的名称

```
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.activiti.org/test">
  <process id="personalTask3" name="PersonalTask3" isExecutable="true">
    <startEvent id="startevent1" name="Start"></startEvent>
    <userTask id="usertask1" name="审批">
      <extensionElements>
        <activiti:taskListener event="create" class="com.activiti.test.TaskListenerImpl"></activiti:taskListener>
      </extensionElements>
    </userTask>
    <sequenceFlow id="flow1" sourceRef="startevent1" targetRef="usertask1"></sequenceFlow>
    <endEvent id="endevent1" name="End"></endEvent>
    <sequenceFlow id="flow2" sourceRef="usertask1" targetRef="endevent1"></sequenceFlow>
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_personalTask3">
    <bpmndi:BPMNPlane bpmnElement="personalTask3" id="BPMNPlane_personalTask3">
      <bpmndi:BPMNShape bpmnElement="startevent1" id="BPMNShape_startevent1">
        <omgdc:Bounds height="35.0" width="35.0" x="310.0" y="20.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="usertask1" id="BPMNShape_usertask1">
        <omgdc:Bounds height="55.0" width="105.0" x="275.0" y="100.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="endevent1" id="BPMNShape_endevent1">
        <omgdc:Bounds height="35.0" width="35.0" x="310.0" y="200.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNEdge bpmnElement="flow1" id="BPMNEdge_flow1"></bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow2" id="BPMNEdge_flow2"></bpmndi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```

重点代码

`<activiti:taskListener event="create" class="com.activiti.test.TaskListenerImpl"></activiti:taskListener>`

监听类 TaskListenerImpl.java

```java
package com.activiti.test;
import org.activiti.engine.delegate.DelegateTask;
import org.activiti.engine.delegate.TaskListener;
public class TaskListenerImpl implements TaskListener {
    /**
     * 指定个人任务和组任务的办理人
     */
    @Override
    public void notify(DelegateTask delegateTask) {
        delegateTask.setAssignee("crystal");// 指派个人任务
    }
}
```

启动流程实例

```java
public void start() {
    ProcessInstance pi = runtimeService().startProcessInstanceByKey("task");
}
```

总结：

```java
个人任务及三种分配方式： 
  1.在taskProcess.bpmn中直接写 assignee=”crystal” 
  2.在taskProcess.bpmn中写 assignee=“#{userID}”，变量的值要是String的（使用流程变量指定办理人）。 
  3.使用TaskListener接口，要使类实现该接口
    在类中定义： delegateTask.setAssignee(assignee);// 指定个人任务的办理人 
  4.使用任务ID和办理人重新指定办理人： taskService.setAssignee(taskId, userId);
```



###  4.3 组任务的三种指派方式

**方式一：直接指定办理人**

（1）.在任务节点设置办理人

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904182327.png)

（2）bpmn中的代码片段

```
 <userTask activiti:candidateUsers="小a,小b,小c" activiti:exclusive="true" id="usertask1" name="提交申请"/>
```

（3）部署流程和启动流程

```java
public void test(){
    repositoryService.createDeployment().addClasspathResource("test1.bpmn").deploy();//部署流程
    ProcessInstance pi = runtimeService.startProcessInstanceByKey("helloword");//启动流程
}
```

（4）查询我的个人任务，没有执行结果

```java
 public void queryTask(String assignee) {
     List<Task> list = taskService.createTaskQuery().orderByTaskCreateTime()
     		.desc().taskAssignee(assignee).list();
     if (list != null && list.size() > 0) {
         for (Task task : list) {
             System.out.println("任务ID：" + task.getId());
             System.out.println("任务的办理人：" + task.getAssignee());
             System.out.println("任务名称：" + task.getName());
             System.out.println("任务的创建时间：" + task.getCreateTime());
             System.out.println("流程实例ID：" + task.getProcessInstanceId());
             System.out.println("#######################################");
         }
     }
}
```

（5）查询组任务，可以查到查询结果

```
 public void findGroupTaskList(String candidateUser) {
        List<Task> list = taskService.createTaskQuery().taskCandidateUser(candidateUser).list();
        if (list != null && list.size() > 0) {
            for (Task task : list) {
                System.out.println("任务ID：" + task.getId());
                System.out.println("任务的办理人：" + task.getAssignee());
                System.out.println("任务名称：" + task.getName());
                System.out.println("任务的创建时间：" + task.getCreateTime());
                System.out.println("流程实例ID：" + task.getProcessInstanceId());
                System.out.println("#######################################");
            }
        }
    }
```

查询结果如下

> 任务ID：15010
> 任务的办理人：null
> 任务名称：提交申请
> 任务的创建时间：Thu Apr 04 10:50:00 CST 2019
> 流程实例ID：15005

（6）查询正在执行的组任务列表

```java
public void findGroupCandidate(String taskId) {
    List<IdentityLink> list = taskService.getIdentityLinksForTask(taskId);
        if (list != null && list.size() > 0) {
        for (IdentityLink identityLink : list) {
            System.out.println("任务ID：" + identityLink.getTaskId());
            System.out.println("流程实例ID："+ identityLink.getProcessInstanceId());
            System.out.println("用户ID：" + identityLink.getUserId());
            System.out.println("工作流角色ID：" + identityLink.getGroupId());
            System.out.println("#########################################");
        }
    }
}
```

执行结果

> 任务ID：15010
> 流程实例ID：null
> 用户ID：小a
> 工作流角色ID：null
>
> 任务ID：15010
> 流程实例ID：null
> 用户ID：小b
> 工作流角色ID：null
>
> 任务ID：15010
> 流程实例ID：null
> 用户ID：小c
> 工作流角色ID：null

（7）查询历史的组任务列表

```java
public void findHistoryGroupCandidate(String processInstanceId) {
    String processInstanceId = "3705";
    List<HistoricIdentityLink> list = historyService
        	.getHistoricIdentityLinksForProcessInstance(processInstanceId);
    if (list != null && list.size() > 0) {
        for (HistoricIdentityLink identityLink : list) {
            System.out.println("任务ID：" + identityLink.getTaskId());
            System.out.println("流程实例ID："+ identityLink.getProcessInstanceId());
            System.out.println("用户ID：" + identityLink.getUserId());
            System.out.println("工作流角色ID：" + identityLink.getGroupId());
            System.out.println("#########################################");
        }
    }
}
```

说明：     

- 小A，小B，小C是组任务的办理人     
- 但是这样分配组任务的办理人不够灵活，因为项目开发中任务的办理人不要放置XML文件中。     
- act_ru_identitylink表存放组任务的办理人，表示正在执行的任务     
- act_hi_identitylink表存放所有任务的办理人，包括个人任务和组任务**，**表示历史任务

**方式二：使用流程变量**

 （1）在任务节点设置变量

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904183359.png)

（2）bpmn中的代码片段

```
<userTask activiti:candidateUsers="#{userIDs}" activiti:exclusive="true" id="usertask1" name="提交申请"/>
```

（3）部署流程和启动流程

启动流程实例的同时，设置流程变量，使用流程变量的方式设置下一个任务的办理人

 流程变量的名称，是task.bpmn中定义activiti:candidateUsers="#{userIDs}"的userIDs  流程变量的值，就是任务的办理人（组任务）

```java
public void test(){
    repositoryService.createDeployment().addClasspathResource("test1.bpmn").deploy();//部署流程
	Map<String, Object> variables = new HashMap<String, Object>();
	variables.put("userIDs", "小a,小b,小c");
	ProcessInstance pi = runtimeService.startProcessInstanceByKey("helloword",variables);//使用流程定义的key的最新版本启动流程
}
```

**方式三：使用监听器**

（1）设置监听器变量

![img](https://img-blog.csdn.net/20151227154130920)

 

（2）编写监听器类

```
public class TaskListenerImpl implements TaskListener {
	/**
	 * 可以设置任务的办理人（个人组人和组任务）
	 */
	@Override
	public void notify(DelegateTask delegateTask) {
		//指定组任务
		delegateTask.addCandidateUser("孙悟空");
		delegateTask.addCandidateUser("猪八戒");
	}
}
```

（3）测试代码

```java
/**将组任务指定个人任务(拾取任务)*/
public void claim(){
    String taskId = "6308";
    //个人任务的办理人
    String userId = "唐僧";
    taskService.claim(taskId, userId);
}

/**将个人任务再回退到组任务（前提：之前这个任务是组任务）*/
public void setAssignee(){
    String taskId = "6308";
    taskService.setAssignee(taskId, null);
}

/**向组任务中添加成员*/
public void addGroupUser(){
    String taskId = "6308";
    //新增组任务的成员
    String userId = "如来";
    taskService.addCandidateUser(taskId, userId);
}

/**向组任务中删除成员*/
public void deleteGroupUser(){
    String taskId = "6308";
    //新增组任务的成员
    String userId = "猪八戒";
    taskService.deleteCandidateUser(taskId, userId);
}
```

总结：      以上就是分配组任务的三种方式，和分配个人任务相对应，同样有三种方式，与个人任务的操作相比，组任务操作增加了组任务分配个人任务（认领任务），个人任务分配给组任务，以及向组任务添加人员和向组任务删除人员的操作。

###  4.4 工作流提供的用户角色

（1）设置用户角色

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904182357.png)

这个【部门经理】相当于一个用户角色，一个角色可以对应多个人，比如有三个人：张三、李四是部门经理，王五是总经理，那我们可以把这三个人录入的我们自己的用户表中，那么工作流也给我们提供了至少三张表：用户表，角色表，用户角色关联表那我们就可以把部门经理这个角色与张三、李四关联起来

（2）具体做法：1、添加用户角色组  2、创建角色3、创建用户4、创建角色用户关联关系测试代码如下：

```java
public void addUser(){	
		//创建角色(两个角色)
		identityService.saveGroup(new GroupEntity("部门经理"));
		identityService.saveGroup(new GroupEntity("总经理"));
		
		//创建用户（三个用户）
		identityService.saveUser(new UserEntity("张三"));
		identityService.saveUser(new UserEntity("李四"));
		identityService.saveUser(new UserEntity("王五"));
		
		//创建用户角色关联关系
		identityService.createMembership("张三", "部门经理");
		identityService.createMembership("李四", "部门经理");
		identityService.createMembership("王五", "总经理");
}
```

（3）部署流程定义

```
public void deployementAndStart(){
	Deployment deployment = repositoryService.createDeployment().name("组任务")
					.addClasspathResource("diagrams/group.bpmn")
					.addClasspathResource("diagrams/group.png").deploy();
	ProcessInstance pi = runtimeService.startProcessInstanceByKey("group");//启动流程  
}				
```

（4）查询张三或者李四的任务

```
public void findGroupTask(){
		String candidateUser = "张三";
		List<Task> list =taskService.createTaskQuery().taskCandidateUser(candidateUser).list();
		if(list!=null && list.size()>0){  
	        for(Task task:list){  
	            System.out.println("任务ID："+task.getId());  
	            System.out.println("任务的办理人："+task.getAssignee());  
	            System.out.println("任务名称："+task.getName());  
	            System.out.println("任务的创建时间："+task.getCreateTime());  
	            System.out.println("流程实例ID："+task.getProcessInstanceId());  
	            System.out.println("#######################################");  
	        }  
	     }
}
```

执行结果

>  任务ID：167504 
>
>  任务的办理人：null 
>
>  任务名称：审核 
>
>  任务的创建时间：Thu Jul 07 10:21:27 GMT+08:00 2016 
>
>  流程实例ID：167501 

（5）候选者不一定真正的参与任务的办理，所以我们需要拾取任务，将组任务分配给个人任务,即指定任务办理人字段

```java
public void cliam(){
		//将组任务分配给个人任务
		String taskId ="167504";
		//分配的个人任务（可以是组任务中的成员，也可以是非组任务的成员）
		String userId ="张三";
		taskService.claim(taskId, userId);
		//当执行完查询正执行的任务表（act_ru_task）可发现ASSIGNEE_字段（指定任务办理人）值为'张三'
	    //此时任务就指定给了张三，再用李四去查个人组任务就查询不出来任何任务【组任务最终也是需要指定一个人办理的，所以需要拾取任务】
}

```

（6）张三完成任务

```java
public void completeTask(){
	String taskId ="167504";
	taskService.complete(taskId);
}

```

当我们部署完流程定义，启动流程实例之后，我们可以查看您一下几张数据表：**表act_ru_task**

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904183255.png)

 可以看见任务办理人的字段值为null,所以可能有两种情况可能没有办理人或者可能这个任务是组任务

**表act_ru_identitylink**   正在执行的任务办理人表

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904183326.png)

在这个表可以看见Task_ID 的值为167504就是正在执行的任务ID，流程实例字段为空，所以这个任务是组任务，处理这个组任务的角色ID为部门经理而张三和李四都是这个角色的用户，所以张三李四都可以查到这个任务，也可以进行任务的拾取，分配等操作。

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904182422.png)

需要说明的是在我们自己项目开发的时候，我们一般都是不用工作流自带的用户表、角色表，用户角色关联表都是自己来建，因为自带的表提供的字段不全。

###  4.5 任务签收与反签收

```java
//任务签收
public void claim(String taskId) {
   String userId = "1111";
   taskService.claim(taskId, userId);
}

//任务反签收
public String unclaim(String taskId) {
    taskService.unclaim(taskId);
}
```

### 4.6 驳回申请



### 4.7  流程变量的设置和获取

**设置流程变量**

流程变量的设置方式有两种，一是通过基本类型设置，第二种是通过JavaBean类型设置。

**（1）基本类型**

```
	public void setProcessVariables(){
		String processInstanceId = "1301";//流程实例ID
		String assignee = "张三";//任务办理人
		//查询当前办理人的任务ID
		Task task = taskService.createTaskQuery()
				.processInstanceId(processInstanceId)//使用流程实例ID
				.taskAssignee(assignee)//任务办理人
				.singleResult();
		//设置流程变量【基本类型】
		taskService.setVariable(task.getId(), "请假人", assignee);
		taskService.setVariableLocal(task.getId(), "请假天数",3);
		taskService.setVariable(task.getId(), "请假日期", new Date());
	}
```

对应数据库表：act_ru_variable

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904182451.png)

**（2）JavaBean类型**

```
public class Person{
	private Integer id;
	private String name;
	private String education;
}
```

然后，通过JavaBean设置流程变量。这里要注意的是，Javabean类型设置获取流程变量，除了需要这个javabean实现了Serializable接口外，还要求流程变量对象的属性不能发生编号，否则抛出异常。 

```java
public void setProcessVariables(){
    String processInstanceId = "1301";//流程实例ID
    String assignee = "张三";//任务办理人
    //查询当前办理人的任务ID
    Task task = taskService.createTaskQuery()
        .processInstanceId(processInstanceId).taskAssignee(assignee).singleResult();
    //设置流程变量【javabean类型】
    Person p = new Person();
    p.setId(1);
    p.setName("周江霄");
    taskService.setVariable(task.getId(), "人员信息", p);
}
```

数据库对应表：act_ru_variable，细心的你可以看到，通过JavaBean设置的流程变量，在act_ru_variable中存储的类型为serializable，变量真正存储的地方在act_ge_bytearray中

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904182524.png) 

 **获取流程变量**

**（1）基本类型**

```java
public void getProcessVariables(){
		String processInstanceId = "1301";//流程实例ID
		String assignee = "张三";//任务办理人
		TaskService taskService = processEngine.getTaskService();
		//获取当前办理人的任务ID
		Task task = taskService.createTaskQuery()
				.processInstanceId(processInstanceId)
				.taskAssignee(assignee)
				.singleResult();
		String person = (String) taskService.getVariable(task.getId(), "请假人");
		Integer day = (Integer) taskService.getVariableLocal(task.getId(), "请假天数");
		Date date = (Date) taskService.getVariable(task.getId(), "请假日期");
		System.out.println(person+"  "+day+"   "+date);
}
```

**（2）JavaBean类型**

```java
/**获取流程变量*/
	@Test
	public void getProcessVariables(){
		String processInstanceId = "1301";//流程实例ID
		String assignee = "张三";//任务办理人
		//获取当前办理人的任务ID
		Task task = taskService.createTaskQuery()
				.processInstanceId(processInstanceId)
				.taskAssignee(assignee)
				.singleResult();
		//获取流程变量【javaBean类型】
		Person p = (Person) taskService.getVariable(task.getId(), "人员信息");
		System.out.println(p.getId()+"  "+p.getName());
		System.out.println("获取成功~~");
	}
```

  **查询历史流程变量** 

```
/**查询历史的流程变量*/
public void getHistoryProcessVariables(){
	List<HistoricVariableInstance> list = processEngine.getHistoryService()
				.createHistoricVariableInstanceQuery()//创建一个历史的流程变量查询
				.variableName("请假天数").list();
	if(list != null && list.size()>0){
		for(HistoricVariableInstance hiv : list){
				System.out.println(
				hiv.getTaskId()+"  "+hiv.getVariableName()+"
				"+hiv.getValue()+"		"+hiv.getVariableTypeName());
		}
	}				
}
```

对应数据库表：act_ru_execution

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904182542.png)

### 4.8 查询所有的正在执行的任务 

```java
public void testQueryTask(){
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        List<Task> tasks = processEngine.getTaskService()
                .createTaskQuery()
                .list();
        for (Task task : tasks) {
            System.out.println(task.getName());
        }
}
```

### 4.9 完成任务

```
public void complete(){
		Task task=taskService.createTaskQuery()
            .processInstanceId(pi.getId()).taskDefinitionKey("task").singleResult();
		taskService.setVariable(task.getId(), "var1", "var1");
         taskService.complete(task.getId());
 }
```

以上代码是查询流程本次执行实例下名为task1的任务

给任务设置全局变量，如果调用的是taskService.setVariableLocal方法，则任务执行完毕后，相关变量数据就会删除，然后再完成任务。

首先向act_ru_variable表中插入变量信息，包含本次流程执行实例的两个id外键，但不包括任务的id，因为setVariable方法设置的是全局变量，也就是整个流程都会有效的变量

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904182930.png)     

此时整个流程执行完毕，act_ru_task，act_ru_execution和act_ru_variable表全被清空

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904182951.png)

## 五、历史任务

### 5.1 查已完成任务和当前在执行的任务 

```java
public void findHistoryTask(){
        ProcessEngine defaultProcessEngine = ProcessEngines.getDefaultProcessEngine();
        //如果只想获取到已经执行完成的，那么就要加入completed这个过滤条件
        List<HistoricTaskInstance> historicTaskInstances1 = defaultProcessEngine.getHistoryService()
                .createHistoricTaskInstanceQuery()
                .taskDeleteReason("completed")
                .list();
        //如果只想获取到已经执行完成的，那么就要加入completed这个过滤条件
        List<HistoricTaskInstance> historicTaskInstances2 = defaultProcessEngine.getHistoryService()
                .createHistoricTaskInstanceQuery()
                .list();
        System.out.println("执行完成的任务：" + historicTaskInstances1.size());
        System.out.println("所有的总任务数（执行完和当前未执行完）：" +historicTaskInstances2.size());
    }
```



Activiti 个人任务（三种指派方式） https://blog.csdn.net/caoyue_new/article/details/52180539



六：关于流程实例的相关API

**涉及到的表：**   

**act_hi_actinst  **

1、说明 *         act:activiti  *         hi:history  *         actinst:activity instance  *            流程图上出现的每一个元素都称为activity  *            流程图上正在执行的元素或者已经执行完成的元素称为activity instance  *      2、字段 *         proc_def_id:pdid  *         proc_inst_id:流程实例ID  *         execution_id_:执行ID  *         act_id_:activity  *         act_name  *         act_type  

**act_hi_procinst **

**      1、说明 *         procinst:process instance  历史的流程实例 *            正在执行的流程实例也在这张表中 *         如果end_time_为null，说明正在执行，如果有值，说明该流程实例已经结束了 *    _

**act_hi_taskinst**

     1、说明 *          taskinst:task instance  历史任务 *             正在执行的任务也在这张表中 *             如果end_time_为null,说明该任务正在执行 *             如果end_time不为null,说明该任务已经执行完毕了 *    **act_ru_execution**

_      1、说明 *         ru:runtime  *         代表正在执行的流程实例表 *         如果当期正在执行的流程实例结束以后，该行在这张表中就被删除掉了，所以该表也是一个临时表 *      2、字段 *         proc_inst_id_:piid  流程实例ID，如果不存在并发的情况下，piid和executionID是一样的 *         act_id:当前正在执行的流程实例(如果不考虑并发的情况)的正在执行的activity有一个，所以act_id就是当前正在执行的流程实例的正在执行的 *           节点 *    **act_ru_task**_

1、代表正在执行的任务表    该表是一个临时表，如果当前任务被完成以后，任务在这张表中就被删除掉了 

2、字段 

 id_:  主键    任务ID      execution_id_:执行ID    *              根据该ID查询出来的任务肯定是一个 *          proc_inst_id:piid  *              根据该id查询出来的任务 *                 如果没有并发，则是一个 *                 如果有并发，则是多个 *          name_:任务的名称 *          assignee_:任务的执行人**



## 六、工作流采购流程如何与业务关联

实例.采购流程的监控

- 查询当前获任务和业务关联

- 查询已经结束的流程
- 查询当前采购流程节点的位置图展示（在流程定义的节点上标出当前节点的位置，使用红色的框标出）
- 查询某个流程下的历史任务（从流程开始到运行结束）
- 查询某个用户所办理的历史任

2.流程变量

- 全局变量
- 局部变量

3.连线分支

设置连线的condition条件实现分支

### 6.1 流程定义图的画法

流程图注意的东西

（1）流程定义key  

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904182636.png)

（2）流程变量 ：

分支条件：**$**(price>=1000) 和**$**(price<1000)

![1567592826450](C:\Users\Administrator\AppData\Local\Temp\1567592826450.png)

（3）人员设置流程变量

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904182721.png)

其他和部门经理的设置方法一样

部门经理审批的代办人流程变量  **$**{u} 

总经理审批的代办人流程变量  **$**{m} 

财务审批的代办人流程变量  **$**{c} 

（4）order.bpmn文件

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:dc="http://www.omg.org/spec/DD/20100524/DC" xmlns:di="http://www.omg.org/spec/DD/20100524/DI" xmlns:tns="Examples" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" expressionLanguage="http://www.w3.org/1999/XPath" id="m1539757531057" name="" targetNamespace="Examples" typeLanguage="http://www.w3.org/2001/XMLSchema">
  <process id="orderKey" isClosed="false" isExecutable="true" processType="None">
    <startEvent id="_2" name="StartEvent"/>
    <userTask activiti:assignee="#{u}" activiti:exclusive="true" id="_3" name="部门经理审批"/>
    <sequenceFlow id="_7" sourceRef="_2" targetRef="_3"/>
    <userTask activiti:assignee="${c}" activiti:exclusive="true" id="_4" name="财务审批"/>
    <userTask activiti:assignee="${m}" activiti:exclusive="true" id="_5" name="总经理审批"/>
    <endEvent id="_6" name="EndEvent"/>
    <sequenceFlow id="_8" sourceRef="_4" targetRef="_6"/>
    <sequenceFlow id="_9" sourceRef="_3" targetRef="_4">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${price < 1000} ]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="_10" sourceRef="_3" targetRef="_5">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${price >= 1000} ]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="_11" sourceRef="_5" targetRef="_4"/>
  </process>
  <bpmndi:BPMNDiagram documentation="background=#32424A;count=1;horizontalcount=1;orientation=0;width=842.4;height=1195.2;imageableWidth=832.4;imageableHeight=1185.2;imageableX=5.0;imageableY=5.0" id="Diagram-_1" name="New Diagram">
    <bpmndi:BPMNPlane bpmnElement="orderKey">
      <bpmndi:BPMNShape bpmnElement="_2" id="Shape-_2">
        <dc:Bounds height="32.0" width="32.0" x="100.0" y="10.0"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="32.0" width="32.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="_3" id="Shape-_3">
        <dc:Bounds height="55.0" width="85.0" x="75.0" y="95.0"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="55.0" width="85.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="_4" id="Shape-_4">
        <dc:Bounds height="55.0" width="85.0" x="70.0" y="220.0"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="55.0" width="85.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="_5" id="Shape-_5">
        <dc:Bounds height="55.0" width="85.0" x="295.0" y="95.0"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="55.0" width="85.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="_6" id="Shape-_6">
        <dc:Bounds height="32.0" width="32.0" x="95.0" y="380.0"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="32.0" width="32.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNEdge bpmnElement="_7" id="BPMNEdge__7" sourceElement="_2" targetElement="_3">
        <di:waypoint x="116.0" y="42.0"/>
        <di:waypoint x="116.0" y="95.0"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="_8" id="BPMNEdge__8" sourceElement="_4" targetElement="_6">
        <di:waypoint x="111.0" y="275.0"/>
        <di:waypoint x="111.0" y="380.0"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="_9" id="BPMNEdge__9" sourceElement="_3" targetElement="_4">
        <di:waypoint x="115.0" y="150.0"/>
        <di:waypoint x="115.0" y="220.0"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="_11" id="BPMNEdge__11" sourceElement="_5" targetElement="_4">
        <di:waypoint x="340.0" y="150.0"/>
        <di:waypoint x="340.0" y="190.0"/>
        <di:waypoint x="155.0" y="247.5"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="_10" id="BPMNEdge__10" sourceElement="_3" targetElement="_5">
        <di:waypoint x="160.0" y="122.5"/>
        <di:waypoint x="295.0" y="122.5"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>

```

> 关键点  <process id="orderKey" isClosed="false" isExecutable="true" processType="None">

###  6.2流程定义的部署

activitiService.java

```java
public void deployByClassPath(String bpmnName) {
        Deployment deploy =       repositoryService.createDeployment().addClasspathResource(bpmnName+".bpmn").deploy();
        printDeploy(deploy);
    }
```

```java
@RequestMapping("/deploy")
@ResponseBody
public String deploy(String bpmnName) {
    activitiService.deployByClassPath(bpmnName);
    activitiService.queryDeployList();
    return "deploy";
}
```

###  6.3 流程实例启动时关联业务

使用activiti自带表act_ru_execution中的BUSINESS_KEY字段我存在业务的唯一表示 

```java
/**
* 流程变量
* 给<userTask activiti:assignee="#{u}" activiti:exclusive="true" name="部门经理审批"/>的u赋值
* 给<userTask activiti:assignee="#{m}" activiti:exclusive="true" name="总经理审批"/>的u赋值
* 给<userTask activiti:assignee="#{c}" activiti:exclusive="true" name="财务审批"/>的u赋值
* 给<conditionExpression xsi:type="tFormalExpression"><![CDATA[${price < 1000}]]>
* </conditionExpression>的price赋值
* 同时设置业务key bussinessKey
*/
public <T> void startProcess(String processDefinitionId,T t){
    HashMap<String, Object> map = new HashMap<>();
    map.put("u", "u");
    map.put("m", "m");
    map.put("c", "c");
    map.put("price", "1200");// 分别测试流程走的分支条件 map.put("price", "300");
    runtimeService.startProcessInstanceByKey(key, map);
    String bussinessKey= t.getClass().getName()+":"+t.getId();
    runtimeService.startProcessInstanceById(processDefinitionId,bussinessKey,map);
}
```

启动流程是第二个参数就是表act_ru_execution中的BUSINESS_KEY字段，我一般喜欢使用业务名+ id来存储当前业务

###  6.4 查询当前获任务和业务关联

通过task获得当前流程，通过当前流程获取当前流程的业务id

查询属性：流程实例id，当前节点，采购名称，采购金额

```java
/**
     * 
     * @param assignee 任务办理人
     */
public void getTaskDetailByAassignee(String assignee){
    List<Task> tasks = taskService.createTaskQuery().taskAssignee(assignee).list();
    for (Task task : tasks) {
        String processInstanceId = task.getProcessInstanceId();
        ProcessInstance processInstance = runtimeService.createProcessInstanceQuery().processInstanceId(processInstanceId).singleResult();
        String businessKey = processInstance.getBusinessKey();
        //当前运行的流程节点
        String activityId = processInstance.getActivityId();
        System.out.println("processInstanceId:"+processInstanceId+" businessKey:"+businessKey+" activityId:"+activityId);
        //业务主键id
        Integer id;
        if(StringUtils.isNotEmpty(businessKey)){
            id=Integer.parseInt(businessKey.split(":")[1]);
            //根据id查询出业务信息
        }
    }
}
```



###  6.5 查询已经结束的流程

通过task获得当前流程，通过当前流程获取当前流程的业务id

查询属性：流程实例id，执行开始时间，执行结束时间，采购名称，采购金额



###  6.6 包含流程变量条件的流程



###  6.6 对采购单的金额进行统计查询 如统计金额的总和

（1）先查询已经结束的流程，用关联sql的方式 不好

（2）最直接的方法，只从业务系统中查询采购单的信息，并统计。因为统计的数据是业务数据，业务系统只负责业务数据。但是面临的一个问题是，并不知道业务系统中的哪些采购单是已经结束了的。

如果，在采购单业务表中加入一个字段status，则表中有采购单结束的标识。

实现方法：

**方法1：** taskListener监听器的方法

在流程定义的最后一个节点定义一个监听器，此监听器在完成任务的时候执行，在监听器中更新采购单业务表中的status字段的值，如：complete完成。

**方法2：**executionListener监听器的方法

在endevent节点上添加executionListener监听器的方法，监听事件选择end

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904182803.png)



## 七、业务开发总结

觉得不需要用activitEntity保存那么多的属性id  processInstanceId和 只需要  查询task列表的时候可以返回一个List<Hashmap>，其实可以不需要 因为有businessEntity

1.新建一个activitEntity.java包含两个字段id和流程实例Id

```
public ActivitEntity{
	private String id;
	private String processInstanceId;
}
```

2.业务表单继承ActivitEntity基类

3.保存业务表单的时候启动一个流程，创建采购单时，在填写新增记录保存时，启动流程实例。

4.业务service层执行的时候，要确定业务key，uuid以及启动流程之后返回的流程实例id

5.启动流程的时候，如果用流程定义Id的话，感觉不太好，应为部署的问题，所以建议使用startBykey的方式

```
public ActivitService{
	private void saveOrder(){
        Order order=new Order();
        String id=UUid.gen();
        order.setId(id);
        //String businessKey=Order.getClass().getName()+":"+id;//业务key 用类名和id组合 order:id
        Map<String, Object> variables=new Hash<>();
        //String processDefinitionId="11";
        String processDefinitionKey="从部署bpmn文件中获取"  //有工具类可以获取到这key 网络搜素一下
        //调用服务的方式
        //ActivitClient.startProcess(processDefinitionId,order,variables)
        ActivitClient.startProcess(processDefinitionKey,order,variables)
	}
}


public <T extends ActivitEntity> void startProcess(String processDefinitionKey,T t,Map<String, Object> variables){
        String bussinessKey= t.getClass().getName()+":"+t.getId(); //业务key 用类名和id组合 order:id
       
         //runtimeService.startProcessInstanceById(processDefinitionId,bussinessKey,variables);
         runtimeService.startProcessInstanceByKey(processDefinitionKey,bussinessKey,variables);
}
```

6.为防止对采购单的业务数据进行修改，所以，创建采购单的人要提交申请，在提交申请之前，是可以对数据进行修改的，提交申请之后，数据不能修改，同时任务流向下一个节点。

7.待提交的采购单如何查询

利用activiti的taskservice查询出当前用户的代办任务

任务的名称，任务的待办人，任务申请人，任务提交的时间，采购金额，采购类型

8.提交采购单  当前任务的taskid  taskService.coomplete(taskid);



9.审核业务（另起数据表--审核表---字段包含 id,采购单,采购申请人,采购类型，审核意见，审核状态0不同意，1通过，审核时间）

进入审核页面  填写审核信息  提交审核  

业务逻辑 ：保存审核意见到审核表中，然后再用activit完成任务









