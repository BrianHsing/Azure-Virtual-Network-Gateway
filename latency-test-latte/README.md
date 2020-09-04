# 使用 Latte 進行延遲測試
Latte 是 Microsoft 用於測量網路延遲的工具，不同於其他 ICMP Potocol 的測量工具(例如 Ping)，Latte 是使用 TCP、
UDP Potocol 測量，由於實際應用程式均使用 TCP、UDP 溝通與交換資料，用 Latte 測量會比較符合真實情況的網路延遲時間。<br><br>

在這個練習中，主要會做幾個比對，分別是**Internet**、**S2S VPN**、**Local Network**的實際網路延遲時間的比較<br>

## 環境說明
 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/Validate-Throughput-iPerf/image/lab.PNG "lab")<br>
 - 使用現有的 Azure S2S Tunnel，請參考[實作 Azure 與 Fortigate 60E 的 S2S 連線](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/tree/master/S2S/Fortigate) <br>
 - 使用 Azure Windows VM (172.16.1.4)與 On Premises Windows VM (192.168.1.112)驗證網路輸送量<br>
 - 使用 On Premises Windows VM (192.168.1.111)與 On Premises Windows VM (192.168.1.112)驗證網路輸送量<br>

## 設定 Latte.exe