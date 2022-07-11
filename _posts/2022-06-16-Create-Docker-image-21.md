---
title:  "各種自製 Docker Image 的方法"
slug: Create-docker-image-21
date:   2022-06-16
excerpt: Dockerfile 像在寫腳本耶。
categories:
  - Docker
tags:
  - centos
  - docker
  - container
---

## 使用的環境

| 系統與使用工具 | 
| ----- |  
| Centos 7.6 | 
| Docker 1.13.1 | 


## 一、從運行中的 Container 產生 Image 檔
> 注意事項：  
> (1) 執行 `docker commit` 時，Container 的狀態是可以停止中的。    
> (2) 掛載在 Container 的儲存空間不會被存到新產生的 Image 的，需要在 `docker run` 時再用 -v 掛載。  
{: .prompt-warning }

先找一個接近自己需要的 Docker Image 來當基底並 `docker run` 起來，並用 `docker exec` 進入這個 Container 來安裝和設定，再用 `docker commit` 把啟動後產生的變動寫到原來所使用的 Docker Image。  

```bash
# 從運行中的 Container 產生 image 檔
$ docker commit <Container 名稱或 ID> <產出的 Docker Image 名稱>

# 透過 -m 來增加說明文字，-a 加上作者資訊
$ docker commit -m "註解" -a "作者" <Container 名稱或 ID> <產出的 Docker Image 名稱>

# 製作一個把 Nginx 預設首頁的訊息從 Welcome 改成 Hello Nginx 的新 Docker Image
$ docker pull nginx
$ docker run --name nginx-welcome -p 8080:80 -d nginx
$ docker cp index.html nginx-welcome:/usr/share/nginx/html
$ docker commit nginx-welcome nginx-hello-commit
$ docker stop nginx-welcome
$ docker images
```

使用 `docker images` 就可以看到剛剛 commit 出來的 image 檔。  
![](/assets/images/2022-06-16-Create-Docker-image-21/1.JPG)  

再來執行剛剛 commit 出來的 docker-hello-commit，成功！      
![](/assets/images/2022-06-16-Create-Docker-image-21/2.JPG)  

## 二、用 Bulid 指令自動化產生新的 Image
> Dockerfile 的檔案名稱有大小寫的分別。   
{: .prompt-info }

透過 `docker build` 和 Dockerfile 就可以以全自動的方式來產生 Image 檔，  
Dockerfile 可以看成是一個腳本，用來明確告訴 `docker build` 產生新 Image 所需要用到的資訊和步驟。  

如果一切都順利，在執行結束後會看到 `Successfully built XXXXX`，表示已經成功建立！  
`docker build` 執行完成後，用 `docker images` 查詢，可以看到剛剛建立的 Image。  

```bash
# 自動化產生新的 Image
$ docker build -t  <新產生的 Image 的名稱：版本> <Dockfile 所在的檔案路徑或網址>

$ cd nginx-docker
$ docker build -t nginx-ayu:demo .
$ docker build -t nginx-ayu:demo http://xxx.xxxx.xxx.xxx
```

## 三、Dockerfile常用的指令和格式
### 1. FROM
**用途：指定要下載的 Image 檔名稱。**  

從 Docker Hub 下載 Debain 的 Jessie 版本的 Image 檔。  
```bash
FROM debain:jessie
```

### 2. ENV
**用途：設定環境變數。**  

在 Container 裡設定指定 Nginx 的 `NGINX_VERSION` 環境變數。  

之後執行 `apt-get install` 安裝指令時，會需要這個 `ENV` 所設定的變數，  
通常會把經常變動的參數設成環境變數，以便後續維護或更新時，可以不用直接修改相關的指令。  
```bash
ENV NGINX_VERSION 1.11.10-1~jessie
```

### 3. EXPOSE
**用途：指定 Container 啟用的連結埠號。**  

設定這個 Container 要啟用 80 和 443  port，透過 `EXPOSE` 啟用的連結埠不能被外部或主機連線存取，必須透過 `docker run` 的 `-p` 選項設定，主機的連線或程式才可以透過該對應的主機連結埠進行連結及存取。  
```bash
EXPOSE 80 443
```

### 4. RUN
**用途：建立 Image 時執行的指令，只有在 `docker build` 產生的時候會有作用，大部分被用在設置和安裝軟體。**  
```bash
# 利用 RUN 指令在這個暫時啟用的 Container 執行安裝的指令
RUN apt-key adv --keyserver ........
```

### 5. CMD
**用途：Container 啟動完成後要執行的指令。**    

> 注意！每個 Dockerfile 只能有一個 `CMD` 指令會有作用，若同一個 Dockfile 裡有多個 `CMD` 指令，只有 **最後一個** `CMD` 指令會有作用。  
{: .prompt-warning }

(1)（官方建議寫法）  
可用在 Dockfile 裡，但不會使用預設的 Shell 執行，不能使用 Shell 所提供的功能，像是取用環境變數或用 Shell 內建的指令。   
```bash
CMD ["<要執行的程式>","<要執行的程式用到的第一組選項>","<要執行的程式用到的第二組選項或第一組選的參數值>"]
```

(2) 可用在 Dockfile 裡，會使用預設的 Shell 執行。 
```bash
CMD <要執行的程式> <要執行的程式用到的第一組選項> <要執行的程式用到的第二組選項或第一組選的參數值>
```

(3) 搭配 `ENTRYPOINT` 指令使用，傳遞指令選項給 `ENTRYPOINT` 指令使用。 
```bash
CMD ["<加在 ENTRYPOINT 執行程式的選項一>","<加在 ENTRYPOINT 執行程式的選項二>"]
```

當 Container 啟動完成後，要求 Container 啟動 Nginx，就等於在 Container 命令列執行 `$ nginx -g daemon off`。
```bash
CMD ["nginx", "-g", "daemon off;"]     

# daemon off 是 -g 選項的參數，且要求須使用雙引號  
CMD nginx -g, "daemon off;"   
```

### 6. ENTRYPOINT
**用途：用法和 `CMD` 相似，都是用來設定 Container 啟動後要執行的指令，但如果是用 `ENTRYPOINT` 指令來啟動程式的話，就不能在 `docker run` 後面加入執行指令來蓋掉 `ENTRYPOINT` 所要求的啟動方式，反而會變成 `ENTRYPOINT` 的後續選項。**  

(1) 官方建議寫法
```bash
ENTRYPOINT ["<要執行的程式>","<要執行的程式用到的第一組選項>","<要執行的程式用到的第二組選項或第一組選的參數值>"]
```

(2) 
```bash
ENTRYPOINT <要執行的程式> <要執行的程式用到的第一組選項> <要執行的程式用到的第二組選項>
```

假如有個 Dockfile 最後一行從 `CMD` 改成 `ENTRYPOINT ["nginx", "-g", "daemon off;"]` 後產生新的 Image 檔，就會發生錯誤，因為 `/bin/bash` 會變成 `ENTRYPOINT` 的後續選項： `[nginx -g "daemon off;"/bin/bash]`，而 Nginx 沒有 `/bin/bash` 選項  

```bash
$ docker build -t  nginx-entrypoint .
$ docker run -it nginx-entrypoint /bin/bash
發生錯誤！
```

### 7. WORKDIR
**用途：指定執行指令的資料夾位置**  

> 注意！當資料夾是相對路徑時，預設會以它上一個 `WORKDIR` 指令所指定的資料夾位置做為目前所在的資料夾位置。  
{: .prompt-warning }

如果用到 `RUN`、`CMD`、`ENTRYPOINT`、`COPY`、`ADD` 指令，便會需要利用 `WORKDIR` 來進行資料夾位置轉換。  
```bash
WORKDIR <資料夾完整路徑或相對路徑>
```

建立 /home/ayubiz/abc 資料夾後，利用 WORKDIR 切換到 /home/ayubiz 資料夾，然後把 hello.html 檔案複製到 /home/ayubiz 裡，透過 `WORKDIR` 指定相對位置 abc 資烙夾將位置切換進 /home/ayubiz/abc，同時這也是使用 `docekr exec` 進入新啟動的 Container 後的所在位置。   

```bash
RUN mkdir -p /home/ayubiz/abc

WORKDIR /home/ayubiz

COPY hello.html ./

WORKDIR abc
```

### 8. ADD
> 需要解自動解壓縮(.tar)時適用。  
{: .prompt-info }

**用途：將一個或多個檔案放進 Image 裡，讓 Dockfile 所產生的 Image 檔能包含這些檔案。**  

```bash
ADD <來源檔案在主機的完整路徑和檔名> <Image 的完整目的路徑和檔名>

ADD <來源檔案完整網址和檔名> <Image 的完整目的路徑和檔名>

ADD 指令支援 * 和 ? 等符號的萬用字元
```

將 Dockfile 所在目錄下的 hello.html 檔案加到新產生的 Image 檔的 /usr/share/nginx/html 資料夾。  
```bash
ADD hello.html /usr/share/nginx/html

ADD *.html /usr/share/nginx/html

ADD hello.htm? /usr/share/nginx/html
```
> 注意！如果在路徑最後加上 / ，都會被認為是一個資料夾位置而不是檔案，若檔案來源路徑是資料夾，資料夾內的檔案會被加到 Image 檔案中，但資料夾本身不會被加入。  
也就是說 `docker build` 不會自動在 Image 檔案裡新增來源檔案的資料夾後再加入來源資料夾內的檔案，但是當目的地檔案是資料夾時，`docker build` 會自動建立不存在的資料夾。  
例如：ADD ./abc/ /demo/  
`docker build` 會新增 demo 資料夾，然後將 abc 資料夾底下的所有檔案加到 demo 資料夾裡，但是 demo 資料夾裡不會有 abc 資料夾的存在。  
{: .prompt-warning }

### 9. COPY
**用途：功能跟 `ADD` 類似，如果沒有要自動解壓縮檔案，官方還是比較建議用`COPY`。**  
```bash
COPY <來源檔案在主機的完整路徑和檔名> <Container 裡的完整目的路徑和檔名>
```

### 10. MAINTAINER
**用途：標註此 Dockerfile 的作者或維護者**  

> 此指令要被淘汰，建議之後用 LABEL 指令。  
{: .prompt-danger }

如果要把 Dockfile 分享出去，建議寫上電子郵件。  
```bash
MAINTAINER NGINX Docker Maintainers "docker-maint@nginx.com"
```
### 11. LABEL
**用途：將 Image 檔的作者、版本、描述、其他自訂資訊儲存到新產生的 Image 檔，建議將多欄位與值都使用一個 `LABEL` 來執行。**  

```bash
LABEL <欄位名稱>="<欄位值>" <欄位名稱>="<欄位值>" <欄位名稱>="<欄位值>" 

LABEL version="0.1" description="demo" organization="arthurtoday.com"
```

存入的資訊可以透過 `docker image inspect` 查看 Labels 欄位。

{% raw %}
```bash
$ docker inspect -f '{{json .ContainerConfig.Labels}}' <Image 名稱或 ID>
```
{% endraw %}

### 12. USER
**用途：指定新產生的 Image 檔的預設使用者帳號（通常都是 root），要先用 `RUN` 建立新使用者，`USER` 指令不會自動建立新帳號，如果指定不存在的使用者帳號是不會有作用的。**  

使用者帳號會影響到 `RUN`、`CMD`、`ENTRYPOINT` 等指令的執行時所用的使用者帳號，也會影響權限，所以若非必要，**不建議**使用 `USER` 指令改變執行軟體的使用者。  

```bash
USER <已建好的使用者名稱>

USER <已建好的使用者 UID>

USER <已建好的使用者名稱>:<已存在的使用者群組名稱>

USER <已建好的使用者 UID>:<已存在的使用者群組 UID>
```

### 13. VOLUME
**用途：提供在 Image 檔案建立 Data Volume 掛載點的功能。**  
```bash
Volume ["<掛載點的資料夾名稱和完整路徑>", ...]

Volume <掛載點的資料夾名稱和完整路徑> ...
```

把 Nginx 的 Image 檔裡的 /usr/share/nginx/html 和 /var/log/nginx 設定為掛載點的方式。  
```bash
Volume ["/usr/share/nginx/html","/var/log/nginx"]

Volume /usr/share/nginx/html /var/log/nginx
```

也可以用 `RUN` 指令建立資料夾，再使用 `VOLUME` 設定掛載點。
```bash
RUN mkdir /temp

Volume /temp
```

### 14. ARG
**用途：可以讓 `docker bulid` 指令利用 `--build-arg` 選項傳入參數到 Dockerfile 裡，可以增加 Dockfile 的彈性和再利用性。**  

有時候某些參數變更就需要修改 Dockerfile，譬如更改環境變數，如果不想經常更改 Dockerfile，可以考慮改用 `ARG` 指令搭配 `docker buil` 的 `--build-arg` 選項。  

(1) 只給定參數名稱
```bash
ARG <參數名稱>
```

(2) 同時給定參數的名稱和預設值，不一定需要設定預設值
```bash
ARG <參數名稱>=<預設值>
```

把 `ENV` 指令的 `NGINX_VERSION` 改用 `ARG` 指定設定為參數和預設值，然後如果要更改，就可以利用 `docker build --build-arg` 指定 `NGINX_VERSION 值`。  
```bash
ARG NGINX_VERSION=1.13.4-1~jessie

$ docker build -t nginx-ayu7 . --build-arg NGINX_VERSION=1.11.20-1~jessie
```

> 注意！如果有相同名稱的變數，`ENV` 指令會優於 `ARG` 指令。  
{: .prompt-warning }
```bash
ARG NGINX_VERSION=1.13.4-1~jessie

ENV NGINX_VERSION=1.11.10-1~jessie

RUN echo $NGINX_VERSION

最後會 echo 出 1.11.10-1~jessie
```

## 四、Dockerfile 撰寫與 docker build 指令建議做法
1. 資料夾
每一個 Dockfile 都放置各自獨立的資料夾，並把所需的檔案都集中放置到資料夾內，執行 `docker build` 前也移到該資料夾後再執行，這樣可以簡化 Dockerfile 的撰寫和 `docker build` 指令的執行。

2. 一次一種用途
Container 和虛擬機器不同，不適合讓一個 Container 當成一個機器來執行很多功能，建議讓多個 Container 來組合成一個完整的架構，這樣 Dockfile 才能易寫易懂易維護，同時還可以利用 Container 的分散運算能力提高整個系統的可用性

3. 只安裝需要的套件
Image 檔案的大小有一部份是取決於安裝軟體數的多寡，改 Dockfile 和 重新產生 Image 的速度很快，有需要再修改 Dockerfile 就好了，不要為了以防萬一而加大 Image 檔案的容量，導致增加檔案傳輸或部屬的時間

4. Build Cache 善用 `docker build` 指令
在產生 Image 檔時，背後有使用到一個快取的機制，所以同一個 Dockerfile 在執行第二次或第三次的 `docker build` 指令時可以發現產生的速度快很多，因為有 Build Cache 機制在運作。
但是 Build Cache 的作用和行為會依 Dockfile 被修改的內容而不同，如果發現執行 `docker build` 後產出的 Image 檔和預期的不同，可以加入 `docker build --no-cache` 停用 Cache，以確定問題不是 Cache 造成

## 五、用匯出和匯入的方法搬移 Image 檔
### 1. 匯出
docker save 指令匯出的檔案為 tar。  
```bash
$ docker save -o <匯出的檔案名稱.tar> <要匯出的 Image>:<版本>

$ docker save -o nginx.tar nginx:latest
$ docker save -o nginx.tar nginx
$ docker save nginx > nginx.tar
```

### 2. 匯入
如果主機上已經有相同的名稱和版本的 Image 檔存在，可能會顯示已匯入但實際上沒覆蓋。  
```bash
$ docker load -i <之前匯出的 image 檔.tar>

$ docker load -i nginx.tar
$ docker load < nginx.tar
```

## 六、用 Container 匯出成新的 Image 檔
除了使用 `docker commit` 指令產生新的 Image 檔，也利用 `docker export` 匯出 Container，再用 `docker import` 產生新的 Image 檔。

### 1. 匯出
不是將運行中的 Container 整個匯出，而只是將 Container 內部使用中的檔案系統匯出，匯出的檔案會覆蓋相同路徑下的同名稱檔案。  

如果 Container 有掛載 Date Volume，則資料卷不會被匯出。

```bash
$ docker export -o [匯出的檔案名稱.tar] [Container 名稱或  ID]

$ docker export [Container 名稱或  ID] > [匯出的檔案名稱.tar] 

$ docker export -o nginx-export.tar nginx-hello
```

### 2. 匯入
匯入時，不會將匯入的檔案內容轉換成運行或停用的 Container，而是直接匯入成 Image 檔，所以可以利用這種特質，將執行中的 Container 檔案內容在同一台或不同主機上，匯出新的 Image 檔。

```bash
$ docker import  [匯入的檔案名稱與路徑] [Image 名稱：版本標籤]

$ docker import nginx-export.tar ayubiz/nginx:hello
$ docker images
```

## 參考資料
- Docker 這樣學才有趣：從入門，到玩直播、挖礦