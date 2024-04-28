---
dg-publish: false
---
假如有张user表，有email字段 ，其中一行记录的值是  'njmoser18@gmail.com\n'   
假如我们dao层有一个方法  是getUserByEmail    然后传入的email是  'njmoser18@gmail.com\n'的话  那么就查询不到值  需要特殊处理  这是比较坑的一点

