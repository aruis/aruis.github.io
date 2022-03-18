---
title: PostgreSQL中执行即时代码段(匿名存储过程)
date: 2019-02-27 09:52:29
categories: 程序人生
tags:
    - PostgreSQL
---
某些时候需要执行过程化的逻辑，单纯靠组织`SQL`语句已经完成不了了，这个时候一般需要引入存储过程用以实现。但是如果只是单纯执行一段逻辑，而不是要封装一个函数，用来接收参数复用，完全可以通过`DO`语句执行一个匿名代码段。这样就可以避免：创建存储过程 -> 调用存储过程 -> 删除存储过程的窘境。
这里给出一个简短的`SQL`演示：
```
DO
  $$
    declare
      i int := 0;
    begin
      raise notice 'il:%',i;
      declare
        i int;
      begin
        raise notice 'i2:%',i;
      end;
    end
    $$ language plpgsql;
```
