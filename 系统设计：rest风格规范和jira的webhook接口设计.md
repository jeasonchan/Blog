# 1 背景

参考文档：

jira官方webhook接口文档    https://developer.atlassian.com/server/jira/platform/webhooks/

pinbox中的rest接口规范文档


# 2 jira中的webhook接口
jira中，webhook用于给第三方应用订阅jira中的变化事件，rest风格的请求以资源为中心，这个实例中webhook就是资源。

## 2.1 增
POST  <JIRA_URL>/rest/webhooks/1.0/webhook

```json
{
"name": "my first webhook via rest",
"url": "http://www.example.com/webhooks",
"events": [
  "jira:issue_created",
  "jira:issue_updated"
],
"filters": {
	"issue-related-events-section": "Project = JRA AND resolution = Fixed"
},
"excludeBody" : false
}
```

返回值就是新增的webhook资源对象的信息，比如ID啥的。

其中，的url就是第三方系统提供的接口，第三方系统通过url接收到变化后，自行处理。

这个行为也可以在jira系统通过图形界面的方式进行手动增加。

## 2.2 删

DELETE  <JIRA_URL>/rest/webhooks/1.0/webhook/{id of the webhook} 

## 2.3 查

查询所有的资源

GET  <JIRA_URL>/rest/webhooks/1.0/webhook

查询某一个的特定资源

GET <JIRA_URL>/rest/webhooks/1.0/webhook/<webhook ID>

## 2.4 改

PUT  <JIRA_URL>/rest/webhooks/1.0/webhook/{id of the webhook}

```json
{
"name": "my first webhook via rest",
"url": "http://www.example.com/webhooks",
"events": [
  "jira:issue_created",
  "jira:issue_updated"
],
"filters": {
	"issue-related-events-section": "Project = JRA AND resolution = Fixed"
},
"excludeBody" : false
}
```

消息体要用和创建时用到的所有属性，此时提交的属性会覆盖原先的值，是一种"赋值"操作

## 2.5 jira中通用的表示结构数据变化的Body

