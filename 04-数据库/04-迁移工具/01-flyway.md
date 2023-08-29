---
dg-publish: true
---
Flyway 是一款开源的数据库版本管理工具
现管理并跟踪数据库变更，**支持数据库版本自动升级**

springboot + mysql为例：
```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
```

```yml
spring:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/flyway?useUnicode=true&characterEncoding=UTF-8&allowMultiQueries=true&rewriteBatchedStatements=true&useSSL=false&serverTimezone=GMT%2B8
    username: root
    password: tiger 
  flyway:
    enabled: true
    # 禁止清理数据库表
    clean-disabled: true
    # 如果数据库不是空表，需要设置成 true，否则启动报错
    baseline-on-migrate: true
    # 与 baseline-on-migrate: true 搭配使用
    baseline-version: 0
    locations: 
      - classpath:db/migration/mysql（根据个人情况设置）
```

这样启动项目后  会自动识别classpath:db/migration/mysql下的迁移脚本文件，根据数据库表
`flyway_schema_history` 中的数据(如果数据库没有这张表会自动创建该表)和配置文件识别哪些脚本需要被执行，并自动执行

```sql
-- auto-generated definition
create table flyway_schema_history
(
    installed_rank int                                 not null
        primary key,
    version        varchar(50)                         null,
    description    varchar(200)                        not null,
    type           varchar(20)                         not null,
    script         varchar(1000)                       not null,
    checksum       int                                 null,
    installed_by   varchar(100)                        not null,
    installed_on   timestamp default CURRENT_TIMESTAMP not null,
    execution_time int                                 not null,
    success        tinyint(1)                          not null
);

create index flyway_schema_history_s_idx
    on flyway_schema_history (success);


```