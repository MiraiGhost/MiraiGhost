---
title: .net使用EFCore
date: 2023-03-14 19:49:57
tags: .net,SqlServer
mermaid: true
---

# .net 搭建 EFCore 开发环境
## 添加 NuGet 包
```bash
Install-Package Microsoft.EntityFrameworkCore.Design
Install-Package Microsoft.VisualStudio.Web.CodeGeneration.Design
Install-Package Microsoft.EntityFrameworkCore.SqlServer
Install-Package Microsoft.EntityFrameworkCore.Tools #生成数据库的工具 
```


## 添加实体类
创建Models文件夹中添加实体类
```bash C#
using System.ComponentModel.DataAnnotation s;
public class User
{
    [key]
    public int Id { get; set; }
    [Required(Error = "{0}不能为空")]
    [Display(Name = "姓名")]
    public string Name { get; set; }
    [Range(0,120)]
    public int Age { get; set; }
}
```
引入 DataAnnotations 以便使用模型验证,
Key 标识表示这个属性在数据库中作为主键
Required 标识表示此属性不能为空，Message 用于自定义错误消息
Range 标识限制值在一定范围


## 添加数据库上下文
在 Data 文件夹添加数据库上下文
```bash ApplicationDbContext
    pulic class ApplicationDbContext : DbContext
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
            : base(options)
            {
            }
        public DbSet<User> User { get; set; }  //User 表名
    }
```

## 依赖注入
ASP.NET Core 通过依赖关系注入 (DI) 生成。 服务（如数据库上下文）在 Program.cs 中向 DI 注册。 这些服务通过构造函数参数提供给需要它们的组件。
```C#
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<AplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnect"))
);
```

## 配置连接字符串
进行本地开发时，ASP.NET Core 配置系统在 appsettings.json 文件中读取 ConnectionString 键。
```json
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "DefaultConnect": "Server = .; Database = dataBaseName; uid = sa; pwd = scce; TrustServerCertificate=true;"  //设置为 true 时，传输层将使用 SSL 来加密通道并跳过证书链以验证信任。
  }
```

## 生成迁移文件 更新数据库
在程序包管理器控制台中执行以下命令:
```bash
Add-Migration InitialCreate
Update-Database
```
 Add-Migration InitialCreate：生成 Migrations/{timestamp}_InitialCreate.cs 迁移文件。 InitialCreate 参数是迁移名称。  

 Update-Database：将数据库更新到上一个命令创建的最新迁移。 此命令在用于创建数据库的 Migrations/{time-stamp}_InitialCreate.cs 文件中运行 Up 方法。

Update-Database XXX：把数据回滚到XXX的状态，迁移脚本不变。
Remove-migration：删除最后一次迁移脚本。
Script-Migration：生成迁移SQL代码。
Script-Migration A B：生成版本A到B的SQL脚本
Script-Migration A：生成版本A到最新版本的SQL脚本

### 反向工程
根据数据库表反向生成实体类
Scaffold-DbContext 'Server=.;Database=demo1;Trusted_Connection=True;MultipleActiveResultSets=true' Micosoft.EntityFrameworkCore.SqlServer

## 数据库种子
在项目中创建 /Data/SeedData.cs ,内容如下
```bash C#
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;
using MvcMovie.Data;
using System;
using System.Linq;

public static class SeedData
{
    public static void Initialize(IServiceProvider serviceProvider)
    {
        using (var context = new ApplicationDbContent(
            serviceProvider.GetRequiredService<
                DbContextOptions<ApplicationDbContent>>()))
        {
            // 如果数据库中有数据，则会返回种子初始值设定项，且不进行种子数据的添加
            if (context.User.Any())
            {
                return;   // DB has been seeded
            }
            context.Movie.AddRange(
                new User
                {
                    Name = "Jack",
                    Age = 12
                },
                new Movie
                {
                    Name = "Alien",
                    Age = 13
                },
                new Movie
                {
                    Name = "John",
                    Age = 11
                }
            );
            context.SaveChanges();
        }
    }
}
```

### 添加种子初始值设定项
```C#
var app = builder.Build();

using (var scope = app.Services.CreateScope())
{
    var services = scope.ServiceProvider;

    SeedData.Initialize(services);
}
```

## EF原理
```mermaid
应用程序 --> Dapper --> ADO.NET Core 
应用程序 --> EF Core --> ADO.NET Core
应用程序 --> FreeSql --> ADO.NET Core
ADO.NET Core --> 数据库
```