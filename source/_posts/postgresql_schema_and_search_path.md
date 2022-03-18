---
title: PostgreSQL中关于SCHEMA和SEARCH_PATH的一些技巧
date: 2018-08-26 08:52:57
categories: 程序人生
tags:
    - PostgreSQL
---
#### 1. `extension`最好放在单独的`schema`里，就像这样
```sql
create schema "extension_schema";
create extension "ltree" schema extension_schema;
```

#### 2.业务上不相干的表，建议也放在单独的`schema`中，比如这样
```sql
create schema "metadata_schema";
create schema "platform_schema";
create schema "platform_schema";
```

#### 3.[*可选*]从数据库设计的角度来说，不同`schema`中的表名是可以重名的。但是我们有时候要反其道而行之，就是要任何情况下表名不同，方便编写`SQL`的时候，可以方便的省略`schema`。这样就需要借助触发器了。像这样
```sql
create function table_create()
  returns event_trigger
language plpgsql
as $$
DECLARE 
        table_name       VARCHAR;
        short_table_name VARCHAR;
BEGIN


  SELECT object_identity
  INTO table_name
  FROM pg_event_trigger_ddl_commands();

  SELECT substr(table_name, position('.' IN table_name) + 1)
  INTO short_table_name;

  SELECT count(*) > 1
  INTO is_exist
  FROM pg_catalog.pg_stat_user_tables
  WHERE relname = short_table_name :: VARCHAR;

  IF is_exist
  THEN
    RAISE EXCEPTION '%  already exists. event:%, command:%. abort.', table_name, TG_EVENT, TG_TAG;
  END IF;

END;
$$;
```
```sql
CREATE EVENT TRIGGER etgr_table_create
ON ddl_command_end
WHEN TAG IN ('CREATE TABLE')
EXECUTE PROCEDURE table_create();
```

#### 4.现在`schema`已经够多了，但是用户连接到`PostgreSQL`的时候，默认只会去`$user`跟`public`这两个`schema`去寻找表，如果要访问别的`schema`还要在用到的时候采用`schema.table`的方式。这点在配置文件`postgresql.conf`中也可以看到
```
#search_path = '"$user", public'        # schema names
```
聪明的你已经猜到，只需要修改配置文件中的这个`search_path`，就可以让用户访问表的时候不必带上`schema`。
但是我倒不建议这么做。因为这个配置文件是全局的。我们可能会有多个`database`运行于此，而他们有着不相干的`schema`，都写在这个配置文件里面显然不合适。我们应该通过下面的方式设置
```sql
alter database "your_database" set search_path = metadata_schema,metadata_schema,platform_schema,public;
```
这样就可以精确到对一个库设置它的默认`search_path`了。设置完毕后，重新建立数据库连接，执行`show search_path;`可以查看设置结果。
除了上面通过`alter database`之外，`alter role`也能起到类似的效果，还靠小伙伴们自行发掘了。