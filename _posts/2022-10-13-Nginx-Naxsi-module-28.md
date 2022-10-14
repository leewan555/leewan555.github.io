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


## 一、WAF 介紹
WAF 會分析網路行為，可以保護自己不小心製造出的漏洞，但只能即時保護應用程式，不能修復漏洞，不過在防禦的同時，可以有緩衝時間修復應用程式的漏洞。

方式：依據事先設計好的安全政策，發掘違反安全政策的封包。  
目的：保護web應用程式，防禦 XSS 及 SQL injection 等攻擊。  

 - 支持 POST/GET  
 - 位於網頁瀏覽者與網頁伺服器中間，專責分析與過濾 「Layer7 應用層」 的網路流量  
 - 有些更強大的甚至可以掃描惡意木馬文件、防竄改、伺服器優化、備份  
 - 如果網站有蒐集 cookie、用戶資料、表單紀錄，建議使用 WAF  
      

## 二、WAF 優缺點

| 優點 | 缺點 |  
| ----- | ----- |   
| 1. 開箱即用，便宜甚至免費 | 1. 誤殺、漏報 |  
| 2. 管理方便，介面友好 | 2. 只適合中小型網站 |  
| 3. 功能豐富 | 3. 必須客製化規則才能有效的抵擋攻擊 |  
|  | 4. 功能強大的 WAF 很貴 |  

## 三、Naxsi 模組介紹
Naxsi 是 Nginx 的第三方模組，與任何 Nginx 版本都相容，採用 GPLv3 授權，可以免費使用，也不需要依賴類似防毒軟體的病毒碼資料庫。  

可以建置簡易的 WAF 系統，阻擋一些常見的 Nginx Anti XSS 及 SQL Injection 攻擊，但只能過濾「GET」及「POST」的請求。    

## 四、Naxsi 的規則檔 naxsi_core.rules

舉例 naxsi_core.rules 內的部份規則：  

{% raw %}
```config
##################################
## SQL Injections IDs:1000-1099 ##
##################################
MainRule "rx:select|union|update|delete|insert|table|from|ascii|hex|unhex|drop|load_file|substr|group_concat|dumpfile" "msg:sql keywords" "mz:BODY|URL|ARGS|$HEADERS_VAR:Cookie" "s:$SQL:4" id:1000;
MainRule "str:\"" "msg:double quote" "mz:BODY|URL|ARGS|$HEADERS_VAR:Cookie" "s:$SQL:8,$XSS:8" id:1001;
MainRule "str:0x" "msg:0x, possible hex encoding" "mz:BODY|URL|ARGS|$HEADERS_VAR:Cookie" "s:$SQL:2" id:1002;
## Hardcore rules
MainRule "str:/*" "msg:mysql comment (/*)" "mz:BODY|URL|ARGS|$HEADERS_VAR:Cookie" "s:$SQL:8" id:1003;
MainRule "str:*/" "msg:mysql comment (*/)" "mz:BODY|URL|ARGS|$HEADERS_VAR:Cookie" "s:$SQL:8" id:1004;
MainRule "str:|" "msg:mysql keyword (|)"  "mz:BODY|URL|ARGS|$HEADERS_VAR:Cookie" "s:$SQL:8" id:1005;
MainRule "str:&&" "msg:mysql keyword (&&)" "mz:BODY|URL|ARGS|$HEADERS_VAR:Cookie" "s:$SQL:8" id:1006;
## end of hardcore rules
MainRule "str:--" "msg:mysql comment (--)" "mz:BODY|URL|ARGS|$HEADERS_VAR:Cookie" "s:$SQL:4" id:1007;
MainRule "str:;" "msg:semicolon" "mz:BODY|URL|ARGS" "s:$SQL:4,$XSS:8" id:1008;
MainRule "str:=" "msg:equal sign in var, probable sql/xss" "mz:ARGS|BODY" "s:$SQL:2" id:1009;
MainRule "str:(" "msg:open parenthesis, probable sql/xss" "mz:ARGS|URL|BODY|$HEADERS_VAR:Cookie" "s:$SQL:4,$XSS:8" id:1010;
MainRule "str:)" "msg:close parenthesis, probable sql/xss" "mz:ARGS|URL|BODY|$HEADERS_VAR:Cookie" "s:$SQL:4,$XSS:8" id:1011;
MainRule "str:'" "msg:simple quote" "mz:ARGS|BODY|URL|$HEADERS_VAR:Cookie" "s:$SQL:4,$XSS:8" id:1013;
MainRule "str:," "msg:comma" "mz:BODY|URL|ARGS|$HEADERS_VAR:Cookie" "s:$SQL:4" id:1015;
MainRule "str:#" "msg:mysql comment (#)" "mz:BODY|URL|ARGS|$HEADERS_VAR:Cookie" "s:$SQL:4" id:1016;
MainRule "str:@@" "msg:double arobase (@@)" "mz:BODY|URL|ARGS|$HEADERS_VAR:Cookie" "s:$SQL:4" id:1017;
```
{% endraw %}

## 五、Naxsi 計分方式與攔截分數介紹
### 1. 計分方式

> 這是 `naxsi_core.rules` 內的部份規則。  
{: .prompt-info }

{% raw %}
| 規則 | 規則解說 |  
| ----- | ----- |       
| id 為 1001 的規則 | `MainRule "str:\"" "msg:double quote" "mz:BODY|URL|ARGS|$HEADERS_VAR:Cookie" "s:$SQL:8,$XSS:8" id:1001;` <br> 表示如果在請求體(BODY)，統一資源定位符(URL)，請求參數(ARGS)，請求標題(Cookie)任何地方出現了雙引號(")，就把該請求可能是 SQL 注入或是跨站腳本攻擊的判斷分數分別設置為 8 |  
| id 為 1002 的規則表示 | `MainRule "str:0x" "msg:0x, possible hex encoding" "mz:BODY|URL|ARGS|$HEADERS_VAR:Cookie" "s:$SQL:2" id:1002;` <br> 表示如果在請求體(BODY)，統一資源定位符(URL)，請求參數(ARGS)，請求標題(Cookie)任何地方出現了雙引號(")，就把該請求可能是 SQL 注入或是跨站腳本攻擊的判斷分數分別設置為2 |     
| id 為 1013 的規則表示 | `MainRule "str:'" "msg:simple quote" "mz:ARGS|BODY|URL|$HEADERS_VAR:Cookie" "s:$SQL:4,$XSS:8" id:1013;` <br> 表示如果在請求體(BODY)，統一資源定位符(URL)，請求參數(ARGS)，請求標題(Cookie)任何地方出現了單引號(')，那麼就把該請求可能是 SQL 注入的判斷分數設置為 4，並且把跨站腳本攻擊的判斷分數設置為 8 |  

{% endraw %}

### 2. 攔截分數（建立網站防護規則）

> 這是在 `nginx.conf` 內的設定，最終分數可以自訂，一旦累積的分數到達設定的標準，就會攔截並回報錯誤，詳細設定方式底下介紹。  
{: .prompt-info }

```config
# 開啟 naxsi
SecRulesEnabled;

# 學習模式 預設關閉
#LearningMode; 

# 透過 libinjection 偵測 SQL 注入和 XSS 攻擊
LibInjectionSql;
LibInjectionXss;

# 拒絕訪問時展示的頁面 
DeniedUrl "/RequestDenied";  

# 設定 naxsi 何時行動，當次數到8(或5)次後就阻擋，設定阻擋的分數，可以自行調整，當請求達到關鍵分數時，該請求將被拒絕
CheckRule "$SQL >= 8" BLOCK;  
CheckRule "$RFI >= 8" BLOCK;
CheckRule "$TRAVERSAL >= 4" BLOCK;
CheckRule "$UPLOAD >= 4" BLOCK;
CheckRule "$XSS >= 8" BLOCK;

# naxsi 的 log 位置設定
error_log /home/wwwlogs/naxsi_attach.log;
```


## 六、安裝 Naxsi 模組

### 1. 安裝必要套件
```bash
$ yum groupinstall "development tools" -y
```

### 2. 安裝 PCRE 以及 openssl 套件
```bash
$ yum install pcre pcre-devel openssl openssl-devel -y
```

### 3. 在想要的路徑下載 Naxsi 
我是安裝在 `/usr/local/nginx_module` 內。

```bash
$ git clone https://github.com/nbs-system/naxsi.git
```
![](/assets/images/2022-10-13-Nginx-Naxsi-module-28/1.JPG)

### 4. 路徑整理
```config
# 從 Github 下載的 naxsi 位置
/usr/local/naxsi

# naxsi主要資料夾
/usr/local/naxsi/naxsi_src

# naxsi 規則檔
/usr/local/naxsi/naxsi_config/naxsi_core.rules
```

### 5. 編譯 Nginx
要在有 configure 的那個資料夾底下編譯，模組安裝路徑可以用絕對路徑比較保險，
使用 `--add-module` 把 naxsi 加進去。  

{% raw %}
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
 --add-dynamic-module=/usr/local/nginx_module/ngx_http_geoip2_module \
 --add-dynamic-module=/usr/local/nginx_module/naxsi/naxsi_src

$ make 

# 有些人說不要 make install，但我用 make module 或純粹 make 都沒有用，不知道原因
$ make install
```
{% endraw %}


編譯成功後長這樣：  

{% raw %}
```bash
$ nginx -V

nginx version: nginx/1.16.1
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC)
built with OpenSSL 1.1.1d  10 Sep 2019
TLS SNI support enabled
configure arguments: --user=www --group=www --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-http_v2_module --with-http_gzip_static_module --with-http_sub_module --with-stream --with-stream_ssl_module --with-openssl=/root/lnmp1.6/src/openssl-1.1.1d --with-openssl-opt=enable-weak-ssl-ciphers --add-dynamic-module=/usr/local/nginx_module/ngx_http_geoip2_module --add-dynamic-module=/usr/local/nginx_module/naxsi/naxsi_src
```
{% endraw %}


![](/assets/images/2022-10-13-Nginx-Naxsi-module-28/2.JPG)

### 6. 將 Naxsi 主要設定檔複製進 Nginx 資料夾中
```bash
$ cp /usr/local/nginx_module/naxsi/naxsi_config/naxsi_core.rules  /usr/local/nginx/conf
```

### 7. 新增一個 Naxsi 規則檔案

這是上面攔截分數把他包成一個檔案 `naxsi_custom.rules`，到時候再 include 到 nginx.conf 裡面。

> 底下設定在上面第 2 節有介紹。  
{: .prompt-info }

{% raw %}
```bash
$ vim /usr/local/nginx/conf/naxsi_custom.rules

SecRulesEnabled;
#LearningMode;
LibInjectionSql;
LibInjectionXss;
DeniedUrl "/RequestDenied";

CheckRule "$SQL >= 8" BLOCK;
CheckRule "$RFI >= 8" BLOCK;
CheckRule "$TRAVERSAL >= 4" BLOCK;
CheckRule "$UPLOAD >= 4" BLOCK;
CheckRule "$XSS >= 8" BLOCK;
error_log /home/wwwlogs/naxsi_attach.log;
```
{% endraw %}

![](/assets/images/2022-10-13-Nginx-Naxsi-module-28/3.JPG)

### 8. 開始設定 
#### (1) nginx.conf
```bash
$ vim /usr/local/nginx/conf/nginx.conf
```

{% raw %}
裡面設定：  
```config
# 在全區塊 pid 下面新增這句：
load_module modules/ngx_http_naxsi_module.so;

# 在 http include rules
http
  {
    include /usr/local/nginx/conf/naxsi_core.rules;
  }


# 在 server include rules
server 
  {
  location /
      {
        include /usr/local/nginx/conf/naxsi_custom.rules;
      }

  location /RequestDenied  # 要搭配上面的 DeniedUrl "/RequestDenied";
      {
        return 400; # 被阻擋後顯示錯誤 400
      }
  }
```
{% endraw %}

![](/assets/images/2022-10-13-Nginx-Naxsi-module-28/4.JPG)
![](/assets/images/2022-10-13-Nginx-Naxsi-module-28/5.JPG)

#### (2) vhost.conf
```bash
$ vim /usr/local/nginx/conf/vhost/xxx.conf
```

裡面設定：  
```config
server 
  {
  location /
      {
        include /usr/local/nginx/conf/naxsi_custom.rules;
      }

  location /RequestDenied  # 要搭配上面的 DeniedUrl "/RequestDenied";
      {
        return 400; # 被阻擋後顯示錯誤 400
      }
  }
```

![](/assets/images/2022-10-13-Nginx-Naxsi-module-28/6.JPG)

### 9. 重新啟動或重新讀取 Nginx
```bash
# 重新啟動 nginx
$ nginx -s reopen

# 重新讀取 nginx
$ nginx -s reload
```
## 七、實際測試
### 1. 瀏覽器上測試

圖片 

在有設定 Naxsi 的網站上面打這些 SQL 注入的請求：
```
http://url/=<>
http://url/?a=<>
http://localhost/?...<>
http://localhost/?id=<>
```
### 2. 主機上測試 
在終端機上面存取有設定 Naxsi 的網站：
```config
$ curl "http://47.52.76.54/?a=<>"
$ wget "https://newgame.fish0915.com/?<>"
```

## 八、攔截成功畫面與分析

### 1. 攔截畫面
觀察網站收到 SQL 注入的請求後有沒有報400。 

圖片 

### 2. log 分析

查看設定的 `naxsi_attach.log`，若在 log 中出現 NAXSI_FMT 開頭，就是啟用成功。    
挑 2020/06/12 05:10:30 那行 log 來當範例。

圖片


且對應到的規則是以下兩行：  

{% raw %}
```config
MainRule "str:;" "msg:semicolon" "mz:BODY|URL|ARGS" "s:$SQL:4,$XSS:8" id:1008;  
MainRule "str:," "msg:comma" "mz:BODY|URL|ARGS|$HEADERS_VAR:Cookie" "s:$SQL:4" id:1015;
```
{% endraw %}


{% raw %}

| log | log 分析 |  
| ----- | ----- |       
| 查看 log 開頭有沒有 NAXSI_FMT | 2020/06/12 05:10:30 [error] 23090#0: *186 NAXSI_FMT:   | 
| 對方IP | ip=2.47.10.131& | 
| 請求伺服器 IP　| server=127.0.0.1& | 
| uri | uri=/adv,/cgi-bin/weblogin.cgi& | 
| 學習模式(有打開才會出現) | learning | 
| naxsi 版本　| vers=0.56& |  
| 總共請求 9 次　| total_processed=9& |  
| 總共阻擋 3 次　| total_blocked=3& |  
| 設定：攔截　| config=block& |  
| 分數標籤：SQL　| cscore0=$SQL& |  
| SQL 分數　| score0=8& |  
| 分數標籤：XSS　| cscore1=$XSS& | 
| XSS分數 | score1=8& |  
| 第一個在 URL 裡　| zone0=URL& |   
| 符合的規則 ID0　| id0=1015& |   
| 符合的變數名0 | var_name0=& |   
| 第二個在 ARGS 裡(GET裡全部的引數)　| zone1=ARGS& | 
| 符合的規則 ID1　 | id1=1008& |
| 符合的變數名1| var_name1=username, |   
| 對方 IP | client: 2.47.10.131, |   
| | server: _, |    
| 請求內容 | request: "GET /adv,/cgi-bin/weblogin.cgi?username=admin%27%3Bls%20%23&password=asdf HTTP/1.1", |    
| | host: "127.0.0.1" |   


{% endraw %}


## 九、(額外)開啟 NAXSI_EXLOG
開啟 NAXSI_EXLOG，可以記錄具體觸發 naxsi 攔截規則的請求內容，內容紀錄正常和異常的請求，方便後續分析攔截的是攻擊請求還是誤判。

{% raw %}

| log | log 分析  
| ----- | ----- |       
| NAXSI_FMT（原本就有） | 僅包含 ID 和異常的位置 | 
| NAXSI_EXLOG | 提供了實際的內容，可以輕鬆確認它是否為誤報 | 

{% endraw %}

### 1. NAXSI_EXLOG 設定

在 conf 檔裡的 server 部分新增設定：
```bash
$ vim /usr/local/nginx/conf/nginx.conf

server 
  {
    set $naxsi_extensive_log 1;  
  }
```

### 2. NAXSI_EXLOG 的 log 畫面 

圖片

## 十、(額外)將 Naxsi 加入 Fail2ban
### 1. Fail2ban 新增 naxsi 過濾器
filter 的 conf 名稱可以自訂。 

圖片
```bash
$ vim /etc/fail2ban/filter.d/nginx-naxsi.conf
```

```config
[INCLUDES]
before = common.conf

[Definition]
failregex = NAXSI_FMT: ip=<HOST>&server=.*&uri=.*&learning=0
            NAXSI_FMT: ip=<HOST>.*&config=block
ignoreregex = NAXSI_FMT: ip=<HOST>.*&config=learning
```

### 2. Fail2ban 主要設定檔設定
```bash
$ vim /etc/fail2ban/jail.local
```

```config
[nginx-naxsi]
enabled = true
filter = nginx-naxsi
action = iptables-multiport[name=nginx-naxsi, port="http,https", protocol=tcp]
logpath = /home/wwwlogs/naxsi_attach.log
maxretry = 6
bantime = -1 
```


## 十一、(額外)白名單
Naxsi 社區提供一些常用的白名單規則，例如:wordpress。  

[naxsi-rules Github](https://github.com/nbs-system/naxsi-rules "naxsi-rules  Github")

然後將規則 include 到 server 內的 location，重啟 nginx 即可，  

不過要注意一些這些白名單的修改日期，有些太老。   
  



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
