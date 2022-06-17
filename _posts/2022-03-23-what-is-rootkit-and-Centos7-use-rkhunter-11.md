---
title:  "找出那些趁人之危的 rootkit－交給專業的 rkhunter！"
slug: what-is-rootkit-and-Centos7-use-rkhunter-11
date:   2022-03-23
excerpt: 很會藏嘛！
categories:
  - Linux 
---

## 使用的環境

| 系統與使用工具 | 
| ----- |  
| Centos 7.6 | 
| Rootkit Hunter 1.4.6 | 

## 一、rootkit 是什麼？
rootkit 又被稱為後門程式 (backdoor) 或木馬程式 (trojan)。  

發現漏洞後與修補程式釋出前的中間空窗期，會被駭客撰寫惡意程式來攻擊該漏洞，他們藉此取得被攻擊主機的控制權，植入木馬程式在受攻擊的主機上，並散播惡意程式，這些惡意程式就被稱為 rootkit。

將 rootkit 這名詞拆開分別是 root 跟 kit，root 是 unix 系統中最高權限管理者的名稱，kit 則是工具的意思。  

由此可知，rootkit 就是被用來攻擊目標主機並取得最高權限的工具，而且還會自我隱藏，不讓真正的管理者發現。   


## 二、如何避免受到 rootkit 的攻擊？
rootkit 主要是藉由主機的漏洞來攻擊的惡意程式，所以要從可能的地方對症下藥。  

1. 關閉不必要的服務  
2. 隨時更新主機上各個工具的修補程式  
3. 使用軟體工具來檢查主機  

而今天要來介紹的就是第三點，使用 rkhunter 來檢查電腦裡是否有 rootkit 潛伏。  

## 三、rkhunter 介紹
rkhunter 是一個檢查運作中的 Unix 機器是否有被植入 Rootkit 的工具。  

而且顧名思義就是 rootkit hunter，rootkit 獵人，聽起來有夠厲害，而且它非常好上手！  

rkhunter 的功能：
1. 利用 MD5 編碼來比對檔案的一致性，檢查文件是否改動
> rkhunter 在釋出的時候，就已經收集了各大知名的 Linux distributions 的重要檔案的 MD5 編碼 (例如 login, ls, ps, top, w 等檔案)， 並製作成資料庫。
2. 檢查 rootkit 經常攻擊的檔案  
3. 檢查是否具有錯誤的檔案權限 (針對 binary files)  
4. 檢查隱藏檔案  
5. 檢查可疑的核心模組 (LKM/KLD)  
6. 作業系統的特殊檢測  
7. 檢查已啟動的監聽埠號 (listening port)  
8. 特定分析 (String scanner)  

## 四、安裝及使用 rkhunter 

當安裝好並且執行後，rkhunter 就會利用它的資料庫的資料去與當下系統的相關檔案進行比對，若比對的結果有問題，則會顯示警示文字，提供系統管理員分析。  

```bash
# 首先安裝 rkhunter
$ yum install rkhunter -y

# 更新資料庫檔案
$ rkhunter --update

# 更新病毒資料庫
$ rkhunter --propupd

# 讓 rkhunter 自動檢測，就不用測試完一部分後就要按 Enter 才能繼續
$ rkhunter --check --skip-keypress

# 檢查系統並執行所有測試令 (需按 Enter 下一步)
$ rkhunter -c
```

## 五、rkhunter 其他指令 (有些需配合 -c 使用，自己判斷)
```bash
# 查看當前版本
$ rkhunter -V

# 檢查是否有新版本
$ rkhunter --versioncheck

# 使用 crontab 定期執行檢查 (會自動拿掉彩色輸出)
$ rkhunter --cronjob

# 將檢測結果用黑白輸出 (在有些情況下，有顏色或延長的顯示符號會有問題)
$ rkhunter --nocolors

# 僅列出警告訊息，正常訊息不列出
$ rkhunter --report-warnings-only  

# 顯示說明及相關參數用法
$ rkhunter -help
```

## 六、rkhunter 的 設定檔位置
```bash
/etc/rkhunter.conf
```

## 七、rkhunter 相關設定
```bash 
# 編輯 rkhunter 設定檔
$ vim /etc/rkhunter.conf  

# 新增白名單，就不會每次都被偵測到有問題
SCRIPTWHITELIST = /usr/bin/egrep

# 同意的隱藏檔
ALLOWHIDDENDIR = /etc/.java
  
# log 檔案位置
LOGFILE=/var/log/rkhunter/rkhunter.log  
```  

## 八、rkhunter 的 log 檔位置
```bash
# 去找吧！都放在那裡了
/var/log/rkhunter/

# 篩選出有問題的部分
$ grep Warning /var/log/rkhunter/rkhunter.log
```

## 九、rkhunter 例外的錯誤狀態
舉例 MD5 編碼這方面：   

rkhunter 在利用 MD5 編碼比對方面，是利用他本身的 MD5 編碼資料庫與當下的系統相關檔案進行比對，但若當下系統不在 rkhunter 支援的範圍之內，rkhunter 會判斷該檔案有問題。  

此外，如果是利用 tarball 的方式自行安裝類似 syslogd, ps 等檔案，因為下達的參數不同，所以這些檔案與 rkhunter 的 MD5 資料庫也會不同，所以會被判定有問題。  

在這種情況下，可以先更新 rkhunter 的資料庫，若問題仍舊存在，可以試著聯絡作者。  


## 十、遇到的問題
### (1) Warning: The file properties have changed

![](/assets/images/2022-03-23-what-is-rootkit-and-Centos7-use-rkhunter-11/1.jpg) 

```bash
Warning: The file properties have changed:
         File: /usr/bin/whoami
         Current inode: 266256    Stored inode: 266257
```

如果遇到這個問題，很有可能是跑了系統更新後 inode 值改變  
可以先去 /var/log/yum.log，確認近期是否有更新過上面被警告的指令  
確認沒問題後，可以用底下指令重置  
```bash
rkhunter --update --propupd
```

## 參考資料
- [rkhunter 木馬&後門偵測](https://blog.xuite.net/beavisliu/blog/15449011) 
- [Day 30 RootKit Hunter監測木馬和後門偵測](https://ithelp.ithome.com.tw/articles/10161775) 
- [找出rootkit](https://www.informationsecurity.com.tw/article/article_detail.aspx?aid=228) 
