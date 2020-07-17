 


哈哈
# 1背景
参考文档1：<https://blog.csdn.net/qq_39295740/article/details/84930254>，比较基础，比较局限，仅仅适用于文章中自己创建出的工作流

参考文档2：<https://blog.csdn.net/chenweifu365/article/details/79032758>，比较详细，涉及了自定义该工作流数据库、使用xml文件配置工作流引擎等

# 2代码实践
## 2.1工作流创建、启动基本流程
此处代码实践主要参考，参考文档1<https://blog.csdn.net/qq_39295740/article/details/84930254>，比较浅显。
```java
package com.zte.工作流练习;

import org.activiti.engine.ProcessEngine;
import org.activiti.engine.ProcessEngineConfiguration;
import org.activiti.engine.RepositoryService;
import org.activiti.engine.RuntimeService;
import org.activiti.engine.repository.Deployment;
import org.activiti.engine.repository.DeploymentBuilder;
import org.activiti.engine.repository.ProcessDefinition;
import org.activiti.engine.repository.ProcessDefinitionQuery;
import org.activiti.engine.runtime.ProcessInstance;

public class Workflow {
    public static ProcessEngine generateProcessEngine() {
        //使用默认的工作流配置模板创建一个工作流引擎配置
        //类似于线程池的创建方式
        ProcessEngineConfiguration processEngineConfiguration =
                ProcessEngineConfiguration.createStandaloneInMemProcessEngineConfiguration();

        //使用配置生成具体的工作流引擎实例
        ProcessEngine processEngine = processEngineConfiguration.buildProcessEngine();

        //给引擎实例做一些修饰和标记
        String engineName = processEngine.getName();
        String engineVersion = processEngine.VERSION;
        System.out.println("引擎名字是：【" + engineName + "】，版本是：【" + engineVersion + "】");
        return processEngine;
    }


    //将resources中的工作流定义文件xml，
    //添加进仓库，并进行部署
    public static String addXmlToRepository(ProcessEngine processEngine, String xmlName) {
        RepositoryService repositoryService = processEngine.getRepositoryService();
        DeploymentBuilder deploymentBuilder = repositoryService.createDeployment();

        //此处直接添加resources中的xml文件
        //还有其他的add方法，添加其他位置或者形式的工作流定义xml文件
        deploymentBuilder.addClasspathResource(xmlName);

        Deployment deployment = deploymentBuilder.deploy();
        return deployment.getId();
    }


    public static ProcessDefinition getProcessDefinition(ProcessEngine processEngine, String deploymentId) {
        RepositoryService repositoryService = processEngine.getRepositoryService();

        //根据部署ID，查询部署的流程定义实例
        ProcessDefinitionQuery processDefinitionQuery = repositoryService.createProcessDefinitionQuery().deploymentId(deploymentId);
        ProcessDefinition processDefinition = processDefinitionQuery.singleResult();//此处只查了一个ID，所以只有一个结果

        String processDefinitionName = processDefinition.getName();
        String processDefinitionId = processDefinition.getId();
        System.out.println("流程定义的名字是：【" + processDefinitionName + "】，ID是：【" + processDefinitionId + "】");
        return processDefinition;
    }

    //启动工作流，返回该工作流对象的实例
    public static ProcessInstance startProcessInstance(ProcessEngine processEngine, ProcessDefinition processDefinition) {
        RuntimeService runtimeService = processEngine.getRuntimeService();
        ProcessInstance processInstance = runtimeService.startProcessInstanceById(processDefinition.getId());
        //也可以使用key启动流程，key就是工作流xml文件里的process id
        System.out.println("启动流程：" + processInstance.getProcessDefinitionKey());
        return processInstance;
    }



}

```


## 2.2工作流创建、启动的更多个性化设置
此处代码实践主要参考，参考文档2<https://blog.csdn.net/chenweifu365/article/details/79032758>，比较浅显。

