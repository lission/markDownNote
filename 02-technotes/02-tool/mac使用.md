[toc]

# 1、java多版本配置

[mac java多版本配置](https://www.cnblogs.com/yyxianren/p/14435118.html)

mac系统中Java默认目录：`/Library/Java/JavaVirtualMachines/`

**(注意：使用终端不同，修改的配置文件也不同，zsh终端请修改 ~/.zprofile 配置文件)**

配置JAVA_HOME：**`vi ~/.bash_profile`**

生效配置：**`source ~/.bash_profile`**

**切换JDK版本就执行对应命令别名** :

- Jdk7
- Jdk8
- Jdk11

# 2、解决xxx.app已损坏，无法打开，您应该将它移到废纸篓

终端中输入命令 sudo xattr -d com.apple.quarantine /Applications/xxx.app解决
xxx.app是出问题的APP名称，有时候app的名字难以输入正确，这个时候只要在应用列表中将该应用拖到终端中，会自动显示该app的名字

**/Applications/Navicat\ for\ SQLite.app**

/Applications/Z-Library.app

sudo xattr -d com.apple.quarantine /Applications/Navicat\ for\ SQLite.app

sudo codesign --force --deep --sign - /Applications/Navicat\ for\ SQLite.app

sudo xattr -d com.apple.quarantine /Applications/Navicat\ Premium.app

sudo xattr -r -d com.apple.quarantine /Applications/Navicat\ Premium.app

sudo codesign --force --deep --sign - /Applications/Navicat\ Premium.app

sudo xattr -d com.apple.quarantine  /Applications/Infuse.app

sudo codesign --force --deep --sign - /Applications/Infuse.app

sudo spctl --master-disable

sudo xattr -r -d com.apple.quarantine /Applications/Navicat\ Premium.app

sudo codesign --force --deep --sign -/Applications/Navicat\ Premium.app

# 3、jd-gui class反编译工具启动方式

https://github.com/java-decompiler/jd-gui/releases

[参考](https://blog.csdn.net/qq_37958845/article/details/121703791)

- cd 进入目录/Users/lission/devTool/jd-gui-osx-1.6.6/JD-GUI.app/Contents/Resources/Java
- java -jar jd-gui-1.6.6-min.jar
