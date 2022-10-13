---
title:  "哇虎！使用 Nginx 的 Naxsi 模組實現 WAF 功能"
slug:  2022-10-13-Nginx-Naxsi-module-28
date:   2022-10-13
excerpt: 人外有天，牆外有牆。
categories:
  - Nginx
tags:
  - nginx
  - naxsi
  - module

---

## 使用的環境

| 系統與使用工具 | 
| ----- |  
| Centos 7.6 | 
| nginx/1.16.1 | 

![](/assets/images/2022-10-13-Nginx-Naxsi-module-28/1.JPG)


[Naxsi 官方 Github](https://github.com/nbs-system/naxsi "Naxsi 官方 Github")


> 你好你好。
{: .prompt-warning }


{% raw %}
```bash
load_module modules/ngx_http_geoip2_module.so;

```
{% endraw %}


還沒打好la


## 一、Naxsi 模組介紹
Naxsi 是 nginx 的第三方模組，與任何 nignx 版本都相容，採用 GPLv3 授權，可以免費使用，也不需要依賴類似防毒軟體的病毒碼資料庫。  

可以建置簡易的 WAF 系統，阻擋一些常見的 Nginx Anti XSS 及 SQL Injection 攻擊，但只能過濾「GET」及「POST」的請求。  


## 二、naxsi_core.rules 規則介紹+白名單放行

### 1. 計分方式
| 1 | 2 | 3 | 4 |  
| ----- | ----- | ----- | ----- |       
| 1 | 2 | 3 | 4 |  
| 1 | 2 | 3 | 4 |  

### 2. 攔截成功畫面


### 3. 攔截 log 與詳細資訊


### 4. (額外)開啟 NAXSI_EXLOG



## 三、安裝 Naxsi 模組


### 1. 安裝必要套件

```bash
$ yum groupinstall "development tools" -y
```

### 2. 安裝 PCRE 以及 open ssl 套件

```bash
$ yum install pcre pcre-devel openssl openssl-devel -y
```

### 3. 在想要的路徑下載 Naxsi 
我是下載到/usr/local

```bash
$ git clone https://github.com/nbs-system/naxsi.git
```

### 4. 各個路徑整理

```bash

```

### 5. 編譯 Nginx
我 LNMP 的 Nginx 解壓縮檔路徑是 /tmp/nginx
使用 --add-module 把 naxsi 加進去
lnmp 壓縮檔 cd /root/lnmp1.6

```bash
$ ./configure --user=www --group=www \
--prefix=/usr/local/nginx \
--with-http_stub_status_module \
--with-http_ssl_module \
--with-http_v2_module \
--with-http_gzip_static_module \
--with-http_sub_module \
--with-stream \
--with-stream_ssl_module \
--with-openssl=/root/lnmp1.6/src/openssl-1.1.1d \
--with-openssl-opt='enable-weak-ssl-ciphers' \
--add-module=/usr/local/naxsi/naxsi_src

$ make 

# 有些人說不要 make install，但我用 make module 或純粹 make 都沒有用，不知道原因
$ make install
```

### 6. 將 Naxsi 主要設定檔複製進 Nginx 資料夾中

```bash
$ cp /usr/local/naxsi/naxsi_config/naxsi_core.rules /usr/local/nginx/conf/
```

### 7. 新增一個 Naxsi 規則檔案

```bash
$ vim /usr/local/nginx/conf/naxsi_custom.rules;
```

### 8. 開始設定 


```bash
$ git clone https://github.com/nbs-system/naxsi.git
```

### 9. 重新啟動或重新讀取 nginx

```bash

```

## 四、實際測試



## 五、(額外)將 naxsi 加入 fail2ban

### 1. a

```bash

```

### 2. b

```bash

```



## 參考資料
- [Naxsi Github](https://github.com/nbs-system/naxsi) 
- [WAF 是什麼？你的網站需要 WAF 嗎？](https://blog.cloudmax.com.tw/waf/) 
- [naxsi-rules Github](https://github.com/nbs-system/naxsi-rules) 
- [Naxsi 配置白名單](https://bolerolily.github.io/2018/08/21/Naxsi-%E9%85%8D%E7%BD%AE%E7%99%BD%E5%90%8D%E5%8D%95/) 
- [Nginx 搭配 Naxsi 實現 WAF 功能](https://dotblogs.com.tw/eric_obay_talk/2018/10/22/150521) 
- [Nginx naxsi + 色情守門員](https://ithelp.ithome.com.tw/articles/10217411) 
- [Install and Configure Nginx With Naxsi](https://medium.com/@icarobichir/install-and-configure-nginx-with-naxsi-9aaa66f20d4e) 
- [NAXSI安裝測試與簡介](https://www.securitypaper.org/5.%E9%99%84%E5%BD%95/04.naxsi%E5%AE%89%E8%A3%85%E6%B5%8B%E8%AF%95%E4%B8%8E%E7%AE%80%E4%BB%8B/) 
- [沒 WAF 防火牆還想混網路？ WAF 防火牆非裝不可的 3 個原因](https://enterprise.fetnet.net/content/ebu/tw/epaper/tech/27th/FETnet-WAF-Firewall.html) 

