---
dg-publish: true
---
想写一个bat  ，批量执行脚本
```bash
sqlplus jubt/jubt@orcl @list.sql>log.txt Exit;
EXIT
```


list.sql 如下：
```bash
@uim.sql
@portal.sql
@bt.sql
@bt_rule.sql
@bt_interface.sql
@batch.sql
```

