# 實作 P2S VPN 用戶端全通道從內部部署連出網際網路
本篇使用 IKEv2 搭配自我簽署憑證實作 P2S VPN 用戶端全通道連線，在開始之前您需要先了解遇到的挑戰：<br>
- S2S 必須是連接狀態，並且已啟用強制通道，作法可以參考[實作 Azure 與 Fortigate 60E 的 S2S VPN Forced Tunneling](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/tree/master/forced-tunneling)<br>
- IKEv2 基於 Policy Base，所以路由不會動態新增，您必須在 P2S VPN 用戶端手動增加路由<br>

## 環境描述
在建立 P2S VPN 時，您需要了解內部部署與 Azure 的網路環境與相關配置，常見的蒐集資訊如下。<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/P2S/customroutelab.png "customroutelab")<br>
 - On Premises 環境
	- Fortigate 60E v6.2.2 build1010，請確認您的 VPN 設備有出現在設備驗證清單中 <br>
	  https://docs.microsoft.com/zh-tw/azure/vpn-gateway/vpn-gateway-about-vpn-devices<br>
	- Public IP : 114.32.191.\*\*\*，**您的會與本篇顯示的不同**<br>
	- VLAN(Subnet) ： 192.168.1.0/24<br>
	- **請注意 Static Route 必須要包含 P2S 位址集區**<br>
	- **確認您設備可以直接連出網際網路**<br>
	- **確認 VPN Tunnel 已啟用 NAT**<br>
 - Azure 虛擬網路環境<br>
	- 虛擬網路名稱 ： VNet<br>
	- 位置空間 ： 10.200.0.0/22<br>
	- 閘道公用 IP 位址：40.83.96.\*\*\*，**您建立的會與本篇顯示的不同**<br>
	- 使用者 VPN 設定位址集區：10.200.4.0/24<br>
	- **確認已經將區域網路閘道啟用強制通道**<br>


## 建立自我簽署憑證
本系列使用 MakeCert 來產生用戶端憑證，如需要了解細節步驟，請參考[使用 MakeCert 來產生並匯出點對站連線的憑證](https://docs.microsoft.com/zh-tw/azure/vpn-gateway/vpn-gateway-certificates-point-to-site-makecert)。<br>
本篇會使用做好的批次檔，快速的產生所需的用戶端憑證，並且可以利用匯出的用戶端憑證個別安裝或透過 GPO 散發。<br>
 - 首先請在您的用戶端先下載工具[點對站憑證批次檔](https://mega.nz/file/pZM03DAY#dGmUTw6EbyFYgQfkXZwS6A5vrNbIx08QsD2mi1B8qks)<br>
 - 解完壓縮您會看到 install Cert 資料夾，其中包含幾個檔案：<br>
   - outputCert：匯出的公開金鑰、用戶端憑證所儲存的資料夾<br>
   - CertMgr.exe：憑證管理工具<br>
   - ClientInstall.bat：在其他用戶端上幫助您快速的將用戶端憑證匯入<br>
   - makecert.exe：使用 MakeCert 來建立自我簽署憑證<br>
   - 使用方法.txt：簡易敘述使用方法<br>
 - 執行 rootInstall.bat<br>
 - 進入 outputCert 資料夾，選擇 updateToAzureCert.cer 按下右鍵選擇記事本開啟，複製公開金鑰，此公開金鑰會在稍後的使用者 VPN 設定中填入<br>
## 虛擬網路閘道設定使用者 VPN 設定
 - 選擇您的虛擬網路閘道，再設定分類中選擇使用者 VPN 設定<br>
 - 位址集區填入：10.200.4.0/24<br>
 - 驗證類型選擇 Azure 憑證<br>
 - 根憑證名稱您可以自行定義，公開憑證資料請填入您剛剛複製的公開金鑰，點選儲存<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/P2S/customroute1.png "customroute1")<br>
 - 儲存後，請點選下載 VPN 用戶端，您會看到下載一個與您虛擬網路閘道相同名字的壓縮檔，以 64 位元系統為例，請選擇 WindowsAmd64 資料夾解壓縮，點選 VpnClientSetupAmd64.exe 安裝<br>
 - 使用 VPN 用戶端連線後，您可以檢查路由表，可以看到 10.200.0.0/22 都會路由至 10.200.4.2，但並沒有將 192.168.1.0/24 路由至 10.200.4.2，您必須再做一些設定才能完成<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/P2S/customroute2.png "customroute2")<br>
 - 確認網際網路連出的 IP 位址<br>

## 使用 On Premises 的位址連出網際網路

- 使用 route add 將路由加入至網路介面<br>
   您可以使用您熟悉的方法，直接將路由加入到現有的路由表中，但每次連線時您必須重新加入，或著您也可以使用持續路由<br>
   - 開啟命令提示字元，新增持續路由`route add 0.0.0.0 mask 0.0.0.0 10.200.4.x -p`<br>
 - 確認路由有加入至路由表<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/P2S/fulltunnelroute1.png "fulltunnelroute1")<br>
 - 確認可以 Firewall forward traffic log 看到流量<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/P2S/fulltunnelroute2.png "fulltunnelroute2")<br>