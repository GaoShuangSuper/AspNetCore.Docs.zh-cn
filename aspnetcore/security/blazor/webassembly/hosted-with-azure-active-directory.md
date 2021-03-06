---
title: Blazor使用 Azure Active Directory 保护 ASP.NET Core WebAssembly 托管应用
author: guardrex
description: ''
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 05/11/2020
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: security/blazor/webassembly/hosted-with-azure-active-directory
ms.openlocfilehash: 6ff95f0c5c925cbafef2b997a6cb23aeb15ff1aa
ms.sourcegitcommit: 1250c90c8d87c2513532be5683640b65bfdf9ddb
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/12/2020
ms.locfileid: "83153966"
---
# <a name="secure-an-aspnet-core-blazor-webassembly-hosted-app-with-azure-active-directory"></a>Blazor使用 Azure Active Directory 保护 ASP.NET Core WebAssembly 托管应用

作者： [Javier Calvarro 使用](https://github.com/javiercn)和[Luke Latham](https://github.com/guardrex)

[!INCLUDE[](~/includes/blazorwasm-preview-notice.md)]

[!INCLUDE[](~/includes/blazorwasm-3.2-template-article-notice.md)]

本文介绍如何创建使用[Azure Active Directory （AAD）](https://azure.microsoft.com/services/active-directory/)进行身份验证的[ Blazor WebAssembly 托管应用](xref:blazor/hosting-models#blazor-webassembly)。

## <a name="register-apps-in-aad-and-create-solution"></a>在 AAD 中注册应用并创建解决方案

### <a name="create-a-tenant"></a>创建租户

按照[快速入门：设置租户](/azure/active-directory/develop/quickstart-create-new-tenant)中的指导在 AAD 中创建租户。

### <a name="register-a-server-api-app"></a>注册服务器 API 应用

按照[快速入门：向 Microsoft 标识平台注册应用程序](/azure/active-directory/develop/quickstart-register-app)和后续 Azure AAD 主题中的指导，在 Azure 门户的**Azure Active Directory**应用注册 "区域中为*服务器 API 应用*注册 AAD 应用  >  **App registrations** ：

1. 选择“新注册”。 
1. 提供应用的**名称**（例如， ** Blazor 服务器 AAD**）。
1. 选择**受支持的帐户类型**。 对于此体验，你可以选择**仅在此组织目录中的帐户**（单租户）。
1. 在这种情况下，*服务器 API 应用*不需要**重定向 uri** ，因此请将下拉集设置为 " **Web** "，并不要输入 "重定向 uri"。
1. 禁用 "**将**  >  **管理员以免授予 openid 并 offline_access 权限**" 复选框。
1. 选择“注册”  。

在 " **API 权限**" 中，删除**Microsoft Graph**  >  **用户. 读取**权限，因为应用程序不需要登录或 uer 配置文件访问。

在中**公开 API**：

1. 选择“添加范围”。 
1. 选择“保存并继续”。 
1. 提供**作用域名称**（例如 `API.Access` ）。
1. 提供**管理员同意显示名称**（例如 `Access API` ）。
1. 提供**管理员同意说明**（例如 `Allows the app to access server app API endpoints.` ）。
1. 确认 "**状态**" 设置为 "**已启用**"。
1. 选择“添加作用域”。 

记录以下信息：

* *服务器 API 应用*应用程序 ID （客户端 ID）（例如， `11111111-1111-1111-1111-111111111111` ）
* 应用 ID URI （例如，、 `https://contoso.onmicrosoft.com/11111111-1111-1111-1111-111111111111` `api://11111111-1111-1111-1111-111111111111` 或提供的自定义值）
* 目录 ID （租户 ID）（例如， `222222222-2222-2222-2222-222222222222` ）
* AAD 租户域（例如 `contoso.onmicrosoft.com` ）
* 默认作用域（例如 `API.Access` ）

### <a name="register-a-client-app"></a>注册客户端应用

按照[快速入门：向 Microsoft 标识平台注册应用程序](/azure/active-directory/develop/quickstart-register-app)和后续 Azure AAD 主题中的指导，在 Azure 门户的**Azure Active Directory**应用注册 "区域中为*客户端应用程序*注册 AAD 应用程序  >  **App registrations** ：

1. 选择“新注册”。 
1. 提供应用的**名称**（例如， ** Blazor 客户端 AAD**）。
1. 选择**受支持的帐户类型**。 对于此体验，你可以选择**仅在此组织目录中的帐户**（单租户）。
1. 将 "**重定向 uri** " 下拉状态设置为 " **Web**"，并提供的重定向 uri `https://localhost:5001/authentication/login-callback` 。
1. 禁用 "**将**  >  **管理员以免授予 openid 并 offline_access 权限**" 复选框。
1. 选择“注册”  。

在 "**身份验证**  >  **平台配置**"  >  **Web**：

1. 确认存在的**重定向 URI** `https://localhost:5001/authentication/login-callback` 。
1. 对于 "**隐式授予**"，选中 "**访问令牌**" 和 " **ID 令牌**" 对应的复选框。
1. 此体验可接受应用的其余默认值。
1. 选择“保存”按钮  。

在 " **API 权限**：

1. 确认应用程序具有**Microsoft Graph**的 "  >  **用户**" 权限。
1. 选择 "**添加权限**"，然后选择 **"我的 api"**。
1. 从 "**名称**" 列中选择*服务器 API 应用*（例如， ** Blazor 服务器 AAD**）。
1. 打开**API**列表。
1. 启用对 API 的访问（例如 `API.Access` ）。
1. 选择“添加权限”  。
1. 选择 "**为 {租户名称} 授予管理内容**" 按钮。 请选择“是”以确认。 

记录*客户端应用*应用程序 Id （客户端 id）（例如 `33333333-3333-3333-3333-333333333333` ）。

### <a name="create-the-app"></a>创建应用程序

将以下命令中的占位符替换为前面记录的信息，然后在命令行界面中执行命令：

```dotnetcli
dotnet new blazorwasm -au SingleOrg --api-client-id "{SERVER API APP CLIENT ID}" --app-id-uri "{SERVER API APP ID URI}" --client-id "{CLIENT APP CLIENT ID}" --default-scope "{DEFAULT SCOPE}" --domain "{DOMAIN}" -ho --tenant-id "{TENANT ID}"
```

若要指定输出位置（如果它不存在，则创建一个项目文件夹），请在命令中包含带有路径的 output 选项（例如， `-o BlazorSample` ）。 文件夹名称还会成为项目名称的一部分。

> [!NOTE]
> 将应用 ID URI 传递给 `app-id-uri` 选项，但请注意，可能需要在客户端应用中进行配置更改，这在 "[访问令牌范围](#access-token-scopes)" 部分中进行了介绍。

## <a name="server-app-configuration"></a>服务器应用配置

*本部分适用于解决方案的**服务器**应用。*

### <a name="authentication-package"></a>身份验证包

提供对 ASP.NET Core Web Api 的身份验证和授权的支持是由提供的 `Microsoft.AspNetCore.Authentication.AzureAD.UI` ：

```xml
<PackageReference Include="Microsoft.AspNetCore.Authentication.AzureAD.UI" 
    Version="{VERSION}" />
```

### <a name="authentication-service-support"></a>身份验证服务支持

`AddAuthentication`方法在应用中设置身份验证服务，并将 JWT 持有者处理程序配置为默认的身份验证方法。 `AddAzureADBearer`方法在验证 Azure Active Directory 发出的令牌所需的 JWT 持有者处理程序中设置特定参数：

```csharp
services.AddAuthentication(AzureADDefaults.BearerAuthenticationScheme)
    .AddAzureADBearer(options => Configuration.Bind("AzureAd", options));
```

`UseAuthentication`并 `UseAuthorization` 确保：

* 应用尝试分析和验证传入请求的令牌。
* 任何试图访问受保护资源的请求均不正确。

```csharp
app.UseAuthentication();
app.UseAuthorization();
```

### <a name="useridentityname"></a>用户。 Identity路径名

默认情况下，服务器应用 API `User.Identity.Name` 使用声明类型中的值 `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name` （例如）进行填充 `2d64b3da-d9d5-42c6-9352-53d8df33d770@contoso.onmicrosoft.com` 。

若要将应用配置为接收来自 `name` 声明类型的值，请在中配置[TokenValidationParameters](xref:Microsoft.IdentityModel.Tokens.TokenValidationParameters.NameClaimType) <xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerOptions> `Startup.ConfigureServices` ：

```csharp
services.Configure<JwtBearerOptions>(
    AzureADDefaults.JwtBearerAuthenticationScheme, options =>
    {
        options.TokenValidationParameters.NameClaimType = "name";
    });
```

### <a name="app-settings"></a>应用设置

*Appsettings*文件包含用于配置用于验证访问令牌的 JWT 持有者处理程序的选项。

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "{DOMAIN}",
    "TenantId": "{TENANT ID}",
    "ClientId": "{SERVER API APP CLIENT ID}",
  }
}
```

示例：

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "contoso.onmicrosoft.com",
    "TenantId": "e86c78e2-8bb4-4c41-aefd-918e0565a45e",
    "ClientId": "41451fa7-82d9-4673-8fa5-69eff5a761fd",
  }
}
```

### <a name="weatherforecast-controller"></a>WeatherForecast 控制器

WeatherForecast 控制器（*控制器/WeatherForecastController*）公开受保护的 API，该 API 的 `[Authorize]` 属性应用到控制器。 **务必**要了解：

* `[Authorize]`此 api 控制器中的属性只是保护此 api 不受未经授权的访问。
* `[Authorize]`WebAssembly 应用程序中使用的属性 Blazor 仅作为对应用程序的提示，用户应授权该应用程序正常工作。

```csharp
[Authorize]
[ApiController]
[Route("[controller]")]
public class WeatherForecastController : ControllerBase
{
    [HttpGet]
    public IEnumerable<WeatherForecast> Get()
    {
        ...
    }
}
```

## <a name="client-app-configuration"></a>客户端应用配置

*本部分适用于解决方案的**客户端**应用。*

### <a name="authentication-package"></a>身份验证包

创建应用以使用工作或学校帐户（ `SingleOrg` ）时，应用会自动接收[Microsoft 身份验证库](/azure/active-directory/develop/msal-overview)（）的包引用 `Microsoft.Authentication.WebAssembly.Msal` 。 包提供一组基元，可帮助应用对用户进行身份验证，并获取令牌以调用受保护的 Api。

如果向应用程序中添加身份验证，请将包手动添加到应用的项目文件中：

```xml
<PackageReference Include="Microsoft.Authentication.WebAssembly.Msal" 
    Version="{VERSION}" />
```

`{VERSION}`将前面的包引用中的替换为 `Microsoft.AspNetCore.Blazor.Templates` 本文中所示的包版本 <xref:blazor/get-started> 。

`Microsoft.Authentication.WebAssembly.Msal`包可传递将 `Microsoft.AspNetCore.Components.WebAssembly.Authentication` 包添加到应用。

### <a name="authentication-service-support"></a>身份验证服务支持

添加了对实例的支持 `HttpClient` ，其中包括对服务器项目发出请求时的访问令牌。

Program.cs  :

```csharp
builder.Services.AddHttpClient("{APP ASSEMBLY}.ServerAPI", client => 
        client.BaseAddress = new Uri(builder.HostEnvironment.BaseAddress))
    .AddHttpMessageHandler<BaseAddressAuthorizationMessageHandler>();

builder.Services.AddTransient(sp => sp.GetRequiredService<IHttpClientFactory>()
    .CreateClient("{APP ASSEMBLY}.ServerAPI"));
```

使用包提供的扩展方法在服务容器中注册对用户进行身份验证的支持 `AddMsalAuthentication` `Microsoft.Authentication.WebAssembly.Msal` 。 此方法设置应用与 Identity 提供程序（IP）交互所需的所有服务。

Program.cs  :

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    builder.Configuration.Bind("AzureAd", options.ProviderOptions.Authentication);
    options.ProviderOptions.DefaultAccessTokenScopes.Add("{SCOPE URI}");
});
```

`AddMsalAuthentication`方法接受回调，以配置对应用进行身份验证所需的参数。 注册应用时，可以从 Azure 门户 AAD 配置获取配置应用所需的值。

配置由*wwwroot/appsettings*文件提供：

```json
{
  "AzureAd": {
    "Authority": "https://login.microsoftonline.com/{TENANT ID}",
    "ClientId": "{CLIENT APP CLIENT ID}",
    "ValidateAuthority": true
  }
}
```

示例：

```json
{
  "AzureAd": {
    "Authority": "https://login.microsoftonline.com/e86c78e2-...-918e0565a45e",
    "ClientId": "4369008b-21fa-427c-abaa-9b53bf58e538",
    "ValidateAuthority": true
  }
}
```

### <a name="access-token-scopes"></a>访问令牌范围

默认访问令牌范围表示访问令牌作用域的列表：

* 默认情况下，在登录请求中包括。
* 用于在身份验证后立即设置访问令牌。

对于每个 Azure Active Directory 规则，所有作用域都必须属于同一应用。 可以根据需要为其他 API 应用添加其他作用域：

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    ...
    options.ProviderOptions.DefaultAccessTokenScopes.Add("{SCOPE URI}");
});
```

> [!NOTE]
> 如果 Azure 门户提供了作用域 URI，并且应用在从 API 收到*401 的未经授权*响应时**引发了未处理的异常**，请尝试使用不包括方案和主机的范围 uri。 例如，Azure 门户可能提供以下作用域 URI 格式之一：
>
> * `https://{ORGANIZATION}.onmicrosoft.com/{API CLIENT ID OR CUSTOM VALUE}/{SCOPE NAME}`
> * `api://{API CLIENT ID OR CUSTOM VALUE}/{SCOPE NAME}`
>
> 提供不含方案和主机的作用域 URI：
>
> ```csharp
> options.ProviderOptions.DefaultAccessTokenScopes.Add(
>     "{API CLIENT ID OR CUSTOM VALUE}/{SCOPE NAME}");
> ```

有关详细信息，请参阅*其他方案*一文中的以下部分：

* [请求其他访问令牌](xref:security/blazor/webassembly/additional-scenarios#request-additional-access-tokens)
* [将令牌附加到传出请求](xref:security/blazor/webassembly/additional-scenarios#attach-tokens-to-outgoing-requests)


### <a name="imports-file"></a>导入文件

[!INCLUDE[](~/includes/blazor-security/imports-file-hosted.md)]

### <a name="index-page"></a>索引页面

[!INCLUDE[](~/includes/blazor-security/index-page-msal.md)]

### <a name="app-component"></a>应用组件

[!INCLUDE[](~/includes/blazor-security/app-component.md)]

### <a name="redirecttologin-component"></a>RedirectToLogin 组件

[!INCLUDE[](~/includes/blazor-security/redirecttologin-component.md)]

### <a name="logindisplay-component"></a>LoginDisplay 组件

[!INCLUDE[](~/includes/blazor-security/logindisplay-component.md)]

### <a name="authentication-component"></a>身份验证组件

[!INCLUDE[](~/includes/blazor-security/authentication-component.md)]

### <a name="fetchdata-component"></a>FetchData 组件

[!INCLUDE[](~/includes/blazor-security/fetchdata-component.md)]

## <a name="run-the-app"></a>运行应用

从服务器项目运行应用。 使用 Visual Studio 时，请在**解决方案资源管理器**中选择服务器项目，并在工具栏中选择 "**运行**" 按钮，或从 "**调试**" 菜单启动应用程序。

<!-- HOLD
[!INCLUDE[](~/includes/blazor-security/usermanager-signinmanager.md)]
-->

[!INCLUDE[](~/includes/blazor-security/troubleshoot.md)]

## <a name="additional-resources"></a>其他资源

* <xref:security/blazor/webassembly/additional-scenarios>
* [使用安全的默认客户端的应用中未经身份验证或未授权的 web API 请求](xref:security/blazor/webassembly/additional-scenarios#unauthenticated-or-unauthorized-web-api-requests-in-an-app-with-a-secure-default-client)
* <xref:security/blazor/webassembly/aad-groups-roles>
* <xref:security/authentication/azure-active-directory/index>
* [Microsoft 标识平台文档](/azure/active-directory/develop/)
