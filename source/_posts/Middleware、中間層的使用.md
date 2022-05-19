---
title: Middleware、中間層的介紹與使用
date: 2022-04-13 11:30:00
updated: 2022-04-13 11:30:00
tags: 
    - .Net
    - Middleware
categories:
    - 心得
---
## 定義
>將個別要求委派指定為內嵌匿名方法 (在內嵌中介軟體中呼叫)，或於可重複使用的類別中加以定義。 這些可重複使用的類別及內嵌匿名方法皆為「中介軟體」。
[ASP.NET Core 中介軟體](https://docs.microsoft.com/zh-tw/aspnet/core/fundamentals/middleware/?view=aspnetcore-6.0) 

## 心得

ASP.NET Core 以Pipeline(管道)的概念，用多個 Middleware 串接起來並具有順序性。

.netcore Middleware 所實現的AOP方式。

主要分成三種不同的擴充方法:
- Run
    - 為最後一個 Middleware
    ``` C#
    var builder = WebApplication.CreateBuilder(args);
    var app = builder.Build();

    app.Run(async context =>
    {
        await context.Response.WriteAsync("Hello.");
    });

    app.Run();
    ```
- Use
    - 自訂 Middleware，next 決定是否呼叫下一層
    ``` C# 
    var builder = WebApplication.CreateBuilder(args);
    var app = builder.Build();

    app.Use(async (context, next) =>
    {
        await context.Response.WriteAsync("Hello Request");
        await next.Invoke();
        await context.Response.WriteAsync("Hello Response");
    });

    app.Run();
    // 最終顯示 Hello Request->下一個Middleware->Hello Response

    ```
    - 擴充方法自訂義 Middleware
    ``` C#
    public class CustomMiddleware
    {
        RequestDelegate _next;

        public CustomMiddleware(RequestDelegate next)
        {
            _next = next;
        }

        public async Task Invoke(HttpContext context)
        {
            // Logic
            await context.Response.WriteAsync("Hello");
            await _next.Invoke(context);
        }
    }

    public static class CustomMiddlewareExtensions
    {
        public static void CustomMiddleware(this IApplicationBuilder app)
        {
            app.UseMiddleware<CustomMiddleware>();
        }
    }

    // Program.cs
    app.CustomMiddleware();

    ```


- Map
    - 用來分支管線
    ``` C#
    app.Map("/map1", map1);

    app.Map("/map2", map2);

    static void map1(IApplicationBuilder app)
    {
        app.Run(async context =>
        {
            await context.Response.WriteAsync("Handle 1");
        });
    }

    static void map2(IApplicationBuilder app)
    {
        app.Run(async context =>
        {
            await context.Response.WriteAsync("Handle 2");
        });
    }

    ```
    - 支援巢狀
    ``` C#
    app.Map("/map1", map1 => {

        map1.Map("/map11", map11 => {
        });
        map1.Map("/map12", map12 => {
        });
    });

    ```
    - 可加入判斷語句
    ``` C#
    app.MapWhen(context => 
        context.Request.Query.ContainsKey("PROD"), HandlePROD
    );

    static void HandlePROD(IApplicationBuilder app)
    {
        app.Run(async context =>
        {
            var environment = context.Request.Query["PROD"];
            await context.Response.WriteAsync($"Environment is  {environment}");
        });
    }

    ```


