---
title:  "利用 Nginx 的 GeoIP2 模組限制特定國家 IP 造訪網站"
slug:  2022-08-11-Nginx-Geoip2-module-25
date:   2022-08-11
excerpt: 一堆奇怪 vpn 來的 ip 通通退散！～
categories:
  - Nginx
tags:
  - nginx
  - geoip2
  - geolite2

---

## 使用的環境

| 系統與使用工具 | 
| ----- |  
| Centos 7.6 | 
| nginx/1.16.1 | 
| geoipupdate 2.5.0 | 

## 一、Nginx GeoIP2 模組介紹
GeoIP2 是 Nginx 的動態模組，搭配 MaxMind 資料庫，可自動辨識 IP 所位於的國家，並設定規則，當特定國家 IP 造訪網站時，做出自訂的回應方式。

## 二、下載 GeoIP2 模組的兩個方式

### 1. GetPageSpeed 的 RPM repo（需訂閱付費）

> 這要付錢，純紀錄，本篇文章不用這個方式。
{: .prompt-warning }

先去下載 GetPageSpeed 的 RPM repo，這樣才能載到新版的 `nginx-module-geoip2`。  

```bash
$ yum -y install https://extras.getpagespeed.com/release-latest.rpm

$ yum install nginx-module-geoip2
```

安裝到一半會說"缺少訂閱所以停止"，並附上一個訂閱的網址，點進去看到自己的 IP，再按訂閱就會跳至付款畫面。  

![](/assets/images/2022-08-11-Nginx-Geoip2-module-25/1.JPG)


### 如果安裝了這個 repo，但反悔想刪掉的話怎麼辦？

```bash
# 先查詢這個 repo 完整的 rpm 套件名稱
$ rpm -qa |grep -i getpagespeed
getpagespeed-extras-release-11-31.noarch

# 如果有找到 rpm 套件名稱，就可加上 -e 刪除
$ rpm -e getpagespeed-extras-release-11-31.noarch
```

### 2. 從 Github 下載

[leev/ngx_http_geoip2_module](https://github.com/leev/ngx_http_geoip2_module "leev/ngx_http_geoip2_module")


```bash
# 建立新資料夾
$ mkdir nginx_module/

$ cd nginx_module/

# 下載 Github 上的 GeoIP2 模組
$ git clone https://github.com/leev/ngx_http_geoip2_module.git

$ cd ngx_http_geoip2_module/
```

## 三、安裝 geoipupdate

因為 IP 時常有變化，所以 MaxMind 提供 `geoipupdate` 工具，可以讓我們更新 IP 清單，但這個工具需要搭配 MaxMind 帳號和 License Key。  

```bash
$ yum install geoipupdate geoipupdate-cron -y

# 查看版本
$ geoipupdate -v
geoipupdate 2.5.0
```
查看版本是因為 License Key 需要對應 `geoipupdate` 的版本。  

## 四、取得免費的 GeoLite2 的 License Key
MaxMind 提供的資料庫有分為商業版 GeoIP2 和免費版 GeoLite2，免費版的資料庫有各國家的 IP 對照表，而商業版有城市、經緯度等等更詳細的資訊。  

網路上有網友實測，免費版的對於城市定位精準度會有些差異，所以就看看自己的需求來選擇囉～  

本篇文章是使用 GeoLite2 資料庫。  

### 1. 註冊會員
[GeoLite2](https://www.maxmind.com/en/geolite2/signup?utm_source=kb&utm_medium=kb-link&utm_campaign=kb-create-account "GeoLite2")

填寫完再去 mail 啟用和設定密碼後就代表註冊成功了！  
![](/assets/images/2022-08-11-Nginx-Geoip2-module-25/2.JPG)

### 2. 建立 License Key
登入 MaxMind 網站後，按左方 Manage License Keys，現在來要建立 License Key。  

依照 `geoipupdate` 版本去選（我的 `geoipupdate` 版本是 2.5.0）。  

![](/assets/images/2022-08-11-Nginx-Geoip2-module-25/3.JPG)

![](/assets/images/2022-08-11-Nginx-Geoip2-module-25/4.JPG)

之後會有 ID 跟 License Key，點選 Download Config，會自動下載一個 `GeoIP.conf` 檔案，將這個檔案覆蓋主機上的 `/etc/GeoIP.conf`。

![](/assets/images/2022-08-11-Nginx-Geoip2-module-25/5.JPG)

### 3. 更新 Maxmind 資料庫

覆蓋完 `GeoIP.conf` 檔案後，用 `geoipupdate` 跑一次 Maxmind 資料庫更新。  

```bash
$ geoipupdate

$ geoipupdate -v
```
![](/assets/images/2022-08-11-Nginx-Geoip2-module-25/6.JPG)


可以將指令寫進 crontab 排程，讓它定時更新 Maxmind 資料庫。  

```bash
1 0 * * 1,4 /usr/bin/geoipupdate
```

### 小技巧：mmdblookup 工具
`mmdblookup` 可以在指定的 Maxmind 資料庫中查詢 IP，IP 的記錄會以類似 JSON 的結構顯示。  

Maxmind 資料庫檔案放在 `/usr/share/GeoIP`。   

[國家清單可以參考此網站](http://www.geonames.org/countries/ "國家清單可以參考此網站")   

```bash
# -f/--file: 指定檔案
# -i/--ip: 指定 IP
# -v: 詳細說明
# country names 表示是用什麼語言來輸出國家名稱
$ mmdblookup --file /usr/share/GeoIP/GeoLite2-Country.mmdb --ip 47.52.76.54 country names en

$ mmdblookup --file /usr/share/GeoIP/GeoLite2-Country.mmdb --ip 47.52.76.54 country iso_code
```
![](/assets/images/2022-08-11-Nginx-Geoip2-module-25/9.JPG)


## 五、Nginx 編譯 GeoIP 模組 

### 1. 找到有含 configure 的資料夾
若 Nginx 是用 yum install 安裝的，會沒有 `configure` 檔案，但如果真的想要新增第三方模組，只需對相同版本的 Nginx 原始碼進行編譯後替換即可。  

> 但模組若也是用 `yum install` 安裝，就可以直接在 nginx.conf 上面新增 `load_module modules/ngx_http_geoip2_module.so;`
{: .prompt-info }

我的 Nginx 是 1.16.1 版本，就找 Nginx 1.16.1 的壓縮檔 `nginx-1.16.1.tar.gz`，並解壓縮，就可以得到 `configure` 以及其他編譯需要用的檔案。   

```bash
$ tar xvpzf nginx-1.16.1.tar.gz

$ cd nginx-1.16.1

$ ll
```
![](/assets/images/2022-08-11-Nginx-Geoip2-module-25/10.JPG)

### 2. 查看 Nginx 原先有的模組
先查看 Nginx 原本有什麼模組，當編譯新的模組時，原有的模組也要原封不動的寫進去，不然到時候編譯完成後會不見（就是底下 `configure arguments` 那段）。  

```bash
$ nginx -V

nginx version: nginx/1.16.1
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC)
built with OpenSSL 1.1.1d  10 Sep 2019
TLS SNI support enabled
configure arguments: --user=www --group=www --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-http_v2_module --with-http_gzip_static_module --with-http_sub_module --with-stream --with-stream_ssl_module --with-openssl=/root/lnmp1.6/src/openssl-1.1.1d --with-openssl-opt=enable-weak-ssl-ciphers
```



### 3. 編譯 GeoIP2 模組
要在有 configure 的那個資料夾底下編譯，模組安裝路徑可以用絕對路徑比較保險。  

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
--with-openssl-opt=enable-weak-ssl-ciphers \
--add-dynamic-module=/usr/local/nginx_module/ngx_http_geoip2_module

$ make 

# 有些人說不要 make install，但我用 make module 或純粹 make 都沒有用，不知道原因
$ make install
```

### 4. 編譯時出現問題

```bash
error:
adding module in /usr/local/nginx_module/ngx_http_geoip2_module
checking for MaxmindDB library ... not found
./configure: error: the geoip2 module requires the maxminddb library.
```

上網爬文，安裝 `libmaxminddb-devel` 後即可正常編譯。 

```bash
$ yum install libmaxminddb-devel
```

安裝好後，重複上面第 3 點的步驟。  

### 5. 查看是否編譯成功
```bash
$ nignx -V
```
![](/assets/images/2022-08-11-Nginx-Geoip2-module-25/7.JPG)

在 Nginx 的安裝路徑也會自動產生一個 `modules` 資料夾，且裡面有兩個檔案，有出現代表新增模組成功。  
```bash 
/usr/local/nginx/modules/ngx_http_geoip2_module.so
/usr/local/nginx/modules/ngx_stream_geoip2_module.so
```
![](/assets/images/2022-08-11-Nginx-Geoip2-module-25/8.JPG)

## 六、Nginx.conf 或 vhost.conf 設定
開始設定 nginx.conf。  
```bash
$ vim /usr/local/nginx/conf/nginx.conf
```
### 1. 全區塊

{% raw %}
在 pid 下面新增這兩句：
```conf
load_module modules/ngx_http_geoip2_module.so;
load_module modules/ngx_stream_geoip2_module.so;
```
{% endraw %}


### 2. http 區塊
在 http 區塊新增下面這串:  

{% raw %}
```conf
geoip2 /usr/share/GeoIP/GeoLite2-Country.mmdb {
auto_reload 5m;
$geoip2_metadata_country_build metadata build_epoch;

$geoip2_data_country_code country iso_code;
$geoip2_data_country_name country names en;
}

geoip2 /usr/share/GeoIP/GeoLite2-City.mmdb {
$geoip2_data_city_name city names en;
}
fastcgi_param COUNTRY_CODE $geoip2_data_country_code;
fastcgi_param COUNTRY_NAME $geoip2_data_country_name;
fastcgi_param CITY_NAME    $geoip2_data_city_name;

map $geoip2_data_country_code $blacklisted_country {
default no;
CN yes;  #yes 就是封鎖
HK yes;  #yes 就是封鎖
} 
```
{% endraw %}

### server 區塊
在 server 區塊新增下面這串:  

{% raw %}
```conf
if ($blacklisted_country = yes) 
    {
      return 400;
    }
```
{% endraw %}

## 七、測試是否成功

上面在 nginx.conf 的 http 區塊有設定 `CN yes;` 和 `HK yes;`，代表：當造訪的 IP 是來自於 CN 跟 HK，就會收到 400 錯誤。  

```conf
CN yes;  #yes 就是封鎖
HK yes;  #yes 就是封鎖
```

可以利用 `curl` 工具來測試這個模組及設定有沒有運作。

```bash
# -I: 顯示 Header
# -L: 是跟著301走（會轉址）

$ curl -IL xxxxx.com
$ curl -IL x.x.x.x
```

### 實際測試

本篇文章使用的主機是來自 HK ，因此可以試著自己造訪自己，看看結果如何？ 

在下圖可以看到，要造訪這台主機，結果會報 400 Bad Request 錯誤，所以 GeoIP2 模組的確有在好好運作～   
![](/assets/images/2022-08-11-Nginx-Geoip2-module-25/11.JPG)


那如果把 `HK yes;` 改成 `HK no;` 會如何呢？

```conf
CN yes;  #yes 就是封鎖
HK no;  #no 就是不封鎖
```

在下圖可以看到主機回覆了 403 錯誤，這是正常的，因為我沒在裡面放東西。   

不過由此可知，如果沒有造訪的 IP 沒有在我們制定的名單內，就可以正常造訪主機或網站。    

![](/assets/images/2022-08-11-Nginx-Geoip2-module-25/12.JPG)



## 參考資料
- [Upgrade to GeoIP2 with NGINX on CentOS/RHEL](https://www.getpagespeed.com/server-setup/nginx/upgrade-to-geoip2-with-nginx-on-centos-rhel) 
- [YUM/DNF Remove Repo – YUM/DNF Disable Repository](https://www.if-not-true-then-false.com/2010/yum-remove-repo-repository-yum-disable-repo-repository/) 
- [GeoLite2 Sign Up](https://www.maxmind.com/en/geolite2/signup?utm_source=kb&utm_medium=kb-link&utm_campaign=kb-create-account) 
- [Sign up for free GeoLite2 databases and web services
](https://support.maxmind.com/hc/en-us/articles/4407099783707-Create-an-Account#h_01G4G4NV169TJWFCJ1KGFAM1CD) 
- [mmdblookup](https://maxmind.github.io/libmaxminddb/mmdblookup.html) 
- [How install the Geoip2 module on a Nginx running in a production environment?](https://stackoverflow.com/questions/62213884/how-install-the-geoip2-module-on-a-nginx-running-in-a-production-environment) 
