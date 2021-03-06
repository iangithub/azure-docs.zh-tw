---
title: "在 Azure AD Domain Services 中設定安全的 LDAP (LDAPS) | Microsoft Docs"
description: "針對 Azure AD 網域服務受管理網域設定安全的 LDAP (LDAPS)"
services: active-directory-ds
documentationcenter: 
author: mahesh-unnikrishnan
manager: stevenpo
editor: curtand
ms.assetid: c6da94b6-4328-4230-801a-4b646055d4d7
ms.service: active-directory-ds
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/06/2017
ms.author: maheshu
translationtype: Human Translation
ms.sourcegitcommit: 44be67cd5c59a57cafd244ce0a49a6fadf44bdda
ms.openlocfilehash: 8abde97ecb906384cd6e62811fcbb833bdc8be51
ms.lasthandoff: 01/25/2017


---
# <a name="configure-secure-ldap-ldaps-for-an-azure-ad-domain-services-managed-domain"></a>針對 Azure AD 網域服務受管理網域設定安全的 LDAP (LDAPS)
本文說明如何為 Azure AD 網域服務受管理網域啟用安全的輕量型目錄存取通訊協定 (LDAPS)。 安全的 LDAP 亦稱為「透過安全通訊端層 (SSL)/傳輸層安全性 (TLS) 的輕量型目錄存取通訊協定 (LDAP)」。

## <a name="before-you-begin"></a>開始之前
若要執行本文中所列的工作，您需要︰

1. 有效的 **Azure 訂用帳戶**。
2. **Azure AD 目錄** - 與內部部署目錄或僅限雲端的目錄同步處理。
3. **Azure AD 網域服務** 必須已針對 Azure AD 目錄啟用。 如果還沒有啟用，請按照 [入門指南](active-directory-ds-getting-started.md)所述的所有工作來進行。
4. **用來啟用安全 LDAP 的憑證**。

   * **建議** - 從受信任的公共憑證授權單位取得憑證。 這是更安全的組態選項。
   * 或者，您也可以選擇 [建立自我簽署憑證](#task-1---obtain-a-certificate-for-secure-ldap) ，如本文稍後所示。

<br>

### <a name="requirements-for-the-secure-ldap-certificate"></a>安全 LDAP 憑證的需求
請先根據下列準則取得有效的憑證，再啟用安全的 LDAP。 如果您嘗試使用無效/不正確的憑證來為受管理的網域啟用安全的 LDAP，您會遭遇失敗。

1. **信任的簽發者** - 憑證必須由需要使用安全的 LDAP 連線到網域的電腦，所信任的授權單位加以發行。 此授權單位可能是受這些電腦信任的公開憑證授權單位。
2. **存留期** - 憑證必須至少在接下來的 3 至 6 個月內都要保持有效。 當憑證過期時，受管理網域的安全 LDAP 存取會中斷。
3. **主體名稱** - 在受管理的網域中，憑證的主體名稱必須是萬用字元。 比方說，如果您的網域名稱為 'contoso100.com'，則憑證的主體名稱必須是 '*.contoso100.com'。 設定此萬用字元名稱的 DNS 名稱 (主體替代名稱)。
4. **金鑰使用方法** - 必須將憑證設定為下列用途 - 數位簽章與金鑰編密。
5. **憑證目的** - 憑證必須有效可進行 SSL 伺服器驗證。

> [!NOTE]
> **企業憑證授權單位︰**Azure AD Domain Services 目前不支援使用您組織的企業憑證授權單位所簽發的安全 LDAP 憑證。 這項限制是因為服務不信任您企業 CA 做為根憑證授權單位。 我們預計在未來會新增對企業 CA 的支援。 如果您一定要使用您企業 CA 所簽發的憑證，請[與我們連絡](active-directory-ds-contact-us.md)以取得協助。
>
>

<br>

## <a name="task-1---obtain-a-certificate-for-secure-ldap"></a>工作 1 - 取得安全 LDAP 的憑證
第一項工作牽涉到取得憑證，該憑證用於對受管理的網域進行安全的 LDAP 存取。 您有兩個選擇：

* 從憑證授權單位取得憑證。 授權單位可以是公開憑證授權單位。
* 建立自我簽署憑證。

### <a name="option-a-recommended---obtain-a-secure-ldap-certificate-from-a-certification-authority"></a>選項 A (建議選項) - 從憑證授權單位取得安全的 LDAP 憑證
如果您的組織從公共憑證授權單位取得其憑證，您必須從該公共憑證授權單位取得安全的 LDAP 憑證。

在要求憑證時，請務必遵循 [安全 LDAP 憑證的需求](#requirements-for-the-secure-ldap-certificate)中所述的需求。

> [!NOTE]
> 需要使用安全的 LDAP 連線到受管理網域的用戶端電腦必須信任安全 LDAPS 憑證的簽發者。
>
>

### <a name="option-b---create-a-self-signed-certificate-for-secure-ldap"></a>選項 B - 為安全的 LDAP 建立自我簽署的憑證
如果您不希望使用公開憑證授權單位的憑證，可以選擇建立安全 LDAP 的自我簽署憑證。

**使用 PowerShell 建立自我簽署憑證**

在您的 Windows 電腦上以 **系統管理員** 的身分開啟新的 PowerShell 視窗，並輸入下列命令以建立新的自我簽署憑證。

    $lifetime=Get-Date

    New-SelfSignedCertificate -Subject *.contoso100.com -NotAfter $lifetime.AddDays(365) -KeyUsage DigitalSignature, KeyEncipherment -Type SSLServerAuthentication -DnsName *.contoso100.com

在上述範例中，將 'contoso100.com' 替換為 Azure AD 網域服務受管理網域的 DNS 網域名稱。

![選取 Azure AD 目錄](./media/active-directory-domain-services-admin-guide/secure-ldap-powershell-create-self-signed-cert.png)

新建立的自我簽署憑證會放在本機電腦的憑證存放區中。

## <a name="task-2---export-the-secure-ldap-certificate-to-a-pfx-file"></a>工作 2 - 將安全的 LDAP 憑證匯出到 .PFX 檔案
在開始這項工作之前，請先確定您已從公共憑證授權單位取得安全的 LDAP 憑證，或已建立自我簽署憑證。

執行下列步驟，將 LDAPS 憑證匯出到 .PFX 檔案。

1. 按 [開始] 按鈕並輸入 **R**。在 [執行] 對話方塊中，輸入 **mmc**，然後按一下 [確定]。

    ![啟動 MMC 主控台](./media/active-directory-domain-services-admin-guide/secure-ldap-start-run.png)
2. 在 [使用者帳戶控制] 提示字元處按一下 [是]，以系統管理員身分啟動 MMC (Microsoft Management Console)。
3. 在 [檔案] 功能表上，按一下 [新增/移除嵌入式管理單元...]。

    ![新增嵌入式管理單元至 MMC 主控台](./media/active-directory-domain-services-admin-guide/secure-ldap-add-snapin.png)
4. 在 [新增或移除嵌入式管理單元] 對話方塊中，選取 [憑證] 嵌入式管理單元，然後按一下 [新增 >] 按鈕。

    ![新增憑證嵌入式管理單元至 MMC 主控台](./media/active-directory-domain-services-admin-guide/secure-ldap-add-certificates-snapin.png)
5. 在 [憑證嵌入式管理單元] 精靈中，選取 [電腦帳戶] 並按 [下一步]。

    ![新增電腦帳戶的憑證嵌入式管理單元](./media/active-directory-domain-services-admin-guide/secure-ldap-add-certificates-computer-account.png)
6. 在 [選取電腦] 頁面上，選取 [本機電腦: (執行這個主控台的電腦)] 並按一下 [完成]。

    ![新增憑證嵌入式管理單元 - 選取電腦](./media/active-directory-domain-services-admin-guide/secure-ldap-add-certificates-local-computer.png)
7. 在 [新增或移除嵌入式管理單元] 對話方塊中，按一下 [確定] 以在 MMC 中新增憑證嵌入式管理單元。

    ![新增憑證嵌入式管理單元至 MMC - 完成](./media/active-directory-domain-services-admin-guide/secure-ldap-add-certificates-snapin-done.png)
8. 在 MMC 視窗中，按一下以展開 [主控台根目錄] 。 您應該會看到已載入 [憑證] 嵌入式管理單元。 按一下 [憑證 (本機電腦)]  加以展開。 按一下以依序展開 [個人] 節點和 [憑證] 節點。

    ![開啟個人憑證存放區](./media/active-directory-domain-services-admin-guide/secure-ldap-open-personal-store.png)
9. 您應該會看到我們所建立的自我簽署憑證。 您可以檢查憑證的屬性以確定指紋符合您在建立憑證時 PowerShell 視窗上所報告的指紋。
10. 選取自我簽署憑證並 **按一下滑鼠右鍵**。 在快顯功能表中，依序選取 [所有工作] 和 [匯出...]。

    ![匯出憑證](./media/active-directory-domain-services-admin-guide/secure-ldap-export-cert.png)
11. 在 [憑證匯出精靈] 中按 [下一步]。

    ![匯出憑證精靈](./media/active-directory-domain-services-admin-guide/secure-ldap-export-cert-wizard.png)
12. 在 [匯出私密金鑰] 頁面上，選取 [是，匯出私密金鑰] 並按 [下一步]。

    ![匯出憑證的私密金鑰](./media/active-directory-domain-services-admin-guide/secure-ldap-export-private-key.png)

    > [!WARNING]
    > 您必須匯出私密金鑰以及憑證。 如果您提供的 PFX 未包含憑證的私密金鑰，則為受管理網域啟用安全的 LDAP 會失敗。
    >
    >
13. 在 [匯出檔案格式] 頁面上，選取 [個人資訊交換 - PKCS #12 (.PFX)] 作為匯出憑證的檔案格式。

    ![匯出憑證檔案格式](./media/active-directory-domain-services-admin-guide/secure-ldap-export-to-pfx.png)

    > [!NOTE]
    > 只有 .PFX 檔案格式受到支援。 請勿將憑證匯出為 .CER 檔案格式。
    >
    >
14. 在 [安全性] 頁面上選取 [密碼] 選項，然後輸入密碼來保護 .PFX 檔案。 請記住此密碼，因為下一個工作會用到它。 按 [下一步]  繼續進行。

    ![憑證匯出的密碼 ](./media/active-directory-domain-services-admin-guide/secure-ldap-export-select-password.png)

    > [!NOTE]
    > 記下此密碼。 在 [工作 3 - 為受管理的網域啟用安全 LDAP](#task-3---enable-secure-ldap-for-the-managed-domain)
    >
    >
15. 在 [要匯出的檔案]  頁面上，指定檔案名稱及接收匯出憑證的位置。

    ![憑證匯出的路徑](./media/active-directory-domain-services-admin-guide/secure-ldap-export-select-path.png)
16. 在接下來的頁面上按一下 [完成]  ，以將憑證匯出至 PFX 檔案。 憑證匯出後，您應該會看到確認對話方塊。

    ![憑證匯出完成](./media/active-directory-domain-services-admin-guide/secure-ldap-exported-as-pfx.png)

## <a name="task-3---enable-secure-ldap-for-the-managed-domain"></a>工作 3 - 為受管理的網域啟用安全 LDAP
若要啟用安全的 LDAP，請執行下列設定步驟：

1. 瀏覽至 **[Azure 傳統入口網站](https://manage.windowsazure.com)**。
2. 在左窗格中，選取 [Active Directory]  節點。
3. 選取已啟用 Azure AD 網域服務的 Azure AD 目錄 (也稱為「租用戶」)。

    ![選取 Azure AD 目錄](./media/active-directory-domain-services-getting-started/select-aad-directory.png)
4. 按一下 [設定]  索引標籤。

    ![設定目錄的索引標籤](./media/active-directory-domain-services-getting-started/configure-tab.png)
5. 向下捲動到標題為 [網域服務] 的區段。 您應該會看到標題為 [安全 LDAP (LDAPS)]  的選項，如下列螢幕擷取畫面所示：

    ![網域服務組態區段](./media/active-directory-domain-services-admin-guide/secure-ldap-start.png)
6. 按一下 [設定憑證...] 按鈕以顯示 [設定安全 LDAP 的憑證] 對話方塊。

    ![設定安全 LDAP 的憑證](./media/active-directory-domain-services-admin-guide/secure-ldap-configure-cert-page.png)
7. 按一下 [含有憑證的 PFX 檔案]  下的資料夾圖示，指定要用於對受管理網域進行安全 LDAP 存取之憑證所在的 PFX 檔案。 此外，也請輸入將憑證匯出到 PFX 檔案時所指定的密碼。 然後，按一下底部的完成按鈕。

    ![指定安全 LDAP 的 PFX 檔案和密碼](./media/active-directory-domain-services-admin-guide/secure-ldap-specify-pfx.png)
8. [設定] 索引標籤的 [網域服務] 區段應該會變成灰色，而且有幾分鐘的時間處在 [擱置...] 狀態。 在此期間，LDAPS 憑證會進行驗證以確認是否正確，並為受管理的網域設定安全 LDAP。

    ![安全 LDAP - 擱置中狀態](./media/active-directory-domain-services-admin-guide/secure-ldap-pending-state.png)

   > [!NOTE]
   > 為受管理的網域啟用安全 LDAP 需要約 10 到 15 分鐘的時間。 如果提供的安全 LDAP 憑證不符合所需的準則，則不會為您的目錄啟用安全 LDAP，而且您會看到失敗。 例如，網域名稱不正確、憑證已到期或即將到期等。
   >
   >

9. 在為受管理的網域成功啟用安全 LDAP 後，[擱置...]  訊息應該會消失。 您應該會看到已顯示憑證的指紋。

    ![安全 LDAP 已啟用](./media/active-directory-domain-services-admin-guide/secure-ldap-enabled.png)

<br>

## <a name="task-4---enable-secure-ldap-access-over-the-internet"></a>工作 4 - 透過網際網路啟用安全 LDAP 存取
**選擇性工作** - 如果您不打算使用 LDAPS 來透過網際網路存取受管理的網域，請略過這項設定工作。

開始這項工作之前，請先確定您已完成 [工作 3](#task-3---enable-secure-ldap-for-the-managed-domain)中所述的步驟。

1. 您應該會在 [設定] 頁面的 [網域服務] 區段中看到 [透過網際網路啟用安全 LDAP 存取] 的選項。 此選項會設定為 [否]  ，因為依預設會停用透過安全 LDAP 對受管理網域的網際網路存取。

    ![安全 LDAP 已啟用](./media/active-directory-domain-services-admin-guide/secure-ldap-enabled2.png)
2. 將 [透過網際網路啟用安全 LDAP 存取] 切換為 [是]。 按一下底部面板上的 [儲存]  按鈕。
    ![安全 LDAP 已啟用](./media/active-directory-domain-services-admin-guide/secure-ldap-enable-internet-access.png)
3. [設定] 索引標籤的 [網域服務] 區段應該會變成灰色，而且有幾分鐘的時間處在 [擱置...] 狀態。 稍後會啟用透過安全 LDAP 對受管理網域的網際網路存取。

    ![安全 LDAP - 擱置中狀態](./media/active-directory-domain-services-admin-guide/secure-ldap-enable-internet-access-pending-state.png)

   > [!NOTE]
   > 為受管理網域啟用透過安全 LDAP 的網際網路存取需要約 10 分鐘的時間。
   >
   >
4. 在成功啟用透過網際網路對受管理網域進行安全 LDAP 存取後，[擱置...]  訊息應該會消失。 您應該會在 [LDAPS 存取的外部 IP 位址] 欄位中，看到可用來透過 LDAPS 存取您目錄的外部 IP 位址。

    ![安全 LDAP 已啟用](./media/active-directory-domain-services-admin-guide/secure-ldap-internet-access-enabled.png)

<br>

## <a name="task-5---configure-dns-to-access-the-managed-domain-from-the-internet"></a>工作 5 - 設定 DNS 以從網際網路存取受管理的網域
**選擇性工作** - 如果您不打算使用 LDAPS 來透過網際網路存取受管理的網域，請略過這項設定工作。

開始這項工作之前，請先確定您已完成 [工作 4](#task-4---enable-secure-ldap-access-over-the-internet)中所述的步驟。

為受管理的網域啟用了透過網際網路的安全 LDAP 存取後，您需要更新 DNS 以便用戶端電腦可以找到此受管理網域。 在工作 4 的最後階段，[設定] 索引標籤的 [LDAPS 存取的外部 IP 位址] 中會顯示外部 IP 位址。

請設定外部 DNS 提供者，讓受管理網域的 DNS 名稱 (例如 'ldaps.contoso100.com') 指向這個外部 IP 位址。 在我們的範例中，我們需要建立下列 DNS 項目︰

    ldaps.contoso100.com  -> 52.165.38.113

這樣就大功告成了。您現在已準備好可使用安全 LDAP 透過網際網路連線到受管理網域。

> [!WARNING]
> 請記住，用戶端電腦必須信任 LDAPS 憑證的簽發者，才能成功使用 LDAPS 連線到受管理網域。 如果您使用企業憑證授權單位或公開的受信任憑證授權單位，您不需要採取任何動作，因為用戶端電腦會信任這些憑證簽發者。 如果您使用自我簽署憑證，則必須在用戶端電腦的受信任憑證存放區中安裝自我簽署憑證的公開部分。
>
>

<br>

## <a name="related-content"></a>相關內容
* [Azure AD Domain Services - 入門指南](active-directory-ds-getting-started.md)
* [Administer an Azure AD Domain Services managed domain (管理 Azure AD 網域服務受管理的網域)](active-directory-ds-admin-guide-administer-domain.md)
* [管理 Azure AD Domain Services 受管理網域上的群組原則](active-directory-ds-admin-guide-administer-group-policy.md)

