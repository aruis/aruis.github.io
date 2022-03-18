---
title: PostgreSQL中实现更新默认值（一）
date: 2018-08-16 07:18:18
categories: 程序人生
tags:
    - PostgreSQL
---
业务系统中，经常会在设计表的时候，考虑这两个字段：新增时间、修改时间。前者用数据库的基础功能即可实现，后者就要采取一些手段了。
在`PostgreSQL`中的最佳实践是采用触发器，捕捉`UPDATE`实践，虽然听起来很可怕，但其实并不难。
1. 首先创建一个函数，作用就是给一行数据，追加某个值（这里我用的字段名是`t_update`）
    
    ```
    CREATE OR REPLACE FUNCTION update_modified_column()
      RETURNS TRIGGER AS $$
    BEGIN
      NEW.t_update = now();
      RETURN NEW;
    END;
    $$
    language 'plpgsql';
    ```
    
2. 然后就是把这个函数怼到触发器上
    
    ```
    CREATE TRIGGER tgr_auto_t_update
      BEFORE UPDATE
      ON a_table
      FOR EACH ROW EXECUTE PROCEDURE update_modified_column()
    ```
    
这样就大功告成了。但是如果有很多表的话，感觉还是有重复劳动，不够优雅。明天我将结合昨天提到的表继承，最大化的减少工作量，用更优雅的方法实现更新默认值的功能。