---
title:  "比 owner, group, others 還要更詳細的 ACL 權限"
slug: Centos7-ACL-permission-10
date:   2022-03-16
excerpt: 就不會門窗都打開開。
categories:
  - Linux 
---

## 使用的環境

| 系統與使用工具 | 
| ----- |  
| Centos 7.6 | 
| setfacl 2.2.51 | 

## 一、ACL 權限介紹
ACL 是 Access Control List 的縮寫。  

主要功能是提供比傳統的 owner, group, others 的 read, write, execute 權限之外的更細部權限設定。  

ACL 可以針對單一使用者、單一檔案或目錄來進行 r, w, x 的權限規範，對於需要特殊權限的使用狀況非常有幫助。

## 二、查看 Linux Kernel 是否啟動 ACL
```bash
$ dmesg | grep -i acl
```

## 三、查看 ACL 版本
```bash
$ setfacl --version
```

## 四、設定 ACL 權限
### 1、ACL 指令結構
setfacl -m 為設定的主要指令與選項。  
而設定的項目則主要有：  

```bash
針對個人： u:[帳號名稱]:rwx- [資料夾或檔案]
針對群組： g:[群組名稱]:rwx- [資料夾或檔案]
u::r-- 當帳號或群組名稱沒有寫的時候，代表為檔案擁有者的帳號與群組之意

# 參數說明：
# -R：遞迴，若是目錄，此選項會讓目錄下的所有都擁有所設定的 ACL 權限
# -d：預設權限，在此目錄下新建立的所有會自動的擁有此「預設權限」。原本就存在該目錄內的徑物並不會被此預設權限影響，是新建立的才會
```

### 2、ACL 指令
#### (1) 遮罩 mask
透過 mask 來規範最大允許的權限，可以避免不小心手誤開放超大的權限給使用者或群組。
```bash
$ setfacl -m m::x /your/file
```
> 基本上如果特定使用者有 rx 的權限，但 mask 只有 x 的話，實際上特定使用者可以取得的權限僅有 x 而已。

#### (2) 列出資料夾或檔案的 ACL 權限資訊
```bash
$ getfacl [folder/file]
```

#### (3) 針對特定使用者設定 ACL 權限
讓特定特定使用者 fred 對檔案 grouptest 擁有 r 跟 x 的 ACL 權限。  

一開始先新增一個 test 檔案，然後查看它的基本權限。  
```bash
$ touch filetest
$ ll filetest
-rw-r--r-- 1 root root 0 Mar 16 11:24 filetest
```

接下來使用 ACL 權限設定：
```bash
$ setfacl -m u:fred:rx filetest
$ ll test
-rw-r-xr--+ 1 root root 0 Mar 16 11:24 filetest*
```

查看 filetest 檔案的 ACL 權限
```bash
$ getfacl filetest
# file: filetest
# owner: root
# group: root
user::rw-      <==預設的擁有者權限
user:fred:r-x  <==針對 fred 的權限
group::r--     <==預設的群組權限
mask::r-x      <==預設的 mask 權限
other::r--
```
就可以知道 filetest 檔案對於使用者 fred 有另外給 r 跟 x 的權限。 

#### (4) 針對特定群組設定 ACL 權限
讓特定群組 fredhome 對檔案 grouptest 擁有 w 跟 x 的 ACL 權限。  

```bash
$ touch grouptest
$ ll grouptest
-rw-r--r-- 1 root root 0 Mar 16 11:35 grouptest
```

接下來使用 ACL 權限設定：
```bash
$ setfacl -m g:fredhome:wx grouptest
$ ll test
-rw-rwxr--+ 1 root root 0 Mar 16 11:35 grouptest*
```

查看 grouptest 檔案的 ACL 權限
```bash
$ getfacl grouptest
# file: grouptest
# owner: root
# group: root
user::rw-
group::r--
group:fredhome:-wx
mask::rwx
other::r--
```
就可以知道 grouptest 檔案對於群組 fredhome 有另外給 w 跟 x 的權限。 


#### (5) 移除指定 ACL 權限
移除指定的 ACL 權限，可以使用 -x 參數，移除 ACL 權限時**不需要指定權限內容**。
```bash
$ setfacl -x u:fred /your/file
```

#### (6) 刪除所有新增的 ACL 權限
但預設的 ACL 規則（owner, group, others）將被保留。  

```bash
$ setfacl -b /your/file
```

#### (7) 刪除預設的 ACL 權限
如果沒有預設規則，將不提示。  
```bash
$ setfacl -k /your/folder
```

#### (8) 繼承 ACL 權限
如果想讓特定資料夾下的新檔案都可以自動繼承特定的 ACL 權限設定，  
可以在資料夾加上預設的 ACL 權限，  
預設 ACL 權限的表示法就是在一般 ACL 權限之前加上 d: 或 default:。  
> 要注意此參數 d 只對資料夾有效。  

```bash
# 讓指定的資料夾有給使用者 fred 擁有 r 跟 x 權限的預設 ACL 權限
$ setfacl -m d:u:fred:rx /your/folder
```

## 參考資料  
- [linux ACL權限](https://crmne0707.pixnet.net/blog/post/322350222-linux-acl%E6%AC%8A%E9%99%90)  
- [[Day 10] Linux 細部權限 ACL](https://ithelp.ithome.com.tw/articles/10221185)  
- [大神親自教你如何用Linux acl命令實現文件權限管理](https://kknews.cc/code/y2kmek.html)  

