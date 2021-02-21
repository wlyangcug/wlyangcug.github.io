---
layout: post
title: InfluxDB HA 方法预研
categories: InfuxDB
description: InfluxDB 服务高可用方法
keywords: InfluxDB, 时序数据库
---

# InfluxDB HA 方法预研

## 1、Influx Proxy

Influx proxy是一个基于高可用、一致性哈希的InfluxDB集群代理方案，基于饿了么开源的influx-proxy进行进一步的开发和优化（[Github地址](https://github.com/chengshiwen/influx-proxy)）

![](https://tcs.teambition.net/storage/31228f393252b32c46d1a17fa6f104bfe281?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IjVmNTRhZDJmZjBhYWI1MjEzNjQ2MmNmYSIsImV4cCI6MTYxNDMwOTUyOCwiaWF0IjoxNjEzNzA0NzI4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjI4ZjM5MzI1MmIzMmM0NmQxYTE3ZmE2ZjEwNGJmZTI4MSJ9.2qLspsJlC9y1uJkLLVWou47klC_IBo1AaTpHf9Dyp-4&download=influx-proxy.PNG "")

**优点：**

influx-proxy相对于饿了么的influx proxy去除了redis等组件依赖，数据写入相对可靠，有对应的后期扩容策略

**缺点**：存在数据不一致的情况。如果某个influxdb实例A挂掉，此时influx-proxy会将未写入到该实例的数据进行缓存到本机，在influxdb实例A起来后重新写入；但是如果在influxdb实例A回复前，influx-proxy挂了或者发生切换，则事前未写入到inflxudb实例A的数据不再写入，则主备节点数据不一致。

## 2、 360 InfluxDB HA

inflxuDB HA是360内部使用研发的高可用方案，目前还没有开源。

![](https://tcs.teambition.net/storage/31228ebe730e9ae1fd313e34b6834d5614b0?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IjVmNTRhZDJmZjBhYWI1MjEzNjQ2MmNmYSIsImV4cCI6MTYxNDMwOTU0OSwiaWF0IjoxNjEzNzA0NzQ5LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjI4ZWJlNzMwZTlhZTFmZDMxM2UzNGI2ODM0ZDU2MTRiMCJ9.iyKoCxCLyIPaZ0x_caY1qPahjUufLCCoWuG5WWb7Ivc&download=influxdb-ha.PNG "")

**优势为**：

- 以measurement为最小拆分单元，从而保证以时序查询influxdb的高效性；

- 支持业务层动态的拆库、拆表操作。

