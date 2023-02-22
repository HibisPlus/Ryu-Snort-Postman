# Ryu-Snort-Postman
**使用工具：**

   1. Ubuntu 20.04.5 LTS
   2. Mininet 2.3.1b1
   3. Snort 2.9.16
   4. Ryu 4.34
   5. Open vSwitch(OpenFlow 1.3) 2.13.8

Mininet
===========

   1. 拓樸建立：
    
     sudo mn --topo single,3 --mac --switch ovsk --controller remote --nat

  topo：單一Switch，連接3台Host。
  
  mac：照順序賦予Host mac位置。
  
  switch ovsk：提升傳輸效能。
  
  controller remote：連接遠端controller，預設為127.0.0.1。
  
  nat：讓Mininet連接外網，使Mininet可以發出DNS請求。


  2. Switch相關指令
  
    ovs-vsctl show
    
 此指令可以顯示各節點連接狀況
 
    ovs-dpctl show
    
 此指令可以顯示 port 連接狀況
 
    ovs-vsctl set Bridge s1 protocols=OpenFlow13
 
 此指令在設定 OpenFlow 協定版本為 1.3
 
    ovs-ofctl -O OpenFlow13 dump-flows s1
 
 此指令在查看 s1 的 Flow table
 
Ryu Controller 配置
==========
   
    sudo Ryu-manager simple_switch_snort.py dns.py

為了使Snort利用unsock模式與Ryu Controller連接，在simple_switch_snort.py中，需進行下列更動：

    socket_config = { 'unixsock' : True}
    # True : unixsock domain socket server
    # False : network socket server

Snort
==========

  1. Snort規則
  
    alert udp $EXTERNAL_NET 53 <> $HOME_NET any (msg:"WannaCry DNS Query"; content:"03 77 77 77 29 69 75 71 65 72|"; offset:36; depth:59; sid:1000024; rev:01)
 
  2. 啟動 Snort
  
分為兩個階段進行：

 一、 檢查規則是否正常
 
    sudo snort -i s1-eth1 -c /etc/snort/conf -A console
 
 二、 與 Ryu Controller連接 (目前無法正常運行)
 
    sudo snort -i s1-eth1 -c /etc/snort/snort.conf -A unsock
 
 
 Postman
 ==========
 
 1.查看連接到的switch 
 
 ![image](https://github.com/HibisPlus/Ryu-Snort-Postman/blob/main/Image/get-switch.jpg)

 2.取得switch的狀態
 
 ![image](https://github.com/HibisPlus/Ryu-Snort-Postman/blob/main/Image/flow-tablestate.jpg)
 
 3.新增flow規則
 
 ![image](https://github.com/HibisPlus/Ryu-Snort-Postman/blob/main/Image/flow-rule.jpg)
  
 4.更新後的flow table
 
 ![image](https://github.com/HibisPlus/Ryu-Snort-Postman/blob/main/Image/new-table.jpg)
 
 
現在進度：
============
 
1. 發現問題：https://stackoverflow.com/questions/71202679/how-to-drop-all-snort-alarts-that-intergration-with-ryu-controller

從上述網址中可發現，在Snort與Ryu測試接通時的顯示是有問題的，可能因此導致後續正式連接時Ryu無法正常顯示。

2. 細微更動：https://www.cnblogs.com/dream397/p/13150677.html

由上述網址中，大致操作相同，但有些細部設定是學生沒有做過的，因此打算由此網站的操作，一步一步進行，期望發現問題所在。
 
 
 
 
 
 
 
 
 
 
 
 
