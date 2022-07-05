---
title:  "守衛家門大作戰！－Centos 7 安裝 DenyHosts 教學"
slug: Centos-7-use-Denyhosts-9
date:   2022-03-15
excerpt: 到處 try 帳密是吃飽太閒喔。
categories:
  - Linux 
tags:
  - centos
  - denyhosts
---

## 使用的環境

| 系統與使用工具 | 
| ----- |  
| Centos 7.6 | 
| iptables v1.4.21 | 
| Python 2.7.5 | 
| Python 3.6.8 | 

## 一、DenyHosts 介紹

DenyHosts 是一個防止暴力攻擊 SSH 的工具,  
當有人想要惡意 try 機器的 SSH 帳號密碼時，它會監看及分析 SSH 的 log file (譬如 /var/log/secure)。   

當發現同一 IP 連續登入失敗，並達到所設定的條件次數時，就會將嘗試登入的 IP 加入到 `/etc/hosts.deny` 並作出封鎖。  

但是它只能單純攔截 ssh 攻擊，  
若想要比較全面一點的防護可以參考這篇文章 (要放 Fail2ban 的連結)。  

## 二、DenyHosts 安裝
```bash
# 在自己想要的資料夾底下將資料夾 clone 下來 (舉例在 /usr/local 資料夾)
$ cd /usr/local
$ git clone https://github.com/denyhosts/denyhosts.git 
# 進入 denyhosts 資料夾
$ cd denyhosts
# 查看現在的路徑
$ pwd
# 開始安裝 DenyHosts 
$ python setup.py install
```

![](/assets/images/2022-03-15-Centos-7-use-Denyhosts-9/1.jpg)   


如果安裝後遇到 `ImportError: No module named ipaddr` 的問題，可以透過安裝 ipaddr 模組來解決。  

![](/assets/images/2022-03-15-Centos-7-use-Denyhosts-9/2.jpg)  

```bash
# 要先安裝 python-pip (已安裝過可跳過此步驟)
$ yum install python-pip
# 安裝 ipaddr 模組
$ pip install ipaddr
# 然後再安裝一次 DenyHosts  
$ python setup.py install
```

![](/assets/images/2022-03-15-Centos-7-use-Denyhosts-9/3.jpg)

## 三、DenyHosts 基本設定
### (1) DenyHosts 的設定檔
```
/etc/denyhosts.conf
```

### (2) 編輯檔案及介紹
```bash 
$ vim /etc/denyhosts.conf
```
```bash
# sshd 登入的 log 位置
SECURE_LOG = /var/log/secure

# 裡面放黑名單 IP 的檔案，預設在 /var/lib/denyhosts
# 此篇範例設在 /usr/local/denyhosts/data
WORK_DIR = /usr/local/denyhosts/data

# DenyHosts 3.0 後就加入了 iptables 同步封鎖的功能
# 如果不要這個功能就將它註解
IPTABLES = /sbin/iptables

# ssh 的 port，若一個以上的 port 要用逗號分隔 (ex: 11,22,33)
BLOCKPORT = 22

# 多久清除一次已封鎖的 IP
PURGE_DENY = 1w

# 是否做域名反解
HOSTNAME_LOOKUP = No

# 如果登入成功，各個 IP 地址的失敗次數將重置為 0
RESET_ON_SUCCESS = yes

# 多久執行一次預設清理，設定值與 PURFE_DENY 一樣即可
DAEMON_PURGE = 1w

# 允許無效用戶登錄失敗的次數
DENY_THRESHOLD_INVALID = 5

# 允許普通用戶登錄失敗的次數
DENY_THRESHOLD_VALID = 10

# 允許 root 登錄失敗的次數
DENY_THRESHOLD_ROOT = 3
```

## 四、WORK_DIR 資料夾底下會有的檔案
裡面是放黑名單 IP 的檔案。
1. hosts
2. hosts-restricted
3. hosts-root
4. hosts-valid
5. users-hosts


## 五、解除被封鎖的 IP
開啟以下六個檔案，個別手動註解或刪除解封的 IP。
```bash
# 此篇範例的 WORK_DIR 路徑在 /usr/local/denyhosts/data 
$ vim /etc/hosts.deny
$ vim /usr/local/denyhosts/data/hosts
$ vim /usr/local/denyhosts/data/hosts-restricted
$ vim /usr/local/denyhosts/data/hosts-root
$ vim /usr/local/denyhosts/data/hosts-valid
$ vim /usr/local/denyhosts/data/users-hosts
```
也有一些指令可以直接解除被封鎖的 IP，但是實際測試不起作用，所以暫且不談。

## 六、Denyhosts 的白名單 IP 設定
若想要設定**不想**被 DenyHosts 封鎖的 IP，可以建立白名單。  

在 WORK_DIR 路徑建立一個新檔案 `allowed-hosts`，檔案內容是一行各一個白名單 IP。

這個 IP 還是會在 /etc/hosts.deny 裡面，只是會被註解。

```bash
# 進入 WORK_DIR 路徑
$ cd /usr/local/denyhosts/data
# 建立並編輯白名單檔案
$ vim allowed-hosts 
```
## 七、開始使用 Denyhosts 

### (1) 啟動 Denyhosts
> 啟動前，務必要先清空或分割 sshd 的 log 檔，不然自己會先被封鎖。 

```bash
# 要先到 Denyhosts 的資料夾底下，再開始做其他動作
$ cd /usr/local/denyhosts

# 啟動 Denyhosts
$ ./daemon-control-dist start

# 重新啟動 Denyhosts
$ ./daemon-control-dist restart

# 查看 Denyhosts 狀態 
$ ./daemon-control-dist status  

# 停止 Denyhosts
$ ./daemon-control-dist stop
``` 
### (2) 分割 sshd 的 log 檔 (/var/log/secure)
想要馬上分割 /var/log/secure 檔案，可以強制執行 logrotate，而不用等定期的分割時間。 

```bash
logrotate -vf /etc/logrotate.conf 
``` 

## 八、若 /var/log/secure 不紀錄 log 
先試著重新啟動 rsyslog，再觀察看看。
```bash
$ service rsyslog restart 
```

## 參考資料
- [sourceforge DenyHosts Files](https://sourceforge.net/projects/denyhosts/files/denyhosts/) 
- [DenyHosts Github](https://github.com/denyhosts/denyhosts) 
- [#64 option --purgeip does not work](https://sourceforge.net/p/denyhosts/bugs/64/?limit=25) 
