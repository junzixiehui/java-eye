


#### 表
- SELECT `TABLE_NAME`, `UPDATE_TIME`  FROM      `information_schema`.`TABLES`  WHERE       `information_schema`.`TABLES`.`TABLE_NAME` = 'tag_10629_20180702';

- show table status where Name = 'tag_10629_20180702';
- 


#### 重复记录

- Select * From 表 Where 重复字段 In (Select 重复字段 From 表 Group By 重复字段 Having Count(*)>1)
- 查找表中多余的重复记录（多个字段）

```
select * from vitae a where (a.peopleId,a.seq) in (select peopleId,seq from vitae group by peopleId,seq having count(*) > 1)
```



#### 建库

CREATE DATABASE sonar CHARACTER SET utf8 COLLATE utf8_general_ci;


#### 权限

本地 Mysql 创建用户并分配权限

```
CREATE USER 'sonar' IDENTIFIED BY 'sonar';
GRANT ALL PRIVILEGES ON *.* TO 'sonar'@'%' IDENTIFIED BY 'sonar' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON *.* TO 'sonar'@'localhost' IDENTIFIED BY 'sonar' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```