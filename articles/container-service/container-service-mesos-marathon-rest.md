---
title: "使用 Marathon REST API 管理 Azure DC/OS 叢集 | Microsoft Docs"
description: "使用 Marathon REST API 將容器部署到 Azure Container Service DC/OS 叢集。"
services: container-service
documentationcenter: 
author: dlepow
manager: timlt
editor: 
tags: acs, azure-container-service
keywords: "Docker、容器、微服務、Mesos、Azure"
ms.assetid: c7175446-4507-4a33-a7a2-63583e5996e3
ms.service: container-service
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/01/2017
ms.author: danlep
translationtype: Human Translation
ms.sourcegitcommit: b2b969500d20d0c840f201ed2cf13a6f2ab38ee5
ms.openlocfilehash: 719f1ea6a6f51d4a787f0465a4bbadb1a6057a8b
ms.lasthandoff: 02/02/2017


---
# <a name="dcos-container-management-through-the-marathon-rest-api"></a>透過 Marathon REST API 的 DC/OS 容器管理
DC/OS 提供環境來部署及調整叢集工作負載，同時將基礎硬體抽象化。 在 DC/OS 之上有架構會管理排程和執行計算工作負載。 雖然許多常見的工作負載都有可用的架構，但這份文件只說明如何使用 Marathon 來建立及調整容器部署。 

## <a name="prerequisites"></a>必要條件

在練習這些範例之前，您需要 Azure 容器服務中設定的 DC/OS 叢集。 您也需要有此叢集的遠端連線。 如需這些項目的詳細資訊，請參閱下列文章。

* [部署 Azure 容器服務叢集](container-service-deployment.md)
* [連接到 Azure 容器服務叢集](container-service-connect.md)

## <a name="access-the-dcos-apis"></a>存取 DC/OS API
連接到 Azure 容器服務叢集之後，您可以透過 http://localhost:local-port 存取 DC/OS 和相關的 REST API。 本文件中的範例假設您的通道為連接埠 80。 例如，在開頭是 `http://localhost/marathon/v2/` 的 URL 可以觸達 Marathon 端點。 

如需各種 API 的詳細資訊，請參閱 [Marathon API](https://mesosphere.github.io/marathon/docs/rest-api.html) 和 [Chronos API](https://mesos.github.io/chronos/docs/api.html) 的 Mesosphere 文件，以及 [Mesos 排程器 API](http://mesos.apache.org/documentation/latest/scheduler-http-api/) 的 Apache 文件。

## <a name="gather-information-from-dcos-and-marathon"></a>從 DC/OS 和 Marathon 收集資訊
將容器部署至 DC/OS 叢集之前，請收集 DC/OS 叢集的一些相關資訊，例如 DC/OS 代理程式的名稱和目前狀態。 若要這樣做，請查詢 DC/OS REST API 的 `master/slaves` 端點。 如果一切順利，查詢會傳回 DC/OS 代理程式清單及每個代理程式的數個屬性。

```bash
curl http://localhost/mesos/master/slaves
```

現在，使用 Marathon `/apps` 端點來檢查目前部署至 DC/OS 叢集的應用程式。 如果這是新的叢集，您會看到空的應用程式陣列。

```bash
curl localhost/marathon/v2/apps

{"apps":[]}
```

## <a name="deploy-a-docker-formatted-container"></a>部署 Docker 格式化容器
您可以使用描述預期部署的 JSON 檔案透過 Marathon 部署 Docker 格式化容器。 下列範例將會部署 Nginx 容器，並將 DC/OS 代理程式的連接埠 80 繫結至容器的連接埠 80。 也請注意 `acceptedResourceRoles` 屬性是設為 `slave_public`。 這會將容器部署到對外公開的代理程式擴展集內的代理程式。

```json
{
  "id": "nginx",
  "cpus": 0.1,
  "mem": 16.0,
  "instances": 1,
    "acceptedResourceRoles": [
    "slave_public"
  ],
  "container": {
    "type": "DOCKER",
    "docker": {
      "image": "nginx",
      "network": "BRIDGE",
      "portMappings": [
        { "containerPort": 80, "hostPort": 80, "servicePort": 9000, "protocol": "tcp" }
      ]
    }
  }
}
```

若要部署 Docker 格式的容器，請將 JSON 檔案儲存在可存取的位置。 接下來，若要部署容器，請執行下列命令。 指定 JSON 檔案的名稱 (在此範例中是 `marathon.json`)。

```bash
curl -X POST http://localhost/marathon/v2/apps -d @marathon.json -H "Content-type: application/json"
```

輸出大致如下：

```json
{"version":"2015-11-20T18:59:00.494Z","deploymentId":"b12f8a73-f56a-4eb1-9375-4ac026d6cdec"}
```

現在，如果您查詢 Marathon 中的應用程式，這個新的應用程式會顯示在輸出中。

```bash
curl localhost/marathon/v2/apps
```

若要確認 Nginx 正在執行，您可以對代理程式集區的完整網域名稱 (`http://<containerServiceName>agents.<region>.cloudapp.azure.com`) 發出 HTTP 要求。

## <a name="scale-your-containers"></a>調整容器的大小
您也可以使用 Marathon API 來相應放大或相應縮小應用程式部署。 在前面的範例中，您已部署一個應用程式執行個體。 讓我們將其相應放大為三個應用程式執行個體。 若要這樣做，請使用下列 JSON 文字建立 JSON 檔案，並將它儲存在可存取的位置。

```json
{ "instances": 3 }
```

執行下列命令來相應放大應用程式。

> [!NOTE]
> URI 是 http://localhost/marathon/v2/apps/ 後接要調整的應用程式的識別碼。 如果您是使用這裡提供的 Nginx 範例，則 URI 會是 http://localhost/marathon/v2/apps/nginx。
> 
> 

```bash
curl http://localhost/marathon/v2/apps/nginx -H "Content-type: application/json" -X PUT -d @scale.json
```

最後，向 Marathon 端點查詢應用程式。 您會看到現在有三個 Nginx 容器。

```bash
curl localhost/marathon/v2/apps
```

## <a name="equivalent-powershell-commands"></a>對等的 PowerShell 命令
您可以在 Windows 系統上使用 PowerShell 命令來執行這些相同的動作。

若要收集 DC/OS 叢集的相關資訊 (例如代理程式名稱和代理程式狀態)，請執行下列命令：

```powershell
Invoke-WebRequest -Uri http://localhost/mesos/master/slaves
```

您可以使用描述預期部署的 JSON 檔案透過 Marathon 部署 Docker 格式化容器。 下列範例將會部署 Nginx 容器，並將 DC/OS 代理程式的連接埠 80 繫結至容器的連接埠 80。

```json
{
  "id": "nginx",
  "cpus": 0.1,
  "mem": 16.0,
  "instances": 1,
  "container": {
    "type": "DOCKER",
    "docker": {
      "image": "nginx",
      "network": "BRIDGE",
      "portMappings": [
        { "containerPort": 80, "hostPort": 80, "servicePort": 9000, "protocol": "tcp" }
      ]
    }
  }
}
```

若要部署 Docker 格式的容器，請將 JSON 檔案儲存在可存取的位置。 接下來，若要部署容器，請執行下列命令。 指定 JSON 檔案的名稱 (在此範例中是 `marathon.json`)。

```powershell
Invoke-WebRequest -Method Post -Uri http://localhost/marathon/v2/apps -ContentType application/json -InFile 'c:\marathon.json'
```

您也可以使用 Marathon API 來相應放大或相應縮小應用程式部署。 在前面的範例中，您已部署一個應用程式執行個體。 讓我們將其相應放大為三個應用程式執行個體。 若要這樣做，請使用下列 JSON 文字建立 JSON 檔案，並將它儲存在可存取的位置。

```json
{ "instances": 3 }
```

執行下列命令來相應放大應用程式：

> [!NOTE]
> URI 是 http://localhost/marathon/v2/apps/ 後接要調整的應用程式的識別碼。 如果您是使用這裡提供的 Nginx 範例，則 URI 會是 http://localhost/marathon/v2/apps/nginx。
> 
> 

```powershell
Invoke-WebRequest -Method Put -Uri http://localhost/marathon/v2/apps/nginx -ContentType application/json -InFile 'c:\scale.json'
```

## <a name="next-steps"></a>後續步驟
* [深入了解 Mesos HTTP 端點](http://mesos.apache.org/documentation/latest/endpoints/)
* [深入了解 Marathon REST API](https://mesosphere.github.io/marathon/docs/rest-api.html)


