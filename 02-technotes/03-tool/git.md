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

## 1.3、Gitlab的用户名和邮箱的设置方法

### idea修改gitLab用户名

1、选择下方的Terminal

2、在cmd中输入

git config user.name可以以查看自己git名字

git config user.email 可以以查看自己邮箱名字

3、修改名字
局部：
git config --local user.name yourname
git config --local user.email youremail
全局：
git config --global user.name liulili
git config --global user.email liulili@cestc.cn



# 2、git 命令



## 2.1、git log 提交记录

### 2.1.1、某段时间内提交的行数
git log --since="2022-10-01" --before="2022-12-31" --author="username" --pretty=tformat: --numstat |awk '{add += $1; subs += $2; loc += $1 - $2 } END {printf "新增行数：%s\n删除行数:%s\n总行数:%s\n", add, subs,loc}'
### 2.1.2、全部
git log  --author="username" --pretty=tformat: --numstat |awk '{add += $1; subs += $2; loc += $1 - $2 } END {printf "新增行数：%s\n删除行数:%s\n总行数:%s\n", add, subs,loc}'
