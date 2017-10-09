---
title: Airflow飞行指南（一）
date: 2017-02-12 12:26:45
tags: Airflow, 定时任务调度
---
# 介绍
airflow是一个任务调度的工具，通过`python`来编写**DAG**来描述配置工作流，airflow就会根据配置,定时的执行任务,此外Airflow也提供了一个简洁明了的UI界面，用于监控运行情况。此项目是由airbnb贡献给apache协会，到目前为止这个项目还在快速迭代孵化当中，与其类似的项目有apache旗下的[Oozie](http://oozie.apache.org/)和[Azkaban](http://data.linkedin.com/opensource/azkaban)。并且airflow也有其自己的特色：

* airflow可以动态的读取配置文件（`python code`),当配置文件发生变化，airflow会立即生成新的任务调度计划
* airflow本身提供了丰富的`operator`用于定义任务和`executor`用于执行任务，但是我们也可以灵活的拓展符合自己需求的组件
* airflow配置整个工作流可谓是优雅，采用了`Jinja`模板引擎做到了运行脚本的参数化
* airflow采用了模块化的架构，可以配合消息队列来管理多个**worker**,从而做到无限的扩展。

# 安装
airflow的安装及其的简单（由于现在此项目处于快速的更新迭代中，版本会更新的很快，目前安装的是**1.7.1.3**版本），使用下面的命令就可以安装airflow
```
pip install airflow
```
此外还可以安装一些额外的功能的组件
```
pip install "airflow[hive, s3]"
```
之后配置一下，airflow的环境变量，初始化之后，就可以开始编写`DAG`来执行调度任务了

```
export AIRFLOW_HOME=/path/to/airflow

# 初始化元数据库，默认使用*sqlite*,可以之后配置成*mysql*
airflow initdb
```

# 对比
在对比了azkaban, oozie, airflow之后，对三者有了以下的感想：
* azkaban的UI界面做的十分炫酷好看，工作流的定义也是使用`java properties`的方式来定义，简洁明了。但是其每次更新配置，都需要打包压缩成一个** *.zip ** 文件，然后上传更新，这就相对来说不是很方便了，而且azkaban的安装部署也十分的麻烦。
* oozie就更不用说了，xml定义配置工作流，任务资源文件还要放在hdfs上等等缺点，甚至有人说你恨他你就告诉他用oozie
* airflow在对比下，UI界面相对来说没有azkaban那样这么的炫酷，但是在其他方面在三者中都更具优势。由于配置文件采用的`python code`的方式以及其动态更新配置的特性，可以直接通过`git`等版本工具管理起来，十分的方便。
