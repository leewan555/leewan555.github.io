---
title:  "架設自己的 Docker Registry"
slug: Create-docker-registry-24
date:   2022-07-11
excerpt: 可以架設自己的小倉庫耶～
categories:
  - Docker
tags:
  - centos
  - docker
  - container
  - docker registry
---

## 使用的環境

| 系統與使用工具 | 
| ----- |  
| Centos 7.6 | 
| Docker 1.13.1 | 


## 一、Docker Registry 是什麼？

- 開放原始碼的軟體
- 提供存放 Docker Image 的空間，可與他人共用
- 可以分為公開或私人的 Registry

## 二、Docker Registry v.s. Docker Distribution

Docker Registry 1.0 - Docker Registry 專案  
Docker Registry 2.0 - Docker Distribution 專案  

Docker Distribution 專案取代並增強 Docker Registry 專案，同時也朝向更完善且有彈性的 Docker Image 整合管理工具。  

### 1. Docker Distribution 專案新增的元件  

- 函式庫：原本由 python 開發，現在改用 golang  
- 規範：相關規範都可以在 Docker Distribution 的 Github Repo 的 docs/spec 資料夾找到  
- 文件：docs 資料夾裡有 docs.docker.com 和 Docker Distribution 相關的說明文件 
  
### 2. Docker Distribution 專案功能的增益項目  

- 更快的 `docker push` 和 `docker pull` 執行速度  
- 改善並提升 Docker Registry 的整體運作效能 
- 簡化 Docker Registry  部署方式  
- 提供可插拔的儲存庫後端，可自行選擇上傳後的 Docker Image 儲存方式和位置，可與多種雲端儲存服務整合（AWS S3...等等）  
- 提供 webhook 的通知機制  

## 三、Docker Hub 和 Docker Cloud
Docker Hub 是 Docker 官方提供的免費服務，主要用途是提供 Docker Image 的儲存庫。  

`docker search` 和 `docker pull` 所存取的 Docker Registry 就是 [Docker Hub](https://hub.docker.com "Docker Hub")。  

Docker Cloud 除了利用 Docker Hub 的儲存庫功能，主要用途是提供 Docker 化應用系統（Dockerized application）的完整架構與自動化服務。  

Docker 化（Dockerized）：把應用程式 Docker 化，把程式跟環境包成一個 image，部署的時候就直接使用這個 image 不需要額外安裝其他東西。  

如果所開發的應用系統已經 Docker 化或準備 Docker 化，且運行在雲端運算環境，那 Docker Cloud 就可以協助自動化和加速整個應用系統在建版、測試、部署和節點管理上的工作。  

Docker Hub 和 Docker Cloud 的帳號是共用的，不需要分別註冊，且可以直接在 Docker Cloud 中使用上傳到 Docker Hub 的 Docker Image。  

## 四、為何不用 Docker Hub 就好？

Docker Hub 是免費且公開的服務，但 Docker Image 的散佈和部署會遇到機密敏感資料與安全考量，以及使用環境的不可用性問題。  

- 機密敏感性的安全問題：Docker Image 可能包含程式碼或不適宜公開的環境設定資訊
- 公司或組織的環境問題：不同環境有不同考量，譬如開發環境、整合測試、使用者驗證環境、內外網連線以及儲存管理部署 Docker Image 的需求，就會需要在公司內架設專用的 Docker Registry 


## 五、架設 Docker Registry

### 1. 下載及啟動 Docker Registry 的 Docker Image

架設 Docker Registry 就是啟用一個 Docker Registry 的 Docker Container。  

```bash
# -p 5000：將主機的 5000 port mapping 到 container 的 5000 port
$ docker pull registry:2
$ docker run -d -p 5000:5000 --name registry registry:2
```

也可以用 `-v` 掛載資料卷，因為如果 Docker Registry 的資料是放在 Container 裡面，刪掉 Container 時裡面的資料就會跟著不見，所以需要使用 `–v` 將主機的檔案路徑 mapping 到 Container 裡面的檔案路徑，這樣就算 Docker Container 被刪除， Docker Registry 的 Image 資料還會存在。  

```bash
$ docker run -d -p 5000:5000 -v /usr/local/docker/registry:/var/lib/registry --name registry registry:2
```

![](/assets/images/2022-07-11-Create-docker-registry-24/1.jpg) 


### 2. 用指令或瀏覽器確認 Docker Registry 是否啟動成功
也可以檢查版本是否為 V2，如果傳回「{ }」，就表示 Docker Registry 已經成功運作。   
```bash
$ curl -LX GET 127.0.0.1:5000/v2
{}
````

![](/assets/images/2022-07-11-Create-docker-registry-24/2.jpg) 


也可以直接用瀏覽器開啟，如果正常啟動一樣會看到「{}」。  
[http://127.0.0.1:5000/v2](http://127.0.0.1:5000/v2 "http://127.0.0.1:5000/v2")


如果不是在架設 Docker Registry 的電腦上執行，可以把 IP 位址換成實際的位址：
[http://x.x.x.x:5000/v2/](http://x.x.x.x:5000/v2 "http://x.x.x.x:5000/v2")


![](/assets/images/2022-07-11-Create-docker-registry-24/3.jpg) 


## 六、將 Docker Image 上傳到 Docker Registry
以下步驟先用 nginx 的 Docker Image 當作範例。  

### 1. 標記 Docker Image

```bash
$ docker tag nginx 127.0.0.1:5000/nginx_local

# 如果不是在架設 Docker Registry 的電腦上執行，可以把 IP 位址換成實際的位址
$ docker tag nginx x.x.x.x:5000/nginx_local

# 查看標記後的 Image
$ docker images
```
![](/assets/images/2022-07-11-Create-docker-registry-24/4.jpg) 

### 2. 將 Docker Image push 到 Docker Registry Server
```bash
$ docker push 127.0.0.1:5000/nginx_local
```

在這邊遇到問題，所以沒有成功將 Docker Image Push 到 Docker Registry 上。

![](/assets/images/2022-07-11-Create-docker-registry-24/5.jpg)

這個錯誤訊息主要是因為安全性上的問題，需要修改 client 端的 Docker 設定。

`insecure-registries`：insecure 的意思是非安全性，所以如果是使用 http 協定的 docker registry 就需要設定此參數。

```bash
$ vim /etc/docker/daemon.json

{
"insecure-registries": ["x.x.x.x:5000"]
}
````

![](/assets/images/2022-07-11-Create-docker-registry-24/6.jpg)


重新啟動 Docker，並重新 push docker image。
```bash
$ systemctl restart docker

$ docker push 127.0.0.1:5000/nginx_local
```

![](/assets/images/2022-07-11-Create-docker-registry-24/7.jpg)

用瀏覽器開啟查詢：  

[http://127.0.0.1:5000/v2/_catalog](http://127.0.0.1:5000/v2/_catalog "http://127.0.0.1:5000/v2/_catalog")

[http://x.x.x.x:5000/v2/_catalog](http://x.x.x.x:5000/v2/_catalog "http://x.x.x.x:5000/v2/_catalog")

![](/assets/images/2022-07-11-Create-docker-registry-24/8.jpg)


如果要下載剛剛上傳的 nginx_local，就用 `docker pull` 抓下來。

```bash
$ docker pull 127.0.0.1:5000/nginx_local
$ docker pull x.x.x.x:5000/nginx_local
```

## 參考資料
- Docker 這樣學才有趣：從入門，到玩直播、挖礦