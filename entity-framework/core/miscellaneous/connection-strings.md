---
title: 连接字符串的 EF Core
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: aeb0f5f8-b212-4f89-ae83-c642a5190ba0
uid: core/miscellaneous/connection-strings
ms.openlocfilehash: 7bb39d260f700e5087673e92a50377dc68151710
ms.sourcegitcommit: 85ccc9ed42d4aaf7525c6312058c5c9ebdaed3ae
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/27/2018
ms.locfileid: "50191337"
---
# <a name="connection-strings"></a>连接字符串

大多数数据库提供程序需要某种形式的连接字符串以连接到数据库。 有时此连接字符串包含必须受到保护的敏感信息。 您可能还需要更改连接字符串，如环境，如开发、 测试和生产环境之间移动你的应用程序。

## <a name="net-framework-applications"></a>.NET framework 应用程序

.NET framework 应用程序，如 WinForms、 WPF、 控制台和 ASP.NET 4 中，具有经过反复测试的连接字符串模式。 应将连接字符串添加到你的应用程序的 App.config 文件 (如果您使用的 ASP.NET Web.config)。 如果你的连接字符串包含敏感信息，如用户名和密码，你可以保护配置文件使用的内容[受保护的配置](https://docs.microsoft.com/dotnet/framework/data/adonet/connection-strings-and-configuration-files#encrypting-configuration-file-sections-using-protected-configuration)。

``` xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>

  <connectionStrings>
    <add name="BloggingDatabase"
         connectionString="Server=(localdb)\mssqllocaldb;Database=Blogging;Trusted_Connection=True;" />
  </connectionStrings>
</configuration>
```

> [!TIP]  
> `providerName`上 EF Core连接字符串存储在 App.config，因为通过代码配置了数据库提供程序不需要设置。

然后可以读取连接字符串使用`ConfigurationManager`在您的上下文中的 API`OnConfiguring`方法。 可能需要添加对引用`System.Configuration`framework 程序集，以便能够使用此 API。

``` csharp
public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
      optionsBuilder.UseSqlServer(ConfigurationManager.ConnectionStrings["BloggingDatabase"].ConnectionString);
    }
}
```

## <a name="universal-windows-platform-uwp"></a>通用 Windows 平台 (UWP)

中的 UWP 应用程序的连接字符串通常是 SQLite 连接，只需指定本地文件名。 它们通常不包含敏感信息，并且无需更改，因为应用程序已部署。 在这种情况下，这些连接字符串是保持在代码中，通常没问题，如下所示。 如果你想要使它们摆脱代码 UWP 支持的设置概念，请参阅[UWP 文档的应用设置部分](https://docs.microsoft.com/windows/uwp/app-settings/store-and-retrieve-app-data)有关详细信息。

``` csharp
public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
            optionsBuilder.UseSqlite("Data Source=blogging.db");
    }
}
```

## <a name="aspnet-core"></a>ASP.NET Core

在 ASP.NET Core 配置系统是非常灵活，并在连接字符串可以存储在`appsettings.json`，环境变量、 用户密钥存储或另一个配置源。 请参阅[配置部分中的 ASP.NET Core 文档](https://docs.asp.net/en/latest/fundamentals/configuration.html)的更多详细信息。 下面的示例演示中存储的连接字符串`appsettings.json`。

``` json
{
  "ConnectionStrings": {
    "BloggingDatabase": "Server=(localdb)\\mssqllocaldb;Database=EFGetStarted.ConsoleApp.NewDb;Trusted_Connection=True;"
  },
}
```

通常在中配置上下文`Startup.cs`正在从配置中读取的连接字符串。 请注意`GetConnectionString()`方法查找配置值，其键与`ConnectionStrings:<connection string name>`。 需要导入[Microsoft.Extensions.Configuration](https://docs.microsoft.com/dotnet/api/microsoft.extensions.configuration)命名空间来使用此扩展方法。

``` csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<BloggingContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("BloggingDatabase")));
}
```
