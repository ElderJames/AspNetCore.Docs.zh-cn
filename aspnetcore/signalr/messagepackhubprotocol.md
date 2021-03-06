---
title: 在 SignalR 中使用 MessagePack 集线器协议 ASP.NET Core
author: bradygaster
description: 将 MessagePack Hub 协议添加到 ASP.NET Core SignalR。
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 11/12/2019
no-loc:
- SignalR
uid: signalr/messagepackhubprotocol
ms.openlocfilehash: cd052a97db1e20d6c7aa00f47cf6a7d01a9bc305
ms.sourcegitcommit: 3fc3020961e1289ee5bf5f3c365ce8304d8ebf19
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/12/2019
ms.locfileid: "73963755"
---
# <a name="use-messagepack-hub-protocol-in-opno-locsignalr-for-aspnet-core"></a>在 SignalR 中使用 MessagePack 集线器协议 ASP.NET Core

作者： [Brennan Conroy](https://github.com/BrennanConroy)

本文假定读者[熟悉入门中](xref:tutorials/signalr)介绍的主题。

## <a name="what-is-messagepack"></a>什么是 MessagePack？

[MessagePack](https://msgpack.org/index.html)是一种快速且紧凑的二进制序列化格式。 当性能和带宽需要考虑时，它很有用，因为它会创建比[JSON](https://www.json.org/)更小的消息。 由于它是二进制格式，因此在查看网络跟踪和日志时会无法读取消息，除非这些字节是通过 MessagePack 分析器传递的。 SignalR 提供对 MessagePack 格式的内置支持，并为客户端和服务器提供要使用的 Api。

## <a name="configure-messagepack-on-the-server"></a>在服务器上配置 MessagePack

若要在服务器上启用 MessagePack 集线器协议，请在应用程序中安装 `Microsoft.AspNetCore.SignalR.Protocols.MessagePack` 包。 在 Startup.cs 文件中，将 `AddMessagePackProtocol` 添加到 `AddSignalR` 调用，以在服务器上启用 MessagePack 支持。

> [!NOTE]
> 默认情况下启用 JSON。 添加 MessagePack 可支持 JSON 和 MessagePack 客户端。

```csharp
services.AddSignalR()
    .AddMessagePackProtocol();
```

若要自定义 MessagePack 如何设置数据的格式，`AddMessagePackProtocol` 获取用于配置选项的委托。 在该委托中，`FormatterResolvers` 属性可用于配置 MessagePack 序列化选项。 有关解析程序工作方式的详细信息，请访问[MessagePack-CSharp](https://github.com/neuecc/MessagePack-CSharp)上的 MessagePack 库。 属性可用于要序列化的对象，以定义应如何处理它们。

```csharp
services.AddSignalR()
    .AddMessagePackProtocol(options =>
    {
        options.FormatterResolvers = new List<MessagePack.IFormatterResolver>()
        {
            MessagePack.Resolvers.StandardResolver.Instance
        };
    });
```

## <a name="configure-messagepack-on-the-client"></a>在客户端上配置 MessagePack

> [!NOTE]
> 默认情况下，为支持的客户端启用 JSON。 客户端只能支持一个协议。 添加 MessagePack 支持将替换任何以前配置的协议。

### <a name="net-client"></a>.NET 客户端

若要在 .NET 客户端中启用 MessagePack，请安装 `Microsoft.AspNetCore.SignalR.Protocols.MessagePack` 包，并 `AddMessagePackProtocol` `HubConnectionBuilder`上调用。

```csharp
var hubConnection = new HubConnectionBuilder()
                        .WithUrl("/chatHub")
                        .AddMessagePackProtocol()
                        .Build();
```

> [!NOTE]
> 此 `AddMessagePackProtocol` 调用采用一个委托来配置与服务器类似的选项。

### <a name="javascript-client"></a>JavaScript 客户端

::: moniker range=">= aspnetcore-3.0"

`@microsoft/signalr-protocol-msgpack` npm 包提供对 JavaScript 客户端的 MessagePack 支持。 通过在命令行界面中执行以下命令来安装包：

```console
npm install @microsoft/signalr-protocol-msgpack
```

::: moniker-end

::: moniker range="< aspnetcore-3.0"

`@aspnet/signalr-protocol-msgpack` npm 包提供对 JavaScript 客户端的 MessagePack 支持。 通过在命令行界面中执行以下命令来安装包：

```console
npm install @aspnet/signalr-protocol-msgpack
```

::: moniker-end

安装 npm 包后，可以通过 JavaScript 模块加载程序直接使用该模块，或通过引用以下文件将该模块导入到浏览器中：

::: moniker range=">= aspnetcore-3.0"

*node_modules\\@microsoft\signalr-protocol-msgpack\dist\browser\signalr-protocol-msgpack.js* 

::: moniker-end

::: moniker range="< aspnetcore-3.0"

*node_modules\\@aspnet\signalr-protocol-msgpack\dist\browser\signalr-protocol-msgpack.js* 

::: moniker-end

在浏览器中，还必须引用 `msgpack5` 库。 使用 `<script>` 标记创建一个引用。 可在*node_modules \msgpack5\dist\msgpack5.js*找到库。

> [!NOTE]
> 使用 `<script>` 元素时，顺序很重要。 如果在*msgpack5*之前引用*signalr-protocol-msgpack* ，则在尝试与 MessagePack 连接时将出现错误。 *signalr*在*signalr-protocol-msgpack*之前也是必需的。

```html
<script src="~/lib/signalr/signalr.js"></script>
<script src="~/lib/msgpack5/msgpack5.js"></script>
<script src="~/lib/signalr/signalr-protocol-msgpack.js"></script>
```

向 `HubConnectionBuilder` 中添加 `.withHubProtocol(new signalR.protocols.msgpack.MessagePackHubProtocol())` 会将客户端配置为在连接到服务器时使用 MessagePack 协议。

```javascript
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/chatHub")
    .withHubProtocol(new signalR.protocols.msgpack.MessagePackHubProtocol())
    .build();
```

> [!NOTE]
> 目前，JavaScript 客户端上没有用于 MessagePack 协议的配置选项。

## <a name="messagepack-quirks"></a>MessagePack 兼容

使用 MessagePack 集线器协议时，需要注意几个问题。

### <a name="messagepack-is-case-sensitive"></a>MessagePack 区分大小写

MessagePack 协议区分大小写。 例如，请看下面C#的类：

```csharp
public class ChatMessage
{
    public string Sender { get; }
    public string Message { get; }
}
```

从 JavaScript 客户端发送时，必须使用 `PascalCased` 属性名称，因为大小写必须与C#类完全匹配。 例如:

```javascript
connection.invoke("SomeMethod", { Sender: "Sally", Message: "Hello!" });
```

使用 `camelCased` 名称不会正确绑定到C#类。 可以通过使用 `Key` 特性为 MessagePack 属性指定一个不同的名称来解决此情况。 有关详细信息，请参阅[MessagePack-CSharp 文档](https://github.com/neuecc/MessagePack-CSharp#object-serialization)。

### <a name="datetimekind-is-not-preserved-when-serializingdeserializing"></a>序列化/反序列化时不保留 DateTime. Kind

MessagePack 协议不提供对 `DateTime`的 `Kind` 值进行编码的方法。 因此，在对日期进行反序列化时，MessagePack Hub 协议假设传入日期为 UTC 格式。 如果你在本地时间使用 `DateTime` 值，则建议在发送这些值之前将其转换为 UTC。 接收到本地时间时将它们从 UTC 转换为本地时间。

有关此限制的详细信息，请参阅 GitHub issue [aspnet/SignalR#2632](https://github.com/aspnet/SignalR/issues/2632)。

### <a name="datetimeminvalue-is-not-supported-by-messagepack-in-javascript"></a>JavaScript 中的 MessagePack 不支持 MinValue

SignalR JavaScript 客户端使用的[msgpack5](https://github.com/mcollina/msgpack5)库不支持 MessagePack 中的 `timestamp96` 类型。 此类型用于编码非常大的日期值（在过去或未来很大程度上）。 `DateTime.MinValue` 的值 `January 1, 0001` 必须用 `timestamp96` 值进行编码。 因此，不支持向 JavaScript 客户端发送 `DateTime.MinValue`。 当 JavaScript 客户端接收到 `DateTime.MinValue` 时，将引发以下错误：

```
Uncaught Error: unable to find ext type 255 at decoder.js:427
```

通常，`DateTime.MinValue` 用于对 "缺少" 或 `null` 值进行编码。 如果需要在 MessagePack 中对该值进行编码，请使用可以为 null 的 `DateTime` 值（`DateTime?`），或对表示日期是否存在的单独 `bool` 值进行编码。

有关此限制的详细信息，请参阅 GitHub issue [aspnet/SignalR#2228](https://github.com/aspnet/SignalR/issues/2228)。

### <a name="messagepack-support-in-ahead-of-time-compilation-environment"></a>"提前" 编译环境中的 MessagePack 支持

.NET 客户端和服务器使用的[MessagePack-CSharp](https://github.com/neuecc/MessagePack-CSharp)库使用代码生成来优化序列化。 因此，默认情况下，在使用 "提前" 编译（如 Xamarin iOS 或 Unity）的环境中，默认情况下不支持此方法。 可以通过 "预生成" 序列化程序/反序列化程序代码，在这些环境中使用 MessagePack。 有关详细信息，请参阅[MessagePack-CSharp 文档](https://github.com/neuecc/MessagePack-CSharp#pre-code-generationunityxamarin-supports)。 预生成序列化程序后，可以使用传递给 `AddMessagePackProtocol`的配置委托注册它们：

```csharp
services.AddSignalR()
    .AddMessagePackProtocol(options =>
    {
        options.FormatterResolvers = new List<MessagePack.IFormatterResolver>()
        {
            MessagePack.Resolvers.GeneratedResolver.Instance,
            MessagePack.Resolvers.StandardResolver.Instance
        };
    });
```

### <a name="type-checks-are-more-strict-in-messagepack"></a>类型检查在 MessagePack 中更加严格

JSON 集线器协议将在反序列化过程中执行类型转换。 例如，如果传入的对象的属性值为数字（`{ foo: 42 }`），但 .NET 类的属性为类型 `string`，则将转换该值。 但是，MessagePack 不会执行此转换，并将引发可在服务器端日志中显示的异常（在控制台中）：

```
InvalidDataException: Error binding arguments. Make sure that the types of the provided values match the types of the hub method being invoked.
```

有关此限制的详细信息，请参阅 GitHub issue [aspnet/SignalR#2937](https://github.com/aspnet/SignalR/issues/2937)。

## <a name="related-resources"></a>相关资源

* [入门](xref:tutorials/signalr)
* [.NET 客户端](xref:signalr/dotnet-client)
* [JavaScript 客户端](xref:signalr/javascript-client)
