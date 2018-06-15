# Mybatis 的一对一查询,一对多查询,以及多对多查询的理解与应用

## (一) 测试表

* db_customer    顾客表
* db_m_s            顾客的每日睡眠数据表
* db_m_s_items  顾客的每日睡眠测量项表
* db_m_s_raw     顾客的每日睡眠走势数据
* db_cust_device 顾客与设备的关系表

| db_cust_device | db_customer |  db_m_s  | db_m_s_items | db_m_s_raw |
| :------------: | :---------: | :------: | :----------: | :--------: |
|       id       |     id      |    id    |      id      |     id     |
|    cust_id     |     ...     | cust_id  |     s_id     |    s_id    |
|   bind_state   |     ...     | dev_code |   dev_code   |    ...     |
|      ...       |     ...     |   ...    |     ...      |    ...     |

一对一查询

db_m_s 与db_m_s_raw 的关系是一对一的关系，即一条睡眠记录对应着一条睡眠走势记录

* 不使用mybatis的查询

```sql

```

