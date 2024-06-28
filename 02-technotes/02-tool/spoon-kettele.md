[toc]

# 1、参考资料

[中文官网](https://www.kettle.net.cn/2989.html)

kettle本地配置文件存储目录

```properties
/Users/lission/.kettle/kettle.properties
```


kettle安装路径：

```properties
/opt/docker-data/webspoon/data-integration/kitchen.sh
```

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
mysql.ip=10.14.2.180
mysql.database=mw_company
mysql.port=13308
mysql.username=yjanquan
mysql.password=sazYWG5NaF0

# 百分点前置机
baifendian.mysql.ip=10.11.13.40
baifendian.mysql.database=zd
baifendian.mysql.port=3306
baifendian.mysql.username=zd
baifendian.mysql.password=zdajj2021

announce.mysql.ip=10.14.2.180
announce.mysql.database=mw_announce
announce.mysql.port=13308
announce.mysql.username=yjanquan
announce.mysql.password=sazYWG5NaF0

announce.down.hazard=/opt/docker-data/webspoon/announce/down_hazard.ktr
announce.down.plant=/opt/docker-data/webspoon/announce/down_plant.ktr
announce.upload=/opt/docker-data/webspoon/announce/announce_info.ktr

```
