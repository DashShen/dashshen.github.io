---
title: morphline的tryRules命令的使用
date: 2017-01-18 20:32:08
tags:
---

在使用`Flume`做`etl`的时候，会使用`morphline interceptor`来做具体的工作。其中，`morphline`提供了一个`if`的`command`来达到对于某一具体的事件来做解析。但是这个`if`并不是和我们平时码代码的时候一样，比如说平时我们使用，`if`的时候，会写**卫语句**

```java
if a == 1:
    do_process_a()
if a == 2:
    do_process_b()
if a == 3:
    do_process_c()
```
可是，`morphline`的`if`虽然可以这么写，但并不能得到我们想要的**卫语句**的效果，一开始使用的时候，这是由于这一点然我在使用的是踩了很多坑，为了达到**卫语句**效果，要写的就需要这么写
```conf
{
    if {
        conditions : [
            { equals : { key : "condition-a" } }
        ]
        then : [
            { logInfo { format : "process condition a" } }
        ]
        else : [
            {
                if {
                    conditions : [
                        { equals : { key : "condition-b" } }
                    ]
                    then : [
                        { logInfo { format : "process condition b" } }
                    ]
                    ...
                }
            }
        ]
    }
}
```
这样写下去，就会写成一个地狱级的条件判断，这样子太难看了。因此，我发现了`morphline`提供了一个叫做`tryRules`的命令，他可以提供**卫语句**的写法，`tryRules`包括了一系列的命令，如果其中一个命令执行成功了，则其他命令就跳过不会被执行，比如说上面的写法，就可以用`tryRules`这么写。
```conf
{
    tryRules {
        commands : [
            { equals : { key : "condition-a" } }
            { logInfo { format : "process condition a" } }
        ]
        commands : [
            { equals : { key : "condition-b" } }
            { logInfo { format : "process condition b" } }
        ]
        ....
    }
}
```
这样子写，一下子就从地狱深度的条件判断，一下就写成了简洁明了的**卫语句**的形式。