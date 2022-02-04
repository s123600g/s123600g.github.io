---
layout: Article/content
title:  "IHttpClientFactory基本用法筆記"
date:   2022-01-02 22:05:00 +0800
categories: [dotnet_core]
tag: dotnet_core
excerpt_separator: <!--more-->
---

此筆記為紀錄工作上碰到開發需要，使用`IHttpClientFactory`來建立管控`HttpClient`生命週期。

依據官方文件(相關參考2)敘述，如果單純使用`HttpClient`進行Http請求，一般作法建立`HttpClient`新實體，再透過此建立實體來進行Http請求動作。

正常使用完畢後，我們都會認為會自動釋放，因為該類別內部有繼承`IDisposable`特性，但是，系統背後通訊端Socket資源佔用問題，卻不會隨著當下實體用完銷毀就跟著被釋放，而後還有其他關於DNS異動調整問題，可以參考相關連結了解詳細。

根據上述狀況，使用官方推薦以`IHttpClientFactory`方式來進行`HttpClient`管控，在此筆記紀錄基本用法與單一實體在生命週期內重複利用。

---

## 主要專案
1. .Net Core 3.1 Console 範例主程式
   * 以此Console進行呼叫Web API。
2. Asp.Net 6 範例Web API
   * Sample API供應範例Console呼叫。

---

## 範例Console主程式

### 安裝相關NutGet套件
* `Microsoft.Extensions.DependencyInjection`
* `Microsoft.Extensions.Http`

### 建立DI與註冊IHttpClientFactory
```csharp
ServiceCollection serviceCollection = new ServiceCollection();

serviceCollection.AddHttpClient();

var serviceBuilder = serviceCollection.BuildServiceProvider();

var httpClientFactory = serviceBuilder.GetService<IHttpClientFactory>();
```

### 實作Http請求範例內容

依據註入`HttpClientFactory`方法`CreateClient`搭配`using`進行範例實作
```csharp
IHttpClientFactory httpClientFactory = GetHttpClientFactory();
var baseAddress = new Uri("http://localhost:5030/");
var requestUrl = "WeatherForecast";

using (var client = httpClientFactory.CreateClient("SampleHttpClient"))
{
   // 設定Time Out
   client.Timeout = TimeSpan.FromSeconds(5);
   client.BaseAddress = baseAddress;

   try
   {
      var response = await client.GetAsync(requestUrl);
      var contentStr = await response.Content.ReadAsStringAsync();

      Console.WriteLine(contentStr);
   }
   catch (TaskCanceledException ex)
   {
      Console.WriteLine("TaskCanceledException");
      Console.WriteLine(ex.Message);
      Console.WriteLine("Call Second Api");

      var postContent = new StringContent("");

      var response2 = await client.PostAsync($"{requestUrl}?name=jyu", postContent);

      var contentStr2 = await response2.Content.ReadAsStringAsync();

      Console.WriteLine(contentStr2);
   }
   catch (Exception ex)
   {
      Console.WriteLine("Exception");
      Console.WriteLine(ex.Message);
   }
}
```

`CreateClient`可以使用命名模式，參數可帶入自訂命名，如果有在Service註冊時，設定指定命名HttpClient配置實體，可以在實際使用時，依據指定命名取得對應已配置好設定實體，如果未有對應則會建立新實體。

詳細可參考 [具名用戶端](https://docs.microsoft.com/zh-tw/aspnet/core/fundamentals/http-requests?view=aspnetcore-6.0#named-clients)

上述Http請求行為如下
* 設定逾時秒數。
* 設定Base Address。
  * 所有呼叫API都在此基底位址下。
* 呼叫第一支API，當逾時發生時，拋出`TaskCanceledException`。
* try-catch攔截到`TaskCanceledException`發生，進行呼叫第二支API。

---

## 範例Web API

使用預設建置`WeatherForecast` Controller模板，原始預設產生`GetWeatherForecast` API調整如下。
```csharp
[HttpGet(Name = "GetWeatherForecast")]
public async Task<IEnumerable<WeatherForecast>> Get()
{
   // 加入此行當收到Request就先等待10秒再進行後續
   await Task.Delay(10000);

   return Enumerable.Range(1, 5).Select(index => new WeatherForecast
   {
      Date = DateTime.Now.AddDays(index),
      TemperatureC = Random.Shared.Next(-20, 55),
      Summary = Summaries[Random.Shared.Next(Summaries.Length)]
   })
   .ToArray();
}
```

在同樣Controller新增第二支API
```csharp
[HttpPost(Name = "PostSecondApi")]
public ActionResult Post(
   [FromQuery] string name
)
{
   return Ok($"Hello {name} !!!");
}
```

---

#### 相關參考

1. [筆記Demo程式碼 - jyu.demo.HttpClientFactory](https://github.com/s123600g/jyu.demo.HttpClientFactory)
2. [在 ASP.NET Core 中使用 IHttpClientFactory 發出 HTTP 要求](https://docs.microsoft.com/zh-tw/aspnet/core/fundamentals/http-requests?view=aspnetcore-3.1#consumption-patterns)
3. [使用 HttpClientFactory 實作復原 HTTP 要求](https://docs.microsoft.com/zh-tw/dotnet/architecture/microservices/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests)
4. [把HttpClient換成IHttpClientFactory之後，放心多了](https://iter01.com/604146.html)
5. [幾種使用IHttpClientFactory方法](https://tiaohsun.netlify.app/posts/%E5%B9%BE%E7%A8%AE%E4%BD%BF%E7%94%A8ihttpclientfactory%E6%96%B9%E6%B3%95/)

