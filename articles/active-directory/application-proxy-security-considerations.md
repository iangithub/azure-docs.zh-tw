---
title: "使用 Azure AD 應用程式 Proxy 遠端存取應用程式的安全性考量 | Microsoft Docs"
description: "涵蓋使用 Azure AD 應用程式 Proxy 的安全性考量"
services: active-directory
documentationcenter: 
author: kgremban
manager: femila
ms.assetid: 
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/12/2017
ms.author: kgremban
translationtype: Human Translation
ms.sourcegitcommit: 5d741836f5defd5d9287b90e53e71aeea439a1df
ms.openlocfilehash: 81a7a57e6b025710660f7d55145ee286b71acf24
ms.lasthandoff: 03/01/2017


---

# <a name="security-considerations-for-accessing-apps-remotely-by-using-azure-ad-application-proxy"></a>使用 Azure AD 應用程式 Proxy 遠端存取應用程式的安全性考量

>[!NOTE]
>在升級至 Premium 或 Basic 版本的 Azure Active Directory 後，才能使用應用程式 Proxy 功能。 如需詳細資訊，請參閱 [Azure Active Directory 版本](active-directory-editions.md)。

本文說明 Azure Active Directory (Azure AD) 應用程式 Proxy 如何提供遠端發佈及存取應用程式的安全服務。

Azure AD 應用程式 Proxy 提供下列安全性優點︰

**已驗證的存取︰**只有已驗證的連線可以存取您的網路。

* Azure AD 應用程式 Proxy 依賴 Azure AD Security Token Service (STS) 來進行所有驗證。 對於使用預先驗證發佈的應用程式，未提供有效 STS 權杖的流量無法通過應用程式 Proxy 服務到達您的環境。
* 預先驗證 (就其本質) 會封鎖大量匿名攻擊，因為只有已驗證的身分識別可以存取後端應用程式。

**條件式存取︰**在建立您的網路連線之前，先套用更豐富的原則控制。

* 使用條件式存取，就可以進一步定義允許哪些流量存取後端應用程式上的限制。 您可以根據位置、驗證強度和使用者風險狀況來定義限制。
* 此功能可使攻擊者受到其他障礙。 如需有關條件式存取的詳細資訊，請參閱[開始使用 Azure AD 條件式存取](https://azure.microsoft.com/en-us/documentation/articles/active-directory-conditional-access-azuread-connected-apps)。

**流量終止︰**終止雲端中所有的流量。

* Azure AD 應用程式 Proxy 是反向 Proxy，因此所有至後端應用程式的流量會在服務終止。 工作階段只能利用後端伺服器來重新建立，也就是說，後端伺服器不會對直接的 HTTP 流量公開。 例如，您可以更輕鬆地降低目標的攻擊。

**所有存取都是輸出︰**不需要開啟連往公司網路的輸入連線。

* Azure AD 連接器會維護連往 Azure AD 應用程式 Proxy 服務的輸出連線，也就是說，您不需要開啟防火牆連接埠以供連入連線使用。
* 傳統方法需要周邊網路 (也稱為「DMZ」、「非軍事區」和「遮蔽式子網路」) 並在網路邊緣開放未經授權連線的存取權。 這種情況會造成需要額外投資許多 Web 應用程式防火牆 (WAF) 產品，以便分析流量並對環境提供額外的保護。 若使用應用程式 Proxy，就可以避免這種情況。 您甚至可以考慮直接進行而不使用周邊網路，因為所有連線皆為輸出方向，並且是透過安全通道來傳輸。

**安全性分析和機器語言為基礎的智慧︰**獲得最新的安全性保護。

* Azure AD Identity Protection 的機器學習導向智慧所使用之資料是來自我們的數位犯罪防治中心和 Microsoft 安全性回應中心。 我們共同主動識別遭入侵的帳戶，並提供來自高風險登入的即時防護。 我們考慮許多因素，例如來自受感染裝置、透過匿名網路以及來自非典型與假位置的存取。
* 這些報告和事件中有許多已可透過 API 與安全性資訊和事件管理 (SIEM) 系統整合。
* 如需詳細資訊，請參閱 [Azure AD Identity Protection](https://azure.microsoft.com/en-us/documentation/articles/active-directory-identityprotection)。

**遠端存取即服務︰**您不必擔心維護及修補內部部署伺服器的事宜。

* Azure AD 應用程式 Proxy 是 Microsoft 自有的網際網路級服務，因此請放心，您永遠會獲得最新的安全性修補程式和升級。
* 未更新的軟體仍需負責處理大量攻擊。 使用我們的服務模型，您不必再承擔邊緣伺服器的管理工作。

依據 [Azure 信任中心](https://azure.microsoft.com/en-us/support/trust-center)所概述之指導方針和標準，使用 Azure AD 運作所提供的遠端存取服務。

下圖顯示 Azure AD 如何讓您在內部部署應用程式實現安全的遠端存取。

 ![透過 Azure AD 應用程式 Proxy 進行安全遠端存取的示意圖](./media/application-proxy-security-considerations/secure-remote-access.png)

>[!NOTE]
>為了改善 Azure AD 應用程式 Proxy 所發佈應用程式的安全性，我們會封鎖 Web 編目程式傀儡程式，使其無法對您的應用程式編製索引和進行保存。 每次 Web 編目程式傀儡程式嘗試擷取已發佈應用程式的傀儡程式設定時，應用程式 Proxy 會以含有下列文字的 robots.txt 檔案回覆：
>
>User-agent: *  
>Disallow: /

## <a name="components-of-the-azure-ad-application-proxy-solution"></a>Azure AD 應用程式 Proxy 解決方案的元件

Azure AD 應用程式 Proxy 是由兩個部分組成︰

* 雲端架構服務︰外部用戶端/使用者會連線到此服務。
* Azure AD 應用程式 Proxy 連接器︰這是內部部署元件，此連接器會接聽來自 Azure AD 應用程式 Proxy 服務的要求，並處理對內部應用程式的連線。 該服務包括處理用於 SSO 的 Kerberos 限制委派 (KCD) 之類的項目。

以下為建立連接器與應用程式 Proxy 服務之間流量的時機︰

* 第一次設定連接器。
* 連接器會從應用程式 Proxy 服務提取組態資訊，包括每個連接器為其成員的連接器群組。
* 使用者會存取發佈的應用程式。

>[!NOTE]
>所有通訊會透過 SSL 發生，且一律源自於至應用程式 Proxy 服務的連接器。 此服務只會輸出。

連接器會使用用戶端憑證來驗證幾乎所有呼叫的應用程式 Proxy 服務。 此程序的唯一例外是可供建立用戶端憑證的初始設定步驟。

### <a name="installing-the-connector"></a>安裝連接器

第一次設定連接器時，會發生下列流程事件：

1. 連接器註冊服務會做為連接器安裝的一部分。 系統目前會提示使用者輸入其 Azure AD 系統管理員認證。 接著會向 Azure AD 應用程式 Proxy 服務顯示必要權杖。
2. 應用程式 Proxy 會評估權杖，以確保使用者在發行權杖的租用戶內是公司系統管理員角色的成員。 如果使用者不是系統管理員角色的成員，此程序就會終止。
3. 連接器會產生用戶端憑證要求，並將此要求連同權杖傳遞至應用程式 Proxy 服務，再由該服務驗證權杖並簽署用戶端憑證要求。
4. 連接器可以使用此用戶端憑證，以便未來與應用程式 Proxy 服務通訊。
5. 連接器會使用其用戶端憑證從服務執行系統設定資料的初始提取，而它現在已準備好接受要求。

### <a name="updating-the-configuration-settings"></a>更新組態設定

每當應用程式 Proxy 服務更新組態設定時，就會發生下列流程事件︰

1. 連接器會使用其用戶端憑證連線至應用程式 Proxy 服務內的組態端點。
2. 用戶端憑證經過驗證後，應用程式 Proxy 服務會將組態資料傳回連接器 (例如，連接器應屬之連接器群組)。
3. 如果目前的憑證已存在超過 30 天，連接器會產生新的憑證要求，有效地每隔 30 天就更新用戶端憑證。

### <a name="accessing-published-applications"></a>存取已發佈的應用程式

當使用者存取已發佈的應用程式時，會發生下列流程事件：

1. 應用程式 Proxy 服務會檢查應用程式的組態設定。 如果應用程式設定為以 Azure AD 使用預先驗證時，使用者會重新導向至 Azure AD STS 驗證。 如果您使用傳遞發佈應用程式，則會略過此步驟。

 a. 在使用 Azure AD 驗證期間，應用程式 Proxy 會檢查特定應用程式的所有條件式存取原則需求。 這個步驟是為了確保已對應用程式指派使用者。 如果需要 Multi-Factor Authentication (MFA)，驗證順序會提示使用者進行第二因素驗證。

 b. 通過所有檢查後，Azure AD STS 會針對應用程式發出已簽署權杖，並將使用者重新導向回到應用程式 Proxy 服務。

 c. 然後應用程式 Proxy 會驗證權杖，以確定它已發行至使用者要求存取的應用程式。 它也會執行其他檢查，例如確保權杖是由 Azure AD 所簽署，且仍在有效期限內。

 d. 應用程式 Proxy 會設定加密的驗證 cookie (例如非持續性 cookie)，以表示已發生應用程式驗證。 此 cookie 包含根據 Azure AD 的權杖和其他資料之到期時間戳記，例如以驗證為基礎的使用者名稱。 會使用僅應用程式 Proxy 服務所知的私密金鑰來加密此 cookie。

 e. 應用程式 Proxy 會將使用者重新導向回到原始要求的 URL。

 >[!NOTE]
 >如果預先驗證步驟的任何部分失敗，使用者的要求會遭到拒絕，且使用者會顯示訊息，指出問題的來源。
 >

2. 在收到來自用戶端的要求後，應用程式 Proxy 便會驗證是否已符合預先驗證條件，且 cookie 仍然有效 (視需要)。 然後，應用程式 Proxy 就會在適當的佇列中提出要求，以供內部部署連接器處理。 

 >[!NOTE]
 >來自連接器的所有要求都會輸出到應用程式 Proxy 服務。 連接器保持對應用程式 Proxy 開啟輸出連線。 當要求傳入時，應用程式 Proxy 會在其中一個開啟連線上佇列要求，以供連接器挑選。

 * 要求包含來自應用程式的項目，例如要求標頭、來自加密 cookie 的資料、提出要求的使用者，以及要求識別碼。 但是，加密的驗證 cookie 不會傳送給連接器。

3. 連接器會根據長時間執行的輸出連線從佇列接收要求。 根據要求，應用程式 Proxy 會執行下列其中一個動作︰

 * 連接器會確認是否可以找出應用程式。 如果無法識別應用程式，連接器會建立連往應用程式 Proxy 服務的連線，以收集關於應用程式的詳細資料，並在本機快取應用程式。

 * 如果要求是簡單的作業 (例如，主體內沒有資料現狀符合 RESTful「GET」要求)，連接器會建立連往目標內部資源的連線，然後等候回應。

 * 如果要求在主體中具有與它相關聯的資料 (例如，RESTful「POST」作業)，連接器會使用用戶端憑證建立與應用程式 Proxy 執行個體的輸出連線。 它會建立此連線來要求資料，並開啟與內部部署資源的連線。 在收到來自連接器的要求後，應用程式 Proxy 服務會開始接受來自使用者的內容，並將資料轉送至連接器。 連接器會依次將資料轉送到內部資源。

4. 完成所有內容的要求並傳輸至後端後，連接器會等候回應。

5. 在收到回應後，連接器會建立對應用程式 Proxy 服務的輸出連線，以傳回標頭詳細資料，並開始串流傳回的資料。

6. 應用程式 Proxy「串流」資料給使用者。 標頭的一些處理可能會視需要發生在這裡，並由應用程式定義。

如果您需要從 Azure web 應用程式以用戶端瀏覽器方式通訊內部部署 Windows 驗證之簡易物件存取通訊協定 (SOAP) 端點的協助，請參閱 [Azure 欄位筆記部落格](http://www.azurefieldnotes.com/2016/12/02/claims-to-windows-identity-translation-solutions-and-its-flaws-when-using-azure-ad-application-proxy)。

## <a name="next-steps"></a>後續步驟

[了解 Azure AD 應用程式 Proxy 連接器](application-proxy-understand-connectors.md)

