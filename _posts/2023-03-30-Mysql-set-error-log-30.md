---
title:  "LNMP MySQL 設定 error_log"
slug: Mysql-set-error-log-30
date:   2023-03-30
excerpt: Log超級重要。
categories:
  - MySQL
tags:
  - MySQL
---

## 使用的環境

| 系統與使用工具 | 
| ----- |  
| Centos 7.6 | 
| mysql  Ver 8.0.13 | 

## 一、設定 error_log 遇到問題
如果 LNMP 安裝的 MySQL 在 `/etc/my.cnf` 當中設定 `log_error=/var/log/mysql/error.log`，會遇到以下錯誤且無法重新啟動成功。

```bash
mysql.serviceJob for mysql.service failed because the control process exited with error code. See "systemctl status mysql.service" and "journalctl -xe" for details.
```

因為 LNMP 在安裝 MySQL 時，會在系統中新增 mysql 的使用者與群組，並用這組權限來控制資料庫。  

但是在 `/etc/my.cnf` 中指定的路徑權限屬於 `root:root`，在無法讀取 log 檔案的情況下，MySQL 就無法正常打開。  


## 二、error_log 設定方式
### 1. 先新增 MySQL 的 log 檔案
```bash
$ touch /var/log/mysql/error.log
```

### 2. 將 `/var/log/mysql` 及其下資料夾和檔案的權限設定為 `mysql:mysql`
```bash
$ chown -R mysql:mysql /var/log/mysql
```

### 3. 設定 /etc/mysql
```bash
vim /etc/my.cnf

[mysqld]
log_error=/var/log/mysql/error.log
```

### 4. 重新啟動 mysql
```bash
$ lnmp restart mysql
```

### 5. 確認設定成功
error_log 設定完成後，可以看看 log 第一行有沒有啟動訊息。

```
2022-12-15T09:53:24.789020Z 0 [System] [MY-010116] [Server] /usr/local/mysql/bin/mysqld (mysqld 8.0.13) starting as process 21119
```

## 參考資料
- [LNMP 設定 MySQL 的錯誤日誌](https://huanyichuang.com/blog/lnmp-failed-to-restart-mysql-after-editing-my-cnf/) 