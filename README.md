# Ryu-Snort-Postman
**使用工具：**

   1. Ubuntu 20.04.5 LTS
   2. Mininet 2.3.1b1
   3. Snort 2.9.16
   4. Ryu 4.34
   5. Open vSwitch(OpenFlow 1.3) 2.13.8

Mininet
===========

安裝：

      sudo apt-get update
      
      sudo apt-get upgrade
      
      reboot
      
      sudo apt-get install git
      
      sudo git https://github.com/mininet/mininet
      
      cd mininet/util/
      
      ./install.sh -a


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

安裝

      sudo apt-get install python3-pip
      
      sudo apt-get install python3-setuptools
      
      sudo pip3 install ryu
      
      sudo git clone https://github.com/osrg/ryu.git
      
      cd ryu/
      
      sudo python3 ./setup.py install
      
      sudo pip3 install -r tools/pip-requires

啟動ryu：

    sudo Ryu-manager simple_switch_snort.py dns.py

為了使Snort利用unsock模式與Ryu Controller連接，在simple_switch_snort.py中，需進行下列更動：

    socket_config = { 'unixsock' : True}
    # True : unixsock domain socket server
    # False : network socket server

Snort
==========

安裝：

  1. 安裝需要的插件

      sudo apt-get install build-essential libpcap-dev libpcre3-dev libnet1-dev zlib1g-dev luajit hwloc libdnet-dev libdumbnet-dev bison flex liblzma-dev openssl libssl-dev pkg-config libhwloc-dev cmake cpputest libsqlite3-dev uuid-dev libcmocka-dev libnetfilter-queue-dev libmnl-dev autotools-dev libluajit-5.1-dev libunwind-dev
      
  2. 安裝daq-2.0.6與snort-2.9.16 (下載壓縮檔至ubuntu中)
  
解壓縮後至daq-2.0.6資料夾輸入下列指令：
      
      ./configure
      
      make
      
      sudo make install
      
安裝完daq後至snort-2.9.16資料夾輸入同樣指令：

      ./configure
      
      make
      
      sudo make install
  
  

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
 
 
2/22：
============
 
1. 發現問題：https://stackoverflow.com/questions/75058364/ryu-snort-integration-ryu-snortlib-py-event-alert-returns-bytes-instead-of-stri

從上述網址中可發現，在Snort與Ryu測試接通時的顯示是有問題的，可能因此導致後續正式連接時Ryu無法正常顯示。

2. 細微更動：https://www.cnblogs.com/dream397/p/13150677.html


2/24：
============
1. 透過2/22發現的問題中，指出會在simple_switch_snort.py檔中修改一段print，程式碼如下：
  
       69  def _dump_alert(self, ev):
       70    msg = ev.msg
       71
       72    print('alertmsg: %s' % ''.join(msg.alertmsg))
       73
       74    self.packet_print(msg.pkt)

而修改內容為在第72行中 ''.join 前加入b，如下：

       72    print('alertmsg: %s' % b''.join(msg.alertmsg))
       
但最終結果會如網址中所呈現的問題，無法正確呈現出網址中的alert，目前正著手於解決此問題。

2. ovs安裝遭遇問題：

前面在進行配置時尚未依據mininet的環境去架設，導致mininet開啟後配置不正確，所以進行第一次的虛機重灌，在install ryu的時候有出錯，嘗試修正錯誤的時候發現沒有用，於是跟澤瑋借用CDX的機器做測試發現可行，於是進行第二次重灌虛機，已成功安裝，目前進行到part3，假日時會再利用時間去完成剩下的部分。
 
 
 
 
 
 
 
 
 
 
 
 
