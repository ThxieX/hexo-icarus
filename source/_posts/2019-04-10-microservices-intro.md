---
title: 微服务专题 — 开篇
thumbnail: /thumbnails/zixingche.jpg
date: 2019-04-10 00:00:00
categories:
  - 微服务
tags:
  - 分布式
  - 架构
excerpt: "微服务专题系列的开篇，梳理分布式、集群、微服务的联系与区别，介绍 Spring Cloud 微服务技术栈的组成。"
aiSummary: "本文是微服务专题系列的开篇，作者反思了过去对微服务认识的不足——仅停留在 Spring Cloud 某些组件的使用层面，未能理解分布式、集群、微服务之间的联系与区别。文章梳理了微服务架构的核心概念，介绍了 Spring Cloud 技术栈的全貌（包括服务治理、配置中心、网关、熔断等组件），为后续专题深入奠定基础。"
---


## 前言

> 2016 年在公司接触到微服务, 并参与了公司基于微服务架构的产品开发, 但是...
> - 开发内容偏业务, 那时我甚至还没有搞清楚分布式、集群、微服务几者之间的联系和区别
> - 对微服务的认识仅限于 Spring Cloud 技术栈中某几个组件的简单使用和配置...

正如那句标语 `Coordinate Anything`, Spring Cloud 为什么简化了分布式开发, 那时完全说不清楚。

---

后来几年里, 有时会关注 Spring Boot、Spring Cloud 的版本迭代, 尝试去做一些 Demo, 例如 xx 整合 xx。

遗憾的是, 依旧记录的是一堆代码、配置... 没有把一些感受和思考及时记录下来。

所以在清理电脑时, 把相关的学习代码全部删掉了。

只留下 GitHub 上一个两年前提交的学习项目, 虽然版本很老旧, 但是组件整合比较完整:

- [spring-cloud-study](https://github.com/thxiex/spring-cloud-study)

所以准备开一个专题, 来记录自己目前阶段对于微服务架构的思考, 以及各开源组件的选型与实践。

<!-- more -->

## 目的

> **微服务是一种架构风格**, 所以专题内容不局限于代码实践, 不局限于Spring Cloud技术栈!

主要目的:

- 对微服务架构体系有较为全面的认识
- 对微服务技术栈中各组件的架构原理进行深入理解
- 理解和实践微服务技术栈中各组件的业务与运维场景
- 能够从源码中获得一些细节和深层次机制

## 目录（持续更新）

开篇先临时列出能想到的目录; 后续补充阶段会把目录标题逐步替换为文章链接。

### 服务注册与发现

- 目标: 梳理分布式系统中服务注册与服务发现的主流技术方案
- 文章:
  - [服务发现的需求与模式](/posts/service-discovery)
  - [源码分析——服务发现组件 Netflix Eureka](/posts/eureka-source-analysis)
  - [源码分析——客户端负载 Netflix Ribbon](/posts/ribbon-source-analysis)
  - [Ribbon——超时与重试](/posts/ribbon-timeout-retry)
  - [Consul 基本使用](/posts/consul-getting-started)
- 实践:
  - Consul + Docker 服务注册与发现

### 服务间的通讯

- 通讯模型
- 服务间的 IPC 机制及各方案对比: HTTP, RPC
- 实践: 服务间通讯 gRPC

### 服务网关

- 服务网关的要素、功能、架构原理
- 业界流行的服务网关方案与实践
- 核心功能重点源码阅读: Netflix Zuul, Zuul 2.0, Spring Cloud Gateway

### 配置中心

- 核心概念与架构原理
- 基础应用场景与高级特性
- 理解与实践: 携程 Apollo

### 服务跟踪

- 分布式追踪系统架构
- OpenTracing 理论基础及其数据模型
- Spring Cloud Sleuth, Zipkin
- CAT 的理解与实践
- 实践: 分布式链路追踪与展示: Sleuth + Zipkin + Kafka + Elasticsearch + Kibana

### 服务容错

- 场景实践: 服务调用熔断、服务降级、容错限流
- 理论基础: 断路器模式
- Netflix Hystrix 原理
- 实践: Hystrix Dashboard

### 服务监控解决方案

- 实践: Prometheus + Grafana

### 微服务安全

- TODO

### 微服务持续集成与部署实践

- TODO: 另开专题
