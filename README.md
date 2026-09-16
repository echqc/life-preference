# 生活优选

## 📌 项目简介

生活服务类平台，类似大众点评。涵盖商户浏览、博客发布、点赞评论、关注推送、附近商户、秒杀下单等核心功能。
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-2.3.12-brightgreen)](https://spring.io/projects/spring-boot)
[![Redis](https://img.shields.io/badge/Redis-red)](https://redis.io/)
[![MySQL](https://img.shields.io/badge/MySQL-blue)](https://www.mysql.com/)
[![Redisson](https://img.shields.io/badge/Redisson-crimson)](https://github.com/redisson/redisson)
[![Java](https://img.shields.io/badge/Java-1.8-yellow)](https://www.java.com/)

---

## 技术栈

Spring Boot 2.3.12 · Redis (Lettuce) · Redisson 3.37.0 · MySQL 5.6 · MyBatis-Plus 3.5.3.1 · Hutool · Lombok

## 核心功能

- **用户模块** — 手机验证码登录 + Redis Token 认证 + 自动续期
- **商铺模块** — 商铺查询 / 更新，缓存三大问题全链路解决
- **秒杀模块** — Lua 脚本原子操作 + Redis Stream 异步下单 + 一人一单
- **探店笔记** — 发布 / 点赞（ZSet）/ 热门分页 / Feed 流推送
- **关注模块** — 关注 / 取关 / 共同关注（Set 交集）/ 滚动分页
- **附近商铺** — 基于 `GEO` 数据结构实现附近商户搜索、距离计算与排序
- **用户签到** — 基于 `BitMap` 实现每日签到、连续签到天数统计
- **UV 统计** — 基于 `HyperLogLog` 实现百万级 UV 去重（极低内存占用）

## Redis 核心应用

| 场景 | 方案 |
|------|------|
| 缓存穿透 | 缓存空值 + 短期 TTL |
| 缓存击穿 | 互斥锁（SetNX）/ 逻辑过期 + 异步重建 |
| 缓存雪崩 | TTL 随机化 |
| 分布式锁 | SimpleRedisLock（自研）+ Redisson RLock |
| 全局 ID | RedisIdWorker（时间戳 + 每日自增序列号） |
| 异步下单 | Lua 原子判断 → Stream 队列 → 单线程异步消费 |
| Feed 流 | ZSet 按时间排序 + 滚动分页（时间戳 + offset） |
| 附近商铺 | 七、附近商铺 | GEO 坐标存储 + `GEORADIUS` 范围查询 |
| 用户签到 | 八、用户签到 | BitMap 位运算（按年存每日1位）+ 连续签到统计 |
| UV 统计 | 九、UV统计 | HyperLogLog 基数估算（百万级仅需≈12KB内存）|

用户请求下单
       │
       ▼
┌──────────────────────────────────┐ │ Lua 脚本原子判断（一次网络请求） │ │ ① 库存充足？ │ │ ② 用户是否重复下单？ │ │ ③ 扣减库存 │ │ ④ 写入下单记录 │ │ ⑤ 发送到 Stream 队列 │ └──────────────┬───────────────────┘ │ 返回订单ID（前端轮询） ▼ ┌──────────────────────────────────┐ │ 后台单线程异步消费（多订单） │ │ ① XREADGROUP 消费 Stream │ │ ② Redisson 锁保证一人一单 │ │ ③ 乐观锁扣数据库库存（stock>0） │ │ ④ 创建秒杀订单 │ │ ⑤ XACK 确认 + PendingList 恢复 │ └──────────────────────────────────┘





## 🔖 License

MIT
