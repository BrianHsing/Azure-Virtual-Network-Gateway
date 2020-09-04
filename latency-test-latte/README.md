# 使用 Latte 進行延遲測試
Latte 是 Microsoft 用於測量網路延遲的工具，不同於其他 ICMP Potocol 的測量工具(例如 Ping)，Latte 是使用 TCP、
UDP Potocol 測量，由於實際應用程式均使用 TCP、UDP 溝通與交換資料，用 Latte 測量會比較符合真實情況的網路延遲時間。
在這個練習中，主要會做幾個比對，分別是 **Internet**、**S2S VPN**、**Local Network** 的實際網路延遲時間的比較。<br>

## 環境說明
 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/latency-test-latte/image/lab.PNG "lab")<br>
 - 使用現有的 Azure S2S Tunnel，請參考[實作 Azure 與 Fortigate 60E 的 S2S 連線](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/tree/master/S2S/Fortigate) <br>
 - 使用 Azure Windows VM (172.16.1.4)與 On Premises Windows VM (192.168.1.112)測試網路延遲<br>
 - 使用 On Premises Windows VM (192.168.1.111)與 On Premises Windows VM (192.168.1.112)測試網路延遲<br>

## 設定 Latte.exe
 - 下載最新版本的 [latte.exe](https://gallery.technet.microsoft.com/Latte-The-Windows-tool-for-ac33093b) ，點選 Download latte.exe，跳出 License 視窗時，請點選 I agree<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/latency-test-latte/image/latte1.PNG "latte1")<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/latency-test-latte/image/latte2.PNG "latte2")<br>
 - 請建立 `C:\tools`，並將 latte.exe 放在此資料夾中<br>
 > **Tips.每個主機上都需要操作上述步驟** <br>
 - 允許 latte.exe 通過 Windows Defender 防火牆，開啟命令提示字元輸入下方命令<br>
 `netsh advfirewall firewall add rule program=c:\tools\latte.exe name="Latte" protocol=any dir=in action=allow enable=yes profile=ANY`
 - 執行延遲測試，主要會分接收者與傳送者<br>
	- 在擔任接收者角色的主機上，開啟命令提示字元，輸入以下命令，使用 65100 次疊代測試<br>
	`latte -a 172.16.1.4:5005 -i 65100`<br>
	 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/latency-test-latte/image/latte3.PNG "latte3")<br>
	-在擔任傳送者角色的主機上，開啟命令提示字元，輸入以下命令，使用 65100 次疊代測試<br>
	`latte -a 172.16.1.4:5005 -i 65100`<br>
	 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/latency-test-latte/image/latte4.PNG "latte4")<br>
 > **Tips.測試可能需要幾分鐘的時間才能完成。在執行較長的測試之前，請考慮以較少的反覆運算開始測試是否成功。** <br>

## 延遲比較

 - 使用 Azure Windows VM (172.16.1.4)與 On Premises Windows VM (192.168.1.112)測試網路延遲<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/latency-test-latte/image/latte.PNG "latte")<br>
 - 使用 On Premises Windows VM (192.168.1.111)與 On Premises Windows VM (192.168.1.112)測試網路延遲<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/latency-test-latte/image/latte0.PNG "latte0")<br>

**參考來源與更詳細的說明**
https://docs.microsoft.com/zh-tw/azure/virtual-network/virtual-network-test-latency