---
title: Freemaker Jpa Generator
date: 2020-03-27 22:33:15
tags: 开源项目
---

<!-- toc -->

[TOC]

#  需求

根据freeMaker模板配置以及数据库字段映射表生成对应的Entity, Repository, CustomRepository,CustomRepositoryImpl等



优先支持Oracle，其次再对mysql进行支持，插件式开发。



# 设计

1.数据库字段和实体类映射关系(oracle和mysql分别读取)

2.配置文件

* 配置生成的类或者interface的freeMaker模板，以及包名

* 配置数据库driver

* 配置需要生成的数据库表，需要支持正则表达式

​     \- 支持Date字段的转换成String的配置(可能需要通用转换器)

* 支持数据表到实体类名称的转换规则配置(replace， 支持正则表达式)

* Id策略看如何配置