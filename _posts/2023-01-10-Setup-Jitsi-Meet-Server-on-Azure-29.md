---
title:  "將 Jitsi meet 架設在 Azure 雲端上"
slug:  2023-01-10-Setup-Jitsi-Meet-Server-on-Azure-29
date:  2023-01-10
excerpt: 唐鳳曾經推薦過的工具耶~
categories:
  - Ubuntu
  - Azure
tags:
  - ubuntu
  - nginx
  - jitsi
---

## 使用的環境

| 系統與使用工具 | 
| ----- |  
| Ubuntu-20.04  | 
| nginx/1.18.0 |
| openjdk 11.0.17 |  


> 嗨。  
{: .prompt-info }


## 一、Jitsi Meet 介紹
### 1. Jitsi Meet 是什麼？
Jitsi 是免費且開源的視訊會議程式，支援多種語言，在電腦上，使用瀏覽器開網頁就可以操作所有功能，無須安裝任何東西。  

自行架設的話則包含 server 和 client 端，client 端包含了網頁瀏覽器、手機 app 等介面。

### 2. 誰適合使用？
適合不想付費、註冊會員、下載或安裝軟體，或是想要長時間開啟視訊會議的人。


## 二、Azure 建立新機器資訊

| 項目 | 設定資訊 |   
| ----- | ----- |   
| 作業系統 | Ubuntu 20.04 |  
| 規格 | Standard B2s (2 vcpu，4 GiB 記憶體) |  
| 資源群組 | jitsi |  
| 電腦名稱 | 美國東部2 East US 2   |  
| DNS名稱標籤 | 80,443,10000,22,3478,5349 |  
| SSH登入指令 | ssh username@jitsi.eastus2.cloudapp.azure.com 

## 三、Azure 開新機器及圖片介紹
1. 建立資源，選擇 Ubuntu Server 20.04 tls。  
![](/assets/images/2023-01-10-Setup-Jitsi-Meet-Server-on-Azure-29/1.JPG)  

2. 自訂資源群組名稱、機器名稱，並選擇區域。  
![](/assets/images/2023-01-10-Setup-Jitsi-Meet-Server-on-Azure-29/2.JPG)  

3. 選擇機器大小，設定登入金鑰或密碼，對外 port 勾選 `[80,443,22]`，再按下左下角的`檢閱+建立`，建立新機器。  
![](/assets/images/2023-01-10-Setup-Jitsi-Meet-Server-on-Azure-29/3.JPG)  

4. 在**網路**新增 Jitst 所需的 port`[80,443,10000,22,3478,5349]`，並命名為`port_jitsi`，前面已經設定過 80,443,22 了，現在又再設定一次是為了集中管理 Jitsi 所需的 port，實際上不影響。   
![](/assets/images/2023-01-10-Setup-Jitsi-Meet-Server-on-Azure-29/4.JPG)  

5. 再**組態**設定 DNS 名稱標籤。
![](/assets/images/2023-01-10-Setup-Jitsi-Meet-Server-on-Azure-29/5.JPG)  

6. 設定皆完成，可以開始連線至機器囉。
![](/assets/images/2023-01-10-Setup-Jitsi-Meet-Server-on-Azure-29/6.JPG)  

## 四、SSH 連線至機器
```bash
$ ssh username@jitsi.eastus2.cloudapp.azure.com
```

## 五、安裝 Jitsi 所需要的工具


### 1. 需要的工具
- gnupg2  
- nginx-full  
- sudo => Only needed if you use sudo   
- curl => Or wget to Add the Jitsi package repository   
- OpenJDK 11 must be used.  

### 2. 把需要的工具安裝起來
```bash
$ sudo -s 
$ apt update -y
$ apt install -y apt-transport-https
$ apt-add-repository universe # jisti需要
$ apt install -y gnupg2
$ apt install -y nginx
$ apt install -y openjdk-11-jdk

# sudo 跟 curl 原本就有所以沒裝
```

### 3.  檢查是否有安裝完成
```bash
$ nginx -V
$ java --version
$ sudo -V
$ curl -V
```

## 六、 編輯 /etc/hosts

```bash
$ vim /etc/hosts
x.x.x.x myjitsi.test123.com myjitsi
# x.x.x.x 是本機IP，myjitsi.test123.com 是自訂的域名，後面的 myjitsi 是電腦名稱

# 重開機吃設定
$ reboot 
```

## 七、 Add the Prosody package repository
```bash
$ echo deb http://packages.prosody.im/debian $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list
$ wget https://prosody.im/files/prosody-debian-packages.key -O- | sudo apt-key add -
$ apt install lua5.2
```

## 八、 Add the Jitsi package repository
```bash
$ curl https://download.jitsi.org/jitsi-key.gpg.key | sudo sh -c 'gpg --dearmor > /usr/share/keyrings/jitsi-keyring.gpg'
$ echo 'deb [signed-by=/usr/share/keyrings/jitsi-keyring.gpg] https://download.jitsi.org stable/' | sudo tee /etc/apt/sources.list.d/jitsi-stable.list > /dev/null
$ apt update
```

## 九、 設定防火牆
```bash
$ ufw allow 80/tcp
$ ufw allow 443/tcp
$ ufw allow 10000/udp
$ ufw allow 22/tcp
$ ufw allow 3478/udp
$ ufw allow 5349/tcp
$ ufw enable # 啟動 ufw
$ ufw status verbose # 查看 ufw 狀態
```

## 十、 安裝 jitsi
```bash
$ apt install -y jitsi-meet
```

安裝過程會詢問的問題：
1. 域名要哪個？—— myjitsi.test123.com
2. SSl 憑證要用哪個？—— Let's Encrypt Certificate
3. 輸入Email(SSL用)—— username@gmail.com
4. Add telephony to your Jitsi meetings?—— no 

## 十一、 在瀏覽器輸入網址
myjitsi.test123.com  
進入網站後即可開始使用基本的 Jitsi。
![](/assets/images/2023-01-10-Setup-Jitsi-Meet-Server-on-Azure-29/7.JPG)   


大功告成！


## 參考資料
- [jitsi](https://meet.jit.si/)
- [Self-Hosting Guide - Debian/Ubuntu server](https://jitsi.github.io/handbook/docs/devops-guide/devops-guide-quickstart)
- [How To Install Jitsi Own Server For Video Conference On Ubuntu 20.04](https://technologyrss.com/how-to-install-jitsi-own-server-for-video-conference-on-ubuntu-20-04/)
- [Jitsi Meet 視訊會議教學：免費無限時間、共享桌面、會議錄影](https://www.playpcesor.com/2020/04/jitsi-meet.html)