---
title: 解决ubuntu中docker无法使用http进行push
date: 2016-11-08 21:24:08
tags:
---
默认情况下docker是使用https进行push的，当我们配置了自己的docker registry且使用http来上传镜像的时候，就需要配置我们ubuntu环境下的docker配置文件。

* 在 `/etc/default/docker` 文件中添加一行

```shell
DOCKER_OPTS="--insecure-registry YOUR_REGISTRY_DOMAIN"
```

然后保存退出，根据[官网的文档](https://docs.docker.com/registry/insecure/),我们只需要重启我们的docker守护进程就可以了。但是再尝试之后一直发现无效，最后才发现原来是docker在ubuntu环境下有坑，不会加载`/etc/default/docker`这个配置文件，下面就是解决办法。

* 修改 `/lib/systemd/system/docker.service` 
    1.增加一行，用于指定配置文件在什么地方
    ```shell
    EnvironmentFile=-/etc/default/docker
    ```

    2.然后将
    ```shell
    ExecStart=/usr/bin/docker daemon -H fd://
    ```
    修改为
    ```shell
    ExecStart=/usr/bin/docker daemon -H fd:// $DOCKER_OPTS
    ```
    注意`fd://`之后又一个空格

* 重启docker服务
    1.执行
    ```shell
    sudo systemctl daemon-reload
    ```

    2.重启docker
    ```
    sudo systemctl restart docker
    ```

    3.这个时候，通过这个命令就可以看到docker的配置文件了
    ```shell
    sudo systemctl show docker | grep EnvironmentFile
    ```

最后再次尝试`docker push`就可以成功上传镜像了。
