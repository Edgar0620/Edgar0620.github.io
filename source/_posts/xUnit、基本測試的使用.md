---
title: xUnit、基本測試的使用
date: 2022-04-14 11:30:00
updated: 2022-04-14 11:30:00
tags: 
    - .Net
    - xUnit
categories:
    - 心得
---
## 定義
>xUnit.net is a free, open source, community-focused unit testing tool for the .NET Framework。
[xUnit.net](https://xunit.net/) 

## 心得

建立一個基本的 ApiController 的 xUnit

- 基本的api並包含 Interface
    ``` C#
    [Route("api/[controller]/[action]")]
    [ApiController]
    public class DeliveryController : ControllerBase
    {
        private readonly IDelivery _dog;
        private readonly IDelivery _cat;

        public DeliveryController(IDog dog, ICat cat)
        {
            _dog = dog;
            _cat = cat;
        }

        [HttpGet]
        public int Get()
        {
            // 100 + 80
            return _dog.GetDeliveryFee()+ _cat.GetDeliveryFee();
        }
    }
    ```
- 建立 xUnit專案
    ``` C#
    [Fact]
        public async void Sum_DeliveryFee_180()
        {
            // Mock
            var mockDogRepo = new Mock<IDog>();
            var mockCatRepo = new Mock<ICat>();

            mockDogRepo.Setup(repo => repo.GetDeliveryFee()).Returns(80);
            mockCatRepo.Setup(repo => repo.GetDeliveryFee()).Returns(100);

            var controller = new DeliveryController(mockDogRepo.Object, mockCatRepo.Object);

            var result =  controller.Get();

            int sum =Assert.IsType<int>(result);

            Assert.Equal(180, sum);
        }
    ```

