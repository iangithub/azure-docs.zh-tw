---
title: "Azure 事件中樞範例 | Microsoft Docs"
description: "事件中樞範例"
services: event-hubs
documentationcenter: na
author: jtaubensee
manager: timlt
editor: 
ms.assetid: 
ms.service: event-hubs
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/07/2017
ms.author: jotaub;sethm
translationtype: Human Translation
ms.sourcegitcommit: cfe4957191ad5716f1086a1a332faf6a52406770
ms.openlocfilehash: f3c8f6e52b8713bcdb58d55e8bbc2301a7c316e4
ms.lasthandoff: 03/09/2017

---

# <a name="event-hubs-samples"></a>事件中樞範例 

事件中樞範例示範 [Azure 事件中樞](/azure/event-hubs/)中的主要功能。 本主題分類及描述可用的範例與每個範例的連結。

在撰寫本文時，事件中樞範例位於數個不同的位置：

- [MSDN 開發人員程式碼範例](https://code.msdn.microsoft.com/site/search?query=event%20hubs&f%5B0%5D.Value=event%20hubs&f%5B0%5D.Type=SearchText&ac=5)
- [GitHub](https://github.com/Azure/azure-event-hubs-dotnet/tree/master/samples)

如需不同版本的 .NET Framework 的詳細資訊，請參閱[架構與目標](/dotnet/articles/standard/frameworks)。

更多的範例會隨時新增，所以請經常回到這裡查看更新。

## <a name="net-standard"></a>.NET Standard

下列範例會示範如何使用 [.NET Standard 程式庫](/dotnet/articles/standard/library)的[事件中樞用戶端](https://github.com/Azure/azure-event-hubs-dotnet/blob/master/readme.md)傳送與接收事件。

### <a name="send-events"></a>傳送事件 

[開始傳送](https://github.com/Azure/azure-event-hubs/tree/master/samples/SampleSender)範例會示範如何撰寫將事件傳送到事件中樞的 .NET Core 主控台應用程式。

### <a name="receive-events"></a>接收事件 

[開始使用 Event Processor Host 接收](https://github.com/Azure/azure-event-hubs/tree/master/samples/SampleEphReceiver)範例是一個 .NET Core 主控台應用程式，可使用 `Event Processor Host` 從事件中樞接收訊息。

## <a name="net-framework"></a>.NET Framework    

這些範例會示範 Azure 事件中樞的各種其他功能，以 [.NET Framework 程式庫](https://msdn.microsoft.com/library/w0x726c2.aspx)為目標。
 
### <a name="notify-users-of-events-received"></a>通知使用者收到的事件

[AppToNotifyUsers](https://github.com/Azure-Samples/event-hubs-dotnet-user-notifications) 範例通知使用者從感應器或其他系統接收到的資料。

### <a name="get-started-with-event-hubs"></a>開始使用事件中心 

[開始使用事件中樞](https://code.msdn.microsoft.com/Service-Bus-Event-Hub-286fd097)範例示範了事件中樞的基本功能，例如如何建立事件中樞、傳送事件到事件中樞，以及使用 [Event Processor Host](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost/) 耗用事件。

### <a name="scale-out-event-processing"></a>相應放大事件處理 

[相應放大事件處理](https://code.msdn.microsoft.com/Service-Bus-Event-Hub-45f43fc3)範例示範如何使用 [Event Processor Host](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost/) 來分散事件中樞資料流耗用量的工作負載。 它示範如何實作 **EventProcessor** 和 **EventProcessorFactory** 物件，以管理事件資料流。 

###  <a name="pull-data-from-sql-into-an-event-hub"></a>將資料從 SQL 提取到事件中樞

[提取 SQL 資料](https://github.com/Azure-Samples/event-hubs-dotnet-import-from-sql)範例示範如何從 SQL 資料表提取資料並推送至事件中樞，以在下游分析應用程式中做為輸入使用。

### <a name="pull-web-data-into-an-event-hub"></a>將網路資料提取到事件中樞 

[從網路匯入資料](https://github.com/Azure-Samples/event-hubs-dotnet-importfromweb)範例示範如何從公用摘要提取資料 (例如交通部的交通資訊摘要) 並將資料推送到事件中樞。

## <a name="next-steps"></a>後續步驟

請造訪下列連結深入了解 .NET Framework 版本︰

- [架構與目標](/dotnet/articles/standard/frameworks)
- [.NET Framework 4.6 和 4.5](https://msdn.microsoft.com/library/w0x726c2.aspx)

您可以參閱下列文章，深入了解事件中樞：

- [事件中樞概觀](event-hubs-what-is-event-hubs.md)
- [建立事件中樞](event-hubs-create.md)
- [事件中樞常見問題集](event-hubs-faq.md)
