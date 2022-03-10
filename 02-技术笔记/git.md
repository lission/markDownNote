[TOC]

# 1、git忽略文件设置

[参考地址](https://www.cnblogs.com/mafeng/p/10141292.html)

在项目根目录下面 添加 .gitignore文件

在.gitignore文件中加入一行.gitignore，否则的话.gitignore将会被传到GIT服务器上

.gitignore文件过滤有两种模式，开放模式和保守模式

## 1.1、开放模式负责设置过滤哪些文件和文件夹

/node_modules/*

## 1.2、保守模式负责设置哪些文件不被过滤

!/node_modules/layer/

