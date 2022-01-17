[TOC]



# 1、参考资料

[中文官网](https://www.kettle.net.cn/2989.html)





# 2、广西部署实践

广西线上887连接方式：

jumpserver：

1、

| 钦州线上186 | 202.103.219.149 |
| ----------- | --------------- |
| ssh227      |                 |
| ssh887      |                 |



后台任务启动命令：

```javascript
nohup ./kitchen.sh -file=/opt/docker-data/webspoon/announce/announce_down_hazard_and_plant.kjb -level=debug -logFile=logs/announce.log >/dev/null &
  
nohup ./kitchen.sh -file=/opt/docker-data/webspoon/announce/announce_upload_to_mw.kjb -level=debug -logFile=logs/announce.log >/dev/null &
```

kettle文件存放路径

```properties
/opt/docker-data/webspoon/announce
```



kettle配置文件路径

```properties
/root/.kettle/kettle.properties
```

kettle配置项，自定义

```properties
guangxi.mysql.ip=10.14.2.180
guangxi.mysql.database=mw_company
guangxi.mysql.port=13308
guangxi.mysql.username=yjanquan
guangxi.mysql.password=sazYWG5NaF0

# 百分点前置机
guangxi.baifendian.mysql.ip=10.11.13.40
guangxi.baifendian.mysql.database=zd
guangxi.baifendian.mysql.port=3306
guangxi.baifendian.mysql.username=zd
guangxi.baifendian.mysql.password=zdajj2021

guangxi.announce.mysql.ip=10.14.2.180
guangxi.announce.mysql.database=mw_announce
guangxi.announce.mysql.port=13308
guangxi.announce.mysql.username=yjanquan
guangxi.announce.mysql.password=sazYWG5NaF0

guangxi.announce.down.hazard=/opt/docker-data/webspoon/announce/down_hazard.ktr
guangxi.announce.down.plant=/opt/docker-data/webspoon/announce/down_plant.ktr
guangxi.announce.upload=/opt/docker-data/webspoon/announce/announce_info.ktr

```



广西安全承诺前置机数据库配置：

```
代理地址
10.11.13.40
port:3306
userName:zd
password:zdajj2021

中电数据库对接mysql数据库
ip:10.7.67.200
port:3306
userName:zd
password:zdajj2021
```

