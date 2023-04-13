---
title:  "Nginx 限制請求的速度"
slug:  2022-12-20-Nginx-http-limit-req-module-29
date:   2022-12-20
excerpt: 人外有天，牆外有牆。
categories:
  - Nginx
tags:
  - nginx
  - limit
  - module

---

## 使用的環境

| 系統與使用工具 | 
| ----- |  
| Centos 7.6 | 
| nginx/1.16.1 |
| Naxsi 1.3 |  


## 一、ngx_http_limit_req_module 寫法介紹

```config
# 保存 10MB 的 ip 的 request 紀錄，每分鐘可以接受 100 個 request。
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=100r/m;
```

- `$binary_remote_addr` 指的是 client 端的 IP，根據 client 端的 IP 進行限流。

- `zone=mylimit:10m` limiter(zone) 叫做 mylimit，並且最多可以使用 10MB 的儲存空間來儲存每個 IP 的請求次數（10MB 可以存十幾萬個 IP）。

- `rate=100r/m` 每分鐘一百個請求。但要注意的是 nginx 會幫你換算成多少毫秒可以接受一個新請求，譬如說每分鐘一百個，就會幫你換算成 600 毫秒一個請求，因此如果你在 600 毫秒內同時發了十個請求，那就只有第一個會被接受，其他九個都會直接被拒絕掉。


## 二、Nginx.conf 設定
> 如果要真實 IP 可以把 `$binary_remote_addr`換成`$http_x_forwarded_for`，除了`remote_addr`，`HTTP_CLIENT_IP`和`http_x_forwarded_for`可以被偽造。  
`remote_addr`雖然可以取得安全可靠的 IP，但無法確定它是否為使用者真正的IP。  
{: .prompt-warning }

```config
http
    {
      limit_req_zone $binary_remote_addr  zone=mylimit:1m rate=2r/s;
    }
```

## 三、Vhost.conf 設定
去網址的 conf 檔設定，可設定 location 路徑再套用限速規則。
```config
location /login
    {
       limit_req zone=mylimit;
    }
```

## 四、 reload Nginx
```bash
$ nginx -s reload
```

## 五、查看是否有設定成功
去網站狂按 F5，看會不會出現 503，意思是「流量過大或正在維修」，有的話即為設定成功。
```
503  Service Temporarily Unavailable 
```
![](/assets/images/2022-12-20-Nginx-http-limit-req-module-29/1.jpg)


## 六、額外：也可以一次宣告多個 zone
```config
# 也可以一次宣告多個 zone
http {
    limit_req_zone $binary_remote_addr zone=limit_verifyPhone:10m rate=2r/m;
    limit_req_zone $binary_remote_addr zone=limit_resetPassword:10m rate=1r/m;


    server {
    # 也可以套用在 POST 請求上
          location /api/verifyPhone 
      {
                limit_req zone=limit_verifyPhone;
              proxy_pass http://backend;
         }    
        location /api/resetPassword 
    {
              limit_req zone=limit_resetPassword;
             proxy_pass http://backend;
            }
     }
}
```


## 參考資料
- [Module ngx_http_limit_req_module](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html) 
- [[Web] Nginx limit_req 指南 – ngx_http_limit_req_module](https://code.yidas.com/nginx-ngx_http_limit_req_module/) 
- [NGINX Rate Limiting: 使用limit_req_zone來限制request量](https://medium.com/evan-fang/nginx-rate-limiting-%E4%BD%BF%E7%94%A8limit-req-zone%E4%BE%86%E9%99%90%E5%88%B6request%E9%87%8F-f72936ebbbac)
- [Day09-流量限制（四）](https://ithelp.ithome.com.tw/articles/10270993?sc=iThelpR) 


