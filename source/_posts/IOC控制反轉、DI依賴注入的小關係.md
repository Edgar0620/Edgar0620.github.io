---
title: IOC控制反轉、DI依賴注入的小關係
date: 2022-04-12 11:30:00
updated: 2022-04-12 11:30:00
tags: 
    - .Net
    - 設計模式
categories:
    - 心得
---

## 定義
>控制反轉（英語：Inversion of Control，縮寫為IoC），是物件導向程式設計中的一種設計原則，可以用來減低電腦代碼之間的耦合度。其中最常見的方式叫做依賴注入（Dependency Injection，簡稱DI）
[維基百科-控制反轉](https://zh.wikipedia.org/wiki/控制反轉) 

## 心得

- IoC是為了解決:
以往 C# 在此用物件前都必須 new 物件名稱()來建立物件,這會對物件產生依賴關係。
ex. 在取得運費的程式中，拿到Cat公司、Dog公司的運費，就必須分別建立兩個物件出來。

- DI 是IoC實作的方法:
藉著建立WebApplicationBuilder，並註冊Services不同生命週期的物件。
ex. 就像ZOO公司(DI Container)買下(註冊)Cat公司、Dog公司，讓取得運費的程式中依賴的變成ZOO公司(DI Container)。

- 生命週期:
    - Transient 每次都重新建立。
    - Scoped 每次新的Request只建立一次並共用一個實體。
    - Singleton 程式建立時建立一次，直到重啟。

## 實作

- 環境
  - .net 6

- 建立 Interface，並由ICat、IDog 繼承IDelivery
``` C#
    public interface IDelivery
    {
        int GetDeliveryFee();
    }

    public interface ICat:IDelivery
    {
    }

    public interface IDog : IDelivery
    {
    }
```


- 建立Service 實作GetDeliveryFee方法
``` C#
    public class CatService : ICat
    {
        public int GetDeliveryFee()
        {
            return 100;
        }
    }

    public class DogService : IDog
    {
        public int GetDeliveryFee()
        {
            return 80;
        }
    }
```


- Program.cs 註冊各種生命週期的物件
``` C#
    //builder.Services.AddTransient<>();
    //builder.Services.AddScoped<>();
    builder.Services.AddScoped<IDog, DogService>();
    builder.Services.AddSingleton<ICat, CatService>();
```

- 使用 api ，並藉由觀察HashCode了解各生命週期
``` C#
    [Route("api/[controller]/[action]")]
    [ApiController]
    public class DeliveryController : ControllerBase
    {
        private readonly IDelivery _dog;
        private readonly IDelivery _cat;

        public DeliveryController(IDog dog,ICat cat)
        {
            _dog=dog;
            _cat=cat;
        }

        [HttpGet]
        public object Get()
        {
            dynamic obj = new ExpandoObject();
            
            obj.dog=_dog.GetDeliveryFee();
            obj.dogHashCode=_dog.GetHashCode();
            obj.cat=_cat.GetDeliveryFee();
            obj.catHashCode=_cat.GetHashCode();

            return obj;
        }
    }
```
