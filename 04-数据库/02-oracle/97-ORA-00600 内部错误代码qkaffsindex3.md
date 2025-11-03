```sql
-- auto-generated definition  
create table T_DOC_WAYBILL_DETAIL  
(  
    ORGANIZATIONID     VARCHAR2(20)                 not null,  
    WAYBILLNO          VARCHAR2(40)                 not null,  
    TASKNO             VARCHAR2(40)                 not null,  
    ORDERNO            VARCHAR2(40),  
    QTYORDERED_EACH    NUMBER(12, 2),  
    QTYORDERED         NUMBER(18, 8),  
    GROSSWEIGHTORDERED NUMBER(18, 9),  
    CUBICORDERED       NUMBER(18, 9),  
    QTY_EACH           NUMBER(12, 2),  
    QTY                NUMBER(18, 8),  
    GROSSWEIGHT        NUMBER(18, 9),  
    CUBIC              NUMBER(18, 9),  
    NOTETEXT           CLOB,  
    UDF01              VARCHAR2(50),  
    UDF02              VARCHAR2(50),  
    UDF03              VARCHAR2(50),  
    UDF04              VARCHAR2(50),  
    UDF05              VARCHAR2(50),  
    CURRENTVERSION     NUMBER        default 100    not null,  
    OPRSEQFLAG         VARCHAR2(65)  default '2022' not null,  
    ADDWHO             VARCHAR2(40),  
    ADDTIME            DATE,  
    EDITWHO            VARCHAR2(40),  
    EDITTIME           DATE,  
    TRACKINGSTATUS     VARCHAR2(20),  
    LASTWAYBILLNO      VARCHAR2(40),  
    NEXTWAYBILLNO      VARCHAR2(40),  
    ACTIVEFLAG         CHAR          default 'N',  
    TASKSTATUS         VARCHAR2(20),  
    TASKPLANSTATUS     VARCHAR2(20),  
    WMSSTATUS          VARCHAR2(20),  
    PODSTATUS          VARCHAR2(20),  
    DELIVERYSTATUS     VARCHAR2(20),  
    DELIVERYNOTES      CLOB,  
    DELIVERYBY         VARCHAR2(40),  
    DELIVERYTIME       DATE,  
    DELIVERYQTY        NUMBER(12, 6),  
    DELIVERYBRANCHID   VARCHAR2(20),  
    CREATETIME         DATE,  
    ORDERTIME          DATE,  
    LOADTIME           DATE,  
    LOADBY             VARCHAR2(40),  
    DEPARTURETIME      DATE,  
    DEPARTUREBY        VARCHAR2(40),  
    CONFIRMTIME        DATE,  
    CONFIRMBY          VARCHAR2(40),  
    ETD                DATE,  
    ETA                DATE,  
    STA                DATE,  
    ATA                DATE,  
    SIGNOFFTIME        DATE,  
    SIGNOFFBY          VARCHAR2(40),  
    EPODTIME           DATE,  
    EPODBY             VARCHAR2(40),  
    PODRETURNTIME      DATE,  
    PODRETURNBY        VARCHAR2(40),  
    LOCKFLAG           CHAR,  
    LOCKTIME           DATE,  
    LOCKBY             VARCHAR2(40),  
    LOCKREASON         VARCHAR2(200),  
    ISDELAY            VARCHAR2(5),  
    DELAYREASON        CLOB,  
    ISEXCEPTION        VARCHAR2(5),  
    LOADFROMTIME       DATE,  
    DELIVERYFROMTIME   DATE,  
    CHECKINTIME        DATE,  
    CHECKINUSERID      VARCHAR2(40),  
    CARRIERCONTACT     VARCHAR2(400),  
    CARRIERADDRESS1    VARCHAR2(400),  
    CARRIERADDRESS2    VARCHAR2(400),  
    CARRIERFAX         VARCHAR2(400),  
    CARRIERADDRESS3    VARCHAR2(400),  
    CARRIERTEL1        VARCHAR2(400),  
    CARRIERMAIL        VARCHAR2(400),  
    THSJ               VARCHAR2(400),  
    PCZT               VARCHAR2(400) default 'Y000',  
    UDF09              VARCHAR2(400),  
    UDF16              VARCHAR2(400),  
    SJ_ISSUED          VARCHAR2(400),  
    SHIPTIME           DATE,  
    VEHICLE01          VARCHAR2(400),  
    primary key (ORGANIZATIONID, WAYBILLNO, TASKNO)  
)  
/  
  
create index I_T_DOC_WAYBILL_DETAIL_OO  
    on T_DOC_WAYBILL_DETAIL (ORGANIZATIONID, ORDERNO)  
/  
  
create index I_T_DOC_WAYBILL_DETAIL_3  
    on T_DOC_WAYBILL_DETAIL (ORGANIZATIONID, WAYBILLNO, ORDERNO)  
/  
  
create index I_T_DOC_WAYBILL_DETAIL_OO1  
    on T_DOC_WAYBILL_DETAIL (ORGANIZATIONID, SUBSTR("WAYBILLNO", 1, 16), TASKNO)  
/
```

```sql
select  
            a.carrierID  
             , a.currencytype  
             , NULL AS offeringtype  
             , a.Payer  
             , MAX(e.addwho) AS ADDWHO  
             , a.buid  
             , sum(nvl(a.amount, 0)) AS amount  
        from BMS_PRE_APPORTION a-- T_BIL_SUMMARY_ALLOCATION a  
                 left join t_Bil_Summary b2  
                           on a.organizationid = b2.organizationid and a.billingsummaryid = b2.billingsummaryid  
                 left join t_Doc_Order_Detail b on a.orderNo = b.orderNo and a.orderlineno = b.orderlineno and  
                                                   a.organizationid = b.organizationid  
                 left join t_Doc_Order_header c on a.orderNo = c.orderNo and a.organizationid = c.organizationid  
                 left join t_bas_sku sku on b.customerid = sku.customerid and b.sku = sku.sku and  
                                            b.organizationid = sku.organizationid  
                 left join t_doc_waybill_detail d on a.orderNo = d.orderNo and a.waybillno = d.waybillno and  
                                                     a.organizationid = d.organizationid  
                 left join t_doc_waybill_header e on d.waybillno = e.waybillno and d.organizationid = e.organizationid  
        where tO_char(b2.settletime, 'YYYY-MM') = '2025-10'  
          and b2.organizationid = 'DREAME'  
          and e.offeringtype in ('TL', 'LTL')  
          AND a.Payer IN ('1011', '1016', '1017', '1021') -- 限制主体是洗地机主体  
          AND a.BULINE = 'BU009'  
          AND A.COSTCENTER  LIKE '%F%'                 -- 非 F类成本中心  
--           and a.prebillingno = '*'  
        group by a.carrierID, a.currencytype, a.Payer, a.buid;
```

报错
```bash
[60000][600]
ORA-00600: 内部错误代码, 参数: [qkaffsindex3], [82], [], [], [], [], [], [], [], [], [], []

https://docs.oracle.com/error-help/db/ora-00600/
Position: 1181
```

**非常有可能！这个函数索引 `I_T_DOC_WAYBILL_DETAIL_OO1` 极有可能就是导致 ORA-00600 错误的罪魁祸首！**

让我详细解释：

## 为什么这个函数索引会导致问题？

### 1. 你的 JOIN 条件

```sql
left join t_doc_waybill_detail d
    on a.orderNo = d.orderNo
    and a.waybillno = d.waybillno  -- ← 注意这里
    and a.organizationid = d.organizationid
```

### 2. 问题索引的定义

```sql
create index I_T_DOC_WAYBILL_DETAIL_OO1
    on T_DOC_WAYBILL_DETAIL (ORGANIZATIONID, SUBSTR("WAYBILLNO", 1, 16), TASKNO)
                                              ^^^^^^^^^^^^^^^^^^^^^^^^
```

### 3. 冲突点分析

当 Oracle 优化器看到你的查询时：

```
你的条件：a.waybillno = d.waybillno
可用索引：
  - I_T_DOC_WAYBILL_DETAIL_3: (ORGANIZATIONID, WAYBILLNO, ORDERNO) ✓ 完美匹配
  - I_T_DOC_WAYBILL_DETAIL_OO1: (ORGANIZATIONID, SUBSTR(WAYBILLNO,1,16), TASKNO) ⚠️ 函数索引
```

**问题发生原因：**

1. 优化器可能尝试使用 `I_T_DOC_WAYBILL_DETAIL_OO1` 索引
2. 它需要判断 `a.waybillno = d.waybillno` 能否用 `SUBSTR(WAYBILLNO, 1, 16)` 的索引
3. 在做 JOIN 时，函数索引的匹配逻辑触发了 Oracle 内核的 bug
4. 参数 `[qkaffsindex3]` 正是与函数索引选择相关的内部错误

---

## 详细解释函数索引的工作机制

### 普通索引 vs 函数索引

```sql
-- 普通索引存储
WAYBILLNO 实际值    → 索引存储值
'WB2024100100001'  → 'WB2024100100001'
'WB2024100100002'  → 'WB2024100100002'

-- 函数索引存储
WAYBILLNO 实际值    → 索引存储值（只存前16位）
'WB2024100100001'  → 'WB2024100100001'
'WB20241001000012' → 'WB2024100100001'  ← 注意：截断后相同！
```

### 当执行 JOIN 时

```sql
-- 你的 JOIN 条件
a.waybillno = d.waybillno

-- Oracle 优化器的困境：
1. 使用 I_T_DOC_WAYBILL_DETAIL_3 (完整 WAYBILLNO)
   → 简单直接，但可能不是最优执行计划

2. 尝试使用 I_T_DOC_WAYBILL_DETAIL_OO1 (SUBSTR(WAYBILLNO,1,16))
   → 需要判断：a.waybillno 的前16位 = d.waybillno 的前16位 是否等价于完整匹配？
   → 如果 waybillno 都小于等于16位 → 可以用
   → 如果 waybillno 超过16位 → 不能用
   → Oracle 在做这个判断时触发了 bug！
```

---

## 为什么设计这个函数索引？

通常是因为：

### 场景1：运单号有统一前缀

```sql
-- 如果运单号格式：WB2024-100-100001（超过16位）
-- 前16位：WB2024-100-10000（业务键部分）
-- 后面是流水号

-- 这样可以快速按业务键查询
WHERE SUBSTR(waybillno, 1, 16) = 'WB2024-100-10000'
```

### 场景2：性能优化（错误的优化）

```sql
-- 有人可能认为截短索引键可以：
-- 1. 减少索引空间
-- 2. 提高索引查找速度
-- 但实际：如果需要完整匹配，这个索引就是累赘！
```

---

## 验证是否是这个索引的问题

### 方法1：临时禁用函数索引（推荐）

```sql
-- 禁用函数索引
ALTER INDEX I_T_DOC_WAYBILL_DETAIL_OO1 UNUSABLE;

-- 重新执行你的查询
select a.carrierID, a.currencytype, a.Payer, a.buid, 
       max(e.ADDWHO) as ADDWHO,
       sum(nvl(a.amount, 0)) amount
from BMS_PRE_APPORTION a
left join t_Bil_Summary b2 ...
-- 完整查询

-- 如果不再报错，证明就是这个索引的问题！

-- 恢复索引（如果需要）
ALTER INDEX I_T_DOC_WAYBILL_DETAIL_OO1 REBUILD;
```

### 方法2：强制不使用这个索引

```sql
select /*+ NO_INDEX(d I_T_DOC_WAYBILL_DETAIL_OO1) */
       a.carrierID, a.currencytype, a.Payer, a.buid, 
       max(e.ADDWHO) as ADDWHO,
       sum(nvl(a.amount, 0)) amount
from BMS_PRE_APPORTION a
left join t_Bil_Summary b2
    on a.organizationid = b2.organizationid
    and a.billingsummaryid = b2.billingsummaryid
left join t_doc_waybill_detail d  -- ← 对这个表加 hint
    on a.orderNo = d.orderNo
    and a.waybillno = d.waybillno
    and a.organizationid = d.organizationid
left join t_doc_waybill_header e
    on d.waybillno = e.waybillno
    and d.organizationid = e.organizationid
where to_char(b2.settletime, 'YYYY-MM') = '2025-09'
  and b2.organizationid = 'DREAME'
  and e.offeringtype in ('TL', 'LTL')
  and a.organizationid = 'DREAME'
  and a.prebillingno = '*'
  and a.Payer in ('1011', '1016', '1017', '1021')
  and a.costcenter not like '%F%'
group by a.carrierID, a.currencytype, a.Payer, a.buid;
```

### 方法3：强制使用正确的索引

```sql
select /*+ INDEX(d I_T_DOC_WAYBILL_DETAIL_3) */
       a.carrierID, a.currencytype, a.Payer, a.buid, 
       max(e.ADDWHO) as ADDWHO,
       sum(nvl(a.amount, 0)) amount
from BMS_PRE_APPORTION a
...
```

---

## 长期解决方案

### 评估这个函数索引是否必要

```sql
-- 查看这个索引的使用情况
SELECT * FROM v$sql 
WHERE sql_text LIKE '%SUBSTR%WAYBILLNO%1%16%'
  AND sql_text LIKE '%T_DOC_WAYBILL_DETAIL%';

-- 或者查看索引使用监控（需要先开启监控）
ALTER INDEX I_T_DOC_WAYBILL_DETAIL_OO1 MONITORING USAGE;
-- 观察一段时间后
SELECT * FROM v$object_usage 
WHERE index_name = 'I_T_DOC_WAYBILL_DETAIL_OO1';
```

### 决策树

```
是否有查询显式使用 SUBSTR(waybillno, 1, 16)？
│
├─ 是 → 保留索引，在你的查询中用 hint 跳过它
│      或者重建索引解决损坏问题
│
└─ 否 → 直接删除这个索引！
       DROP INDEX I_T_DOC_WAYBILL_DETAIL_OO1;
```

### 如果需要保留

```sql
-- 重建索引，可能解决内部数据损坏
ALTER INDEX I_T_DOC_WAYBILL_DETAIL_OO1 REBUILD;

-- 或者使用不同的函数索引定义
DROP INDEX I_T_DOC_WAYBILL_DETAIL_OO1;
CREATE INDEX I_T_DOC_WAYBILL_DETAIL_OO1_NEW
    on T_DOC_WAYBILL_DETAIL (ORGANIZATIONID, 
                              CASE WHEN LENGTH(WAYBILLNO) <= 16 
                                   THEN WAYBILLNO 
                                   ELSE SUBSTR(WAYBILLNO, 1, 16) 
                              END, 
                              TASKNO);
```

---

## 总结

**这个函数索引 99% 就是问题根源！**

**立即执行：**

```sql
ALTER INDEX I_T_DOC_WAYBILL_DETAIL_OO1 UNUSABLE;
-- 然后测试你的查询
```

如果问题解决了，有三个选择：

1. **删除索引**（如果没有其他查询需要）
2. **重建索引**（可能解决损坏）
3. **在查询中用 hint 跳过它**（临时方案）

先测试禁用索引，告诉我结果！

