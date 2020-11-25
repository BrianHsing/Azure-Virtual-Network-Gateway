# 實作 P2S VPN 用戶端全通道至內部部署
本篇使用 實作 P2S VPN 用戶端自訂至內部部署，在開始之前您需要先了解遇到的挑戰：<br>
- S2S 必須是連接狀態，並且已啟用強制通道，作法可以參考[實作 Azure 與 Fortigate 60E 的 S2S VPN Forced Tunneling](https://github.com/BrianHsing/Azure-Virtual-Network-Gateway/tree/master/forced-tunneling)<br>
- IKEv2 基於 Policy Base，所以路由不會動態新增，您必須在 P2S VPN 用戶端手動增加路由<br>

##