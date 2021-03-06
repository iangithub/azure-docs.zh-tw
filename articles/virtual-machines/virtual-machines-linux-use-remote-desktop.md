---
title: "在 Azure 中使用遠端桌面連接至 Linux VM | Microsoft Docs"
description: "了解如何安裝和設定遠端桌面 (xrdp)，以使用圖形化工具在 Azure 中連接至 Linux VM"
services: virtual-machines-linux
documentationcenter: 
author: iainfoulds
manager: timlt
editor: 
ms.assetid: 
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 03/07/2017
ms.author: iainfou
translationtype: Human Translation
ms.sourcegitcommit: 356de369ec5409e8e6e51a286a20af70a9420193
ms.openlocfilehash: 1b8ad526ee59bd1547cbd7f4a208720fb6557fc2
ms.lasthandoff: 03/27/2017


---
# <a name="install-and-configure-remote-desktop-to-connect-to-a-linux-vm-in-azure"></a>在 Azure 中安裝和設定遠端桌面，以連接至 Linux VM
在 Azure 中的 Linux 虛擬機器 (VM) 通常是使用安全殼層 (SSH) 連接從命令列管理。 如果是 Linux 的新手，或者是快速疑難排解的案例，使用遠端桌面可能會比較容易。 本文將詳細說明如何使用 Resource Manager 部署模型為您的 Linux VM 安裝和設定桌面環境 ([xfce](https://www.xfce.org)) 和遠端桌面 ([xrdp](http://www.xrdp))。 您也可以[對使用傳統部署模型的 VM 執行這些步驟](linux/classic/remote-desktop.md?toc=%2fazure%2fvirtual-machines%2flinux%2fclassic%2ftoc.json)。


## <a name="prerequisites"></a>必要條件
這篇文章需要在 Azure 中有現有的 Linux VM。 如果您需要建立 VM，請使用下列其中一個方法︰

- [Azure CLI 2.0](virtual-machines-linux-quick-create-cli.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) 或 [Azure CLI 1.0](virtual-machines-linux-quick-create-cli-nodejs.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
- [Azure 入口網站](virtual-machines-linux-quick-create-portal.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

您也需要登入[有效的 Azure 帳戶](https://azure.microsoft.com/pricing/free-trial/)。


## <a name="quick-commands"></a>快速命令
如果您需要快速完成工作，下列章節詳細說明在您的 VM 上安裝和設定遠端桌面的基本命令。 每個步驟的詳細資訊和內容可在文件其他地方找到，從[這裡](#install-graphical-environment-on-linux-vm)開始。

下列範例會在 Ubuntu VM 上安裝輕量型 [xfce4](https://www.xfce.org/) 桌面環境。 其他的散發 套件和桌面環境會稍微不同。 例如，使用 **yum** 在 Red Hat Enterprise Linux 上安裝及設定適當的 **selinux** 規則，或使用 **zypper** 在 SUSE 上安裝。

SSH 連線到您的 VM。 安裝 xfce 桌面環境，如下所示︰

```bash
sudo apt-get update
sudo apt-get install xfce4
```

安裝 xrdp，如下所示︰

```bash
sudo apt-get install xrdp
```

設定 xrdp 以使用 xfce 做為您的桌面環境，如下所示︰

```bash
echo xfce4-session >~/.xsession
```

重新啟動 xrdp 服務：

```bash
sudo service xrdp restart
```

如果目前只使用 SSH 金鑰進行驗證，請設定您的使用者帳戶密碼︰

```bash
sudo passwd ops
```

結束 Linux VM 的 SSH 工作階段。 使用本機電腦上的 Azure CLI 建立網路安全性群組規則，以允許遠端桌面流量。 透過 Azure CLI 2.0 使用 [az network nsg rule create](/cli/azure/network/nsg/rule#create)。 下列範例會在 `myNetworkSecurityGroup` 中建立名為 `myNetworkSecurityGroupRule` 的規則以允許 TCP 通訊埠 3389 上的流量︰
    
```azurecli
az network nsg rule create --resource-group myResourceGroup \
    --nsg-name myNetworkSecurityGroup --name myNetworkSecurityGroupRule \
    --protocol tcp --direction inbound --priority 1010 \
    --source-address-prefix '*' --source-port-range '*' \
    --destination-address-prefix '*' --destination-port-range 3389 \
    --access allow
```

或使用 Azure CLI 1.0：

```azurecli
azure network nsg rule create --resource-group myResourceGroup \
    --nsg-name myNetworkSecurityGroup --name myNetworkSecurityGroupRule \
    --protocol tcp --direction inbound --priority 1010 \
    --destination-port-range 3389 --access allow
```

使用您所選擇的遠端桌面用戶端連接至 Linux VM。

![使用遠端桌面用戶端連接至 xrdp](./media/virtual-machines-linux-use-remote-desktop/remote-desktop-client.png)

## <a name="install-a-desktop-environment-on-your-linux-vm"></a>在 Linux VM 上安裝桌面環境
在 Azure 中大多數 Linux VM 預設沒有安裝桌面環境。 Linux VM 通常是使用 SSH 連接來管理，而不是使用桌面環境來管理。 在 Linux 中您有各種不同的桌面環境可以選擇。 根據您選擇的桌面環境，可能會使用 1 到 2 GB 的磁碟空間，並且需要 5 到 10 分鐘，才能安裝及設定所有必要的套件。

下列範例會在 Ubuntu VM 上安裝輕量型 [xfce4](https://www.xfce.org/) 桌面環境。 對於其他散發的命令稍微不同 (例如，使用 **yum** 以在 Red Hat Enterprise Linux 上安裝及設定適當的 **selinux** 規則，或使用 **zypper** 在 SUSE 上安裝)。

首先，SSH 連接至您的 VM。 下列範例會連接至名為 `myvm.westus.cloudapp.azure.com` 且具有使用者名稱 `ops` 的 VM：

```bash
ssh ops@myvm.westus.cloudapp.azure.com ~/.ssh/id_rsa.pub
```

如果您使用 Windows，並且需要使用 SSH 的詳細資訊，請參閱[如何使用 SSH 金鑰與 Windows](virtual-machines-linux-ssh-from-windows.md)。

接下來，使用 `apt` 安裝 xfce，如下所示︰

```bash
sudo apt-get update
sudo apt-get install xfce4
```

## <a name="install-and-configure-a-remote-desktop-server"></a>安裝及設定遠端桌面伺服器
現在您已安裝桌面環境，設定遠端桌面服務來接聽連入連線。 [xrdp](http://xrdp.org) 是開放原始碼遠端桌面通訊協定 (RDP) 伺服器，適用於大部分的 Linux 散發套件，而且很適合處理 xfce。 在您的 Ubuntu VM 上安裝 xrdp，如下所示：

```bash
sudo apt-get install xrdp
```

告訴 xrdp 當您啟動工作階段時要使用的桌面環境。 設定 xrdp 以使用 xfce 做為您的桌面環境，如下所示︰

```bash
echo xfce4-session >~/.xsession
```

重新啟動 xrdp 服務，讓變更生效，如下所示︰

```bash
sudo service xrdp restart
```


## <a name="set-a-local-user-account-password"></a>設定本機使用者帳戶密碼
如果您在建立 VM 時已為您的使用者帳戶建立密碼，請略過此步驟。 如果您只使用 SSH 金鑰驗證，並且尚未設定本機帳戶密碼，請先指定密碼，再使用 xrdp 登入您的 VM。 xrdp 無法接受 SSH 金鑰進行驗證。 下列範例會指定使用者帳戶 `ops` 的密碼：

```bash
sudo passwd ops
```

> [!NOTE]
> 如果目前不允許密碼登入，則指定密碼不會將 sshd 組態更新為允許密碼登入。 從安全性的觀點而言，您可能想要使用金鑰型驗證的 SSH 通道連接至 VM，然後連接到 xrdp。 如果是這樣，請略過下一個步驟 (建立網路安全性群組規則) 以允許遠端桌面流量。


## <a name="create-a-network-security-group-rule-for-remote-desktop-traffic"></a>建立遠端桌面流量的網路安全性群組規則
若要允許遠端桌面流量觸達您的 Linux VM，必須建立網路安全性群組規則，允許連接埠 3389 上的 TCP 觸達您的 VM。 如需有關網路安全性群組規則的詳細資訊，請參閱[什麼是網路安全性群組？](../virtual-network/virtual-networks-nsg.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) 您也可以[使用 Azure 入口網站建立網路安全性群組規則](virtual-machines-windows-nsg-quickstart-portal.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

下列範例會對 `tcp` 連接埠 `3389` 上的流量，將名為 `myNetworkSecurityGroupRule` 的網路安全性群組規則建立為 `allow`。

- 透過 Azure CLI 2.0 使用 [az network nsg rule create](/cli/azure/network/nsg/rule#create)：
    
    ```azurecli
    az network nsg rule create --resource-group myResourceGroup \
        --nsg-name myNetworkSecurityGroup --name myNetworkSecurityGroupRule \
        --protocol tcp --direction inbound --priority 1010 \
        --source-address-prefix '*' --source-port-range '*' \
        --destination-address-prefix '*' --destination-port-range 3389 \
        --access allow
    ```

- 或使用 Azure CLI 1.0：

    ```azurecli
    azure network nsg rule create --resource-group myResourceGroup \
        --nsg-name myNetworkSecurityGroup --name myNetworkSecurityGroupRule \
        --protocol tcp --direction inbound --priority 1010 \
        --destination-port-range 3389 --access allow
    ```

## <a name="connect-your-linux-vm-with-a-remote-desktop-client"></a>連接 Linux VM 與遠端桌面用戶端
開啟您的本機遠端桌面用戶端，並連接至 Linux VM 的 IP 位址或 DNS 名稱。 在您的 VM 上輸入使用者帳戶的使用者名稱和密碼，如下所示︰

![使用遠端桌面用戶端連接至 xrdp](./media/virtual-machines-linux-use-remote-desktop/remote-desktop-client.png)

驗證之後，會載入 xfce 桌面環境，看起來類似下列範例︰

![透過 xrdp 的 xfce 桌面環境](./media/virtual-machines-linux-use-remote-desktop/xfce-desktop-environment.png)


## <a name="troubleshoot"></a>疑難排解
如果無法使用遠端桌面用戶端連接至 Linux VM，請在 Linux VM 上使用 `netstat`，以檢查您的 VM 是否正在接聽 RDP 連線，如下所示︰

```bash
sudo netstat -plnt | grep rdp
```

下列範例顯示 VM 如預期地接聽 TCP 連接埠 3389：

```bash
tcp     0     0      127.0.0.1:3350     0.0.0.0:*     LISTEN     53192/xrdp-sesman
tcp     0     0      0.0.0.0:3389       0.0.0.0:*     LISTEN     53188/xrdp
```

如果未接聽 xrdp 服務，在 Ubuntu VM 上重新啟動服務，如下所示︰

```bash
sudo service xrdp restart
```

在您的 Ubuntu VM 上檢閱 `/var/log` 中的記錄檔，以取得為何服務沒有回應的指示。 您也可以在遠端桌面連線嘗試期間監視 syslog，以檢視任何錯誤：

```bash
tail -f /var/log/syslog
```

Red Hat Enterprise Linux 和 SUSE 等其他 Linux 散發重新啟動服務的方式可能有所不同，要檢閱的記錄檔位置也不同。

如果您未在遠端桌面用戶端收到任何回應，而且在系統記錄檔中看不到任何事件，這種行為表示遠端桌面流量無法觸達 VM。 檢閱網路安全性群組規則，以確保您擁有規則以允許連接埠 3389 上的 TCP。 如需詳細資訊，請參閱[針對應用程式連線能力問題進行疑難排解](virtual-machines-linux-troubleshoot-app-connection.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。


## <a name="next-steps"></a>後續步驟
如需有關建立和使用 SSH 金鑰與 Linux VM 的詳細資訊，請參閱[在 Azure 中為 Linux VM 建立 SSH 金鑰](virtual-machines-linux-mac-create-ssh-keys.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

如需從 Windows 使用 SSH 的詳細資訊，請參閱[如何以 Windows 使用 SSH 金鑰](virtual-machines-linux-ssh-from-windows.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。


