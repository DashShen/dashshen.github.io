---
title: morphlne-interceptor使用解析
date: 2017-04-22 21:09:01
tags:
---
# morphline interceptor
morphline interceptor是Flume中的一个灵活的数据ETL拦截器，类似于**logstash**中的`filter`。本身提供了许多的命令用于进行数据解析，并且也可以根据需要开发自己定制的命令。

## morphlines文件的结构

morphilnes配置文件采用[HOCON](https://github.com/typesafehub/config/blob/master/HOCON.md)语法来进行配置文件的编写，其中每一个morphlines的配置文件可以包含多个morphline,也就是morphline command的集合。每个morphline都有一个唯一的Id用于标识, 以及commands配置具体的命令执行流程,命令的执行会根据commands里面配置的情况，从上到下依次执行。

```hocon
morphlines : [
   {
       # morphline唯一标识
       id : morphline1
       # 导入morhline command jar包，包括自定义命令的jar包
       importCommands : [ "org.kietesdk.**", "com.aispeech.libra.morphlines.**"]

       commands : [
           {
               readJson {}
           }
           ....
       ]
   } 

   {
        id : morphline2
        importCommands : ["org.kietesdk.**", "com.aispeech.libra.morphlines.**"]
        commands : [
            {
                readLine {
                    charset : UTF-8
                }
            }
            .....
        ]
   }
]
```


## 使用
* 在flume配置文件中，指定morphlines文件，并且指定具体morphline

```
agent.sources.s1.interceptors = morphlineinterceptor
agent.sources.s1.interceptors.morphlineinterceptor.type = org.apache.flume.sink.solr.morphline.MorphlineInterceptor$Builder
# 配置morphlines文件路径
agent.sources.s1.interceptors.morphlineinterceptor.morphlineFile = /opt/flume/conf/morphline.conf
# 指定具体morphline
agent.sources.s1.interceptors.morphlineinterceptor.morphlineId = morphline1
```

# 部分命令分析


## readJson 
使用jackson库，读取包含json内容的`InputStream`或者`byte array`,读取后的结果会放在输出的record的body当中。
* 参数列表

参数 | 默认值 | 描述
----| ----- | -----
outputClass | com.fasterxml.jackson.databind.JsonNode | 读取之后转换为的Java对象，默认为JsonNode


* 使用样例

```
{
    readJson {}
}
```
* 将输入的json数据转换为Map
```
{
    readJson { 
        outputClass : java.util.Map
    }
}
```

## tryRules
`tryRules`专门用于处理各种格式数据的一套规则引擎命令，此命令可以包含零个或者多个个规则，每个规则又可以包含零个或者多个命令。规则按照从上之下的顺序进行执行，
如果当前规则无法处理这个数据，则这条数据会交给下一个规则去执行。如果一条数据被其中的一个规则成功处理了，那么剩下的所有规则都会被跳过执行。

* 参数列表

参数 | 默认值 | 描述
----| ----- | -----
catchExceptions | false | 是否捕获异常，并且交给传播至执行链上
throwExceptionIfAllRulesFailed | true | 如果没有规则执行成功，是否抛出异常

* 使用样例
```
tryRules {
  catchExceptions : false
  throwExceptionIfAllRulesFailed : true
  rules : [
        {
            commands : [
                { equals : { key : "foo"} }
                {
                    # do somthinge
                }
            ]
        }
        {
            commands : [
                { equals : { key : "bar"} }
                {
                    # do somthinge
                }
            ]
        }
        {
            commands : [
                { equals : { key : "zoo"} }
                {
                    # do somthinge
                }
            ]
        }
  ]
}
```

## equals
`equals`用于判断字段是否等于指定内容

* 使用样例
```
# 判断key字段中的值是否等于foo
{
    equals  {
        key : "foo" 
    }
}
```

## setValues
`setValues`用于给字段赋值

* 使用样例

```
{
    setValues {
        # 给source_type字段赋值foo和bar
        source_type : [foo, bar]

        # 情况url中的内容
        url : []


        # 将first_name字段中的内容赋值给name
        name : "@{first_name}"
    }
}
```

## split
`split`通过指定分隔符分割字符串，分隔符可以是一个字符，词（字符串），或者是正则表达式，以及`grok`模板

* 参数列表

参数 | 默认值 | 描述
----| ----- | -----
inputField | n/a | 需要进行分割的字段
outputField | null | 存放分割结果的字段, **与outputFields参数不可以同时配置**
outputFields | null | 存放分割结果的字段列表，如[firstName, lastName, "", age], ""空字符串标识，忽略这个位置的内容
separator | n/a | 分隔符
isRegex | false | 分隔符是否是正则表达式或者是`grok`模板
dictionaryFiles | [] | `grok`模板文件的列表
dictionaryString | null | 自定义`grok`模板
trim | true | 是否对结果调用String.trim()方法
addEmptyStrings | false | 是否对结果添加空字符串
limit | -1 | 是否限制结果的最大元素个数，-1代表不限制

* 使用样例
```
{
    split { 
        inputField : message
        outputFields : [first_name, last_name, "", age]          
        separator : ","        
        isRegex : false      
        #separator : """\s*,\s*"""        
        #isRegex : true      
        addEmptyStrings : false
        trim : true          
    }
}
```
* 输入数据
```
message:"Nadja,Redwood,female,8"
```
* 输出数据
```
first_name:Nadja
last_name:Redwood
age:8
```

## toAvro
`toAvro`将record中的body数据，序列化成avro数据

* 参数列表

参数 | 默认值 | 描述
----| ----- | -----
schemaFile | null | avro schema模板文件路径
schemaString | null | avro schema模板的具体内容
schemaField | null | 将record中的某个字段序列化成avro
mappings | [] | 配置record中的字段与schema中的字段的映射关系

* 使用样例

```
{
    toAvro {
        #schemaFile : /path/to/interop.avsc 
        #schemaField : _dataset_descriptor_schema 
        schemaString : """
            {
            "type" : "record",
            "name" : "Rating",
            "fields" : [ 
                {
                "name" : "userId",
                "type" : "int"
                }, 
                {
                "name" : "rating",
                "type" : ["int","null"]
                }, 
                {
                "name" : "reviews",
                "type" : {"type": "array", "items": "string"}
                }, 
                {
                "name" : "history",
                "type" : ["null", {"type": "map", "values":     
                                    {"type": "record", "name": "Foo",
                                        "fields": [{"name": "timestamp", "type": "long"}]}}]
                } 
            ]
            }
        """
        
        mappings : { 
            userId : morphlineUserId
        }          
    } 
}
```

## extractJsonPaths
`extractJsonPaths`使用`XPath`的方式，从json对象中抽取出指定的值。需要注意的是，该命令只能抽取`com.fasterxml.jackson.databind.JsonNode`及其子类的内容。
抽取的内容可以在paths中配置，指定需要抽取的内容，以及存放结果的字段。

* 参数列表

参数 | 默认值 | 描述
----| ----- | -----
flatten | true | 是否以抽取结果是否维持原有的树形结构，默认不维持
paths | [] | 去要抽取的json表达式

* 使用样例
```
{
    extractJsonPaths {
        flatten : true
        paths : {
            my_price : /price
            my_docId : /docId
            my_links : /links
            my_links_backward : "/links/backward"
            my_links_forward : "/links/forward"
            my_name_language_code : "/name[]/language[]/code"
            my_name_language_country : "/name[]/language[]/country"
            my_name : /name    
        }
    }
}
```
    
