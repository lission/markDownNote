[TOC]

# 解压缩

## *.tar.gz

### 1.压缩命令：

　　命令格式：tar -zcvf  压缩文件名.tar.gz  被压缩文件名

   可先切换到当前目录下。压缩文件名和被压缩文件名都可加入路径。 

### 2.解压缩命令：

　　命令格式：tar -zxvf  压缩文件名.tar.gz

　　解压缩后的文件只能放在当前的目录。

# 查找

## find命令

　　　　基本格式：find  path expression

### 1.按照文件名查找

　　　　(1)find / -name httpd.conf　　#在根目录下查找文件httpd.conf，表示在整个硬盘查找
　　　　(2)find /etc -name httpd.conf　　#在/etc目录下文件httpd.conf
　　　　(3)find /etc -name '*srm*'　　#使用通配符*(0或者任意多个)。表示在/etc目录下查找文件名中含有字符串‘srm’的文件
　　　　(4)find . -name 'srm*' 　　#表示当前目录下查找文件名开头是字符串‘srm’的文件

### 2.按照文件特征查找 　　　　

　　　　(1)find / -amin -10 　　# 查找在系统中最后10分钟访问的文件(access time)
　　　　(2)find / -atime -2　　 # 查找在系统中最后48小时访问的文件
　　　　(3)find / -empty 　　# 查找在系统中为空的文件或者文件夹
　　　　(4)find / -group cat 　　# 查找在系统中属于 group为cat的文件
　　　　(5)find / -mmin -5 　　# 查找在系统中最后5分钟里修改过的文件(modify time)
　　　　(6)find / -mtime -1 　　#查找在系统中最后24小时里修改过的文件
　　　　(7)find / -user fred 　　#查找在系统中属于fred这个用户的文件
　　　　(8)find / -size +10000c　　#查找出大于10000000字节的文件(c:字节，w:双字，k:KB，M:MB，G:GB)
　　　　(9)find / -size -1000k 　　#查找出小于1000KB的文件

### 3.使用混合查找方式查找文件

　　　　参数有： ！，-and(-a)，-or(-o)。

　　　　(1)find /tmp -size +10000c -and -mtime +2 　　#在/tmp目录下查找大于10000字节并在最后2分钟内修改的文件
   　　    (2)find / -user fred -or -user george 　　#在/目录下查找用户是fred或者george的文件文件
   　　    (3)find /tmp ! -user panda　　#在/tmp目录中查找所有不属于panda用户的文件