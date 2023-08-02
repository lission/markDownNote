批量模糊删除命令：

```shell
redis-cli -n 4 -a cestc@redis --scan --pattern "*1689*"  | xargs redis-cli -n 4 -a cestc@redis del
```

