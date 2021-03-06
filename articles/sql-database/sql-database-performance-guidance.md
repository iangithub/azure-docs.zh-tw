---
title: "單一資料庫的 Azure SQL Database 效能 | Microsoft Docs"
description: "本文可協助您判斷要為您的應用程式選擇哪個服務層。 文中也會建議微調應用程式以充分利用 Azure SQL Database 的方式。"
services: sql-database
documentationcenter: na
author: CarlRabeler
manager: jhubbard
editor: 
ms.assetid: dd8d95fa-24b2-4233-b3f1-8e8952a7a22b
ms.service: sql-database
ms.custom: monitor and tune
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: data-management
ms.date: 03/06/2017
ms.author: carlrab
translationtype: Human Translation
ms.sourcegitcommit: 07635b0eb4650f0c30898ea1600697dacb33477c
ms.openlocfilehash: 03d38dbce86711395a967cf8bad440fd50a38631
ms.lasthandoff: 03/28/2017


---
# <a name="azure-sql-database-and-performance-for-single-databases"></a>Azure SQL Database 和單一資料庫效能
Azure SQL Database 提供四個[服務層](sql-database-service-tiers.md)：基本、標準、進階和進階 RS。 每個服務層會嚴格地隔離出 SQL Database 可以使用的資源，並保證該服務層級會有可預測的效能。 在本文中，我們會提供指引來協助您選擇應用程式的服務層。 我們也會討論您可以微調應用程式以充分利用 Azure SQL Database 的方式。

> [!NOTE]
> 本文著重在 Azure SQL Database 中單一資料庫的效能指引。 如需彈性集區的相關效能指引，請參閱[彈性集區的價格和效能考量](sql-database-elastic-pool-guidance.md)。 但請注意，您可以將本文的諸多微調建議套用到彈性集區中的資料庫，並獲得類似的效能優點。
>
>

## <a name="why-service-tiers"></a>為何使用服務層？
雖然每個資料庫的工作負載可能不同，但服務層的目的是要在各種效能等級提供效能可預測性。 對資料庫有大規模資源需求的客戶，則可以在更專用的運算環境中工作。

### <a name="common-service-tier-use-cases"></a>常見的服務層使用案例
#### <a name="basic"></a>基本
* **您剛開始使用 Azure SQL Database**。 開發中的應用程式通常不需要很高的效能等級。 基本資料庫的價格便宜，是適合用來開發資料庫的理想環境。
* **您的資料庫只有單一使用者**。 將單一使用者與資料庫相關聯的應用程式通常沒有高度的並行存取和效能需求。 這些應用程式適合使用基本服務層。

#### <a name="standard"></a>標準
* **您的資料庫有多個並行要求**。 一次服務多名使用者的應用程式通常需要較高的效能等級。 例如，中等流量的網站或需要更多資源的部門應用程式，都很適合使用標準服務層。

#### <a name="premium"></a>進階
大多數進階服務層使用案例具有下列一或多項特性︰

* **高尖峰負載**。 需要大量 CPU、記憶體或輸入/輸出 (I/O) 以完成其作業的應用程式，需要的是專用、高效能的等級。 例如，已知會長時間取用數個 CPU 核心的資料庫作業，就適合使用進階服務層。
* **許多並行要求**。 某些資料庫應用程式會為許多並行要求提供服務，例如，在為具有高流量的網站提供服務時。 基本和標準服務層會限制每個資料庫的並行要求數目。 需要更多連線的應用程式必須選擇適當的預留大小才能處理最大數目的所需要求。
* **低延遲**。 某些應用程式必須保證在最短的時間內傳回資料庫回應。 如果呼叫特定的預存程序做為更廣泛客戶作業的一部分，您可能需要該呼叫在 20 毫秒 (也就是 99% 的時間) 內傳回回應。 這類應用程式會受益於高階資料庫，以確定所需的運算能力可供使用。

* **進階 RS**。 專為需要大量 IO 的工作負載，但不需要高可用性保證的客戶所設計。 範例包括測試高效能工作負載，或資料庫不是記錄系統的分析工作負載。

您需要的 SQL Database 服務等級取決於每個資源維度的尖峰負載需求。 有些應用程式雖使用少量的某一資源，但卻對其他資源有大量需求。

## <a name="service-tier-capabilities-and-limits"></a>服務層的功能和限制
每個服務層和效能層級都關聯到不同的限制和效能特性。 下表描述單一資料庫的這些特性。

[!INCLUDE [SQL DB service tiers table](../../includes/sql-database-service-tiers-table.md)]

> [!IMPORTANT]
> 使用 P11 和 P15 效能等級的客戶不需額外付費就能使用最多 4 TB 的內含儲存體。 這個 4 TB 選項目前在下列區域為公開預覽狀態：美國東部 2、美國西部、西歐、東南亞、日本東部、澳大利亞東部、加拿大中部和加拿大東部。
>

### <a name="maximum-in-memory-oltp-storage"></a>記憶體內部 OLTP 儲存體上限
您可以使用 **sys.dm_db_resource_stats** 檢視來監視 Azure 記憶體內部儲存體的使用情形。 如需有關監視的詳細資訊，請參閱 [監視記憶體內部 OLTP 儲存體](sql-database-in-memory-oltp-monitoring.md)。

### <a name="maximum-concurrent-requests"></a>並行要求數上限
若要查看並行要求數目，請在 SQL Database 上執行下列 Transact-SQL 查詢：

    SELECT COUNT(*) AS [Concurrent_Requests]
    FROM sys.dm_exec_requests R

若要分析內部部署 SQL Server 資料庫的工作負載，請修改此查詢來篩選您要分析的特定資料庫。 比方說，如果您擁有名為 MyDatabase 的內部部署資料庫，則下列 Transact-SQL 查詢會傳回該資料庫中並行要求的計數：

    SELECT COUNT(*) AS [Concurrent_Requests]
    FROM sys.dm_exec_requests R
    INNER JOIN sys.databases D ON D.database_id = R.database_id
    AND D.name = 'MyDatabase'

這只是單一時間點的快照。 若要進一步了解您的工作負載和並行要求需求，您必須收集一段時間內的許多範例。

### <a name="maximum-concurrent-logins"></a>並行登入數上限
您可以分析您的使用者和應用程式模式以了解登入頻率。 您也可以在測試環境中執行真實世界的負載，藉此確定您不會達到我們在本文中討論的這項限制或其他限制。 沒有任何單一查詢或動態管理檢視 (DMV) 可以向您顯示並行登入計數或歷程記錄。

如果這些用戶端使用相同的連接字串，服務仍會驗證每一個登入。 如果有 10 位使用者同時以相同的使用者名稱和密碼連接到資料庫，將會有 10 個並行登入。 這項限制只適用於登入和驗證期間。 如果相同的 10 位使用者依序連接到資料庫，並行登入數目絕對不會大於 1。

> [!NOTE]
> 這項限制目前不適用於彈性集區中的資料庫。
>
>

### <a name="maximum-sessions"></a>工作階段數上限
若要查看目前的作用中工作階段數目，請在 SQL Database 上執行下列 Transact-SQL 查詢：

    SELECT COUNT(*) AS [Sessions]
    FROM sys.dm_exec_connections

如果您要分析內部部署 SQL Server 的工作負載，請修改查詢以專注於特定資料庫。 如果您考慮將資料庫移至 Azure SQL Database，此查詢可協助您判斷該資料庫可能的工作階段需求。

    SELECT COUNT(*)  AS [Sessions]
    FROM sys.dm_exec_connections C
    INNER JOIN sys.dm_exec_sessions S ON (S.session_id = C.session_id)
    INNER JOIN sys.databases D ON (D.database_id = S.database_id)
    WHERE D.name = 'MyDatabase'

同樣地，這些查詢傳回的是某一時間點的計數。 如果您收集一段時間內的多個範例，您就可以充分了解您的工作階段使用量。

若要進行 SQL Database 分析，您可以查詢 [sys.resource_stats](https://msdn.microsoft.com/library/dn269979.aspx)檢視，並檢閱 **active_session_count** 資料行，以取得工作階段的歷史統計資料。

## <a name="monitor-resource-use"></a>監視資源使用量

您可以使用 [SQL Database 查詢效能深入解析](sql-database-query-performance.md)和[查詢存放區](https://msdn.microsoft.com/library/dn817826.aspx)來監視資源用量。

您也可以使用以下兩種檢視來監視用量：

* [sys.dm_db_resource_stats](https://msdn.microsoft.com/library/dn800981.aspx)
* [sys.resource_stats](https://msdn.microsoft.com/library/dn269979.aspx)

### <a name="sysdmdbresourcestats"></a>sys.dm_db_resource_stats
您可以在每一個 SQL Database 中使用 [sys.dm_db_resource_stats](https://msdn.microsoft.com/library/dn800981.aspx)檢視。 **sys.dm_db_resource_stats** 檢視可顯示相對於服務層的最新資源使用量資料。 每隔 15 秒鐘就會記錄一次 CPU、資料 I/O、記錄檔寫入和記憶體的平均百分比，並且會維持 1 小時。

因為此檢視會提供更細微的資源使用量資訊，請先使用 **sys.dm_db_resource_stats** 來進行任何現狀分析或疑難排解。 例如，下列查詢會顯示目前的資料庫在過去一小時的平均和最大資源使用量：

    SELECT  
        AVG(avg_cpu_percent) AS 'Average CPU use in percent',
        MAX(avg_cpu_percent) AS 'Maximum CPU use in percent',
        AVG(avg_data_io_percent) AS 'Average data I/O in percent',
        MAX(avg_data_io_percent) AS 'Maximum data I/O in percent',
        AVG(avg_log_write_percent) AS 'Average log write use in percent',
        MAX(avg_log_write_percent) AS 'Maximum log write use in percent',
        AVG(avg_memory_usage_percent) AS 'Average memory use in percent',
        MAX(avg_memory_usage_percent) AS 'Maximum memory use in percent'
    FROM sys.dm_db_resource_stats;  

如需其他查詢的資訊，請參閱 [sys.dm_db_resource_stats](https://msdn.microsoft.com/library/dn800981.aspx) 中的範例。

### <a name="sysresourcestats"></a>sys.resource_stats
**master** 資料庫中的 [sys.resource_stats](https://msdn.microsoft.com/library/dn269979.aspx) 檢視有其他資訊可協助您監視 SQL Database 在其特定服務層和效能等級的效能。 這項資料每隔 5 分鐘就會收集一次，並且會維持大約 14 天。 這個檢視適合用於進行 SQL Database 如何使用資源的長期歷史分析。

下圖顯示在一週中 P2 效能等級之高階資料庫每小時的 CPU 資源使用量。 此圖從星期一開始，顯示 5 個工作天，然後顯示較少發生在應用程式的週末。

![SQL Database 的資源使用量](./media/sql-database-performance-guidance/sql_db_resource_utilization.png)

從資料中，這個資料庫目前相對於 P2 效能等級的尖峰 CPU 負載剛好超過 50% 的 CPU 使用量 (星期二中午)。 如果 CPU 是應用程式的資源設定檔中的主要因素，您可能會決定 P2 是正確的效能等級，以確保永遠符合工作負載。 如果您預期應用程式會隨著時間成長，最好預留額外的資源緩衝，讓應用程式永遠不會達到效能等級限制。 如果您提升效能等級，則可協助避免資料庫沒有足夠能力來有效處理要求時可能發生的客戶可見錯誤，尤其是在延遲敏感的環境中。 其中一例便是支援可根據資料庫呼叫結果繪製網頁之應用程式的資料庫。

其他應用程式類型可能會以不同方式解譯相同的圖形。 例如，如果應用程式嘗試每天處理薪資資料而且具有同一張圖表，這種「批次作業」模型可能會在 P1 效能等級中正常執行。 相較於 P2 效能等級的 200 個 DTU，P1 效能等級有 100 個 DTU。 P1 效能等級提供的效能是 P2 效能等級的一半。 因此，P2 中 50% 的 CPU 使用量等於 P1 中 100% 的 CPU 使用量。 如果應用程式沒有逾時，就算作業花了 2 個小時或 2.5 個小時才能完成也沒關係，只要它在今天內完成即可。 這個類別中的應用程式或許可以使用 P1 效能等級。 一天中有好幾個時段的資源使用量較低，您可以善用這個事實，讓任何「巨量尖峰」可以溢出到當天稍後的一個低谷。 只要作業可以每天及時完成，P1 效能等級可能就很適合這類應用程式 (並節省經費)。

Azure SQL Database 會在每個伺服器 **master** 資料庫的 **sys.resource_stats** 檢視中公開每個作用中資料庫的耗用資源資訊。 資料表中的資料會以 5 分鐘的間隔彙總。 利用基本、標準和進階服務層，資料可能要花 5 分鐘以上的時間才會出現在資料表中，因此這項資料比較適合進行歷程記錄分析而不是近乎即時的分析。 查詢 **sys.resource_stats** 檢視可查看資料庫的近期歷程記錄，並驗證所選的保留是否會在必要時提供所需的效能。

> [!NOTE]
> 您必須連接到邏輯 SQL Database 伺服器的 **master** 資料庫才能在下列範例中查詢 **sys.resource_stats**。
>
>

下列範例會示範如何公開此檢視中的資料：

    SELECT TOP 10 *
    FROM sys.resource_stats
    WHERE database_name = 'resource1'
    ORDER BY start_time DESC

![sys.resource_stats 目錄檢視](./media/sql-database-performance-guidance/sys_resource_stats.png)

下列範例示範不同方式，以供您用來使用 **sys.resource_stats** 目錄檢視取得有關 SQL Database 如何使用資源的相關資訊：

1. 若要查看 userdb1 資料庫在過去一週的資源使用量，您可以執行下列查詢：

        SELECT *
        FROM sys.resource_stats
        WHERE database_name = 'userdb1' AND
              start_time > DATEADD(day, -7, GETDATE())
        ORDER BY start_time DESC;
2. 若要評估您的工作負載與效能等級的符合程度，您必須鑽研資源度量的每個層面：CPU、讀取、寫入、背景工作角色數目和工作階段數目。 以下是使用 **sys.resource_stats** 修訂過的查詢，可報告這些資源度量的平均值和最大值：

        SELECT
            avg(avg_cpu_percent) AS 'Average CPU use in percent',
            max(avg_cpu_percent) AS 'Maximum CPU use in percent',
            avg(avg_data_io_percent) AS 'Average physical data I/O use in percent',
            max(avg_data_io_percent) AS 'Maximum physical data I/O use in percent',
            avg(avg_log_write_percent) AS 'Average log write use in percent',
            max(avg_log_write_percent) AS 'Maximum log write use in percent',
            avg(max_session_percent) AS 'Average % of sessions',
            max(max_session_percent) AS 'Maximum % of sessions',
            avg(max_worker_percent) AS 'Average % of workers',
            max(max_worker_percent) AS 'Maximum % of workers'
        FROM sys.resource_stats
        WHERE database_name = 'userdb1' AND start_time > DATEADD(day, -7, GETDATE());
3. 利用這項有關各資源度量平均值和最大值的資訊，您可以評估您的工作負載與您所選之效能等級的符合程度。 通常，來自 **sys.resource_stats** 的平均值可提供您對目標大小所使用的理想基準。 它應該是您主要的量尺。 例如，您可能使用標準服務層搭配 S2 效能等級。 CPU 以及 I/O 讀取和寫入的平均使用量百分比低於 40%，背景工作角色平均數目低於 50，而且工作階段平均數目低於 200。 您的工作負載可能符合 S1 效能等級。 要看到您的資料庫是否符合背景工作和工作階段限制範圍內非常容易。 若要查看資料庫在 CPU、讀取和寫入方面是否符合較低的效能等級，請將較低效能等級的 DTU 數目除以目前效能等級的 DTU 數目，然後將結果乘以 100：

    **S1 DTU / S2 DTU * 100 = 20 / 50 * 100 = 40**

    此結果是以百分比表示之兩個效能等級的相對效能差異。 如果您的資源使用量未超過這個數量，您的工作負載可能符合較低的效能等級。 不過，您需要查看所有範圍的資源使用量值，並以百分比判斷資料庫工作負載符合較低效能等級的頻率。 下列查詢會根據我們在此範例中計算的 40% 臨界值，輸出每個資源維度的相符百分比：

        SELECT
            (COUNT(database_name) - SUM(CASE WHEN avg_cpu_percent >= 40 THEN 1 ELSE 0 END) * 1.0) / COUNT(database_name) AS 'CPU Fit Percent'
            ,(COUNT(database_name) - SUM(CASE WHEN avg_log_write_percent >= 40 THEN 1 ELSE 0 END) * 1.0) / COUNT(database_name) AS 'Log Write Fit Percent'
            ,(COUNT(database_name) - SUM(CASE WHEN avg_data_io_percent >= 40 THEN 1 ELSE 0 END) * 1.0) / COUNT(database_name) AS 'Physical Data IO Fit Percent'
        FROM sys.resource_stats
        WHERE database_name = 'userdb1' AND start_time > DATEADD(day, -7, GETDATE());

    根據您的資料庫服務等級目標 (SLO)，您可以決定您的工作負載是否符合較低的效能等級。 如果您的資料庫工作負載 SLO 是 99.9%，且上述查詢針對三個資源維度傳回的值都大於 99.9，您的工作負載可能會符合較低的效能等級。

    查看相符百分比也可讓您深入了解是否必須移到下一個較高的效能等級來滿足您的 SLO。 例如，userdb1 會顯示過去一週的下列 CPU 使用量：

   | 平均 CPU 百分比 | 最大 CPU 百分比 |
   | --- | --- |
   | 24.5 |100.00 |

    平均 CPU 大約是效能等級限制的四分之一，完全符合資料庫的效能等級。 不過，此最大值會顯示資料庫達到效能等級的限制。 您需要移至下一個較高的效能等級嗎？ 請查看工作負載達到 100% 的次數，然後將其與您的資料庫工作負載 SLO 做比較。

        SELECT
        (COUNT(database_name) - SUM(CASE WHEN avg_cpu_percent >= 100 THEN 1 ELSE 0 END) * 1.0) / COUNT(database_name) AS 'CPU fit percent'
        ,(COUNT(database_name) - SUM(CASE WHEN avg_log_write_percent >= 100 THEN 1 ELSE 0 END) * 1.0) / COUNT(database_name) AS 'Log write fit percent'
        ,(COUNT(database_name) - SUM(CASE WHEN avg_data_io_percent >= 100 THEN 1 ELSE 0 END) * 1.0) / COUNT(database_name) AS 'Physical data I/O fit percent'
        FROM sys.resource_stats
        WHERE database_name = 'userdb1' AND start_time > DATEADD(day, -7, GETDATE());

    如果此查詢針對三個資源維度傳回的值小於 99.9%，請考慮移到下一個較高的效能等級或使用應用程式微調技術減少 SQL Database 的負載。
4. 此練習也會考慮您預計的未來工作負載增加量。

## <a name="tune-your-application"></a>微調應用程式
在傳統的內部部署 SQL Server 中，初始容量規劃的程序通常會和在生產環境中執行應用程式的程序分開。 先購買硬體和產品授權，之後再微調效能。 當您使用 Azure SQL Database 時，最好是將執行和微調應用程式的程序交織在一起。 透過容量隨選的付費模型，您可以微調應用程式以使用目前需要的最少資源，而不是根據對應用程式未來成長計劃的猜測 (這通常是不正確的) 來過度佈建。 某些客戶可能會選擇不微調應用程式，而改為選擇過度佈建硬體資源。 若您不想在忙碌時期變更關鍵應用程式，這種方法可能很適合。 但是，當您使用 Azure SQL Database 中的服務層時，微調應用程式可以最小化資源需求並降低每月帳單。

### <a name="application-characteristics"></a>應用程式特性
雖然 Azure SQL Database 服務層的設計可以改善應用程式的效能穩定性和可預測性，但某些最佳做法可以協助您微調應用程式，以更充分利用某一效能等級的資源。 雖然許多應用程式只藉由切換至較高的效能等級或服務層就能有顯著的效能提升，但是某些應用程式需要額外的微調才能從較高的服務等級獲益。 為了提高效能，請考慮為具有下列特性的應用程式進行額外的應用程式微調︰

* **因為「多對話」行為而使效能變慢的應用程式**。 多對話應用程式會產生過多對網路延遲敏感的資料存取作業。 您可能需要修改這類應用程式以減少 SQL Database 的資料存取作業數目。 比方說，您可以藉由使用某些技術來改善應用程式效能，例如批次處理特定查詢或將查詢移至預存程序。 如需詳細資訊，請參閱 [批次查詢](#batch-queries)。
* **無法由整部單一電腦支援之具有大量工作負載的資料庫**。 超過最高進階效能等級資源的資料庫可能會受益於相應放大工作負載。 如需詳細資訊，請參閱[跨資料庫分區化](#cross-database-sharding)和[功能資料分割](#functional-partitioning)。
* **具有次佳查詢的應用程式**。 其查詢未經適當微調的應用程式，尤其是位在資料存取層中者，可能無法受益於較高的效能等級。 這包括缺少 WHERE 子句、具有遺漏的索引或具有過時統計資料的查詢。 這些應用程式會受益於標準查詢效能微調技術。 如需詳細資訊，請參閱[遺漏索引](#missing-indexes)和[查詢微調和提示](#query-tuning-and-hinting)。
* **具有次佳資料存取設計的應用程式**。 具有內在資料存取並行問題的應用程式，例如死結，可能無法受益於較高的效能等級。 請考慮藉由使用 Azure 快取服務或其他快取技術來快取用戶端的資料，以減少對 Azure SQL Database 的往返作業。 請參閱 [應用程式層快取](#application-tier-caching)。

## <a name="tuning-techniques"></a>微調技術
在本節中，我們會討論一些技術，您可以用這些技術來微調 Azure SQL Database 以獲取應用程式的最佳效能，並且盡可能在最小的效能等級中執行。 其中有些技術符合傳統的 SQL Server 微調最佳做法，但是其他技術則是專屬於 Azure SQL Database。 在某些情況下，您可以檢查資料庫已取用的資源來尋找要進一步微調的區域，並擴充傳統的 SQL Server 技術以使其適用於 Azure SQL Database。

### <a name="azure-portal-tools"></a>Azure 入口網站工具
Azure 入口網站中有下列工具協助您分析及修正 SQL Database 的效能問題：

* [查詢效能深入解析](sql-database-query-performance.md)
* [SQL Database 建議程式](sql-database-advisor.md)

Azure 入口網站有關於這兩個工具以及其使用方式的詳細資訊。 若要有效率地診斷並更正問題，建議您先嘗試使用 Azure 入口網站中的這兩個工具。 我們建議您使用接下來要討論的手動微調方法，以處理特殊情況下的遺漏索引和查詢微調。

### <a name="missing-indexes"></a>遺漏的索引
OLTP 資料庫效能中常見的問題與實體資料庫設計相關。 資料庫結構描述的設計和轉移通常不會經過大規模測試 (無論是在負載或資料數量)。 不幸的是，查詢計劃的效能可能只有小規模可以接受，但是在面臨實際執行等級的資料數量時會大幅降低。 此問題最常見的來源是缺少適當的索引來滿足篩選或查詢中的其他限制。 遺漏的索引通常會在索引搜尋可以滿足時，以資料表掃描的形式呈現。

在此範例中，選取的查詢計劃會在搜尋即可滿足需要時使用掃描︰

    DROP TABLE dbo.missingindex;
    CREATE TABLE dbo.missingindex (col1 INT IDENTITY PRIMARY KEY, col2 INT);
    DECLARE @a int = 0;
    SET NOCOUNT ON;
    BEGIN TRANSACTION
    WHILE @a < 20000
    BEGIN
        INSERT INTO dbo.missingindex(col2) VALUES (@a);
        SET @a += 1;
    END
    COMMIT TRANSACTION;
    GO
    SELECT m1.col1
    FROM dbo.missingindex m1 INNER JOIN dbo.missingindex m2 ON(m1.col1=m2.col1)
    WHERE m1.col2 = 4;

![具有遺漏索引的查詢計劃](./media/sql-database-performance-guidance/query_plan_missing_indexes.png)

Azure SQL Database 可協助您尋找和修正常見的遺漏索引狀況。 Azure SQL Database 內建的 DMV 會查看查詢編譯，其中的索引會大幅減少執行查詢的估計成本。 執行查詢期間，SQL Database 會追蹤每個查詢計劃的執行頻率，並追蹤執行查詢計劃和其中存在該索引之假設查詢計劃之間的預估落差。 您可以使用這些 DMV 快速推測哪些實體資料庫設計變更可能會改善資料庫和其實際工作負載的整體工作負載成本。

您可以使用下列查詢來評估潛在的遺漏索引︰

    SELECT CONVERT (varchar, getdate(), 126) AS runtime,
        mig.index_group_handle, mid.index_handle,
        CONVERT (decimal (28,1), migs.avg_total_user_cost * migs.avg_user_impact *
                (migs.user_seeks + migs.user_scans)) AS improvement_measure,
        'CREATE INDEX missing_index_' + CONVERT (varchar, mig.index_group_handle) + '_' +
                  CONVERT (varchar, mid.index_handle) + ' ON ' + mid.statement + '
                  (' + ISNULL (mid.equality_columns,'')
                  + CASE WHEN mid.equality_columns IS NOT NULL
                              AND mid.inequality_columns IS NOT NULL
                         THEN ',' ELSE '' END + ISNULL (mid.inequality_columns, '')
                  + ')'
                  + ISNULL (' INCLUDE (' + mid.included_columns + ')', '') AS create_index_statement,
        migs.*,
        mid.database_id,
        mid.[object_id]
    FROM sys.dm_db_missing_index_groups AS mig
    INNER JOIN sys.dm_db_missing_index_group_stats AS migs
        ON migs.group_handle = mig.index_group_handle
    INNER JOIN sys.dm_db_missing_index_details AS mid
        ON mig.index_handle = mid.index_handle
    ORDER BY migs.avg_total_user_cost * migs.avg_user_impact * (migs.user_seeks + migs.user_scans) DESC

在此範例中，查詢導致了這項建議︰

    CREATE INDEX missing_index_5006_5005 ON [dbo].[missingindex] ([col2])  

在其建立之後，同一個 SELECT 陳述式會挑選不同的計劃，其使用搜尋而不是掃描，然後更有效率地執行計劃：

![具有已更正索引的查詢計劃](./media/sql-database-performance-guidance/query_plan_corrected_indexes.png)

重要的觀念是共用商品系統的 I/O 容量會比專用伺服器電腦的容量受到更多限制。 重點在於要最小化不必要的 I/O，才能在 Azure SQL Database 的服務層中，於每個效能等級的 DTU 內充分利用此系統。 適當的實體資料庫設計選項可以大幅改善個別查詢的延遲、改善每個縮放單位中可處理的並行要求輸送量，並最小化滿足查詢所需的成本。 如需有關遺漏索引 DMV 的詳細資訊，請參閱 [sys.dm_db_missing_index_details](https://msdn.microsoft.com/library/ms345434.aspx)。

### <a name="query-tuning-and-hinting"></a>查詢微調和提示
Azure SQL Database 內的查詢最佳化工具類似於傳統的 SQL Server 查詢最佳化工具。 微調查詢和了解查詢最佳化工具之推論模型限制的大多數最佳做法也適用於 Azure SQL Database。 如果您微調 Azure SQL Database 中的查詢，您可以因為減少彙整資源需求而獲得額外好處。 您的應用程式能夠以低於未經微調之查詢的成本執行，因為它可以在較低的效能等級中執行。

SQL Server 中常見並且也適用於 Azure SQL Database 的範例是查詢最佳化工具如何「探查」參數。 在編譯期間，查詢最佳化工具會評估參數值以判斷該值是否可以產生較佳的查詢計劃。 雖然這個策略產生的查詢計劃通常在速度上明顯優於不知道參數值就編譯的計劃，但是此策略目前在 SQL Server 和 Azure SQL Database 中的運作狀況並不完美。 有時參數並未經過探查，有時則是參數已經過探查，但產生的計劃對工作負載中的整組參數值而言只是次佳的。 Microsoft 包括查詢提示 (指示詞)，因此您可以更刻意地指定意圖及覆寫參數探查的預設行為。 如果您使用提示，您通常可以修正指定客戶工作負載的預設 SQL Server 或 Azure SQL Database 行為不完美的情況。

下一個範例會示範查詢處理器如何產生對效能和資源需求都不盡理想的計劃。 此範例也會示範如果您使用查詢提示，則可以降低 SQL Database 的查詢執行時間和資源需求︰

    DROP TABLE psptest1;
    CREATE TABLE psptest1(col1 int primary key identity, col2 int, col3 binary(200));

    DECLARE @a int = 0;
    SET NOCOUNT ON;
    BEGIN TRANSACTION
    WHILE @a < 20000
    BEGIN
        INSERT INTO psptest1(col2) values (1);
        INSERT INTO psptest1(col2) values (@a);
        SET @a += 1;
    END
    COMMIT TRANSACTION
    CREATE INDEX i1 on psptest1(col2);
    GO

    CREATE PROCEDURE psp1 (@param1 int)
    AS
    BEGIN
        INSERT INTO t1 SELECT * FROM psptest1
        WHERE col2 = @param1
        ORDER BY col2;
    END
    GO

    CREATE PROCEDURE psp2 (@param2 int)
    AS
    BEGIN
        INSERT INTO t1 SELECT * FROM psptest1 WHERE col2 = @param2
        ORDER BY col2
        OPTION (OPTIMIZE FOR (@param2 UNKNOWN))
    END
    GO

    CREATE TABLE t1 (col1 int primary key, col2 int, col3 binary(200));
    GO

設定程式碼會建立一個具有扭曲資料散發的資料表。 最佳的查詢計劃會根據選取的參數而有差異。 不幸的是，計劃快取行為並不一定會根據最常見的參數值重新編譯查詢。 因此，即使不同的計劃會是平均來說較好的計劃選項，仍有可能會快取不盡理想的計劃並用於許多值。 然後，查詢計劃會建立兩個完全相同的預存程序，唯一的差異是其中一個有特殊的查詢提示。

**範例 (第 1 部分)**

    -- Prime Procedure Cache with scan plan
    EXEC psp1 @param1=1;
    TRUNCATE TABLE t1;

    -- Iterate multiple times to show the performance difference
    DECLARE @i int = 0;
    WHILE @i < 1000
    BEGIN
        EXEC psp1 @param1=2;
        TRUNCATE TABLE t1;
        SET @i += 1;
    END

**範例 (第 2 部分)**

(建議您先等待至少 10 分鐘再開始範例的第 2 部分，這樣一來，所產生之遙測資料的結果才會截然不同)。

    EXEC psp2 @param2=1;
    TRUNCATE TABLE t1;

    DECLARE @i int = 0;
    WHILE @i < 1000
    BEGIN
        EXEC psp2 @param2=2;
        TRUNCATE TABLE t1;
        SET @i += 1;
    END

此範例中的每個部分都嘗試執行參數化的 insert 陳述式 1000 次 (以產生足夠的負載來做為測試資料集)。 當它執行預存程序時，查詢處理器會檢查第一次編譯 (參數「探查」) 期間傳遞至程序的參數值。 即使參數值不同，處理器還是會快取產生的計劃並用於稍後的叫用。 最佳的計劃可能不會用於所有情況。 有時候您必須引導最佳化工具挑選計劃，相較於第一次編譯查詢時的特定案例，此計劃較適合平均案例。 在此範例中，初始計劃會產生「掃描」計劃，此計劃會讀取所有資料列來尋找符合參數的每個值：

![使用掃描計劃微調查詢](./media/sql-database-performance-guidance/query_tuning_1.png)

因為我們以值 1 執行此程序，所以產生的計劃會是值 1 的最佳選擇，但是對資料表中的其他值而言未必是最佳的。 如果您隨機挑選每個計劃，因為該計劃執行得更慢而且使用更多資源，因此結果可能不是您要的。

如果您以 `SET STATISTICS IO` 設為 `ON` 來執行測試，此範例會在幕後完成邏輯掃描工作。 您可以看到計劃完成了 1,148 次讀取 (如果平均案例是只傳回一個資料列，則此計劃是沒效率的)：

![使用邏輯掃描微調查詢](./media/sql-database-performance-guidance/query_tuning_2.png)

此範例的第二個部分使用查詢提示告訴最佳化工具，在編譯處理程序期間使用特定值。 在此情況下，它會強制查詢處理器忽略傳遞做為參數的值，並改為假設 `UNKNOWN`。 這表示資料表中具有平均頻率的值 (忽略扭曲)。 產生的計劃是以搜尋為基礎的計劃，平均而言會比此範例第 1 部分的計劃更快速並使用較少資源：

![使用查詢提示微調查詢](./media/sql-database-performance-guidance/query_tuning_3.png)

您可以在 **sys.resource_stats** 資料表中看到這個影響 (從您執行測試的時間到資料填入資料表的時間會發生延遲)。 在此範例中，第 1 部分會在 22:25:00 時間範圍期間執行，而第 2 部分會在 22:35:00 執行。 較早的時間範圍在該時間範圍內使用的資源比較晚的時間範圍還多 (因為計劃效率改善)。

    SELECT TOP 1000 *
    FROM sys.resource_stats
    WHERE database_name = 'resource1'
    ORDER BY start_time DESC

![查詢微調範例結果](./media/sql-database-performance-guidance/query_tuning_4.png)

> [!NOTE]
> 雖然此範例中的數量已刻意簡化，次佳參數的影響還是很明顯，尤其是對於較大型的資料庫。 在極端的情況下，快速案例之間的差異可達數秒，而緩慢案例之間的差異可達數小時。
>
>

您可以檢查 **sys.resource_stats** 來判斷測試使用的資源比另一個測試多或少。 在比較資料時，請將測試的時間隔開，讓它們不在 **sys.resource_stats** 檢視的同一個 5 分鐘時間範圍內。 練習的目標是要最小化使用的資源總量，而不是最小化尖峰資源。 一般而言，最佳化一段延遲的程式碼也可減少資源耗用量。 請確定您對應用程式所做的變更是必要的，而且這些變更不會讓使用應用程式的某人在使用查詢提示時對客戶體驗造成負面影響。

如果工作負載具有一組重複的查詢，擷取並驗證您計劃選項的最適化通常是合理的，因為這會讓主控資料庫所需的資源大小單位降到最低。 在驗證之後，請偶爾重新檢查計劃以確保它們不會降級。 您可以深入了解 [查詢提示 (Transact-SQL)](https://msdn.microsoft.com/library/ms181714.aspx)。

### <a name="cross-database-sharding"></a>跨資料庫分區化
因為 Azure SQL Database 會在商用硬體上執行，所以單一資料庫的容量限制會比傳統的內部部署 SQL Server 安裝更低。 有些客戶會在資料庫作業不符合 Azure SQL Database 中的單一資料庫限制時，使用分區化技術在多個資料庫散佈這些作業。 在 Azure SQL Database 上使用分區化技術的大部分客戶都會在跨多個資料庫的單一維度上分割其資料。 針對此方法，您必須了解 OLTP 應用程式通常會執行的交易只會套用到結構描述內的一個資料列或一小組資料列。

> [!NOTE]
> SQL Database 現在提供可協助分區化的程式庫。 如需詳細資訊，請參閱 [彈性資料庫用戶端程式庫概觀](sql-database-elastic-database-client-library.md)。
>
>

例如，如果資料庫有客戶名稱、訂單及訂單詳細資料 (如 SQL Server 隨附的傳統範例 Northwind 資料庫)，則您可以透過以相關訂單和訂單詳細資料等資訊來分組客戶，將這項資料分割至多個資料庫。 您可以保證客戶的資料會保留在單一資料庫內。 應用程式會跨資料庫分割不同的客戶，跨多個資料庫有效分配負載。 透過分區化，客戶不但可以避免資料庫大小上限，Azure SQL Database 也能處理明顯大於不同效能等級限制的工作負載，前提是每個個別資料庫符合其 DTU。

雖然資料庫分區化不會減少方案的彙整資源容量，但可以非常有效地支援分配到多個資料庫的非常大型方案。 每個資料庫可以在不同的效能等級執行以支援具有高資源需求且非常大型的「有效」資料庫。

### <a name="functional-partitioning"></a>功能資料分割
SQL Server 使用者通常會在單一資料庫內結合許多功能。 例如，如果應用程式有管理商店庫存的邏輯，該資料庫可能具有的邏輯會與庫存、追蹤訂單、預存程序和管理月底報告的索引或具體化檢視相關聯。 這項技術可以簡化備份等作業的資料庫管理，但也需要您調整硬體大小來跨應用程式的所有功能處理尖峰負載。

如果您在 Azure SQL Database 中使用相應放大架構，則最好將應用程式的不同功能分割至不同資料庫。 使用這項技術時，每個應用程式都可以獨立縮放。 隨著應用程式變得更忙碌 (而且資料庫上的負載增加)，系統管理員可為應用程式內的每個功能選擇獨立的效能等級。 此架構在達到限制時，應用程式會比單一商用電腦可處理的應用程式還大，因為負載會分配給多部電腦。

### <a name="batch-queries"></a>批次查詢
對於使用大量、頻繁的特定查詢存取資料的應用程式而言，大量的回應時間都花費在應用程式層和和 Azure SQL Database 層之間的網路通訊上。 即使應用程式和 Azure SQL Database 都位於相同的資料中心，兩者之間的網路延遲還是可能因為大量資料存取作業而放大。 若要減少資料存取作業的網路往返，請考慮選擇批次處理特定查詢或將其編譯成預存程序。 如果您批次處理特定查詢，您可以將多個查詢當做一個大型批次，在單一往返中傳送到 Azure SQL Database。 如果您在預存程序中編譯特定查詢，您可以達到和對其進行批次處理時相同的結果。 使用預存程序也能讓您獲益，因為在 Azure SQL Database 中快取查詢計劃的機會增加，因此您可以再次使於預存程序。

某些應用程式是寫入密集型的。 有時候，您可以藉由考慮如何一併批次處理寫入來減少資料庫上的 I/O 總負載。 這通常與在預存程序及特定查詢內使用明確的交易而不是自動認可的交易一樣容易。 如需評估您可以使用的不同技術，請參閱 [Azure 中 SQL Database 應用程式的批次處理技術](https://msdn.microsoft.com/library/windowsazure/dn132615.aspx)。 實驗您自己的工作負載來尋找正確的批次處理模型。 請務必了解，模型在交易一致性保證上可能稍有不同。 要尋找可最小化資源使用的適當工作負載，就必須找出一致性與效能權衡取捨的正確組合。

### <a name="application-tier-caching"></a>應用程式層快取
某些資料庫應用程式具有大量讀取工作負載。 對應用程式層進行快取可能會減少資料庫上的負載，並可能使用 Azure SQL Database 而有機會降低支援資料庫所需的效能等級。 使用 [Azure Redis 快取](https://azure.microsoft.com/services/cache/)時，如果您具有大量讀取工作負載，您可以讀取資料一次 (或可能每個應用程式層電腦讀取一次，取決於設定方式)，然後將該資料儲存在 SQL Database 之外。 此方式可減少資料庫負載 (CPU 和讀取 I/O)，但會對交易一致性造成影響，因為從快取讀取的資料可能不會與資料庫中的資料同步。 雖然許多應用程式可接受一定程度的不一致性，但並非所有工作負載都是如此。 您應該充分了解應用程式的任何需求，然後再實作應用程式層快取策略。

## <a name="next-steps"></a>後續步驟
* 如需服務層的詳細資訊，請參閱 [SQL Database 選項和效能](sql-database-service-tiers.md)
* 如需彈性集區的詳細資訊，請參閱[什麼是 Azure 彈性集區？](sql-database-elastic-pool.md)
* 如需效能和彈性集區的相關資訊，請參閱 [考慮使用彈性集區的時機](sql-database-elastic-pool-guidance.md)

