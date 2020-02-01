# 1 配置文件
使用spring initializr创建spring boot工程时，默认的app全局配置文件是：src/main/resources/application.properties，但是，还支持src/main/resources/application.yaml，配置的**名字是固定的**。(很久之前，大多使用XML文件作为配置文件，现在仍然有这中方式。)目前，主流使用yaml书写配置文件。

配置文件的作用是：修改springboot的默认配置（auto configuration）
## 1.1 YAML语法
* key:[**空格**]value

* 层级关系划分：左端对齐的为同一级，上下级之前可以用一个或者多个空格进行缩进划分

* 大小写敏感

* value的类型：

字面量（数字、字符串、布尔）：	字面量所见即所得，直接写即可。但是，双引号的和单引号包括字符串时有稍有区别。

​	英文双引号：不会进行任何转译，真正的所见即所得

​	引文单引号：会像java中的字符串输出，进行转译



对象（属性和值）、Map（键值对）：对象直接写成key:[空格]value的形式，比如：

```yaml
person:
	name: jeason
	age: 16
	country: China
```

或者

```yaml
person: {name: jeason, age: 16, country: China}
```



数组（List、Set）：

使用一个短横表示的数组的中的一个元素的，短横和value之间也必须有空格，比如：


```yaml
animalList:
    - cat
    - dog
    - monkey
```

# 2 加载顺序

# 3 配置原理