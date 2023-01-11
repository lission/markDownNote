[TOC]

# 1、Docker是什么

Docker作为一个**开源的应用容器引擎**，可以让开发者**打包他们的应用及依赖环境**到一个**可移植的容器**中，然后发布到任何运行有Docker引擎的机器上。

Docker集**版本控制、克隆继承、环境隔离**等特性于一身，提出一整套**软件构建、部署和维护的解决方案**。

Docker使用容器引擎**解决平台依赖**的问题，它在每台宿主机上都启动一个**Docker守护进程**，守护进程屏蔽与具体平台相关的信息，**对上层应用提供统一的接口**。

Docker针对不同平台，解析给不同平台下的执行驱动、存储驱动和网络驱动去执行。

Java曾提出“Write Once, Run Anywhere”，而Docker则提出了“Build once, Run anywhere,Configure once, Run anything“。

如果把软件部署的应用看作Android的App，Docker简直和Android一模一样，Docker是一个开源的容器引擎，也有自己的生态圈，它的**应用以镜像(image)的形式发布**，可以运行在任何装有Docker引警的操作系统上。它有一个**官方的镜像仓库**，提供各种各样的应用，当需要某个应用时，就从官方的仓库搜素并下载，个人开发者也可以提交镜像到官方仓库，分享给别人使用。**Docker也允许使用第三方的镜像仓库。**

Docker用来管理软件部署的应用，Docker把应用打包成一个镜像，镜像带有版本控制功能，应用的每次修改迭代对应镜像的一个版本。制作好的镜像可以发布到镜像仓库

Docker官方仓库地址：https://hub.docker.com/

Docker通过**分层共享和增量变更**把应用的运行环境（包括操作系统在内）的庞大体量实现瘦身，让应用运行环境的安装和修改在大多数情况下与只安装软件包一样轻量、简单。

# 2、Docker基础

Docker三大基础组件，仓库、镜像、容器。





# 1、容器



镜像文件存放路径

```
/Users/{YourUserName}/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/Docker.qcow2
```

docker run -d -p 80:80 docker/getting-started