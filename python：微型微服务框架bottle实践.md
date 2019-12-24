# 1 背景
在公司跑FT，需要一些桩服务来配合整个测试流程，桩服务本身没啥处理逻辑，唯一的要求就是写起来方便的，能快速开发。
发现了Bottle这个框架，感觉还可以，把第一次使用的代码实践贴出来
# 2 代码实践
资源接口类**MyWeb.py**，定义了资源接口，代码时python2的代码，和3语法略有不同！
```python
# coding: utf-8
import json
import logging
import os

from bottle import get, run, post, request, response, put

class MyWeb(object):
    excepted_return = "success"
    base_url = "/api/MyWeb/v1"
    test_url = base_url + "/test"
    set_response_url = base_url + "/set_response"
    send_motep_url = base_url + "/channel"

    @staticmethod
    @post(send_motep_url)
    def send_json_body():
        response.content_type = 'application/json'
        return MyWeb.excepted_return

    @staticmethod
    @put(set_response_url)
    def set_response():
        response.content_type = 'application/json'
        body = request.json  # request对象里的，json属性，是字典类型
        if body is None:
            return {'code': 1, 'message': 'body param is null'}
        if type(body) is not dict:
            return {'code': 1, 'message': 'body param is not dict'}
        MyWeb.excepted_return = body
        # 返回值既可以直接发送字典类型对象，框架支持自动转换
        # 也可以用json.dumpes(object)转成字符串发送，客户端收到的是完全相同的
        return {"code": 0, "message": "success"}

    @staticmethod
    @get(test_url)
    def check_connection():
        response.content_type = 'application/json'
        return {"code": 0, "message": {"excepted_return": MyWeb.excepted_return}}

```
服务启动类，**Main.py**：
```python
# coding=utf-8

import json
import logging
import os

from bottle import get, run, post, request, response

import MyWeb # 必须显式导入，才能使用该类中定义好的资源接口

if __name__ == '__main__':
    logging.info("start pa-tcp-rnc fake service")
    run(host='0.0.0.0', port=8087, debug=True)

```
使用：python Main.py，即可启动微服务，使用可使用postman对上面定义好的接口进行使用。
