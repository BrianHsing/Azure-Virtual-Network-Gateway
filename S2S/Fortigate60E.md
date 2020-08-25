# 實作 Azure 與 Fortigate 60E 的 S2S VPN 連線
 本篇使用 Fortigate 60E 來實作與 Azure 建立單一 S2S VPN 連線通道，閱讀完此篇文章您將會：<br>
 - 使用 Azure 入口網站建立與設定虛擬網路、虛擬網路閘道、區域網路閘道<br>
 - 設定 Fortigate 60E IPsec Tunnels、Policy<br>
 - 驗證傳輸到虛擬網路的 VPN 輸送量<br>

## 環境說明
 - On Premises 環境
	- Fortigate 60E v6.2.2 build1010，請確認您的 VPN 設備有出現在設備驗證清單中 <br>
	  https://docs.microsoft.com/zh-tw/azure/vpn-gateway/vpn-gateway-about-vpn-devices<br>
	- Public IP : 
	- Subnet ： 192.168.1.0/24

 - Azure 環境
	- 虛擬網路名稱 ： VNet<br>
	- 位置空間 ： 172.16.0.0/16<br>
	- 子網路 ： WorkloadSubnet 172.16.1.0/24<br>
	- 閘道子網路 ： GatewaySubnet 172.16.10.0/27，請將範圍設為至少/27的空間位置<br>
	- 虛擬網路閘道名稱： VNetGW
	- 公用 IP 位址名稱： VNetGWPip
	- VPN 類型： 以路由為基礎 (路由 Route Base = 動態路由、原則 Policy Base = 靜態路由)
	- 區域網路閘道名稱： Site1
	- 連接名稱： VNet1toSite1

## Azure 入口網站
 - 建立虛擬網路 VNET<br>
	- 在 Azure 入口網站搜尋欄中搜尋虛擬網路，並點選
	![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/S2S/image/createvnet1.PNG "createvnet1")<br>
	- 點選「新增」，選擇您的訂用帳戶、資源群組、名稱輸入 VNET、區域選擇日本東部後，點選「下一步：IP 位址」<br>
	![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/S2S/image/createvnet3.PNG "createvnet3")<br>
	- IPv4 位址空間請輸入 172.16.0.0/16，建立兩個子網路，分別為 WorkloadSubnet 172.16.1.0/24、GatewaySubnet 172.16.10.0/27，請將 GatewaySubnet 範圍設為至少/27的空間位置。請按「檢閱 + 建立」，按下「建立」後完成。<br>
	![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/S2S/image/createvnet4.PNG "createvnet4")<br>
 - 建立虛擬網路閘道 VNETGW<br>
	- 在 Azure 入口網站搜尋欄中搜尋虛擬網路閘道，並點選
	![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/S2S/image/createvnetgw1.PNG "createvnetgw1")<br>
	- 點選「新增」，填入以下資訊，請按「檢閱 + 建立」，再次按下「建立」後完成。**需要大約 40 分鐘的部署時間**<br>
		- 名稱輸入 VNETGW<br>
		- 區域選擇日本東部<br>
		- 閘道類型選擇路由，(路由 Route Base = 動態路由、原則 Policy Base = 靜態路由)<br>
		- SKU選擇 VpnGW1，規格說明請參考 https://docs.microsoft.com/zh-tw/azure/vpn-gateway/vpn-gateway-about-vpngateways#gateway-skus-by-tunnel-connection-and-throughput<br>
		- 虛擬網路選擇 VNET<br>
		- 建立新的公用 IP 位置，輸入公用 IP 位址名稱 VNTGWPIP。**您只需要輸入名稱，就會自動幫您建立適合的公用 IP 位址**<br>
		- 停用主動模式<br>
		- 停用 BGP ASN<br>
		![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/S2S/image/createvnetgw2.PNG "createvnetgw2")<br>

## Fortigate 60E

## 驗證傳輸到虛擬網路的 VPN 輸送量

**參考來源與更詳細的說明**