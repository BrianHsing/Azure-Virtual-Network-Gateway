# 使用 iPerf 驗證網路輸送量
在實作 Azure VPN Tunnel 時，常常會遇到客戶想透過 SMB Potocol 拖拉檔案來測試 VPN 的傳輸速率。其實這種方法是不太正確的，因為 SMB Potocol 傳輸
一次只會使用一個 Connection 傳輸，假設 VPN Tunnel 實際的傳輸速率為 30 Mbps，這樣您只會有 3.75 MB/s 的傳輸速率，但是如果是使用 FTP 等續傳軟體，
使用多執行序傳輸，那這樣傳輸速率就會好非常多，可以透過以下的練習來體驗一下。<br><br>

iPerf 會產生從一端到另一端自我產生的 TCP 流量。根據用來測量用戶端和伺服器節點之間可用頻寬的實驗來產生統計資料。
在兩個節點之間進行測試時，一個節點會當做伺服器，而另一個節點則是做為用戶端。 
此測試完成之後，建議您反轉節點的角色，以測試兩個節點上的上傳和下載輸送量，強烈建議非尖峰時段執行此驗證比較準確。<br>

## 環境說明
 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/Validate-Throughput-iPerf/image/lab.PNG "lab")<br>
 - 使用現有的 Azure S2S Tunnel，請參考[實作 Azure 與 Fortigate 60E 的 S2S 連線](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/tree/master/S2S/Fortigate) <br>
 - 使用 Azure Windows VM (172.16.1.4)與 On Premises Windows VM (192.168.1.112)驗證網路輸送量
 
## 設定 iPerf 

 - 伺服器端(Azure VM)<br>
	- 下載 https://iperf.fr/iperf-download.php，並解壓縮至 C:\  <br>
	![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/Validate-Throughput-iPerf/image/lab.PNG "lab")<br>
	![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/Validate-Throughput-iPerf/image/iperfset1.PNG "iperfset1")<br>
	![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/Validate-Throughput-iPerf/image/iperfset2.PNG "iperfset2")<br>
	- 啟用連接埠 5001 的防火牆例外狀況，因為稍後也要測試透過 Internet 傳輸速率，所以也記得要將網路安全性群組輸入規則加入一筆 5001 的規則<br>
	`netsh advfirewall firewall add rule name="Open Port 5001" dir=in action=allow protocol=TCP localport=5001`<br>
	![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/Validate-Throughput-iPerf/image/iperfset3.PNG "iperfset3")<br>
	- 執行命令提示字元，執行iPerf，並將它設定為在埠5001上接聽，完成<br>
	![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/Validate-Throughput-iPerf/image/iperfset4.PNG "iperfset4")<br>
 - 用戶端 (On Premises VM)<br>
 	- 下載 https://iperf.fr/iperf-download.php，並解壓縮至 C:\，圖示說明請參閱伺服器端<br>
	- 執行命令提示字元，執行iPerf<br>
	`iperf3.exe -c <IP of the iperf Server> -t 30 -p 5001 -P 32`<br>
		- -t 指定傳輸測試持續時間<br>
		- -p 指定連接埠<br>
		- -P 指定連線數量<br>
		- -u 指定UDP傳輸協定<br>
		- -R 反向傳輸<br>
		- -4 IPv4<br>
		- -6 IPv6<br>
		- --logfile 儲存測試結果<br>
	![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/Validate-Throughput-iPerf/image/iperfset5.png "iperfset5")<br>
 - 網路輸送量交叉比較 <br>
	- 單一 Connection 的速率<br>
	![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/Validate-Throughput-iPerf/image/iperfset6.png "iperfset6")<br>
	- 32 個 Connection 的速率 <br>
 	![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/Validate-Throughput-iPerf/image/iperfset7.png "iperfset7")<br>
	由上面的比較圖來看，多個 Connection 的速率遠優於單一 Connection，所以非常不適合使用大檔案利用 SMB Potocol 拖拉測試速率。如果這邊測試出來的速率不滿意，那這樣傳輸速率就會好非常多，可以透過以下的練習來體驗一下。
	可能需要檢查 VPN 設備的 phase 2 演算法、ISP 線路穩定度與實際速率、 Azure VM Size 的頻寬限制的方向來查找。<br>

**參考來源與更詳細的說明**<br>
https://docs.microsoft.com/zh-tw/azure/vpn-gateway/vpn-gateway-validate-throughput-to-vnet<br>