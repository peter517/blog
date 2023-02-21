title: Java类模板 约束验证
date: 2018-12-12 15:08:50
tags: [Java,Tools]
description: 介绍java-object-field-check库，利用Java注解定义一个模板，表示类属性标准，验证Json或者类实例是否符合要求

----



# 解决问题

在许多服务中，需要对客户端传入的字段进行验证，比如下面这个API请求为一个Json串

```Json
{
  "version":"1.0.0",
  "student":{
  "name":"Jack",
  "age":18,
  "hobby":"basketball"
  },
}
```

对参数有如下要求:

- key为“version”对应一个版本号，不能为空
- key为“student”对应的value是一个Json对象
- key为“name”的value对应是String类型
- key为“age”的value对应是Integer类型，且不能为负数
- key为“hobby”的value对应是String类型，但允许为空

一般的会把这些约束写到代码中，这样，代码里面出现很多if else语句，十分冗余和难以维护，Json对象t代表整个结构体，如下：

```Java
if (!jsonObject.containsKey("version") || jsonObject.getString("version").equals("")) {
    throw new Exception("version can't be empty");
}
if (!jsonObject.containsKey("student")) {
    throw new Exception("name can't be empty");
}
if (!studentJsonObject.containsKey("name")) {
    throw new Exception("name can't be empty");
}
if (!studentJsonObject.containsKey("age")) {
    throw new Exception("name can't be empty");
}
if(!studentJsonObject.getInteger("age") < 0) {
    throw new Exception("age can't be < 0");
}
```

这时候，就可以利用Java注解定义一个模板，表示类属性约束，验证Json是否符合要求，java-object-field-check这个库就是用来解决这个问题

# 现有方案

Java中有很多开源的validator，比如：

- Apache中的[commons-validator](http://commons.apache.org/proper/commons-validator/)，通过XML来验证Bean的属性约束

- J2EE中的[Bean Validation](http://docs.oracle.com/javaee/6/tutorial/doc/gircz.html)，提供一种注解的方案来验证Bean的属性约束

- Android的[android.support.annotation](https://developer.android.com/studio/write/annotations?hl=zh-CN)，一种注解的平台定制化方案，比如有对方法有权限类的注解，RequiresPermission

  

java-object-field-check也是通过注解的方案来验证Bean的属性约束，它支持通过验证Json对象得到类实例，也支持通过验证类实例得到Json对象，优势如下：

- 注解种类更多，比如支持对嵌套对象、列表的支持注解
- 功能更加丰富，比如自定义验证方法，正则表达的支持
- 更加易于使用，一行代码进行验证，验证识别可以在异常类中获取信息



# java-object-field-check接口

从上面的例子可以看出，FieldCheck的调用都是通过静态方法调用，一行代码搞定，支持验证三类参数：

- Json对象是否符合模板要求
- Json字符串是否符合模板要求
- Object实例是否符合模板要求

所有接口如下，Class表示模板对应的类型信息：

```java
 /**
  * @param c the template of class
  * @param jsonObject json object about class
  * @return the instance of class
  * @throws FieldCheckException
  */
 public static Object checkObjectByJsonObject(Class c, JSONObject jsonObject) throws FieldCheckException;
 /**
  * @param c the template of class
  * @param jsonString json string about class
  * @return the instance of class
  * @throws FieldCheckException
  */
 public static Object checkObjectByJson(Class c, String jsonString) throws FieldCheckException;
 /**
  * @param object the instance of class
  * @return json object about class
  * @throws FieldCheckException
  */
 public static JSONObject checkObjectByInstance(Object object) throws FieldCheckException;
```

验证Object实例的代码和验证Json的代码类似，验证一个对象实例来，如果验证通过，则获得对应的Json对象，可以用在API的返回内容中

```java
Request request = new Request();
request.name = "Jack";
request.age = 18;
request.hobby = "basketball";

JSONObject requestJsonObject;
try {
    requestJsonObject = FieldCheck.checkObjectByInstance(request);
} catch (FieldCheckException e) {
    //TODO
    logger.info("field check error " + e.getInfo());
}
```



# 一个例子

为了解决上面的问题，我们可以定义这么一个类：

```Java
public class Request {

    @CheckString(allowEmpty = false)
    private String version;
    
    @CheckObject
    private Student student;

    public static class Student{
        @CheckString
        private String name;

        @CheckInteger(allowNegative = false)
        private Integer age;

        @CheckString(allowEmpty = true)
        private String hobby;
    }
    
    public String getVersion() {
        return version;
    }

    public void setVersion(String version) {
        this.version = version;
    }

    public Student getStudent() {
        return student;
    }

    public void setStudent(Student student) {
        this.student = student;
    }
    
}
```

模板表示的内容为：

- 注解version字段不能为空，allowEmpty = false也可以不写，是默认值
- 注解CheckObject表示key为“student”对应的value是一个Json对象
- name上的注解CheckString表示key为“name”的value对应是String类型
- age上的注解CheckInteger表示key为“age”的value对应是Integer类型，注解的属性allowNegative = false表示该元素不能为负数
- hobby上的注解CheckString表示：key为“hobby”的value对应是String类型，注解的属性allowEmpty = true表示可以为空

定义好这个模板后，就可以用java-object-field-check来验证这个每个Json是否满足约束，通过工具类FieldCheck的静态方法checkObjectByJsonObject来验证。

如果满足需求，则返回已从json串提取内容的类实例，否则抛出FieldCheckException异常，异常中可以打印哪个字段发生验证错误

如果是Post请求，如下body为requestJsonObject，一个json对象，验证方法为：

```Java
@RequestMapping(value = "/xxx/xxx", method = RequestMethod.POST)
public @ResponseBody void postRequest(@RequestBody JSONObject requestJsonObject, HttpServletResponse response) {
	Request request = null;
	try {
    	request = (Request) FieldCheck.checkObjectByJsonObject(Request.class, requestJsonObject);
	} catch (FieldCheckException e) {
    	//TODO
    	logger.info("field check error " + e.getInfo());
	}
}
```

如果是Get请求，先忽略student字段验证，如下：

```java
@RequestMapping(value = "/xxx/xxx", method = RequestMethod.GET)
public @ResponseBody void getRequest(@RequestParam("version") String version, HttpServletResponse response) {
	Request request = new Request();
	request.setVersion("xxx");
	try {
    	FieldCheck.checkObjectByInstance(Request.class, request);
	} catch (FieldCheckException e) {
    	//TODO
    	logger.info("field check error " + e.getInfo());
	}
}
```



# Java注解类型

对于不同的类型，注解属性也有一些不同

## Field

Field注解里面有一些属性，可以表示一个类型的一些常见约束

### String类型

| 名称                 | 类型        | 描述                   | 默认值                            |
| -------------------- | ----------- | ---------------------- | --------------------------------- |
| optional             | boolean     | 是否可选               | false                             |
| allowEmpty           | boolean     | 字符串内容是否可以为空 | false                             |
| availableValue       | array       | 有效值集合             | {}                                |
| availableLengthRange | LengthRange | 有效长度               | {min = 0,max = Integer.MAX_VALUE} |
| regexp               | Regexp      | 匹配正则表达           | {value = "",errorInfo=""}         |

实例：

```Java
@CheckString(availableValue = {"1", "2"},
             availableLengthRange = @LengthRange(min = 2, max = 4))
private String key0;
```

### Boolean类型

| 名称           | 类型    | 描述       | 默认值 |
| -------------- | ------- | ---------- | ------ |
| optional       | boolean | 是否可选   | false  |
| availableValue | array   | 有效值集合 | {}     |

实例：

```Java
@CheckBoolean(optional = true, availableValue = {true})
private Boolean key0;
```

### Number类型

主要包括Integer、Short、Long、Float、Double，下面以Long为例子

| 名称           | 类型    | 描述           | 默认值         |
| -------------- | ------- | -------------- | -------------- |
| optional       | boolean | 是否可选       | false          |
| max            | long    | 最大值         | Long.MAX_VALUE |
| min            | long    | 最小值         | Long.MIN_VALUE |
| allowNegative  | boolean | 是否可以为负数 | true           |
| availableValue | array   | 有效值集合     | {}             |

实例：

```Java
@CheckLong(allowNegative = false, min = 1, max = 300)
private Long key0;
```

### JsonObject类型

| 名称     | 类型    | 描述     | 默认值 |
| -------- | ------- | -------- | ------ |
| optional | boolean | 是否可选 | false  |

实例：

```Java
@CheckObject
private Student student;
```

### JsonArray类型

| 名称                 | 类型        | 描述     | 默认值                            |
| -------------------- | ----------- | -------- | --------------------------------- |
| optional             | boolean     | 是否可选 | false                             |
| availableLengthRange | LengthRange | 有效长度 | {min = 0,max = Integer.MAX_VALUE} |

实例：

```Java
@CheckArray(availableLengthRange = @LengthRange(min = 2, max = 4))
private List<Student> student;
```


## Method

Field中的常见约束无法满足时，可以使用Method注解来自定义需求

### JsonCheckRule

| 名称      | 类型   | 描述             |
| --------- | ------ | ---------------- |
| errorInfo | String | 验证错误提示信息 |

JsonCheckRule用来在验证Json或者Json字符串时，即调用checkObjectByJsonObject或者checkObjectByJson时，提供约束规则，**参数必须JSONObject类型。**如key1Rule方法，用来验证key1字段的合法性，输入是被解析后的整个Json对象，返回true表示验证通过，返回false表示验证失败，errorInfo用来返回错误信息。具体使用如下：

```Java
public class InterObject {
        @CheckString
        private String key1;

        @JsonCheckRule(errorInfo = "key1 should not be 11")
        public Boolean key1Rule(JSONObject jsonObject) throws FieldCheckException {
            return jsonObject.getString("key1").equals("11") == false;
        }
    }
```

### ObjectCheckRule

| 名称      | 类型   | 描述             |
| --------- | ------ | ---------------- |
| errorInfo | String | 验证错误提示信息 |

ObjectCheckRule用来在验证Object实例时，即调用checkObjectByInstance时，提供约束规则，**参数必须为空。**如key2Rule方法，验证key2字段的合法性，返回true表示验证通过，返回false表示验证失败。具体使用如下：

```Java
public class InterObject {
        @CheckString
        private String key2 = "1";

        @ObjectCheckRule(errorInfo = "key2 should not be 12")
        public Boolean key2Rule() throws FieldCheckException {
            return key2.equals("12") == false;
        }

    }
```
