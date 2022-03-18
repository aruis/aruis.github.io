---
title: PostgreSQL中实现更新默认值（二）
date: 2018-08-17 08:11:01
categories: 程序人生
tags:
    - PostgreSQL
---
今天我们用`表继承`+`触发器`的方案，来实现表中的更新默认值。这也许是`PostgreSQL`里最佳的解决方案。
#### 一. 创建一张表，作为父表

```sql
create table basic_update
(
  t_update timestamp
);
```
#### 二. 创建一个函数，用作最后负责修改`t_update`使用

```sql
CREATE OR REPLACE FUNCTION update_modified_column()
  RETURNS TRIGGER AS $$
BEGIN
  NEW.t_update = now();
  RETURN NEW;
END;
$$
language 'plpgsql';
```

#### 三. 创建一个函数，用来给继承了`basic_update`的表新增一个触发器，怼上第二步的函数

```sql
create function table_create()
  returns event_trigger
language plpgsql
as $$
DECLARE oid           INT;
        table_name    VARCHAR;
        parent_tables VARCHAR [];
        is_update     BOOL;

BEGIN

  SELECT object_identity
      INTO table_name FROM pg_event_trigger_ddl_commands();

  SELECT objid
      INTO oid FROM pg_event_trigger_ddl_commands();


  SELECT get_parent_tables_by_oid(oid)
      INTO parent_tables;

  SELECT parent_tables :: TEXT [] @> '{basic_update}' :: TEXT []
      INTO is_update;


  IF is_update

  THEN
    EXECUTE 'CREATE TRIGGER tgr_auto_t_update'
            || ' BEFORE UPDATE ON  '
            || table_name
            || ' FOR EACH ROW EXECUTE PROCEDURE update_modified_column()';
  END IF;

END;
```

#### 四. 创建一个建表时的触发器，接管创建表的时机，怼上第三步的函数

```sql
CREATE EVENT TRIGGER etgr_table_create
ON ddl_command_end
WHEN TAG IN ('CREATE TABLE')
EXECUTE PROCEDURE table_create();
```

#### 五. 所有操作已经完成，可以创建一个表测试了

```sql
create table test
(
  id        varchar default uuid_generate_v4() not null
    constraint test_pkey
    primary key,
  text      varchar,
  t_create  timestamp default now()
)
  inherits (basic_update);
```


```
insert into test (text)
values ('a');
```


```
update test
set text = 'b'
```