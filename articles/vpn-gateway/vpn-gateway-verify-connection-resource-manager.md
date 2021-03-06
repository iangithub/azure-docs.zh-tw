---
title: "確認 VPN 閘道連線 | Microsoft Docs"
description: "本文說明如何確認虛擬網路「VPN 閘道」連線。"
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: timlt
editor: 
tags: azure-service-management,azure-resource-manager
ms.assetid: 7e3d1043-caa9-4472-96d3-832f4e2c91ee
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/30/2017
ms.author: cherylmc
translationtype: Human Translation
ms.sourcegitcommit: b5bad71095e4b7e3b26df15780467526200ffa10
ms.openlocfilehash: 68d94a6402b1497f65c4d03fb987ba800e86c2a3


---
# <a name="verify-a-vpn-gateway-connection"></a>確認 VPN 閘道連線
您可以使用入口網站和 PowerShell 來確認虛擬網路「VPN 閘道」連線。 本文包含同時適用於 Resource Manager 和傳統部署模型的步驟。

## <a name="verify-using-the-azure-portal"></a>使用 Azure 入口網站進行確認

[!INCLUDE [Azure portal](../../includes/vpn-gateway-verify-connection-portal-rm-include.md)]

## <a name="verify-using-powershell"></a>使用 PowerShell 進行確認

若要使用 PowerShell 進行確認，請安裝最新版的 Azure Resource Manager PowerShell Cmdlet。 如需有關安裝 PowerShell Cmdlet 的資訊，請參閱[如何安裝和設定 Azure PowerShell](/powershell/azureps-cmdlets-docs)。 如需使用 Resource Manager Cmdlet 的詳細資訊，請參閱 [搭配使用 Windows PowerShell 與 Resource Manager](../powershell-azure-resource-manager.md)。

### <a name="log-in-to-your-azure-account"></a>登入您的 Azure 帳戶
1. 以提高的權限開啟 PowerShell 主控台並連接到您的帳戶。
   
        Login-AzureRmAccount
2. 檢查帳戶的訂用帳戶。
   
        Get-AzureRmSubscription 
3. 指定您要使用的訂用帳戶。
   
        Select-AzureRmSubscription -SubscriptionName "Replace_with_your_subscription_name"

### <a name="verify-your-connection"></a>確認您的連接

[!INCLUDE [Powershell](../../includes/vpn-gateway-verify-connection-ps-rm-include.md)]

## <a name="verify-using-the-azure-portal-classic"></a>使用 Azure 入口網站進行確認 (傳統)
[!INCLUDE [Azure portal](../../includes/vpn-gateway-verify-connection-azureportal-classic-include.md)]


## <a name="verify-using-powershell-classic"></a>使用 PowerShell 進行確認 (傳統)
若要使用 PowerShell 進行確認，請安裝最新版的 Azure PowerShell Cmdlet。 請務必將 Resource Manager 和「服務管理」(SM) 版本都下載並安裝。 如需有關安裝 PowerShell Cmdlet 的資訊，請參閱[如何安裝和設定 Azure PowerShell](/powershell/azureps-cmdlets-docs)。 

### <a name="log-in-to-your-azure-account"></a>登入您的 Azure 帳戶
1. 以提高的權限開啟 PowerShell 主控台並連接到您的帳戶。
   
        Login-AzureRmAccount
2. 檢查帳戶的訂用帳戶。
   
        Get-AzureRmSubscription 
3. 指定您要使用的訂用帳戶。
   
        Select-AzureRmSubscription -SubscriptionName "Replace_with_your_subscription_name"
4. 登入以使用適用於傳統部署模型的「服務管理」Cmdlet。

        Add-AzureAccount

### <a name="verify-your-connection"></a>確認您的連接
[!INCLUDE [Classic PowerShell](../../includes/vpn-gateway-verify-connection-ps-classic-include.md)]

## <a name="next-steps"></a>後續步驟
* 您可以將虛擬機器加入您的虛擬網路。 請參閱 [建立網站的虛擬機器](../virtual-machines/virtual-machines-windows-hero-tutorial.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) 以取得相關步驟。




<!--HONumber=Jan17_HO5-->


