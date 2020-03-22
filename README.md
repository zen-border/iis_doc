# IIS関連のまとめ

## Windows認証をサイト毎に変更

1. 設定をバックアップする  

    管理者権限でコマンドプロンプトを起動しコマンドを実行する。

``` bash
%systemroot%\system32\inetsrv\appcmd.exe add backup
```

2. 認証設定の継承を許可する  

``` bash
%systemroot%\system32\inetsrv\appcmd.exe unlock config -section:anonymousAuthentication
%systemroot%\system32\inetsrv\appcmd.exe unlock config -section:windowsAuthentication
```

---

## CORS設定

CORS設定(Windows認証)をする。asp.net coreの[ミドルウェア設定](https://docs.microsoft.com/ja-jp/aspnet/core/security/cors?view=aspnetcore-3.1#cors-in-iis)では設定は不可。
匿名認証の場合のみミドルウェア設定で済む。

1. IIS Core Moduleを[インストール](https://www.iis.net/downloads/microsoft/iis-cors-module)する
2. Web.configを設定する  

originは環境に応じて設定を変更

``` xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <location path="." inheritInChildApplications="false">
    <system.webServer>
      <security>
        <authentication>
          <anonymousAuthentication enabled="false" />
          <windowsAuthentication enabled="true" />
        </authentication>
      </security>
      <cors enabled="true">
        <add origin="http://*" allowCredentials="true" />
      </cors>
    </system.webServer>
  </location>
</configuration>
```

---

## ASP. NET Coreで環境変数を読み込む

1. ConfigureAppConfigurationの設定  

    AddEnvironmentVariables()を追加する

``` cs
// Program.cs
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureAppConfiguration((hostingContext, config) =>
        {
            // 環境変数WEBAPP_*を読み込む
            config.AddEnvironmentVariables("WEBAPP_");
        })
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
```

2. ConfigureServicesの設定

    DI(Dependency Injection)するオプション設定クラスを設定する

``` cs
// AppEnvironment.cs
public class AppEnvironment
{
    public string Variable1 { get; set; }
}

// Startup.cs
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllers();
        services.Configure<AppEnvironment>(p => {
            p.Variable1 = Configuration["WEBAPP_VAR_1"];
        });
    }
}
```

