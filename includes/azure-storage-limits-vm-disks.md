Azure 虛擬機器支援附加資料磁碟數目。 為達最佳效能，您將需要連接至虛擬機器的磁碟數目限制，以避免可能的節流。 只要不是全部磁碟在同一時間高度使用，儲存體帳戶可支援更大的磁碟數目。

* **Azure 受控磁碟︰**訂用帳戶的受控磁碟計數限制為區域性限制。 每個訂用帳戶每個區域的預設軟性限制為 2,000。 若要提高限制，請連絡 Azure 支援服務。

    受空快照集和映像會計入受控磁碟限制。

* **標準儲存體帳戶：** 標準儲存體帳戶的總要求率上限為 20000 IOPS。 在標準儲存體帳戶中，所有虛擬機器磁碟的 IOPS 總數不得超過此限。
  
    您可以根據要求率的限制，大致計算單一標準儲存體帳戶可支援的高度使用磁碟數目。 例如，針對基本層 VM，高度使用的磁碟數目上限約為 66 (每一磁碟 20,000/300 IOPS)；而針對標準層 VM，約為 40 (每一磁碟 20,000/500 IOPS)，如下表所示。 
* **進階儲存體帳戶：** 進階儲存體帳戶的總輸送量速率上限為 50 Gbps。 所有 VM 磁碟的總輸送量不應該超過此限。

