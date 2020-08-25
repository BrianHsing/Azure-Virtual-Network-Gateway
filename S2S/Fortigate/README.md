# 實作 Azure 與 Fortigate 60E 的 S2S VPN 連線
 本篇使用 Fortigate 60E 來實作與 Azure 建立單一 S2S VPN 連線通道，閱讀完此篇文章您將會：<br>
 - 使用 Azure 入口網站建立與設定虛擬網路、虛擬網路閘道、區域網路閘道<br>
 - 設定 Fortigate 60E IPsec Tunnels、Static Route、Policy、TCP MSS、MTC<br>

## 環境說明

 在建立 S2S VPN 期間，您需要了解內部部署與 Azure 的網路環境與相關配置，常見的蒐集資訊如下。

 - On Premises 環境
	- Fortigate 60E v6.2.2 build1010，請確認您的 VPN 設備有出現在設備驗證清單中 <br>
	  https://docs.microsoft.com/zh-tw/azure/vpn-gateway/vpn-gateway-about-vpn-devices<br>
	- Public IP : 114.32.191.\*\*\* <br>
	- VLAN(Subnet) ： 192.168.1.0/24<br>
	- Pre-shared Key ：20200825<br>

 - Azure 虛擬網路環境<br>
	- 虛擬網路名稱 ： VNet<br>
	- 位置空間 ： 172.16.0.0/16<br>
	- 子網路 ： WorkloadSubnet 172.16.1.0/24<br>
	- 閘道子網路 ： GatewaySubnet 172.16.10.0/27，請將範圍設為至少/27的空間位置<br>
	- 閘道公用 IP 位址：20.46.122.\*\*\*，**您建立的會與本篇顯示的不同**<br>
	- Pre-shared Key ：20200825<br>

## Azure 入口網站
 - 建立虛擬網路 VNET<br>
	- 在 Azure 入口網站搜尋欄中搜尋虛擬網路，並點選
	![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/S2S/Fortigate/image/createvnet1.PNG "createvnet1")<br>
	- 點選「新增」，選擇您的訂用帳戶、資源群組、名稱輸入 VNET、區域選擇日本東部後，點選「下一步：IP 位址」<br>
	![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/S2S/Fortigate/image/createvnet3.PNG "createvnet3")<br>
	- IPv4 位址空間請輸入 172.16.0.0/16，建立兩個子網路，分別為 WorkloadSubnet 172.16.1.0/24、GatewaySubnet 172.16.10.0/27，請將 GatewaySubnet 範圍設為至少/27的空間位置。請按「檢閱 + 建立」，按下「建立」後完成。<br>
	![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/S2S/Fortigate/image/createvnet4.PNG "createvnet4")<br>
 - 建立虛擬網路閘道 VNETGW<br>
	- 在 Azure 入口網站搜尋欄中搜尋虛擬網路閘道，並點選
	![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/S2S/Fortigate/image/createvnetgw1.PNG "createvnetgw1")<br>
	- 點選「新增」，填入以下資訊，請按「檢閱 + 建立」，再次按下「建立」後完成。**需要大約 40 分鐘的部署時間**<br>
		- 名稱輸入 VNETGW<br>
		- 區域選擇日本東部<br>
		- 閘道類型選擇路由，(路由 Route Base = 動態路由、原則 Policy Base = 靜態路由)<br>
		- SKU選擇 VpnGW1，規格說明請參考 https://docs.microsoft.com/zh-tw/azure/vpn-gateway/vpn-gateway-about-vpngateways#gateway-skus-by-tunnel-connection-and-throughput<br>
		- 虛擬網路選擇 VNET<br>
		- 建立新的公用 IP 位址，輸入公用 IP 位址名稱 VNTGWPIP。**您只需要輸入名稱，就會自動幫您建立適合的公用 IP 位址**<br>
		- 停用主動模式<br>
		- 停用 BGP ASN<br>
		![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/S2S/Fortigate/image/createvnetgw2.PNG "createvnetgw2")<br>
 - 建立區域網路閘道 LOCALGW<br>
	- 在 Azure 入口網站搜尋欄中搜尋區域網路閘道，並點選
	![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/S2S/Fortigate/image/createlocalnetgw1.PNG "createlocalnetgw1")<br>
	- 點選「新增」，填入以下資訊，請按「檢閱 + 建立」，再次按下「建立」後完成。<br>
		- 名稱輸入 LOCALGW<br>
		- IP 位址輸入 114.32.191.\*\*\*<br>
		- 位址空間請填入 192.168.1.0/24，**這邊填入的網段，代表您希望能從 Azure 虛擬網路中路由到的網段**<br>
		- 不勾選 BGP 設定<br>
		- 選擇對應的訂用帳戶、資源群組、位置後，點選「建立」<br>
		![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/S2S/Fortigate/image/createlocalnetgw2.PNG "createlocalnetgw2")<br>
 - 建立連線 connection
	- 點選虛擬網路閘道 VNETGW，選擇設定欄位下的連接，新增連線<br>
	![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/S2S/Fortigate/image/connetion1.png "connetion1")<br>
	- 輸入名稱、連接類型「站對站(IPsec)」、選擇區域網路閘道 LOCALGW、輸入共用金鑰(PSK)「20200825」、IKE通訊協定選擇「IKEv2」<br>
	![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/S2S/Fortigate/image/connetion2.PNG "connetion2")<br>

## Fortigate 60E
 - 建立 IPsec Tunnels
	- 決定您要使用哪一個公用 IP 與 Azure 建立 Tunnel，我這邊只有一個 IP 位址，所以選擇使用 WAN1 的 IP 位址<br>
	![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/S2S/Fortigate/image/fortigate2.png "fortigate2")<br>
	- 在左邊導覽列中，展開 VPN，選擇 IPsec Wizard，建立 IPsec Tunnels，輸入 Name 並選擇 Template Type Custom<br>
	![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/S2S/Fortigate/image/fortigate3.PNG "fortigate3")<br>
	- 在 Network 區塊中，填入虛擬網路閘道 VNETGW 的公用 IP 位址，並將 NAT Traversal Disable(如果未來想要啟用強制通道，則切換為 Enable)、Dead Peer Detection On Idle (系統將判定與遠端站台的連線是否斷線)<br>
	![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/S2S/Fortigate/image/fortigate4.png "fortigate4")<br>	
	- Pre-shared Key 與 Azure 連線設定一致，IKE請選擇 Version 2<br>
	![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/S2S/Fortigate/image/fortigate5.PNG "fortigate5")<br>	
	- Phase 1 為主要連線，Encryption 請設定 AES256、Authentication 請設定為 SHA1、Diffie-Hellman Group 請選擇 2 (密碼交換強度)、Key Lifetime 請設定 28800 (不影響傳輸速率)<br>
	![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/S2S/Fortigate/image/fortigate6.PNG "fortigate6")<br>	
	- Phase 2 為快速連線，Local Address 代表內部佈署的網段、Remote Address 代表 Azure 虛擬網路的子網路，Encryption 請選擇 AES256GCM、AES265，Authentication 請選擇 SHA1(Encryption、Authentication 會影響傳輸速率)，取消勾選 Enable Replay Detection(偵測已無回應之連線對象)、PFS(啟動此功能可能會略為影響效能，但可加強安全性)<br>
 	![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/S2S/Fortigate/image/fortigate7.PNG "fortigate7")<br>	
 - 設定 Static Route (靜態路由)，必須要讓設備知道 Azure 虛擬網路子網路該往哪送<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/S2S/Fortigate/image/fortigate8.PNG "fortigate8")<br>	
 - 設定 Policy (原則)，這裡設定 Azure 子網路到內部部署子網段的連線管理原則，**NAT 請關閉**<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/S2S/Fortigate/image/fortigate9.PNG "fortigate9")<br>	
 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/S2S/Fortigate/image/fortigate10.PNG "fortigate10")<br>	
 - TCP MSS 設定為 1350 (TCP資料封包每次能夠傳輸的最大資料分段)<br>
	- 設定方式是將剛剛新增的兩筆 Policy 透過命令列的方式設定 TCP MSS 1350<br>
	```
	config firewall policy 
		edit <policy-id> 
			set tcp-mss-sender 1350 
			set tcp-mss-receiver 1350 
		next 
	end
	```
	
	![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/S2S/Fortigate/image/fortigate11.PNG "fortigate11")<br>	
 - MTU 設定為 1400 (最大傳輸單元，此數值會影響傳輸效能)<br>
	- 找到 IPsec Tunnel 的 MTU <br>
	`fnsysctl ifconfig -a`<br>
	`config system interface edit <interface_name> set mtu-override enable set mtu 1400 end`<br>



**參考來源與更詳細的說明**
https://docs.microsoft.com/zh-tw/azure/vpn-gateway/vpn-gateway-about-vpn-devices <br>
https://docs.microsoft.com/zh-tw/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal<br>
https://docs.fortinet.com/document/fortigate/6.2.0/azure-cookbook/989216/connecting-a-local-fortigate-to-an-azure-vnet-vpn <br>
https://docs.fortinet.com/document/fortigate/6.0.0/cookbook/566499/configuring-the-fortigate <br>
