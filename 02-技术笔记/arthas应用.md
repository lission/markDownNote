[TOC]

# 1、参考内容

https://www.jianshu.com/p/507f7e0cc3a3

# 2、启动命令

```shell
java -jar arthas-boot.jar
```



# 3、常用命令

## 3.1、watch

watch命令，可以用来查看方法的入参及返回值

示例：

```shell
watch com.etcc.monitor.warning.business.company.provider.SCompanyDataReportStatProviderImpl getReportDataCount  "{params,returnObj}" -x 3 
```



