---
layout: page
title: 关于
permalink: /about/
---

# 关于 Storage Lock

## 设计理念

在分布式系统中，分布式锁是一个基础且关键的需求。然而，现有的分布式锁实现通常与特定的存储后端强耦合——比如 Redis 的 Redlock、Zookeeper 的临时节点、etcd 的租约等。

**Storage Lock 的核心理念是：将锁的逻辑与存储后端完全解耦。** 锁的算法应该是通用的，存储介质只是锁状态的持久化载体。只要存储介质满足基本的读写接口，就可以用来实现分布式锁。

这意味着：

- 你不需要为了使用分布式锁而引入新的存储组件——使用你现有的 MySQL、PostgreSQL、MongoDB 即可
- 你可以在不同的存储后端之间自由切换，而无需修改业务代码
- 你可以在开发环境用内存存储，生产环境切换到 Redis 或数据库，零代码改动

---

## 架构设计

```
┌─────────────────────────────────────────────┐
│              应用层 (Application)             │
│   HTTP API / gRPC API / CLI / UI / SDK      │
├─────────────────────────────────────────────┤
│           锁核心 (go-storage-lock)           │
│                                             │
│   ┌─────────────┐  ┌──────────────────┐    │
│   │  Lock/Unlock │  │  Watch Dog 续期   │    │
│   └─────────────┘  └──────────────────┘    │
│   ┌─────────────┐  ┌──────────────────┐    │
│   │  可重入锁     │  │  事件通知         │    │
│   └─────────────┘  └──────────────────┘    │
├─────────────────────────────────────────────┤
│          存储抽象 (go-storage)               │
│                                             │
│   Storage 接口：                              │
│   - GetName()                                │
│   - Init(ctx)                                │
│   - Get(ctx, lockId)                         │
│   - Create(ctx, lockId, lockInfo)            │
│   - UpdateWithVersion(ctx, lockId, ...)      │
│   - DeleteWithVersion(ctx, lockId, ...)      │
├─────────────────────────────────────────────┤
│              存储实现 (50+ 种)                │
│  MySQL / Redis / MongoDB / S3 / ...          │
└─────────────────────────────────────────────┘
```

### 核心模型

#### LockInformation

锁的信息模型，持久化保存在存储介质中：

```go
type LockInformation struct {
    LockId         string    // 锁的唯一 ID
    OwnerId        string    // 当前持有者的全局唯一 ID
    Version        Version   // 版本号，乐观锁避免 ABA 问题
    LockCount      int       // 加锁次数，支持可重入锁
    LockBeginTime  time.Time // 锁开始持有的时间
    LeaseExpireTime time.Time // 租约过期时间
}
```

#### Storage 接口

定义了存储介质必须实现的五个核心方法：

- **GetName()** — 返回存储名称
- **Init(ctx)** — 初始化操作（如创建表）
- **Get(ctx, lockId)** — 读取锁信息
- **Create(ctx, lockId, lockInfo)** — 创建锁记录
- **UpdateWithVersion(ctx, lockId, expectedVersion, newInfo)** — 带版本号的更新（CAS）
- **DeleteWithVersion(ctx, lockId, expectedVersion)** — 带版本号的删除

#### 乐观锁（CAS）

通过版本号（Version）实现乐观锁机制，每次锁状态变更时版本号递增。更新和删除操作都要求传入期望的版本号，只有版本号匹配时操作才会成功，从而避免 ABA 问题。

#### 看门狗（Watch Dog）

看门狗是一个后台协程，在锁的持有期间自动续期租约。当业务逻辑执行时间超过初始租约时间时，看门狗会自动延长锁的过期时间，防止锁意外过期导致并发问题。当业务完成后释放锁，看门狗会自动停止。

---

## 抽象层

为了减少重复代码，项目提供了几层通用抽象：

| 抽象层 | 包 | 说明 |
|--------|---|------|
| SQL 通用 | [go-sql-based-storage](https://github.com/storage-lock/go-sql-based-storage) | 所有 SQL 数据库的公共实现 |
| 对象存储通用 | [go-object-based-storage](https://github.com/storage-lock/go-object-based-storage) | 所有对象存储的公共实现 |
| 文件系统通用 | [go-filesystem-storage](https://github.com/storage-lock/go-filesystem-storage) | 所有文件系统存储的公共实现 |

---

## License

[MIT License](https://github.com/storage-lock/.github/blob/main/LICENSE)

Copyright (c) 2023 Storage Lock
