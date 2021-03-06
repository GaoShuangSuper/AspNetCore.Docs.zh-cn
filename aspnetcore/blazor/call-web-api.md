---
title: 从 ASP.NET Core Blazor WebAssembly 调用 Web API
author: guardrex
description: 了解如何使用 JSON 帮助程序从 Blazor WebAssembly 应用调用 Web API，包括发出跨域资源共享 (CORS) 请求。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 05/04/2020
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: blazor/call-web-api
ms.openlocfilehash: d823db3688e05f6befefacc9f390e0dcdbf329a7
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/04/2020
ms.locfileid: "82767143"
---
# <a name="call-a-web-api-from-aspnet-core-blazor"></a>从 ASP.NET Core Blazor 调用 Web API

作者：[Luke Latham](https://github.com/guardrex)、[Daniel Roth](https://github.com/danroth27) 和 [Juan De la Cruz](https://github.com/juandelacruz23)

[!INCLUDE[](~/includes/blazorwasm-preview-notice.md)]

[Blazor WebAssembly](xref:blazor/hosting-models#blazor-webassembly) 应用使用预配置的 `HttpClient` 服务调用 Web API。 使用 Blazor JSON 帮助程序或通过 <xref:System.Net.Http.HttpRequestMessage> 撰写请求，其中可以包含 JavaScript [Fetch API ](https://developer.mozilla.org/docs/Web/API/Fetch_API) 选项。 Blazor WebAssembly 应用中的 `HttpClient` 服务侧重于使请求返回源服务器。 本主题中的指导仅适用于 Blazor WebAssembly 应用。

[Blazor 服务器](xref:blazor/hosting-models#blazor-server)应用使用 <xref:System.Net.Http.HttpClient> 实例（通常是使用 <xref:System.Net.Http.IHttpClientFactory> 创建）调用 Web API。 本主题中的指导不适用于 Blazor 服务器应用。 开发 Blazor 服务器应用时，请按照 <xref:fundamentals/http-requests> 中的指导进行操作。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/)（[如何下载](xref:index#how-to-download-a-sample)）&ndash; 选择 BlazorWebAssemblySample  应用。

请查看 BlazorWebAssemblySample  示例应用中的以下组件：

* 调用 Web API (Pages/CallWebAPI.razor  )
* HTTP 请求测试器 (Components/HTTPRequestTester.razor  )

## <a name="packages"></a>package

在项目文件中引用 [System.Net.Http.Json](https://www.nuget.org/packages/System.Net.Http.Json/) NuGet 包。

## <a name="add-the-httpclient-service"></a>添加 HttpClient 服务

在 `Program.Main` 中个，如果 `HttpClient` 服务尚不存在，则添加它：

```csharp
builder.Services.AddTransient(sp => 
    new HttpClient
    {
        BaseAddress = new Uri(builder.HostEnvironment.BaseAddress)
    });
```

## <a name="httpclient-and-json-helpers"></a>HttpClient 和 JSON 帮助程序

在 Blazor WebAssembly 应用中，[HttpClient](xref:fundamentals/http-requests) 作为预配置服务提供，用于使请求返回源服务器。

默认情况下，Blazor 服务器应用不包含 `HttpClient` 服务。 使用 [HttpClient 工厂基础结构](xref:fundamentals/http-requests)向应用提供 `HttpClient`。

`HttpClient` 和 JSON 帮助程序还用于调用第三方 Web API 终结点。 `HttpClient` 使用浏览器 [Fetch API](https://developer.mozilla.org/docs/Web/API/Fetch_API) 来实现，受其限制制约，包括强制实施相同初始策略。

客户端的基址设置为原始服务器的地址。 使用 `@inject` 指令插入 `HttpClient` 实例：

```razor
@using System.Net.Http
@inject HttpClient Http
```

在下面的示例中，Todo Web API 处理创建、读取、更新和删除 (CRUD) 操作。 这些示例基于存储以下内容的 `TodoItem` 类：

* ID（`Id`，`long`）&ndash; 项目的唯一 ID。
* 名称（`Name`，`string`）&ndash; 项目的名称。
* 状态（`IsComplete``bool`）&ndash; 指示 Todo 项是否已完成。

```csharp
private class TodoItem
{
    public long Id { get; set; }
    public string Name { get; set; }
    public bool IsComplete { get; set; }
}
```

JSON 帮助程序方法将请求发送到 URI（以下示例中的 Web API）并处理响应：

* `GetFromJsonAsync` &ndash; 发送 HTTP GET 请求并分析 JSON 响应正文以创建对象。

  在下面的代码中，`_todoItems` 由组件显示。 当组件完成呈现 ([OnInitializedAsync](xref:blazor/lifecycle#component-initialization-methods)) 时，会触发 `GetTodoItems` 方法。 有关完整的示例，请参阅示例应用。

  ```razor
  @using System.Net.Http
  @inject HttpClient Http

  @code {
      private TodoItem[] _todoItems;

      protected override async Task OnInitializedAsync() => 
          _todoItems = await Http.GetFromJsonAsync<TodoItem[]>("api/TodoItems");
  }
  ```

* `PostAsJsonAsync` &ndash; 发送 HTTP POST 请求（包括 JSON 编码的内容），并分析 JSON 响应正文以创建对象。

  在下面的代码中，`_newItemName` 由组件的绑定元素提供。 通过选择 `<button>` 元素来触发 `AddItem` 方法。 有关完整的示例，请参阅示例应用。

  ```razor
  @using System.Net.Http
  @inject HttpClient Http

  <input @bind="_newItemName" placeholder="New Todo Item" />
  <button @onclick="@AddItem">Add</button>

  @code {
      private string _newItemName;

      private async Task AddItem()
      {
          var addItem = new TodoItem { Name = _newItemName, IsComplete = false };
          await Http.PostAsJsonAsync("api/TodoItems", addItem);
      }
  }
  ```
  
  调用 `PostAsJsonAsync` 会返回 <xref:System.Net.Http.HttpResponseMessage>。 若要对响应消息中的 JSON 内容进行反序列化处理，请使用 `ReadFromJsonAsync<T>` 扩展方法：
  
  ```csharp
  var content = response.content.ReadFromJsonAsync<WeatherForecast>();
  ```

* `PutAsJsonAsync` &ndash; 发送 HTTP PUT 请求（包括 JSON 编码的内容）。

  在下面的代码中，`Name` 和 `IsCompleted` 的 `_editItem` 值由组件的绑定元素提供。 当在 UI 的另一个部分中选择项并调用 `EditItem` 时，会设置项的 `Id`。 通过选择 Save `<button>` 元素来触发 `SaveItem` 方法。 有关完整的示例，请参阅示例应用。

  ```razor
  @using System.Net.Http
  @inject HttpClient Http

  <input type="checkbox" @bind="_editItem.IsComplete" />
  <input @bind="_editItem.Name" />
  <button @onclick="@SaveItem">Save</button>

  @code {
      private TodoItem _editItem = new TodoItem();

      private void EditItem(long id)
      {
          var editItem = _todoItems.Single(i => i.Id == id);
          _editItem = new TodoItem { Id = editItem.Id, Name = editItem.Name, 
              IsComplete = editItem.IsComplete };
      }

      private async Task SaveItem() =>
          await Http.PutAsJsonAsync($"api/TodoItems/{_editItem.Id}, _editItem);
  }
  ```
  
  调用 `PutAsJsonAsync` 会返回 <xref:System.Net.Http.HttpResponseMessage>。 若要对响应消息中的 JSON 内容进行反序列化处理，请使用 `ReadFromJsonAsync<T>` 扩展方法：
  
  ```csharp
  var content = response.content.ReadFromJsonAsync<WeatherForecast>();
  ```

<xref:System.Net.Http> 包括用于发送 HTTP 请求和接收 HTTP 响应的附加扩展方法。 [HttpClient.DeleteAsync](xref:System.Net.Http.HttpClient.DeleteAsync*) 用于将 HTTP DELETE 请求发送到 Web API。

在下面的代码中，Delete `<button>` 元素调用 `DeleteItem` 方法。 绑定 `<input>` 元素提供要删除的项的 `id`。 有关完整的示例，请参阅示例应用。

```razor
@using System.Net.Http
@inject HttpClient Http

<input @bind="_id" />
<button @onclick="@DeleteItem">Delete</button>

@code {
    private long _id;

    private async Task DeleteItem() =>
        await Http.DeleteAsync($"api/TodoItems/{_id}");
}
```

## <a name="cross-origin-resource-sharing-cors"></a>跨域资源共享 (CORS)

浏览器安全可防止网页向不同域（而不是向网页提供服务的域）进行请求。 此限制称为同域策略  。 同域策略可防止恶意站点从另一站点读取敏感数据。 若要从浏览器向具有不同源的终结点进行请求，终结点  必须启用[跨域资源共享 (CORS)](https://www.w3.org/TR/cors/)。

[Blazor WebAssembly 示例应用 (BlazorWebAssemblySample)](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/) 演示如何在调用 Web API 组件 (Pages/CallWebAPI.razor  ) 中使用 CORS。

若要允许其他站点对应用进行跨域资源共享 (CORS) 请求，请参阅 <xref:security/cors>。

## <a name="additional-resources"></a>其他资源

* <xref:security/blazor/webassembly/index>
* <xref:security/blazor/webassembly/additional-scenarios>
* <xref:fundamentals/http-requests>
* <xref:security/enforcing-ssl>
* [Kestrel HTTPS 终结点配置](xref:fundamentals/servers/kestrel#endpoint-configuration)
* [W3C 上的跨域资源共享 (CORS)](https://www.w3.org/TR/cors/)
