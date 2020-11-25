# 實作 P2S VPN 用戶端自訂路由至內部部署
本篇使用 實作 P2S VPN 用戶端自訂至內部部署，在開始之前您需要先了解遇到的挑戰：<br>
- S2S 必須是連接狀態，並且您的內部部署的設備靜態路由必須要包含 P2S 網段<br>
- IKEv2 基於 Policy Base，所以路由不會動態新增，您必須在 P2S VPN 用戶端手動增加路由，或著您也可以在虛擬網路匣道中自訂路由<br>
## 環境描述
在建立 P2S VPN 時，您需要了解內部部署與 Azure 的網路環境與相關配置，常見的蒐集資訊如下。<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/P2S/customroutelab.png "customroutelab")<br>
 - On Premises 環境
	- Fortigate 60E v6.2.2 build1010，請確認您的 VPN 設備有出現在設備驗證清單中 <br>
	  https://docs.microsoft.com/zh-tw/azure/vpn-gateway/vpn-gateway-about-vpn-devices<br>
	- Public IP : 114.32.191.\*\*\*，**您的會與本篇顯示的不同**<br>
	- VLAN(Subnet) ： 192.168.1.0/24<br>
	- **請注意 Static Route 必須要包含 P2S 位址集區**<br>
 - Azure 虛擬網路環境<br>
	- 虛擬網路名稱 ： VNet<br>
	- 位置空間 ： 10.200.0.0/22<br>
	- 閘道公用 IP 位址：40.83.96.\*\*\*，**您建立的會與本篇顯示的不同**<br>
	- 使用者 VPN 設定位址集區：10.200.4.0/24<br>
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
 - 確認是無法 Ping 到防火牆的內部 IP 192.168.1.99<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/P2S/customroute3.png "customroute3")<br>
 
## 新增 On-Premises 路由至您的 P2S VPN 用戶端
由於 IKEv2 是 Policy Base，所以您必須手動自訂路由至您的 P2S VPN 用戶端，主要會有 3 種方式，您可以選擇其中一種達成新增路由的目的<br>
 - 選項 1.公告 P2S VPN 用戶端的自訂路由<br>
   使用此方法好處是您不需要再 VPN 用戶端新增路由，但如果路由變更，您必須重新下載 VPN 用戶端，並重新安裝<br>
   - 啟用 CloudShell<br>
   - 輸入`Connect-AzAccount` 登入<br>
   - 輸入以下命令<br>
      ```
      $gw = Get-AzVirtualNetworkGateway -Name <name of gateway> -ResourceGroupName <name of resource group>
      Set-AzVirtualNetworkGateway -VirtualNetworkGateway $gw -CustomRoute 192.168.1.0/24
      ``` 
   - 查詢自訂路由，請輸入命令`$gw.CustomRoutes | Format-List`
   ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/P2S/customroute6.png "customroute6")<br> 
   - 移除 VPN 用戶端，重新下載 VPN 用戶端後，重新安裝後連線<br>

 - 選項 2.將路由加入至 VPN 設定檔 route.txt<br>
   使用此方法您必須已經安裝 VPN 用戶端，並且在更新 route.txt 後，要重新連線<br>
   - 找到路徑`%appdata%/Microsoft/Network/Connections/Cm//routes.txt`的 route.txt 檔案並開啟<br>
   - 加入一行`ADD 192.168.1.0 MASK 255.255.255.0 default METRIC default IF default`<br>
   - 重新連線 VPN 用戶端<br>
 - 選項 3.使用 route add 將路由加入至網路介面<br>
   您可以使用您熟悉的方法，直接將路由加入到現有的路由表中，但每次連線時您必須重新加入，或著您也可以使用持續路由<br>
   - 開啟命令提示字元，新增路由`route add 192.168.1.0 mask 255.255.255.0 10.200.4.2`、新增持續路由`route add 192.168.1.0 mask 255.255.255.0 10.200.4.2 -p`<br>
 - 確認路由有加入至路由表<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/P2S/customroute4.png "customroute4")<br>
 - 確認可以 Ping 到防火牆的內部 IP 192.168.1.99<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/P2S/customroute5.png "customroute5")<br>