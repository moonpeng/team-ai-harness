# 数据库表设计规范

## 设计规范

1. 表名不超过 50 个字符的小写字母和下划线组成，且必须要有注释。由 `业务简称_模块简称_功能` 名组成，例：`aos_order_item`
2. 列名由小写字母和下划线组成，不超过 40 个字符，避开 MySQL 关键字，且必须要有注释。
3. 主键命名为 `id` 需要设置无符号 `unsigned`，类型为 `bigint(20)`，注释 "自增主键"，建议为 `MYSQL` 自增.
4. 索引命名，普通索引 `idx_fields`，唯一索引 `ux_fields`。
6. 禁止使用 `ENUM` 类型，存储过程、函数、触发器、外键，大字段 `text/json` 类型。
7. 时间类型不要使用 `TIMESTAMP`，容易溢出。高精度数字类型使用 `decimal` 类型。
8. 同一个库字符集和排序规则要一致，同一个系统不同环境之间字符集和排序规则要一致。

## 必备字段

以下为字段清单，类型与注释要求以上述规范为准；自增为推荐策略，采用其他主键生成策略时相应调整 `AUTO_INCREMENT`。

```sql
id          BIGINT(20) UNSIGNED NOT NULL AUTO_INCREMENT     -- 自增主键
is_delete   TINYINT     NOT NULL DEFAULT '0'                -- '是否删除（0-未删除，1-已删除）'
create_time DATETIME    NOT NULL DEFAULT CURRENT_TIMESTAMP  -- 创建时间
update_time DATETIME    NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE    CURRENT_TIMESTAMP   -- 更新时间
```

