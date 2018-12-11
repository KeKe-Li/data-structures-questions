### 数据库

数据库这块我们先从在开发中常用的数据库Mysql开始，MySQL是一个关系型数据库管理系统，由瑞典MySQL AB 公司开发，目前属于 Oracle 旗下公司。MySQL 最流行的关系型数据库管理系统，在 WEB 应用方面MySQL是最好的 RDBMS (Relational Database Management System，关系数据库管理系统) 应用软件之一。

MySQL基础:

* [MySQL的多存储引擎架构](#MySQL的多存储引擎架构)
* [数据库的事务](#数据库的事务)
* [数据库的事务](#数据库的事务)

#### MySQL的多存储引擎架构
<p align="center">
<img width="500" align="center" src="../images/2.jpg" />
</p>

MySQL作为一个大型的网络程序、数据管理系统，架构非常复杂。

#### 数据库的事务
<p align="center">
<img width="500" align="center" src="../images/3.jpg" />
</p>

数据库的事务指的是满足数据库的 ACID 特性的一组操作，可以通过 Commit 提交一个事务，也可以使用 Rollback 进行回滚。

MySQL 中默认采用自动提交(AUTOCOMMIT)模式。如果不显式使用 START TRANSACTION 语句来开始一个事务，那么每个查询都会被当做一个事务自动提交。
