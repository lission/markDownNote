[toc]

# 一、综合类

## 1、pingWurth 个人学习笔记

https://gitee.com/pingWurth/study-notes

## 2、java全栈知识体系

https://www.pdai.tech/

## 3、go

官网博客：https://go.dev/blog/

中文文档：https://www.topgoer.com/

代码实践：https://gobyexample.com/

go组件：https://github.com/avelino/awesome-go

go框架：https://github.com/avelino/awesome-go#web-frameworks

### 基于Go+Vue实现的openLDAP后台管理项目

基于Go+Vue实现的openLDAP后台管理项目：http://ldapdoc.eryajf.net/

## 4、通用测试用例大全

https://www.cnblogs.com/wysk/archive/2018/01/05/8193091.html

## 5、freeCodeCamp

https://www.freecodecamp.org/learn/

https://forum.freecodecamp.org/

## 6、面试之彻底击碎行为问题

[彻底击碎行为问题](https://docs.google.com/document/d/112HBiMNvu6TYbDUOfVRe_MS4A-fKaWYrpMlmnsiMNiA/edit)

## 7、面试之系统设计面试题精选

[系统设计面试题精选](https://soulmachine.gitbooks.io/system-design/content/cn/)

# 二、网络

## 1、图解网络

图解网络作者博客：https://xiaolincoding.com/network/1_base/tcp_ip_model.html#%E5%BA%94%E7%94%A8%E5%B1%82

# 三、专项类

## 1、RabbitMQ

RabbitMQ基础博客：https://juejin.cn/post/7054448694321479711

## 2、Redission

redission wiki 目录：https://github.com/redisson/redisson/wiki/%E7%9B%AE%E5%BD%95

## 3、kafka Tool

offset explore下载地址：

https://www.kafkatool.com/download.html

使用教程：

https://blog.csdn.net/m0_67400972/article/details/126074724

https://www.cnblogs.com/xingxia/p/kafka_tool.html

## 4、jsonPath

github地址：https://github.com/json-path/JsonPath

```xml
<dependency>
    <groupId>com.jayway.jsonpath</groupId>
    <artifactId>json-path</artifactId>
    <version>2.7.0</version>
</dependency>
```

使用简介：https://blog.csdn.net/weixin_44150794/article/details/111560428

```java
    public static void jsonPathTest(){
        String msg = "{\"name\":\"赵仲\",\"mobile\":\"13522002789\",\"info\":{\"address\":\"test\"},\"gender\":\"male\"}";
        String relation = "{\"username\":\"$.name\",\"sex\":\"$.gender\",\"location\":\"$.info.address\"}";
        JSONObject jsonObject = JSONObject.parseObject(relation);
        String username = jsonObject.getString("username");
        String location = jsonObject.getString("location");
        Object document = Configuration.defaultConfiguration().jsonProvider().parse(msg);
        String usernameVal = JsonPath.read(document,username);
        String locationVal = JsonPath.read(document,location);
        System.out.println(usernameVal);
        System.out.println(locationVal);
    }
```

## 5、springdoc中文文档

https://springdoc.cn/

https://springboot.io/

## 6、分布式定时任务轻量级解决方案ShedLock

https://www.modb.pro/db/212154

## 7、redis连接工具

### 7.1、tiny-craft

- https://github.com/tiny-craft/tiny-rdm
- https://redis.tinycraft.cc/

## 8、git

### 8.1、git命令练习工具

- https://learngitbranching.js.org/?locale=zh_CN

## 9、正则表达式

- https://wangwl.net/static/projects/visualRegex#
- https://regex.ai/

## 10、markdown

- 教程：https://markdown.com.cn/

# 四、github 资源

todo 待整理

https://github.com/PKUanonym/REKCARC-TSC-UHT

https://github.com/USTC-Resource/USTC-Course

https://github.com/kxxwz/SJTU-Courses

https://github.com/zjdx1998/seucourseshare

https://github.com/QSCTech/zju-icicles

https://github.com/PKUanonym/REKCARC-TSC-UHT

https://github.com/tongtzeho/PKUCourse

https://github.com/lib-pku/libpku

https://github.com/OpenBMB/ChatDev/blob/main/README-Chinese.md

https://github.com/Liubsyy/HotSecondsIDEA/blob/master/install/%E4%BD%BF%E7%94%A8%E6%96%87%E6%A1%A3.md

https://github.com/nl8590687/ASRT_SpeechRecognition
https://github.com/yeyupiaoling/PPASR
https://github.com/k2-fsa/sherpa-onnx

https://gitee.com/dotnetchina/SmartSQL

https://github.com/dunwu/linux-tutorial
https://github.com/imthenachoman/How-To-Secure-A-Linux-Server

https://github.com/webp-sh/webp_server_go

https://github.com/facebookresearch/seamless_communication

https://github.com/lucasg/Dependencies

https://lbs.amap.com/demo/javascript-api-v2/example/marker/replaying-historical-running-data

https://github.com/ismartcoding/plain-app

https://github.com/dotnet-easy/easy-kit/issues/1

- https://github.com/xiangyuecn/AreaCity-JsSpider-StatsGov  省市区数据采集并标注拼音、坐标和边界范围
- https://github.com/open-telemetry/opentelemetry-java-instrumentation  类似于arthas的java进程诊断工具
- https://github.com/bzsome/idcard_generator  身份证图片构造

## 1、ansj分词

真正java实现.分词效果速度都超过开源版的ict. 中文分词,人名识别,词性标注,用户自定义词典

- https://github.com/NLPchina/ansj_seg

# 五、工具

## 1、showdoc

开源地址： https://github.com/star7th/showdoc

官网： https://www.showdoc.com.cn/

lission@foxmail.com

## 2、ppzhilian

https://www.ppzhilian.com/

## 3、个人头像

logo制作：https://www.designevo.com/cn/apps/logo/

图片转素描：https://javier.xyz/pintr/

图片像素化：https://pixel-me.tokyo/en/

## 4、淘宝手工皮具

https://www.zhihu.com/question/24958060/answer/1491000411

## 5、clash for windows

https://clashforwindows.org/

https://github.com/Fndroid/clash_for_windows_pkg/releases

## 6、cursor 代码生成工具

https://www.cursor.so/

## 7、GPT

- https://c.binjie.fun/#/chat/1693451554887
- https://yiyan.baidu.com/

https://chat.openai.com/

接码平台：https://sms-activate.org/

注册教程：https://intercom.help/letsvpn-world/en/articles/6973491-%E7%83%AD%E9%97%A8%E4%B8%93%E9%A2%98chatgpt-%E4%BF%9D%E5%A7%86%E7%BA%A7%E6%B3%A8%E5%86%8C%E6%95%99%E7%A8%8B

https://study.zwjjiaozhu.top/posts/chatgpt-mirror-sites.html

https://chat.yougan.cc/

## 8、识屏 · 搜索·截屏

- https://esearch.vercel.app/#download
- shareX

## 9、markdown工具

https://miaoyan.app/

## 10、文档网站工具

-  https://docusaurus.io/zh-CN/docs
- vuepress-theme-vdoing
- https://github.com/docsifyjs/docsify

## 11、个人博客

- https://halo.run/

## 12、wsl

- https://zhuanlan.zhihu.com/p/386590591

## 13、openWrt-家庭路由配置，全局翻墙

家庭路由配置，全局翻墙

https://xtrojan.pro/client/openwrt-openclash-tutorial.html

https://xtrojan.pro/client/openwrt-passwall-tutorial.html

https://openwrt.org/zh/docs/guide-quick-start/start

旁路由配置：https://blog.csdn.net/kangzeru/article/details/115373587


# 六、生活

## 1、12315投诉平台

https://www.12315.cn/
