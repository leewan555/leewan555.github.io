---
title:  "Linux 的風紀委員！－認識 iptables 及常用指令介紹"
slug: Linux-iptables-7
date:   2022-03-09
excerpt: iptables 很厲害內。
categories:
  - Linux 
tags:
  - centos
  - iptables
---

## 使用的環境

| 系統與使用工具 | 
| ----- |  
| Centos 7.6 | 
| iptables v1.4.21 | 


## 一、iptables 介紹
CentOS 7 的防火牆套件有 iptables 和 firewalld，兩者的核心都是以 netfilter 來實現。  

可藉由設定規則來過濾傳入或傳出的封包，然後寫好的規則會被送往 netfilter，告訴核心如何去處理封包。  

而此篇筆記要來介紹 iptables。

![](/assets/images/2022-03-09-Linux-iptables-7/1.jpg)



## 二、iptables 可以做到的事情
1. 
2. 
3. 


## 三、開始使用 iptables
### (1) 安裝 iptables 
```bash
yum install iptables -y
```
### (2) 啟動 iptables
```bash
systemctl start iptables
```

### (3) 重新啟動 iptables
```bash
systemctl restart iptables
```

### (4) 查看 iptables 狀態
```bash
systemctl status iptables
```

### (5) 停止 iptables
```bash
systemctl stop iptables
```

### (6) 設定開機自動啟動
```bash
systemctl enable iptables
```

## 四、iptables 的主要設定檔位置
```bash
/etc/sysconfig/iptables
```

## 五、iptables 狀態說明
iptables 查看狀態時是 `Active: active (exited)`   
而不是 `Active: active (running)`  

去查看 Loaded 時的讀取檔後，可以發現以下這行   
```bash
$ vim /usr/lib/systemd/system/iptables.service

ExecStart=/usr/libexec/iptables/iptables.init start
````

得知當執行 iptables 時是執行 `iptables.init start` 這個指令   
當執行完後就結束了，因此對 systemd 來說是有成功執行 (active)  

而只要 ExecStart 執行的指令結束後，就會變成 exited  
沒有 daemon 會持續執行，狀態就會是 active (excited)  


## 六、iptables 定義規則的方式 (以下皆可)
### (1) iptables 設定檔直接編輯修改
```bash
$ vim /etc/sysconfig/iptables
```
若直接修改此檔案，建議修改前先保存目前的防火牆規則  
規則會自動保存到 /etc/sysconfig/iptables  
```bash
# 此命令保存的規則開機會自動生效
$ service iptables save
````


修改後要重新啟動 iptables  
```bash
$ systemctl restart iptables
```


### (2) 用 iptables-restore 還原先前已保存的規則
底下有介紹  
[按這裡！](https://notes.lookfred.com/linux/Linux-iptables-7/#iptables%E5%82%99%E4%BB%BD)  

### (3) 使用 iptables 指令
底下有介紹  
[按這裡！](https://notes.lookfred.com/linux/Linux-iptables-7/#iptables-%E6%8C%87%E4%BB%A4)  

## 七、iptables 結構
iptables 主要分為三個部分：  
- Tables (表)：就是一份防火牆的規則表，可包含多組 Chain  
- Chains (鏈)：是 Rules 規則的鏈群組，可包含多個 Rule  
- Rules (規則)：每一個單獨設立的規則  

## 八、iptables 預設的三個 Tables
內建的表有三個，分別是：nat、mangle 和 filter，當未指定規則表時，則一律視為是 filter。 
 
另外，還有兩個 Tables (Raw、 Security) 暫時先不談。  

|  Tables |  包含的 Chains 與功能 |  
| ----- | ----- |  
| **filter** |  **INPUT、OUTPUT、FORWARD** | 
|  | 最常用的 table，一般的過濾功能 | 
| **nat** | **PREROUTING、POSTROUTING** | 
|  | 全名是 Network Address Translation 的縮寫，主要在進行來源與目的之 IP 或 port 的轉換，或是處理 Routing 轉換前/後的封包 | 
|  | 與 Linux 本機較無關，主要與 Linux 主機後的區域網路內電腦較有相關 | 
| **mangle** | **INPUT、OUTPUT、FORWARD、PREROUTING、POSTROUTING** |  
|  | 主要是與特殊的封包的路由旗標有關，可以修改封包的 IP 位址及其他值  |  
|  |  早期僅有 PREROUTING 及 OUTPUT，不過從 kernel 2.4.18 之後加入了 INPUT 及 FORWARD   |  
|  | 由於這個表格與特殊旗標相關性較高，所以在單純的環境當中，較少使用 mangle 這個表格  |  

## 九、五種階段的 Chains 介紹
- INPUT：經網卡進入的封包  
- OUTPUT：經網卡出去的封包  
- FORWARD：經網卡進入 / 出去轉送的封包 (proxy 類型)  
- PREROUTING：改變經網卡進入的封包狀態 (DNAT / REDIRECT)  
- POSTROUTING：改變經網卡出去的封包狀態 (SNAT / MASQUERADE)

> FORWARD 處理的封包會繞過 INPUT 和 OUTPUT，因為處理路徑不同  

## 十、iptables 處理流程
![](/assets/images/2022-03-09-Linux-iptables-7/2.jpg)

1、當封包進入網卡後，會先進入 PREROUTING，然後根據目的地址進行路由決策，如果目的地址是本機，就會走 INPUT，不是本機則走 FORWARD，然後再走 POSTROUTING 轉出去。  

2、進入 INPUT 的封包會轉給本機的程式，程式處理後會傳送新的封包，走 OUTPUT，然後經過 POSTROUTING 轉出去。 

3、過程中，當封包每經過一個 chain，都要按照 chain 的 rule 順序來走，只要遇到一個 match 的 rule 就要按照這個 rule 進行處理，而後面的 rule 對這個封包資料就不再起作用。  

## 十一、iptables 指令
用 iptables 指令新增的規則會立即生效，不用重新啟動服務  

### (1) iptables備份
```bash
$ iptables-save > /your-path/iptables-backup.bak
```

### (2) iptables還原
```bash
# 鳥哥建議使用 iptables-save 這個指令來觀察防火牆規則，因為 iptables-save 會列出完整的防火牆規則，只是並沒有規格化輸出而已
$ iptables-restore < /your-path/iptables-backup.bak
```

### (3) 列出預設 filter 表格的三個鏈的規則
```bash
$ iptables -L -n

# 單獨列出 nat 表格的規則
$ iptables -t nat -L -n
# 參數說明：
# -L ：列出目前的 table 的規則
# -n ：不進行 IP 與 hostname 的反查，速度較快
# -t ：用來指定規則表
```

輸出的結果解釋：
![](/assets/images/2022-03-09-Linux-iptables-7/3.jpg)

target：代表進行的動作， ACCEPT (接受)、REJECT (拒絕)以及 DROP (丟棄) 

prot：代表使用的封包協定，主要有 tcp、udp 以及 icmp 三種封包格式  

opt：額外的選項說明  

source ：代表此規則是針對哪個『來源 IP』進行限制  

destination ：代表此規則是針對哪個『目標 IP』進行限制  


### (4) 顯示 iptables 版本
```bash
$ iptables -V
```

### (5) 刪除所有的規則
```bash
$ iptables -F
```

### (6) 刪除指定的 chain
```bash
$ iptables -X
```

### (7) 將 iptables 計數器歸零
```bash
$ iptables -Z
```

### (8) 新增規則 (放在所有規則的最後面)
```bash
$ iptables -A
```

### (9) 新增規則 (放在指定規則的上一行，沒有指定就放第一行)
```bash
$ iptables -I
```

### (10) 刪除某條規則 (指定行數)
```bash
$ iptables -D INPUT 1
```

### (11) 查看指令行數
```bash
$ iptables --line-numbers -L INPUT
```

### (12) 定義 chain 的預設過濾政策
```bash
# 設定預訂政策 INPUT 為丟棄
$ iptables -P INPUT DROP
# 設定預訂政策 OUTPUT 為接受
$ iptables -P OUTPUT ACCEPT
# 設定預訂政策 FORWARD 為接受
$ iptables -P FORWARD ACCEPT
```

## 十二、iptables 那些比較長的指令
### (1) 允許來自 lo 介面的封包
```bash
# 不論封包來自何處或去到哪裡，只要是來自 lo (loopback) 這個介面，就予以接受
$ iptables -A INPUT -i lo -j ACCEPT
```

### (2) 接受由本機發出的回應封包
```bash
$ iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
# 參數說明：
# -m ： iptables 的外掛模組，主要有：state (狀態模組)、mac (網路卡硬體位址)
# --state ：一些封包的狀態，主要有：INVALID  (無效)、ESTABLISHED (連線成功)、NEW (新建立連線)、RELATED (最常用，主機發送出去的封包)
```

### (3) 讓 icmp 封包變成可接受的封包類型
```bash
$ iptables -A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT
```

### (4) 開啟 22/80/443 port
```bash
$ iptables -A INPUT -p tcp --dport 22 -j ACCEPT
$ iptables -A INPUT -p tcp --dport 80 -j ACCEPT
$ iptables -A INPUT -p tcp --dport 443 -j ACCEPT
```

### (5) 阻擋來自特定 IP 或 port 號的封包
```bash
# 阻擋來自 192.168.1.0/24 的 1024:65535 埠口的封包，且想要連線到本機的 ssh port
$ iptables -A INPUT -i eth0 -p tcp -s 192.168.1.0/24  --sport 1024:65534 --dport ssh -j DROP
# 參數說明：
# -p 協定：設定此規則適用於哪種封包格式，主要的封包格式有：tcp、udp、icmp 以及 all
# --sport / -s ：限制來源 port，port可連續，ex 1024:65535
# --dport / -d ：限制目標 port
# -j ：後面接動作，主要的動作有ACCEPT、DROP、REJECT及LOG
```


## 參考資料
- [IPTABLES 應用與管理](https://blog.xuite.net/towns/hc/81675609)  
- [[Ubuntu]iptables 設定](https://blog.johnsonlu.org/ubuntuiptables-%E8%A8%AD%E5%AE%9A/) 
- [iptables](https://hoohoo.top/blog/iptables/) 
- [iptables系列教程（一）iptables入門篇](https://www.gushiciku.cn/pl/pA7I/zh-tw) 
- [邁向 RHCE 之路 (Day22) - IPTables 防火牆 ](https://ithelp.ithome.com.tw/articles/10079931) 
- [iptables基礎原理和使用簡介](https://iter01.com/550867.html) 
 


