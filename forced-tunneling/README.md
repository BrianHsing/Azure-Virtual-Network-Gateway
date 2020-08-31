# 實作 Azure 與 Fortigate 60E 的 S2S VPN Forced Tunneling
強制通道(Forced Tunneling)可以讓您透過站對站將所有網際網路流量傳回內部部署，以便方便進行稽核。如果沒使用強制通道，一律會從 Azure 網路基礎結構直接到網際網路，
無法使用企業熟悉的方式，您無法選擇檢查或稽核流量。要在現有的 S2S VPN 啟用強制通道，主要有幾項工作：<br>
 - 設定路由表(UDR)<br>
 - 使用 Powershell 命令設定 GatewayDefaultSite <br>
 - 啟用 Fortigate VPN Tunnel NAT<br>
 - 設定 Phase 2 Selector 0.0.0.0/24<br>
 - 啟用 Fortigate Policy NAT<br>
 
## 環境說明
 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/forced-tunneling/image/lab.PNG "lab")<br>
 - 使用現有的 Azure S2S Tunnel，請參考[實作 Azure 與 Fortigate 60E 的 S2S 連線](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/tree/master/S2S/Fortigate) <br>
 - 盤點會用到的資訊<br>
	- 虛擬網路閘道名稱：VNETGW<br>
	- 區域網路閘道：LOCALGW<br>
	- 子網路：WorkloadSubnet(172.16.1.0/24)<br>
## Azure 入口網站
 - 設定路由表(UDR)<br>
	- 在 Azure 入口網站搜尋欄中搜尋路由，並點選
	
	- 建立路由表需填入訂用帳戶、資源群組、區域、名稱、**傳播閘道路由請點選 Yes**
	
	-
	-
	-
	-
 - 使用 Powershell 命令設定 GatewayDefaultSite<br>
	- 啟用 CloudShell<br>
    - 輸入`Connect-AzAccount` 登入<br>
	- 使用以下命令設定 GatewayDefaultSite，這邊會需要填入兩個資源名稱虛擬網路閘道 VNETGW、區域網路閘道 LOCALGW<br>
	 ```
		$LocalGateway = Get-AzLocalNetworkGateway -Name "LOCALGW" -ResourceGroupName "F60E-S2S-Lab"
		$VirtualGateway = Get-AzVirtualNetworkGateway -Name "VPNGW" -ResourceGroupName "F60E-S2S-Lab"
		Set-AzVirtualNetworkGatewayDefaultSite -GatewayDefaultSite $LocalGateway -VirtualNetworkGateway $VirtualGateway
	 ```

## Fortigate 60E
 - 啟用 Fortigate VPN Tunnel NAT<br>
 - 設定 Phase 2 Selector 0.0.0.0/24<br>
 - 啟用 Fortigate Policy NAT<br>
 
## 啟用後測試

**參考來源與更詳細的說明**<br>
https://docs.microsoft.com/zh-tw/azure/vpn-gateway/vpn-gateway-forced-tunneling-rm