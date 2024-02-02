---
title: SqlSugar
date: 2023-04-26 00:20:01
tags: .net,database
---

## 介绍
1.  开箱即用，ORM功能齐全,无需第三方组件,多种数据库兼容好，相比EF Core 学习成本低一天学会  【视频教程】 
2.  支持 .NET 百万级【大数据】写入和更新、分表和几十亿查询和统计等 拥有成熟方案
3.  支持 完整的SAAS一套应用 跨库查询 、租户分库 、租户分表 和 租户数据隔离
4.  支持【低代码】+工作流  （动态建类 、动态建表、无实体多库兼容CRUD 、 JSON TO SQL 、自定义XML等）
5.  语法最爽的ORM、优美的表达式、仓储、UnitOfWork、DbContext、AOP 
6.  支持 DbFirst、CodeFirst和【WebFirst】 3种模式开发
7. 简单易用、功能齐全、高性能、轻量级、服务齐全、官网教程文档、有专业技术支持一天18小时服务

## 数据库支持
|关系型数据库|MySql、SqlServer、Sqlite、Oracle 、 postgresql、达梦、

人大金仓(国产推荐)、神通数据库、瀚高、Access 、OceanBase

MySqlConnector、华为 GaussDB 、南大通用 GBase、MariaDB、Odbc、自定义|
|时序性数据库|QuestDb （适合几十亿数据分析,模糊查询，自动分表存储 ，缺点不支持删除）|
|列式存储库|Clickhouse（适用于商业智能领域(BI)，缺点大小写必须和库一样，不支持事务）|

## 配置
### 安装SqlSugarCore包
```bash
Package install SqlSugarCore
```
### 注入
```C#
builder.Service.AddTransient<ISqlSugarClient>(options =>{
    SqlSugarClient db = new SqlSugarClient(new ConnectionConfig(){
        DbType = DbType.SqlServer, // 数据库类型
        ConnectionString = builder.Configuration.GetConnerctionStrgin("DefaultConnection"), // 配置连接字符串
        IsAutoCloseConnection = true // 自动管理连接
    });
    return db
})
```
### 配置连接字符串
appsettings.json
```json
ConnectionString:{
    "DefalultConnection":"Server=服务器地址;DataBase=数据库名称;uid=用户名;pwd=密码"
}
```

## 使用
### 创建实体模型
```C#
public class Stu{
    [SugarColumn(IsPrimaryKey = true, IsIdentity = true)]
    public long Id { get; set; }
    [SugarColumn(IsNullable = false)]
    public string Name { get; set; }
    [SugarColumn()]
    public int age { get; set; }
}
```
### 生成数据库
```C#
_db.DbMaintenance.CreateDatabase();
```
### 生成表
```C#
// 获取程序运行时的程序集，然后获取其命名空间Modle中的所有实体模型，放在数组中
Type[] ass = Assembly.LoadFrom(AppContext.BaseDiectory + "web.dll").GetTypes().Where(t => t.Namespace == "web.Models").ToArry();
// CodeFirst初始化数据库表，设置其字符串默认长度为 200
_db.CodeFirst.SetStringDefaultLength(200).InitTable(ass);
```

### 添加数据
```C#
// 添加数据返回被影响的行数
_db.Insertable(stus).ExecuteCommand();
```
