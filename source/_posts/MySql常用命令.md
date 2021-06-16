---
title: MySql常用命令
date: 2020-03-27 22:29:44
tags: mysql
---

[TOC]

## 启动关闭mysql

net start mysql
net stop mysql

## 创建表空间

create database 空间名 default character set utf8 collate utf8_bin;

## 创建表

CREATE TABLE `MEETING_ROOM` (
`UNIQUEID`  int(20) NOT NULL ,
`NAME`  varchar(255) NULL ,
`ABBREVIATION`  varchar(255) NULL ,
`LIMIT`  integer(255) NULL ,
`LOCATION`  varchar(500) NULL ,
PRIMARY KEY (`UNIQUEID`)
);

# **问题**

**授权方式解决Host '192.168.1.180' is not allowed to connect to this MySQL server**

​	GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'; 

https://blog.csdn.net/vin_1991216/article/details/82632710

