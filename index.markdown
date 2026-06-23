---
layout: home
title: 首页
---

# 🔒 Storage Lock

**一个存储后端无关的分布式锁框架** — 基于 Go 语言实现，通过抽象 Storage 接口，支持任何可分布式访问的存储介质来实现分布式锁。

---

## ✨ 核心特性

- 🔄 **存储后端无关** — 同一套锁逻辑可运行在 50+ 种不同的存储后端上
- 🔁 **可重入锁** — 支持同一持有者多次获取同一把锁，按加锁次数释放
- 🔒 **乐观锁（CAS）** — 基于版本号的乐观锁机制，避免 ABA 问题
- 🐕 **看门狗（Watch Dog）** — 自动续期机制，防止业务未完成时锁过期
- 📡 **事件通知** — 锁的获取、释放等事件可通过事件监听器处理
- 📊 **监控集成** — 内置 Prometheus 指标支持

---

## 🚀 快速开始

### 安装

```bash
go get -u github.com/storage-lock/go-storage-lock
```

### 使用示例（以 MySQL 为例）

```go
package main

import (
    "context"
    "log"

    mysql_storage "github.com/storage-lock/go-mysql-storage"
    mysql_locks "github.com/storage-lock/go-mysql-locks"
    storage_lock "github.com/storage-lock/go-storage-lock"
)

func main() {
    // 1. 创建 Storage
    storage, err := mysql_storage.NewMysqlStorage(mysql_storage.NewMysqlStorageOptions{
        Host:     "127.0.0.1",
        Port:     3306,
        Username: "root",
        Password: "password",
        Database: "test",
    })
    if err != nil {
        log.Fatal(err)
    }

    // 2. 创建 Lock
    lock, err := mysql_locks.NewMysqlLocks(mysql_locks.NewMysqlLocksOptions{
        Storage: storage,
    })
    if err != nil {
        log.Fatal(err)
    }

    // 3. 加锁
    err = lock.Lock(context.Background(), "my-lock-key")
    if err != nil {
        log.Fatal(err)
    }
    defer lock.Unlock(context.Background(), "my-lock-key")

    // 4. 执行业务逻辑...
}
```

---

## 🗄️ 支持的存储后端

### 关系型数据库

| 存储 | 包 |
|------|---|
| MySQL | [go-mysql-locks](https://github.com/storage-lock/go-mysql-locks) |
| PostgreSQL | [go-postgresql-locks](https://github.com/storage-lock/go-postgresql-locks) |
| MariaDB | [go-mariadb-locks](https://github.com/storage-lock/go-mariadb-locks) |
| SQL Server | [go-sqlserver-locks](https://github.com/storage-lock/go-sqlserver-locks) |
| TiDB | [go-tidb-locks](https://github.com/storage-lock/go-tidb-locks) |
| Oracle | [go-oracle-storage](https://github.com/storage-lock/go-oracle-storage) |
| SQLite3 | [go-sqlite3-storage](https://github.com/storage-lock/go-sqlite3-storage) |
| OceanBase | [go-oceanbase-storage](https://github.com/storage-lock/go-oceanbase-storage) |
| 达梦 | [go-dameng-storage](https://github.com/storage-lock/go-dameng-storage) |
| ClickHouse | [go-clickhouse-storage](https://github.com/storage-lock/go-clickhouse-storage) |

### ORM 框架

| 框架 | 包 |
|------|---|
| GORM | [go-gorm-locks](https://github.com/storage-lock/go-gorm-locks) |
| sqlx | [go-sqlx-locks](https://github.com/storage-lock/go-sqlx-locks) |
| xorm | [go-xorm-locks](https://github.com/storage-lock/go-xorm-locks) |
| gorp | [go-gorp-locks](https://github.com/storage-lock/go-gorp-locks) |
| beego | [go-beego-locks](https://github.com/storage-lock/go-beego-locks) |
| *sql.DB 通用 | [go-sqldb-locks](https://github.com/storage-lock/go-sqldb-locks) |

### NoSQL

| 存储 | 包 |
|------|---|
| MongoDB | [go-mongodb-locks](https://github.com/storage-lock/go-mongodb-locks) |
| Redis | [go-redis-storage](https://github.com/storage-lock/go-redis-storage) |
| HBase | [go-hbase-storage](https://github.com/storage-lock/go-hbase-storage) |
| Cassandra | [go-cassandra-storage](https://github.com/storage-lock/go-cassandra-storage) |
| Memcache | [go-memcache-storage](https://github.com/storage-lock/go-memcache-storage) |
| DynamoDB | [go-dynamodb-storage](https://github.com/storage-lock/go-dynamodb-storage) |

### 对象存储

| 存储 | 包 |
|------|---|
| S3 | [go-s3-storage](https://github.com/storage-lock/go-s3-storage) |
| MinIO | [go-minio-storage](https://github.com/storage-lock/go-minio-storage) |
| Ceph | [go-ceph-storage](https://github.com/storage-lock/go-ceph-storage) |
| 阿里云 OSS | [go-oss-storage](https://github.com/storage-lock/go-oss-storage) |
| 腾讯云 COS | [go-cos-storage](https://github.com/storage-lock/go-cos-storage) |
| 百度云 BOS | [go-bos-storage](https://github.com/storage-lock/go-bos-storage) |
| 七牛云 Kodo | [go-kodo-storage](https://github.com/storage-lock/go-kodo-storage) |
| 华为云 OBS | [go-obs-storage](https://github.com/storage-lock/go-obs-storage) |
| 金山云 KS3 | [go-ks3-storage](https://github.com/storage-lock/go-ks3-storage) |
| 又拍云 USS | [go-uss-storage](https://github.com/storage-lock/go-uss-storage) |
| 青云 QingStor | [go-qingcloud-object-storage](https://github.com/storage-lock/go-qingcloud-object-storage) |
| US3 | [go-us3-storage](https://github.com/storage-lock/go-us3-storage) |

### 嵌入式 / KV 存储

| 存储 | 包 |
|------|---|
| LevelDB | [go-leveldb-storage](https://github.com/storage-lock/go-leveldb-storage) |
| RocksDB | [go-rocksdb-storage](https://github.com/storage-lock/go-rocksdb-storage) |
| BerkeleyDB | [go-berkeleydb-storage](https://github.com/storage-lock/go-berkeleydb-storage) |
| LedisDB | [go-ledisdb-storage](https://github.com/storage-lock/go-ledisdb-storage) |
| 内存 | [go-memory-locks](https://github.com/storage-lock/go-memory-locks) |

### 分布式文件系统

| 存储 | 包 |
|------|---|
| HDFS | [go-hdfs-storage](https://github.com/storage-lock/go-hdfs-storage) |
| GlusterFS | [go-glusterfs-storage](https://github.com/storage-lock/go-glusterfs-storage) |
| JuiceFS | [go-juicefs-storage](https://github.com/storage-lock/go-juicefs-storage) |
| FastDFS | [go-fastdfs-storage](https://github.com/storage-lock/go-fastdfs-storage) |
| MooseFS | [go-moosefs-storage](https://github.com/storage-lock/go-moosefs-storage) |
| FTP | [go-ftp-storage](https://github.com/storage-lock/go-ftp-storage) |

---

## 🌐 生态项目

| 项目 | 说明 |
|------|------|
| [go-storage-lock](https://github.com/storage-lock/go-storage-lock) | 核心库 — 分布式锁模型定义与算法实现 |
| [go-storage](https://github.com/storage-lock/go-storage) | 存储抽象接口定义 |
| [storage-lock-http-api](https://github.com/storage-lock/storage-lock-http-api) | HTTP API 服务 |
| [storage-lock-grpc-api](https://github.com/storage-lock/storage-lock-grpc-api) | gRPC API 服务 |
| [storage-lock-cli](https://github.com/storage-lock/storage-lock-cli) | 命令行工具 |
| [storage-lock-ui](https://github.com/storage-lock/storage-lock-ui) | Web UI 管理界面 |
| [java-storage-lock](https://github.com/storage-lock/java-storage-lock) | Java SDK |
| [python-storage-lock](https://github.com/storage-lock/python-storage-lock) | Python SDK |
