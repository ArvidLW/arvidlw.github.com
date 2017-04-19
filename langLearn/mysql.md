# mysql学习记录

***

## 基本命令

show databases;

use database;

show tables;

## 远程访问权限

use mysql ;select * from user;

然后里面有不同的用户，以及host, host表示其登录权限是什么。

%：表示可以在任意ip登录

ip：表示只可以在这个ip登录

localhost：表示只可以在本地登录

::1：表示本机，只ipv6地址的缩写。

## 创建用户

create user  'newuser'@'%' identified by 'password';

eg: create user 'discuz'@'%' identified by 'discuz';