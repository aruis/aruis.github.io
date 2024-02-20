---
title: 给已存在 PostgreSQL 服务安装 TimescaleDB 插件
date: 2024-02-01 15:15:15
categories: 程序人生
tags:
    - PostgreSQL
---

仅以`docker`版的`PostgreSQL 16`为例。

```
apt update
```
```
apt install lsb-release wget
```
```
echo "deb https://packagecloud.io/timescale/timescaledb/debian/ $(lsb_release -c -s) main" | sudo tee /etc/apt/sources.list.d/timescaledb.list
```
```
wget --quiet -O - https://packagecloud.io/timescale/timescaledb/gpgkey | sudo apt-key add -
```
```
apt update
```
```
apt install timescaledb-2-postgresql-16
```

```
timescaledb-tune
```

```
docker restart xxx
```

```
CREATE EXTENSION IF NOT EXISTS timescaledb;
```

