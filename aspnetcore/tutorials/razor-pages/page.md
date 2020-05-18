---
title: ASP.NET Core 中已搭建基架的 Razor Pages
author: rick-anderson
description: 介绍通过搭建基架生成的 Razor Pages。
ms.author: riande
ms.date: 08/17/2019
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: tutorials/razor-pages/page
ms.openlocfilehash: 22afbc729cc73427b3d04bee379534cda38b39bd
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/04/2020
ms.locfileid: "82774842"
---
# <a name="scaffolded-razor-pages-in-aspnet-core"></a>ASP.NET Core 中已搭建基架的 Razor 页面

::: moniker range=">= aspnetcore-3.0"

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)

本教程介绍在[上一教程](xref:tutorials/razor-pages/model)中通过搭建基架创建的 Razor 页面。

[!INCLUDE[View or download sample code](~/includes/rp/download.md)]

## <a name="the-create-delete-details-and-edit-pages"></a>“创建”、“删除”、“详细信息”和“编辑”页面

检查 Pages/Movies/Index.cshtml.cs  页面模型：

[!code-csharp[](razor-pages-start/snapshot_sample3/RazorPagesMovie30/Pages/Movies/Index.cshtml.cs)]

Razor 页面派生自 `PageModel`。 按照约定，`PageModel` 派生的类称为 `<PageName>Model`。 此构造函数使用[依赖关系注入](xref:fundamentals/dependency-injection)将 `RazorPagesMovieContext` 添加到页。 所有已搭建基架的页面都遵循此模式。 请参阅[异步代码](xref:data/ef-rp/intro#asynchronous-code)，了解有关使用实体框架的异步编程的详细信息。

对页面发出请求时，`OnGetAsync` 方法向 Razor 页面返回影片列表。 调用 `OnGetAsync` 或 `OnGet` 以初始化页面的状态。 在这种情况下，`OnGetAsync` 将获得影片列表并显示出来。

当 `OnGet` 返回 `void` 或 `OnGetAsync` 返回 `Task` 时，不使用任何返回语句。 当返回类型是 `IActionResult` 或 `Task<IActionResult>` 时，必须提供返回语句。 例如，Pages/Movies/Create.cshtml.cs `OnPostAsync` 方法  ：

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Create.cshtml.cs?name=snippet)]

检查 Pages/Movies/Index.cshtml<a name="index"></a> Razor 页面  ：

[!code-cshtml[](razor-pages-start/snapshot_sample3/RazorPagesMovie30/Pages/Movies/Index.cshtml)]

Razor 可以从 HTML 转换为 C# 或 Razor 特定标记。 当 `@` 符号后跟 [Razor 保留关键字](xref:mvc/views/razor#razor-reserved-keywords)时，它会转换为 Razor 特定标记，否则会转换为 C#。

### <a name="the-page-directive"></a>@page 指令

`@page` Razor 指令将文件转换为一个 MVC 操作，这意味着它可以处理请求。 `@page` 必须是页面上的第一个 Razor 指令。 `@page` 是转换到 Razor 特定标记的一个示例。 有关详细信息，请参阅 [Razor 语法](xref:mvc/views/razor#razor-syntax)。

检查以下 HTML 帮助程序中使用的 Lambda 表达式：

```cshtml
@Html.DisplayNameFor(model => model.Movie[0].Title)
```

`DisplayNameFor` HTML 帮助程序检查 Lambda 表达式中引用的 `Title` 属性来确定显示名称。 检查 Lambda 表达式（而非求值）。 这意味着当 `model`、`model.Movie` 或 `model.Movie[0]` 为 `null` 或为空时，不会存在任何访问冲突。 对 Lambda 表达式求值时（例如，使用 `@Html.DisplayFor(modelItem => item.Title)`），将求得该模型的属性值。

<a name="md"></a>

### <a name="the-model-directive"></a>@model 指令

[!code-cshtml[](razor-pages-start/snapshot_sample3/RazorPagesMovie30/Pages/Movies/Index.cshtml?range=1-2&highlight=2)]

`@model` 指令指定传递给 Razor 页面的模型类型。 在前面的示例中，`@model` 行使 `PageModel` 派生的类可用于 Razor 页面。 在页面上的 `@Html.DisplayNameFor` 和 `@Html.DisplayFor` [HTML 帮助程序](/aspnet/mvc/overview/older-versions-1/views/creating-custom-html-helpers-cs#understanding-html-helpers)中使用该模型。

### <a name="the-layout-page"></a>布局页

选择菜单链接（“RazorPagesMovie”  、“主页”  和“隐私”  ）。 每页显示相同的菜单布局。 菜单布局是在 Pages/Shared/_Layout.cshtml  文件中实现。 打开 Pages/Shared/_Layout.cshtml  文件。

[布局](xref:mvc/views/layout)模板允许 HTML 容器具有如下布局：

* 在一个位置指定。
* 应用于站点中的多个页面。

查找 `@RenderBody()` 行。 `RenderBody` 是显示全部页面专用视图的占位符，已包装  在布局页中。 例如，选择“隐私”  链接后，Pages/Privacy.cshtml  视图在 `RenderBody` 方法中呈现。

<a name="vd"></a>

### <a name="viewdata-and-layout"></a>ViewData 和布局

考虑来自 Pages/Movies/Index.cshtml  文件中的以下标记：

[!code-cshtml[](razor-pages-start/snapshot_sample3/RazorPagesMovie30/Pages/Movies/Index.cshtml?range=1-6&highlight=4-999)]

前面突出显示的标记是 Razor 转换为 C# 的一个示例。 `{` 和 `}` 字符括住 C# 代码块。

`PageModel` 基类包含 `ViewData` 字典属性，可用于将数据传递到某个视图。 可以使用键/值模式将对象添加到 `ViewData` 字典。 在前面的示例中，`"Title"` 属性被添加到 `ViewData` 字典。

`"Title"` 属性用于 Pages/Shared/_Layout.cshtml 文件  。 以下标记显示 _Layout.cshtml 文件的前几行  。

<!-- we need a snapshot copy of layout because we are
changing in in the next step.
-->
[!code-cshtml[](razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/NU/_Layout.cshtml?highlight=6)]

行 `@*Markup removed for brevity.*@` 为 Razor 注释。 与 HTML 注释不同 (`<!-- -->`)，Razor 注释不会发送到客户端。

### <a name="update-the-layout"></a>更新布局

更改 Pages/Shared/_Layout.cshtml 文件中的 `<title>` 元素以显示 Movie 而不是 RazorPagesMovie    。

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie30/Pages/Shared/_Layout.cshtml?range=1-6&highlight=6)]

在 Pages/Shared/_Layout.cshtml  文件中，查找以下定位点元素。

```cshtml
<a class="navbar-brand" asp-area="" asp-page="/Index">RazorPagesMovie</a>
```

将前面的元素替换为以下标记：

```cshtml
<a class="navbar-brand" asp-page="/Movies/Index">RpMovie</a>
```

前面的定位点元素是一个[标记帮助程序](xref:mvc/views/tag-helpers/intro)。 此处它是[定位点标记帮助程序](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper)。 `asp-page="/Movies/Index"` 标记帮助程序属性和值可以创建指向 `/Movies/Index` Razor 页面的链接。 `asp-area` 属性值为空，因此在链接中未使用区域。 有关详细信息，请参阅[区域](xref:mvc/controllers/areas)。

保存所做的更改，并通过单击“RpMovie”  链接测试应用。 如果遇到任何问题，请参阅 GitHub 中的 [_Layout.cshtml](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Pages/Shared/_Layout.cshtml) 文件。

测试其他链接（“主页”  、“RpMovie”  、“创建”  、“编辑”  和“删除”  ）。 每个页面都设置有标题，可以在浏览器选项卡中看到标题。将某个页面加入书签时，标题用于该书签。

> [!NOTE]
> 可能无法在 `Price` 字段中输入十进制逗号。 若要使 [jQuery 验证](https://jqueryvalidation.org/)支持使用逗号（“,”）表示小数点的的非英语区域设置，以及支持非美国英语日期格式，必须执行使应用全球化的步骤。 有关添加十进制逗号的说明，请参阅 [GitHub 问题 4076](https://github.com/dotnet/AspNetCore.Docs/issues/4076#issuecomment-326590420)。

在 Pages/_ViewStart.cshtml  文件中设置 `Layout` 属性：

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie30/Pages/_ViewStart.cshtml)]

前面的标记针对所有 Razor 文件将布局文件设置为 Pages  文件夹下的 Pages/Shared/_Layout.cshtml  。 请参阅[布局](xref:razor-pages/index#layout)了解详细信息。

### <a name="the-create-page-model"></a>“创建”页面模型

检查 *Pages/Movies/Create.cshtml.cs* 页面模型：

[!code-csharp[](razor-pages-start/snapshot_sample3/RazorPagesMovie30/Pages/Movies/Create.cshtml.cs?name=snippetALL)]

`OnGet` 方法初始化页面所需的任何状态。 “创建”页没有任何要初始化的状态，因此返回 `Page`。 在本教程的后面部分中，将介绍 `OnGet` 初始化状态的示例。 `Page` 方法创建用于呈现 Create.cshtml  页的 `PageResult` 对象。

`Movie` 属性使用 `[BindProperty]` 特性来选择加入[模型绑定](xref:mvc/models/model-binding)。 当“创建”表单发布表单值时，ASP.NET Core 运行时将发布的值绑定到 `Movie` 模型。

当页面发布表单数据时，运行 `OnPostAsync` 方法：

[!code-csharp[](razor-pages-start/snapshot_sample3/RazorPagesMovie30/Pages/Movies/Create.cshtml.cs?name=snippetPost)]

如果不存在任何模型错误，将重新显示表单，以及发布的任何表单数据。 在发布表单前，可以在客户端捕获到大部分模型错误。 模型错误的一个示例是，发布的日期字段值无法转换为日期。 本教程后面讨论了客户端验证和模型验证。

如果不存在模型错误，将保存数据，并且浏览器会重定向到索引页。

### <a name="the-create-razor-page"></a>创建 Razor 页面

检查 Pages/Movies/Create.cshtml  Razor 页面文件：

[!code-cshtml[](razor-pages-start/snapshot_sample3/RazorPagesMovie30/Pages/Movies/Create.cshtml)]

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Visual Studio 以用于标记帮助程序的特殊加粗字体显示以下标记：

* `<form method="post">`
* `<div asp-validation-summary="ModelOnly" class="text-danger"></div>`
* `<label asp-for="Movie.Title" class="control-label"></label>`
* `<input asp-for="Movie.Title" class="form-control" />`
* `<span asp-validation-for="Movie.Title" class="text-danger"></span>`

![Create.cshtml 页的 VS17 视图](page/_static/th3.png)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

以下标记帮助程序显示在前面的标记中：

* `<form method="post">`
* `<div asp-validation-summary="ModelOnly" class="text-danger"></div>`
* `<label asp-for="Movie.Title" class="control-label"></label>`
* `<input asp-for="Movie.Title" class="form-control" />`
* `<span asp-validation-for="Movie.Title" class="text-danger"></span>`

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

Visual Studio 以用于标记帮助程序的特殊加粗字体显示以下标记：

* `<form method="post">`
* `<div asp-validation-summary="ModelOnly" class="text-danger"></div>`
* `<label asp-for="Movie.Title" class="control-label"></label>`
* `<input asp-for="Movie.Title" class="form-control" />`
* `<span asp-validation-for="Movie.Title" class="text-danger"></span>`

---

`<form method="post">` 元素是一个[表单标记帮助程序](xref:mvc/views/working-with-forms#the-form-tag-helper)。 表单标记帮助程序会自动包含[防伪令牌](xref:security/anti-request-forgery)。

基架引擎在模型中为每个字段（ID 除外）创建 Razor 标记，如下所示：

[!code-cshtml[](~/tutorials/razor-pages/razor-pages-start/snapshot_sample3/RazorPagesMovie30/Pages/Movies/Create.cshtml?range=15-20)]

[验证标记帮助程序](xref:mvc/views/working-with-forms#the-validation-tag-helpers)（`<div asp-validation-summary` 和 `<span asp-validation-for`）显示验证错误。 本系列后面的部分将更详细地讨论有关验证的信息。

[标签标记帮助程序](xref:mvc/views/working-with-forms#the-label-tag-helper) (`<label asp-for="Movie.Title" class="control-label"></label>`) 生成标签描述和 `Title` 属性的 `for` 特性。

[输入标记帮助程序](xref:mvc/views/working-with-forms) (`<input asp-for="Movie.Title" class="form-control">`) 使用 [DataAnnotations](/aspnet/mvc/overview/older-versions/mvc-music-store/mvc-music-store-part-6) 属性并在客户端生成 jQuery 验证所需的 HTML 属性。

有关标记帮助程序（如 `<form method="post">`）的详细信息，请参阅 [ASP.NET Core 中的标记帮助程序](xref:mvc/views/tag-helpers/intro)。

## <a name="additional-resources"></a>其他资源

> [!div class="step-by-step"]
> [上一篇：添加模型](xref:tutorials/razor-pages/model)
> [下一步：数据库](xref:tutorials/razor-pages/sql)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)

本教程介绍在[上一教程](xref:tutorials/razor-pages/model)中通过搭建基架创建的 Razor 页面。

[查看或下载](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22)示例。

## <a name="the-create-delete-details-and-edit-pages"></a>“创建”、“删除”、“详细信息”和“编辑”页面

检查 Pages/Movies/Index.cshtml.cs  页面模型：

[!code-csharp[](razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/Movies/Index.cshtml.cs)]

Razor 页面派生自 `PageModel`。 按照约定，`PageModel` 派生的类称为 `<PageName>Model`。 此构造函数使用[依赖关系注入](xref:fundamentals/dependency-injection)将 `RazorPagesMovieContext` 添加到页。 所有已搭建基架的页面都遵循此模式。 请参阅[异步代码](xref:data/ef-rp/intro#asynchronous-code)，了解有关使用实体框架的异步编程的详细信息。

对页面发出请求时，`OnGetAsync` 方法向 Razor 页面返回影片列表。 在 Razor 页面上调用 `OnGetAsync` 或 `OnGet` 以初始化页面状态。 在这种情况下，`OnGetAsync` 将获得影片列表并显示出来。

当 `OnGet` 返回 `void` 或 `OnGetAsync` 返回 `Task` 时，不使用任何返回方法。 当返回类型是 `IActionResult` 或 `Task<IActionResult>` 时，必须提供返回语句。 例如，Pages/Movies/Create.cshtml.cs `OnPostAsync` 方法  ：

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Create.cshtml.cs?name=snippet)]

检查 Pages/Movies/Index.cshtml<a name="index"></a> Razor 页面  ：

[!code-cshtml[](razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/Movies/Index.cshtml)]

Razor 可以从 HTML 转换为 C# 或 Razor 特定标记。 当 `@` 符号后跟 [Razor 保留关键字](xref:mvc/views/razor#razor-reserved-keywords)时，它会转换为 Razor 特定标记，否则会转换为 C#。

`@page` Razor 指令将文件转换为一个 MVC 操作，这意味着它可以处理请求。 `@page` 必须是页面上的第一个 Razor 指令。 `@page` 是转换到 Razor 特定标记的一个示例。 有关详细信息，请参阅 [Razor 语法](xref:mvc/views/razor#razor-syntax)。

检查以下 HTML 帮助程序中使用的 Lambda 表达式：

```cshtml
@Html.DisplayNameFor(model => model.Movie[0].Title)
```

`DisplayNameFor` HTML 帮助程序检查 Lambda 表达式中引用的 `Title` 属性来确定显示名称。 检查 Lambda 表达式（而非求值）。 这意味着当 `model`、`model.Movie` 或 `model.Movie[0]` 为 `null` 或为空时，不会存在任何访问冲突。 对 Lambda 表达式求值时（例如，使用 `@Html.DisplayFor(modelItem => item.Title)`），将求得该模型的属性值。

<a name="md"></a>

### <a name="the-model-directive"></a>@model 指令

[!code-cshtml[](razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/Movies/Index.cshtml?range=1-2&highlight=2)]

`@model` 指令指定传递给 Razor 页面的模型类型。 在前面的示例中，`@model` 行使 `PageModel` 派生的类可用于 Razor 页面。 在页面上的 `@Html.DisplayNameFor` 和 `@Html.DisplayFor` [HTML 帮助程序](/aspnet/mvc/overview/older-versions-1/views/creating-custom-html-helpers-cs#understanding-html-helpers)中使用该模型。

### <a name="the-layout-page"></a>布局页

选择菜单链接（“RazorPagesMovie”  、“主页”  和“隐私”  ）。 每页显示相同的菜单布局。 菜单布局是在 Pages/Shared/_Layout.cshtml  文件中实现。 打开 Pages/Shared/_Layout.cshtml  文件。

[布局](xref:mvc/views/layout)模板使你能够在一个位置去指定网站的 HTML 容器布局，然后将它应用到网站中的多个页面。 查找 `@RenderBody()` 行。 `RenderBody` 是显示所创建的全部页面专用视图的占位符，已包装  在布局页中。 例如，如果选择“隐私”  链接，Pages/Privacy.cshtml  视图在 `RenderBody` 方法中呈现。

<a name="vd"></a>

### <a name="viewdata-and-layout"></a>ViewData 和布局

考虑来自 Pages/Movies/Index.cshtml  文件中的以下代码：

[!code-cshtml[](razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/Movies/Index.cshtml?range=1-6&highlight=4-999)]

前面突出显示的代码是 Razor 转换为 C# 的一个示例。 `{` 和 `}` 字符括住 C# 代码块。

`PageModel` 基类具有 `ViewData` 字典属性，可用于添加要传递到某个视图的数据。 可以使用键/值模式将对象添加到 `ViewData` 字典。 在前面的示例中，“Title”属性被添加到 `ViewData` 字典。

“Title”属性用于 Pages/Shared/_Layout.cshtml 文件  。 以下标记显示 _Layout.cshtml 文件的前几行  。

<!-- we need a snapshot copy of layout because we are
changing in in the next step.
-->
[!code-cshtml[](razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/NU/_Layout.cshtml?highlight=6-99)]

行 `@*Markup removed for brevity.*@` 是不会出现在布局文件中的 Razor 注释。 与 HTML 注释不同 (`<!-- -->`)，Razor 注释不会发送到客户端。

### <a name="update-the-layout"></a>更新布局

更改 Pages/Shared/_Layout.cshtml 文件中的 `<title>` 元素以显示 Movie 而不是 RazorPagesMovie    。

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie22/Pages/Shared/_Layout.cshtml?range=1-6&highlight=6)]

在 Pages/Shared/_Layout.cshtml  文件中，查找以下定位点元素。

```cshtml
<a class="navbar-brand" asp-area="" asp-page="/Index">RazorPagesMovie</a>
```

将前面的元素替换为以下标记。

```cshtml
<a class="navbar-brand" asp-page="/Movies/Index">RpMovie</a>
```

前面的定位点元素是一个[标记帮助程序](xref:mvc/views/tag-helpers/intro)。 此处它是[定位点标记帮助程序](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper)。 `asp-page="/Movies/Index"` 标记帮助程序属性和值可以创建指向 `/Movies/Index` Razor 页面的链接。 `asp-area` 属性值为空，因此在链接中未使用区域。 有关详细信息，请参阅[区域](xref:mvc/controllers/areas)。

保存所做的更改，并通过单击“RpMovie”  链接测试应用。 如果遇到任何问题，请参阅 GitHub 中的 [_Layout.cshtml](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Pages/Shared/_Layout.cshtml) 文件。

测试其他链接（“主页”  、“RpMovie”  、“创建”  、“编辑”  和“删除”  ）。 每个页面都设置有标题，可以在浏览器选项卡中看到标题。将某个页面加入书签时，标题用于该书签。

> [!NOTE]
> 可能无法在 `Price` 字段中输入十进制逗号。 若要使 [jQuery 验证](https://jqueryvalidation.org/)支持使用逗号（“,”）表示小数点的的非英语区域设置，以及支持非美国英语日期格式，必须执行使应用全球化的步骤。 有关添加十进制逗号的说明，请参阅 [GitHub 问题 4076](https://github.com/dotnet/AspNetCore.Docs/issues/4076#issuecomment-326590420)。

在 Pages/_ViewStart.cshtml  文件中设置 `Layout` 属性：

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie22/Pages/_ViewStart.cshtml)]

前面的标记针对所有 Razor 文件将布局文件设置为 Pages  文件夹下的 Pages/Shared/_Layout.cshtml  。 请参阅[布局](xref:razor-pages/index#layout)了解详细信息。

### <a name="the-create-page-model"></a>“创建”页面模型

检查 *Pages/Movies/Create.cshtml.cs* 页面模型：

[!code-csharp[](razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/Movies/Create.cshtml.cs?name=snippetALL)]

`OnGet` 方法初始化页面所需的任何状态。 “创建”页没有任何要初始化的状态，因此返回 `Page`。 教程的后面部分将介绍 `OnGet` 方法初始化状态。 `Page` 方法创建用于呈现 Create.cshtml  页的 `PageResult` 对象。

`Movie` 属性使用 `[BindProperty]` 特性来选择加入[模型绑定](xref:mvc/models/model-binding)。 当“创建”表单发布表单值时，ASP.NET Core 运行时将发布的值绑定到 `Movie` 模型。

当页面发布表单数据时，运行 `OnPostAsync` 方法：

[!code-csharp[](razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/Movies/Create.cshtml.cs?name=snippetPost)]

如果不存在任何模型错误，将重新显示表单，以及发布的任何表单数据。 在发布表单前，可以在客户端捕获到大部分模型错误。 模型错误的一个示例是，发布的日期字段值无法转换为日期。 本教程后面讨论了客户端验证和模型验证。

如果不存在模型错误，将保存数据，并且浏览器会重定向到索引页。

### <a name="the-create-razor-page"></a>创建 Razor 页面

检查 Pages/Movies/Create.cshtml  Razor 页面文件：

[!code-cshtml[](razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/Movies/Create.cshtml)]

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Visual Studio 以用于标记帮助程序的特殊加粗字体显示 `<form method="post">` 标记：

![Create.cshtml 页的 VS17 视图](page/_static/th.png)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

有关标记帮助程序（如 `<form method="post">`）的详细信息，请参阅 [ASP.NET Core 中的标记帮助程序](xref:mvc/views/tag-helpers/intro)。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

Visual Studio for Mac 以用于标记帮助程序的特殊加粗字体显示 `<form method="post">` 标记。

---

`<form method="post">` 元素是一个[表单标记帮助程序](xref:mvc/views/working-with-forms#the-form-tag-helper)。 表单标记帮助程序会自动包含[防伪令牌](xref:security/anti-request-forgery)。

基架引擎在模型中为每个字段（ID 除外）创建 Razor 标记，如下所示：

[!code-cshtml[](~/tutorials/razor-pages/razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/Movies/Create.cshtml?range=15-20)]

[验证标记帮助程序](xref:mvc/views/working-with-forms#the-validation-tag-helpers)（`<div asp-validation-summary` 和 `<span asp-validation-for`）显示验证错误。 本系列后面的部分将更详细地讨论有关验证的信息。

[标签标记帮助程序](xref:mvc/views/working-with-forms#the-label-tag-helper) (`<label asp-for="Movie.Title" class="control-label"></label>`) 生成标签描述和 `Title` 属性的 `for` 特性。

[输入标记帮助程序](xref:mvc/views/working-with-forms) (`<input asp-for="Movie.Title" class="form-control">`) 使用 [DataAnnotations](/aspnet/mvc/overview/older-versions/mvc-music-store/mvc-music-store-part-6) 属性并在客户端生成 jQuery 验证所需的 HTML 属性。

## <a name="additional-resources"></a>其他资源

* [本教程的 YouTube 版本](https://youtu.be/zxgKjPYnOMM)

> [!div class="step-by-step"]
> [上一篇：添加模型](xref:tutorials/razor-pages/model)
> [下一步：数据库](xref:tutorials/razor-pages/sql)

::: moniker-end
