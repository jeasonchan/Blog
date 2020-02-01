# 1 配置文件
使用spring initializr创建spring boot工程时，默认的app全局配置文件是：src/main/resources/application.properties，但是，还支持src/main/resources/application.yaml，配置的**名字是固定的**。(很久之前，大多使用XML文件作为配置文件，现在仍然有这中方式。)目前，主流使用yaml书写配置文件。

配置文件的作用是：修改springboot的默认配置（auto configuration）
## 1.1 YAML语法
* key:[**空格**]value

* 层级关系划分：左端对齐的为同一级，上下级之前可以用一个或者多个空格进行缩进划分

* 大小写敏感

* value的类型：

字面量（数字、字符串、布尔）：


对象（属性和值）、Map（键值对）：


数组（List、Set）


```yaml
sever:
    port: 6666
    path: /hello 

```

# 2 加载顺序

# 3 配置原理