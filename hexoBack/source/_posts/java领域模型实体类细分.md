---
title: java领域模型实体类细分
date: 2017-07-20 16:52:05
tags: java
---
领域模型中的实体类对象可以细分为4种类型：VO,DTO,DO,PO
<!-- more -->
## 4种类型
 - VO(View Object)：视图对象，展示视图状态
 - DTO(Data Transfer Object)：视图和服务器之间的传输对象
 - DO(Domain Object)：业务实体对象
 - PO(Persistent Object)：持久化对象，直接对应数据库表
 
从分层的角度来说，PO,DO/DTO,VO分别属于持久层，服务处和展现层。
