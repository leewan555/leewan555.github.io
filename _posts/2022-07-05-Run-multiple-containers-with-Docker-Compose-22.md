---
title:  "利用 Docker Compose 管理多個 Container "
slug: Run-multiple-containers-with-Docker-Compose-22
date:   2022-07-05
excerpt: 呱呱。
categories:
  - Docker
tags:
  - centos
  - docker
  - container
  - docker-compose
---

## 使用的環境

| 系統與使用工具 | 
| ----- |  
| Centos 7.6 | 
| Docker 1.13.1 | 


## 一、Docker Compose 介紹
Docker Compose 主要是來管理需要同時運行多個 Docker Container 的狀況。  

透過 docker-compose.yml 和 docker-compose 指令，加上對 Dockerfile 的支援，快速將多個 Docker Container 分別建立並串接在同一個網路下進行整合，並用簡便的指令來管理同一個 docker-compose.yml 所啟動的 Docker Container。  

## 二、什麼情況會需要用到 Docker Compose？

- 開發、測試環境或正式運行的生產環境皆可使用
- 所使用的應用系統會需要連結或整合一個以上的 Container  
- 想達到更彈性和自動化的部屬 

## 三、一個 Container 或多個 Container？
不建議將應用系統需要的服務和元件都放進同一個 Docker Container 裡面，因為設定工作會變複雜，而且這種 Docker Image 在啟動時，會需要更多的環境變數和啟動設定才能順利啟動。  

建議把應用系統需要的服務和元件都製作成不同的 Docker Image，然後透過 Docker Compose 設定、整合和啟動所需的 Container，這樣能善用分散式運算的能力，每項服務都可以依需要獨立擴充和抽拔，更有效率。  


## 四、在 CentOS 安裝 Docker Compose
Docker 和 Docker Compose 是各自獨立的程式，版本號碼也不同步，所以需要分別手動安裝。

### 1. 用 yum 套件安裝
```bash
$ yum install docker compose
```

### 2. 用 pip 安裝
```bash
# 安裝 pip
$ yum install python-pip

# 升級 pip
$ pip install --upgrade pip

# 安裝 docker compose
$ pip install docker-compose

# 建立連結檔
$ ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

安裝完後，查看 Docker Compose 版本：  
```bash
$ docker-compose -v
$ docker-compose version

docker-compose version 1.18.0, build 8dd22a9
````

## 五、用範例了解 Docker Compose 的運作方式
Docker Compose 功能主要是由 docker-compose 指令和 docker-compose.yml 組成。  

docker-compose 指令可以執行大部分的 docker 指令的選項和功能，但可以直接使用 docker-compose.yml 裡的設定值，而不用像 docker 指令是由選項一一指定。  

docker-compose.yml 是採用 YAML 檔案格式來撰寫，是一種以「鍵/值 (Key/Value)」方式指定設定變數與設定值的格式，除了 `version` 標籤外，主要是由 `services（服務）`、`volumes（資料卷）`、`networks（網路）` 等三大標籤所構成。  

- services：定義和設定此 docker-compose.yml 需要啟動的 Container 資訊，功能和所使用的標籤和 `docker run` 相似，都是用於啟動 Container。  
- volumes：定義給服務使用資料儲存資訊。  
- networks：定義給服務的連結網路資訊。  

而每個 docker-compose.yml 的檔案結構會有 `version`、`services`、`volumes`、`networks`、`configs`、`secrets` 等六個頂層指令組成，後面會講到。  

### 1. docker-compose.yml 範例

![](/assets/images/2022-07-05-Run-multiple-containers-with-Docker-Compose-22/1.JPG)  

### 2. 驗證 docker-compose.yml 內容（不會真正執行）
```bash
$ docker-compose config
# -f：指定檔案
# -q：如果一切正常，檢查完成後不顯示出任何訊息（原本會顯示完整的 docker-compose.yml 內容）

$ docker-compose config -q
$ docker-compose -f /usr/local/docker-test/node-red/docker-compose.yml config -q
```
![](/assets/images/2022-07-05-Run-multiple-containers-with-Docker-Compose-22/2.JPG)  

### 3. 啟動 docker-compose.yml 內所有的服務
> 用 docker-compose.yml 啟動服務，services 名稱前面預設是資料夾名，可以加上 `-p` 選項指定專案名稱。  
{: .prompt-info }

```bash
# -d：在背景模式執行，如果沒有加上 -d，Container 就會在前景執行
$ docker-compose up -d
$ docker-compose -p 專案名稱 up -d
```
![](/assets/images/2022-07-05-Run-multiple-containers-with-Docker-Compose-22/3.JPG)    

![](/assets/images/2022-07-05-Run-multiple-containers-with-Docker-Compose-22/4.JPG)    

![](/assets/images/2022-07-05-Run-multiple-containers-with-Docker-Compose-22/5.JPG)    

![](/assets/images/2022-07-05-Run-multiple-containers-with-Docker-Compose-22/6.JPG)    

可以看到剛建立的資料卷及網路前面都會加上「nodered_」字串，是因為那個字串預設會是 docker-compose.yml 所在資料夾的名稱，並會移除資料夾名稱中的非英數字元，如果想指定專案名稱可以加上 `-p` 選項。    

### 4. 執行 docker-compose.yml 內的服務
> docker-compose 指令幾乎只接受 **services 名稱**，而不是 Container 名稱，所以不一定要特別設定 Container 名稱，這樣也可以避免 Container 名稱衝突而無法建立成功。  
{: .prompt-info }

```bash
$ docker-compose exec 
$ docker-compose exec -it db /bin/bash （db 是 services 名稱）
```
> 注意！如果是用 `docker exec` 就要使用 Container 名稱。 
> `docker exec -it node-red-counchdb`
{: .prompt-warning }


## 六、Dockerfile 和 docker-compose.yml 差異

| 名稱 | 所屬的腳本檔 | 用途 | 
| ---- | ---- | ---- | 
| Dockerfile | docker build | 注重在如何產生一個新的 Image | 
| docker-compose.yml | Docker Compose | 比較像 `docker run`，用來告訴 Docker 如何啟動、運行和連結 Container | 

## 七、docker-compose.yml 常用指令
### 1. version
**用途：指定該 docker-compose.yml 檔案所使用的格式版本。**

不同的 Docker 及 Docker Compose 版本所支援的 docker-compose.yml 格式和指令略有不同，而 docker-compose.yml 共有三個主要版本，最新版本是 Version 3.8 (2022/07)，各版本差異主要是在 yaml 檔案可以使用的指令數不同，版本越新支援越多指令。  

docker-compose.yml 版本和 Docker 版本相關，舊版 Docker 上無法用新版的 docker-compose.yml 格式。  

```yml
version: "3"
version: "3.0"
version: "3.8"
```

### 2. services
**用途：指定該 docker-compose.yml 檔案需要啟動的所有的 Container 的啟動及設定資訊。**

```yml
services:
  node-red: 
    image: xxxxxxxx
    container_name: xxxxxxxxx
```

### 3. container_name
**用途：指定每一個 service 下的 Container 名稱。**

不過由 docker-compose.yml 啟動的 Container 都可以透過 docker-compose 指令和 services 名稱進行操作，因此 Container 的名稱就不用非要指定。  

```yml
services:
  node-red: 
    image: xxxxxxxx
    container_name: node-red-web
```

### 4. image
**用途：指定某一個 service 所要使用的 image 檔案名稱和標籤。**

> 用法跟 `docker run` 和 Dockerfile 的 `FROM` 相同。
{: .prompt-info }

```yml
image: nginx
image: nginx:1.13.3
image: ayubiz/nginx
image: ayubiz/nginx:1.13.3
image: f4114aea
```


### 5. build
**用途：另一種取得 image 的方式，只要指定 Dockerfile 所在的資料夾即可。**

#### (1) 如果檔案名稱叫做 Dockerfile
```yml
services:
  web:
    build: /home/ayubiz
```

#### (2) 如果檔案名稱是自訂的，要多加一個 dockerfile 指令
```yml
services:
  web:
    build: /home/ayubiz
    dockerfile: ayu.txt
```

#### (3) 如果需要更改產出的 image 檔案的名稱和標籤，可以同時使用 build 和 image 指令
```yml
services:
  web:
    build: . (目前所在資料夾)
    image: ayubiz:demo
```

### 6. command
**用途：指定 Container 啟動後執行的指令，用以取代掉 Image 檔案內原有的啟動指令（改變 Container 原有的執行指令）。**

> 用法跟 Dockerfile 的 `CMD` 相同。
{: .prompt-info }

```yml
services:
  http:
    image: nginx
    command: ["nginx", "-g", "daemon off;"]
```
或是
```yml
services:
  http:
    image: nginx
    command: nginx -g "daemon off;"
```

### 7. entrypoint
**用途：覆寫掉 Image 檔案內原有的 ENTRYPOINT 指令。**

> 用法跟 Dockerfile 的 `ENTRYPOINT` 類似。
{: .prompt-info }

```yml
services:
  http:
    image: nginx
    entrypiont:
      - nignx
      - g
      - "daemon off;"
```
或是
```yml
services:
  http:
    image: nginx
    command: nginx -g "daemon off;"
```

### 8. environment
**用途：用以設定該 services 的環境變數。**

> 用法跟 docker run -e 或是 Dockerfile 的 `ENV` 類似。
{: .prompt-info }

平常設定值不需要使用雙引號，不過 YAML 檔會解析 `true/false` 及 `yes/no` 這兩組關鍵字，所以會需要加上雙引號避免被轉換成布林值而產生錯誤。

```yml
services:
  http:
    image: nginx
    environment:
      USER: ayubiz
      ROOT: "true"
      NO_PW:
```
或是
```yml
# 此種方式能免去減少雙引號的問題
services:
  http:
    environment:
      USER: ayubiz
      ROOT: "true"
      NO_PW:
```


### 9. env_file
**用途：將變數檔案路徑及檔案名稱事先寫在文字檔，並直接引用到 docker-compose.yml 內，可以省去用 `environment` 逐一設定的麻煩。**

> 注意！同樣的變數名稱，會以 `environment` 指令的設定為準，如果多個變數檔都有設定同一個變數名時，以最後加入的那個檔案的變數值為主。
{: .prompt-warning }

```yml
services:
  http:
    image: nginx
    env_file: aybiz.txt
```
或是
```yml
services:
  http:
    image: nginx
    env_file: 
      - ./ayubiz.txt
      - ./env/ayubiz.txt
      - /tmp/ayubiz.txt
```

### 10. expose
**用途：用以設定要啟用的連結埠號。**

> 用法跟 Dockerfile 的 `EXPOSE` 相同。
{: .prompt-info }

啟用的連結埠號除了提供給同一個 Docker Network 的其他 Container/Service 存取外，並不能直接給主機的程式使用
避免不必要的錯誤發生，加雙引號會比較安全。

```yml
services:
  http:
    image: nginx
    expose: 
      - "80"
      - "443"
```

### 11. ports
**用途：將主機的連結埠號對應到 Container/Service 所啟用（Expose）的連結埠號，如果沒有要將 Container/Service 所啟用的連結埠號開給主機上的程式或是其他不在同一個 Docker Network 的 Container 所使用就不需要用到。**

> 用法跟 docker run -p 相似。
{: .prompt-info }

```yml
services:       
  http:       
    image: nginx       
    ports:       
      - "80"  # 指定主機的 80 埠對應到 Container 的 80 埠      
      - "8080-8088"   # 指定主機的 8080 至 8088 之間的埠號對應到 Container 的 8080 至 8088 之間的埠號    
      - "80:80"   # 指定主機的 80 埠對應到 Container 的 80 埠    
      - "8080-8090:80-90"   # 指定主機的 8080 至 8090 之間的埠號對應到 Container 的 80 至 90 之間的埠號    
      - "127.0.0.1:8080:80"   # 指定主機的 127.0.0.1 網址的 8080 埠號對應到 Container 的 80 埠號    
      - "127.0.0.1:8080-8090:80-90"     # 指定主機的 127.0.0.1 網址的 8080 至 8090 之間的埠號對應到 Container 的 80 至 90 之間的埠號  
      - "16882:16882/udp"   # 指定主機的 16882 埠號對應到 Container 的 16882 埠號的 UDP 通訊協定    
```

### 12. restart
**用途：設定 Container 自動重新啟動的模式。**

> 用法跟 `docker run --restart` 相同。
{: .prompt-info }

`restart` 預設值為 `no`，其他設定值可以是 `no`、`always`、`on-failure`、`unless-stopped`。

```yml
services:
  image: nginx
  restart: always
```


### 13. volumes
**用途：用於掛載主機的資料夾和檔案或外部資料卷到 Container 的指定資料夾或檔案裡**

> 用法跟 `docker run -v` 相同。
{: .prompt-info }

#### (1) services 指令下的 volumes：指定 Container 要掛載的資料卷
```yml
services:
  http:
    image: nginx
    volumes:
      # 由 docker 自動建立相同路徑名稱的資料卷掛載到此路徑
      - /usr/share/nginx/html

      # 指定主機的檔案路徑掛載到 Container 的指定路徑
      - /home/ayubiz/nginx/html:/usr/share/nginx/html

      # 將 docker volume 建立的資料卷掛載到 Container 上
      - nginx-data:/usr/share/nginx/html
```

#### (2) 與 services 指令同層的 volumes （頂層）：建立一組新的資料卷
用頂層的 `volumes` 建立一個名為 `nginx-data` 的資料卷後，就可以使用 services 下的 `volumes` 將命名好的資料卷進行掛載。  

```yml
services:
  http:
    image: nginx
    volumes:
      - nginx-data:/usr/share/nginx/html
volumes:
    nginx-data
```

#### (3) 3.2 版以上 docker-compose.yml 檔案格式支援條列式
```yml
version: "3.2"
services:
  http:
    image: nginx
    volumes:
      - type: volume
        source: nginx-data
        target: /usr/share/nginx/html
        volume:
          nocopy: true
volumes:
  nginx-data
```


### 14. networks
**指定 Container 要加入的 Docker Network 名稱。**
#### (1) 指定 Container 要加入的 Docker Network 名稱

#### (2) 與 services 指令同層的 networks（頂層）：建立一組新的 Docker Network，提供給 services 指令所定義的 Container/Service 使用

> 可以使用頂層的 `networks` 定義多個 Docker Network，且每一個 services 下的 Container 也可以透過 `networks` 同時加入多個 Docker Network。
{: .prompt-info }

```yml
services:
  node-red:
    image: nodered/node-red-docker:slim
    networks:
      - node-red-net

  db:
    image: counchdb:latest
    networks:
      - node-red-net

networks:
  node-red-net
```

### 15. depends_on
**用途：可以改變 docker-compose.yml 檔案裡面各個 Container/Service 的啟動順序。**

只要在 `node-red` 服務下方加上下面的 `depends_on`，就可以讓 `node_red` 在 `db` 啟動完成後再啟動，但可能會發生 `node-red` 在 `db` 啟動完成後要啟動，但 Couchdb Server 還沒有啟動完成而無法連線的問題。

> 注意！啟動完成是指 Container 本身，不是 Container 內的程式或服務啟動完成。
{: .prompt-warning }

```yml
services:
  node-red:
    image: nodered/node-red-docker:slim
    depends_on:
      - "db"

  db:
    image: counchdb:latest
```

### 16. external_links | link
這兩個指令大部分已經可以透過 Docker Nerwork 功能自動達成，建議改用 `networks` 建立 Docker Network 達成相關功能。


### 17. extra_hosts
**用途：設定其他主機的 IP 和網域名稱做對應，設定完後打開 /etc/hosts 檔案會發現多出剛剛設定的紀錄。**

> 和直接去 Container 的 /etc/hosts 檔案裡加入新的主機的作用相同。
{: .prompt-info }

```yml
services:
  node-red:
    image: nodered/node-red-docker:slim
    extra_hosts:
      - "webapp1:192.168.1.10"
      - "webapp2:192.168.99.101"
```

### 18. dns
**用途：設定 DNS 伺服器，該服務啟動的 Container 的 DNS 伺服器就會被變更為設定的 DNS 伺服器位址。**

```yml
services"
  node-red:
    image: nodered/node-red-docker:slim
    dns: 9.9.9.9

  db:
    image: couchdb:latest
    dns: 
      - 168.95.1.1
       - 168.95.192.1
```

### 19. labels
**用途：將 Image 檔的作者、版本、描述、其他自訂資訊儲存到新產生的 Image 檔。**
> 用途和 Dockerfile 的 `LABEL` 相同。
{: .prompt-info }


## 八、Docker Compose 常用指令
用 docker-compose 指令操作的好處在於可以對指定的 docker-compose.yml 檔案內所啟動的 Container 做整批的操作，而不需要一個一個處理。  

docker-compose 指令必須在有 docker-compose.yml 檔案路徑和名稱才能夠正常使用。

### 1. config 
**用途：驗證或檢視  docker-compose.yml 的內容，而不用透過實際執行才能發現錯誤。**  
```bash        
$ docker-compose config
```

### 2. build
**用途：如果 docker-compose.yml 檔案裡有使用到 Dockerfile 來產生 Docker Image 的話，可以使用 `build` 來產生所需的 Docker Image，但如果 docker-compose.yml 檔案裡沒有指定 Dockerfile，用 `build` 就不會有任何產出。**

如果所使用的 Dockerfile 的內容有變動時，也可以使用 `build` 重新產生 docker image。

```bash     
$ docker-compose build          
```

### 3. pull
**用途：下載 docker-compose.yml 中指定要取得的 Image，可指定一或多個服務的名稱，若沒指定就會下載 docker-compose.yml 檔案裡所有指定的 Image 檔。**
```bash         
$ docker-compose pull node-red db         
$ docker-compose pull
```

### 4. start | stop | restart | pause | unpause | kill            
**用途：啟動、停止、重新啟動、暫停、取消暫停和強制停止 docker-compose.yml 檔案內的 Container。**          
如果不指定 Container，會預設對 docker-compose.yml 檔案內所有 Container 做操作（譬如：不指定就會停止所有 Container 的運行）。

```bash         
$ docker-compose stop         
$ docker-compose stop node-red db
```

### 5. rm
**用途：移除已經停止運行的 Container，可指定一或多個 Container 的名稱，若沒指定就會移除 docker-compose 檔案裡全部停用的 Container。**
```bash        
-s 可以移除仍在運行中的 Container         
$ docker-compose rm -v          
$ docker-compose rm -v node-red           
$ docker-compose rm -s node-red
```

### 6. run  
**用途：主要是利用 `services` 所設定好的各項設定值，然後搭配要執行的命令來啟動服務。**

docker-compose run 的選項有：`-d`、`--name`、`--rm`、`-p`、`--entrypoint`、`--workdir`、`-v`、`--no-deps`。  

`--no-deps`：啟動服務時，不去啟動相連或相依的服務。   

```bash         
# node-red 是 services 名稱不是 Image 名稱
$ docker-compose run node-red         
```

### 7. up（最常用的指令）
**用途：將撰寫好的 docker-compose.yml 建立，並啟動所有的服務。**      

用 docker-compose.yml 啟動 Container，若沒另外指定專案名稱，預設是資料夾名。

`--no-recreate`：無論  docker-compose.yml 或 Image 檔有無調整或改變，都不需要重製每個服務所使用的 Container          
`--force-recreate`：無論  docker-compose.yml 或 Image 檔有無調整或改變，都要強迫執行重新建立服務所使用的 Container 的動作         

> 注意！ `--no-recreate` 和 `--force-recreate` 對於已經啟動的服務的 Container 都會先停止運行後，再依要求只重新啟動或重建後重新啟動。 
{: .prompt-info }

```bash
# -d：在背景模式執行，如果沒有加上 -d，Container 就會在前景執行          
$ docker-compose up -d          
$ docker-compose -p 專案名稱 up  -d         
```

### 8. version  
**用途：查看 docker-compose 版本的方式。**          

### 9. logs
**用途：查看 Container 所輸出的 log 訊息。**         
跟 `docker logs` 用途和用法相同，但 `docker-compose logs` 不支援以指定的起始時間查詢 log 及額外詳細的 log 顯示方式。

### 10. port
**用途：查詢 Container 內部的通訊埠所對應的外部 IP 位址和通訊埠號。**        
與 `docker ps` 差別在於 `docker ps` 不用指定要查詢的通訊埠號。

```bash
# 查詢 node-red 服務的 1880 埠號在 Container 外所對應的 IP 位址和通訊埠號       
$ docker-compose port node-red 1880  
```

### 11. ps
**用途：顯示所使用的 docker-compose.yml 在 services 項目下定義的 Container 的狀態（包含啟動中和非啟動中），並不是所有的 Container 的狀態。**    

```bash
$ docker-compose ps         

# # 只顯示 Container ID 資訊
$ docker-compose ps -q    
```

### 12. help
**用途：查訊個別指令選項的使用方式和參數**    

```bash    
$ docker-compose start --help     

# docker 指令不支援這個操作      
$ docker-compose start -?       
```

## 九、何時要使用 Docker Compose 指令？
docker-compose 指令和 docker 指令相同，所以時常會搞不清楚要用那個指令。  

經過上面可知，docker-compose 指令操作都是基於 docker-compose.yml，如果想要操作的 Container 是由某一個 docker-compose.yml 檔案所定義和產生時，就用 docker-compose 指令來操作，可以少打一些字，因為它會自動去參考 docker-compose.yml 的相關設定。  

docker-compose 指令可以容易的進行批次操作，可以一次啟用或停用 docker-compose.yml 所定義的所有的 Container，不用像 docker 指令要一一列出想用啟用或停用的 Container。  
 
另外，docker 指令使用 Container 名稱作為指令的參數，而 docker-compose 指令則多是以 docker-compose.yml 檔案裡的「服務名稱」來作為參數使用，不直接給 Container 的名稱。  


## 參考資料
- Docker 這樣學才有趣：從入門，到玩直播、挖礦