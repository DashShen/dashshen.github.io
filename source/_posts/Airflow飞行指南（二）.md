---
title: Airflow飞行指南（二）
date: 2017-02-26 12:38:31
tags:
---
# Airflow的基础使用
下面会举一个airflow的基础使用的例子，可能一上来看一大串代码会让人懵，但是没关系后面会一行行的解释的。
```python
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from datetime import datetime, timedelta


default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': datetime(2015, 6, 1),
    'email': ['airflow@airflow.com'],
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
    # 'queue': 'bash_queue',
    # 'pool': 'backfill',
    # 'priority_weight': 10,
    # 'end_date': datetime(2016, 1, 1),
}

dag = DAG('tutorial', default_args=default_args)

# t1, t2 and t3 are examples of tasks created by instantiating operators
t1 = BashOperator(
    task_id='print_date',
    bash_command='date',
    dag=dag)

t2 = BashOperator(
    task_id='sleep',
    bash_command='sleep 5',
    retries=3,
    dag=dag)

templated_command = """
    {% for i in range(5) %}
        echo "{{ ds }}"
        echo "{{ macros.ds_add(ds, 7)}}"
        echo "{{ params.my_param }}"
    {% endfor %}
"""

t3 = BashOperator(
    task_id='templated',
    bash_command=templated_command,
    params={'my_param': 'Parameter I passed in'},
    dag=dag)

t2.set_upstream(t1)
t3.set_upstream(t1)
```

## 默认参数
当我们创建一个具体的任务（operator)的时候，都会需要传入一些参数，而且会有些参数是共有的。因此我们就可以定义一个设置了公有参数的`dict`,然后这些参数会在构建任务的时候，传递给每一个任务的构造方法。
```python
default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': datetime(2015, 6, 1),
    'email': ['airflow@airflow.com'],
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
    # 'queue': 'bash_queue',
    # 'pool': 'backfill',
    # 'priority_weight': 10,
    # 'end_date': datetime(2016, 1, 1),
}
```

## 实例化DAG 
我们需要有向无环图(DAG)来容纳我们定义的任务，指定他们的运行顺序。实例化DAG时我们需要给他一个唯一标识的`dag_id`,同时把我们刚才定义的默认参数传递给DAG的构造方法。
```
dag = DAG('tutorial', default_args=default_args)
```

## 定义任务
任务的我们可以通过实例化一个`operator`来定义，airflow提供了很多种`operator`,在这里我们使用`bash_operator`来定义任务。首先，每一个任务也必须有一个某一标识的任务ID(`task_id`),因此第一个参数我们填的就是任务ID

```python
t1 = BashOperator(
    task_id='print_date',
    bash_command='date',
    dag=dag)

t2 = BashOperator(
    task_id='sleep',
    bash_command='sleep 5',
    retries=3,
    dag=dag)
```
我们看到当我们在实例化一个t2这个`bash_operator`的时候，不仅传入了需要执行的bash命令，还传入了`retries=3`这样一个参数，因此这个时候t2这个失败重试次数就变成了3次。

参数处理的优先级：
1. 显示传入的参数
2. 默认参数
3. `operator`本身的默认参数

## Jinja模板的使用
airflow使用了Jinja模板引擎提供了一系列宏和内建参数的命令模板，不仅如此用户还可以传入自己的参数以及定义自己的宏。其中`ds`就是内建参数，`macros.ds_add(ds,7)`就是宏，后面的`params.my_param`就是用户自定义的参数,只需要在实例化`operator`时传入一个params的`dict`即可。

```python
templated_command = """
    {% for i in range(5) %}
        echo "{{ ds }}"
        echo "{{ macros.ds_add(ds, 7)}}"
        echo "{{ params.my_param }}"
    {% endfor %}
"""

t3 = BashOperator(
    task_id='templated',
    bash_command=templated_command,
    params={'my_param': 'Parameter I passed in'},
    dag=dag)
```

## 配置任务间的依赖
最后就是设置任务间前后执行关系,我们只需要设置的`upstream`(或者`downstream`)即可

```python
t2.set_upstream(t1)
t3.set_upstream(t1)

# 或者
# t1.set_downstream(t2)
# t1.set_downstream(t2)
```

# 执行任务
秩序任务我们需要开启`scheduler`和`webserver`,用下面的命令开启。
```shell
# 启动任务调度器
airflow scheduler -D

# 启动监控页面
airflow webserver -D -p 9090
```
这个时候就会在[localhost:9090](http://localhost:9090)看到我们的airflow的监控页面，之后需要点击任务的打开按钮就可以启动定时任务了。
