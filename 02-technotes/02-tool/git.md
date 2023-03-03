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



# 2、git 命令



## 2.1、git log 提交记录

### 2.1.1、某段时间内提交的行数
git log --since="2022-10-01" --before="2022-12-31" --author="username" --pretty=tformat: --numstat |awk '{add += $1; subs += $2; loc += $1 - $2 } END {printf "新增行数：%s\n删除行数:%s\n总行数:%s\n", add, subs,loc}'
### 2.1.2、全部
git log  --author="username" --pretty=tformat: --numstat |awk '{add += $1; subs += $2; loc += $1 - $2 } END {printf "新增行数：%s\n删除行数:%s\n总行数:%s\n", add, subs,loc}'



## 2.2、Gitlab的用户名和邮箱的设置方法

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



  git config --global user.email "lission@fxmail.com"
  git config --global user.name "lission"

## 2.3、Gitlab 生成sshKey

首先，你需要确认自己是否已经拥有密钥。 默认情况下，用户的 SSH 密钥存储在其 `~/.ssh` 目录下。 进入该目录并列出其中内容，你便可以快速确认自己是否已拥有密钥：

```console
$ cd ~/.ssh
$ ls
authorized_keys2  id_dsa       known_hosts
config            id_dsa.pub
```

我们需要寻找一对以 `id_dsa` 或 `id_rsa` 命名的文件，其中一个带有 `.pub` 扩展名。 `.pub` 文件是你的公钥，另一个则是与之对应的私钥。 如果找不到这样的文件（或者根本没有 `.ssh` 目录），你可以通过运行 `ssh-keygen` 程序来创建它们。 在 Linux/macOS 系统中，`ssh-keygen` 随 SSH 软件包提供；在 Windows 上，该程序包含于 MSysGit 软件包中。

```console
$ ssh-keygen -o
Generating public/private rsa key pair.
Enter file in which to save the key (/home/schacon/.ssh/id_rsa):
Created directory '/home/schacon/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/schacon/.ssh/id_rsa.
Your public key has been saved in /home/schacon/.ssh/id_rsa.pub.
The key fingerprint is:
d0:82:24:8e:d7:f1:bb:9b:33:53:96:93:49:da:9b:e3 schacon@mylaptop.local
```

首先 `ssh-keygen` 会确认密钥的存储位置（默认是 `.ssh/id_rsa`），然后它会要求你输入两次密钥口令。 如果你不想在使用密钥时输入口令，将其留空即可。 然而，如果你使用了密码，那么请确保添加了 `-o` 选项，它会以比默认格式更能抗暴力破解的格式保存私钥。 你也可以用 `ssh-agent` 工具来避免每次都要输入密码。

现在，进行了上述操作的用户需要将各自的公钥发送给任意一个 Git 服务器管理员 （假设服务器正在使用基于公钥的 SSH 验证设置）。 他们所要做的就是复制各自的 `.pub` 文件内容，并将其通过邮件发送。 公钥看起来是这样的：

```console
$ cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAklOUpkDHrfHY17SbrmTIpNLTGK9Tjom/BWDSU
GPl+nafzlHDTYW7hdI4yZ5ew18JH4JW9jbhUFrviQzM7xlELEVf4h9lFX5QVkbPppSwg0cda3
Pbv7kOdJ/MTyBlWXFCR+HAo3FXRitBqxiX1nKhXpHAZsMciLq8V6RjsNAQwdsdMFvSlVK/7XA
t3FaoJoAsncM1Q9x5+3V0Ww68/eIFmb1zuUFljQJKprrX88XypNDvjYNby6vw/Pb0rwert/En
mZ+AW4OZPnTPI89ZPmVMLuayrD2cE86Z/il8b+gw3r3+1nKatmIkjn2so1d01QraTlMqVSsbx
NrRFi9wrf+M7Q== schacon@mylaptop.local
```



## 2.4、git上传代码报错ssh: connect to host github.com port 22: Connection timed out解决办法

当在远程库上设置了SSH 之后还是报错连接超时，问题如下

```shell
$ git push origin master
ssh: connect to host [github](https://so.csdn.net/so/search?q=github&spm=1001.2101.3001.7020).com port 22: Connection timed out
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

这个时候需要检查一下SSH是否能够连接成功，输入以下命令

ssh -T git@github.com

稍等片刻如果继续报错，如下：

ssh: connect to host github.com port 22: Connection timed out
则，可以使用一下解决办法

打开存放ssh的目录

cd ~/.ssh 

ls

查看是否存在 id_rsa  id_rsa.pun known_hosts 三个文件，如果没有移步解决办法：

https://blog.csdn.net/u014344668/article/details/78931031

如果存在，则新建config文件输入下面内容。

```cpp
Host github.com
User YourEmail@163.com
Hostname ssh.github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa
Port 443
```

其中User后面为GitHub的账号名称

创建方法：

vim comfig

然后编辑，最后:wq退出

保存之后再次执行"ssh -T git@github.com"时，会出现如下提示，回车"yes"即可。

这时验证就可以通过。