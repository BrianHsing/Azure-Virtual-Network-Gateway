# 使用 iPerf 驗證網路輸送量
在實作 Azure VPN Tunnel 時，常常會遇到客戶想透過 SMB Potocol 拖拉檔案來測試 VPN 的傳輸速率。其實這種方法是不太正確的，因為 SMB Potocol 傳輸
一次只會使用一個 Connection 傳輸，假設 VPN Tunnel 實際的傳輸速率為 30 Mbps，這樣您只會有 3.75 MB/s 的傳輸速率，但是如果是使用 FTP 等續傳軟體，
使用多執行序傳輸，那這樣傳輸速率就會好非常多，可以透過以下的練習來體驗一下。<br><br>
會影響
iPerf 會產生從一端到另一端自我產生的 TCP 流量。根據用來測量用戶端和伺服器節點之間可用頻寬的實驗來產生統計資料。
在兩個節點之間進行測試時，一個節點會當做伺服器，而另一個節點則是做為用戶端。 
此測試完成之後，建議您反轉節點的角色，以測試兩個節點上的上傳和下載輸送量，強烈建議非尖峰時段執行此驗證比較準確。<br>

## 環境說明
 
 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/blob/master/Validate-Throughput-iPerf/image/lab.PNG "lab")<br>
 - 使用現有的 Azure S2S Tunnel，請參考[實作 Azure 與 Fortigate 60E 的 S2S 連線](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/tree/master/S2S/Fortigate) <br>
 - 使用 Azure VM (172.16.1.4)與 On Premises VM (192.168.1.112)驗證網路輸送量