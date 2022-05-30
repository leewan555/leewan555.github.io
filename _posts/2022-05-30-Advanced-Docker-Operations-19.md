---
title:  "Docker Container 的進階操作"
date:   2022-05-30
excerpt: 都是很值得學習的操作！
categories:
  - Docker
---

## 使用的環境

| 系統與使用工具 | 
| ----- |  
| Centos 7.6 | 
| Docker 1.13.1 | 

## 一、幫 Container 取名字
```bash
# 執行 Container 時，加上 --name 可同時命名該 Container 的名字
# 若是不取名，就會有 Docker 自動產生的名字
$ docker run --name <Container 名稱> -p <對外的埠號>:<預設的埠號> -d <Docker Image 名稱>

# 把 nginx Container 取名成 nginx
$ docker run --name nginx -p 8080:80 -d nginx
$ docker ps -a
```
![](/assets/images/2022-05-30-Advanced-Docker-Operations-19/1.JPG)  

Container 有自訂名稱後，就可以用指令去操作。  
```bash
$ docker stop nginx
$ docker start nginx
$ docker restart nginx
$ docker rm nginx
```

## 二、將 Container  設定環境變數
有些軟體的 Container 啟動時要同時設定好一些變數讓它可以讀取來做為初始的設定值，  
例如：預設的管理者帳號密碼、預設執行目錄、同意軟體授權等參數設定。  
```bash
# 執行 Container 的同時加上 -e 可以設定變數值
docker run -e <變數名稱>=<變數值> -p <對外的埠號>:<預設的埠號> -d <Docker Image 名稱>

# MySQL 資料庫需要事先透過 MYSQL_ROOT_PASSWORD 指定資料庫管理者密碼
# 一開始先指定 root 密碼是 123456789，成功啟動後再用 docker exec 執行 mysql 操作 MySQL，就可以用剛剛設定的密碼成功登入
$ docker run --name mysql -e MYSQL_ROOT_PASSWORD=123456789 -d mysql
$ docker exec -it mysql mysql -p
```
![](/assets/images/2022-05-30-Advanced-Docker-Operations-19/2.JPG)  

## 三、docker run v.s. docker create?
docker create 要建立一個新的 Container，需搭配 docker start 指令才會啟動
docker run 在執行之後就會立刻運行

`docker run --name nginx-run -p 8080:80 -d nginx`
-> 下指令後，Container 的狀態會是 `UP`

`docker create --name nginx-create -p 8081:80 nginx`
-> 下指令後，Container 的狀態會是 `Create`

![](/assets/images/2022-05-30-Advanced-Docker-Operations-19/3.JPG)  

## 四、外掛 Container 的儲存空間，把資料存下來
如果使用資料庫的 Container，資料庫檔案位置會是在 Container 內部，當不斷把資料寫進資料庫，用 docker ps -a 就會發現 Container 佔用的空間會不斷的長大。  

Docker Container 的檔案系統是用疊層（Layer）的方式儲存，會比常見的檔案儲存更耗用空間（用一種層疊的方式來新增資料，每次新增資料，就會產生一層新資料層來疊在原來所使用的 Docker Image 上）。  

如果直接把資料庫檔案存在 Container 內部，就會因為產生大量的新資料層而快速吃掉硬碟空間，如果之後想把資料帶走，就要匯出這個 Container 來產生新的 Docker Image，超級麻煩。  

所以就可以外掛 Container 的儲存空間，把資料另外存下來。  

```bash
# 執行 Container 的同時加上 -v 可以掛上外部的硬碟空間
$ docker run -v <外部資料夾或檔案的完整路徑>:<Container 內部要被取代的資料夾完整路徑> -p <對外的埠號>:<預設的埠號> -d <Docker Image 名稱>

# nginx Container 內的 /usr/share/nginx/html 資料夾改成主機上的 /var/www 來取代
# 如果主機上沒有 /var/www 資料夾存在，docker run 會自己新增
$ docker run -v /var/www:/usr/share/nginx/html -name nginx-store -p 8080:80 -d nginx

# 除了可以取代資料夾，也可以取代檔案，常見用法是取代設定檔，用預先自訂好的設定檔取代 Docker Image 內建的設定檔
# 將本機上的 /home/nginx-test/nginx.conf 取代 Container 內的 /etc/nginx.nginx.conf
$ docker run -v /home/nginx-test/nginx.conf:/etc/nginx/nginx.conf -v /var/www:/usr/share/nginx/html -name nginx-store -p 8080:80 -d nginx
```
![](/assets/images/2022-05-30-Advanced-Docker-Operations-19/4.JPG)  

## 五、直接執行 Container 內的程式或指令
有些 Container 用途是被當作軟體工具包，docker run 可以直接在最後面加上想要執行的指令，讓 Container 一啟動完成後就執行指定的指令。  
{% raw %}

```bash
$ docker run <Docker Image 名稱> <指令完整檔案路徑與名稱及指令選項>

# 想在 Windows 環境用 ls 操作
# 使用 ubuntu 的 ls 指令，但這樣只會把 ubuntu Container 跟目錄的檔案列出來
# 所以想要把 Windows 上的某個資料夾檔案清單列出就要搭配 -v，將主機的資料夾掛載到 Container 裡面，讓 Container 內的程式可以看的到它，進而用做各項操作
$ docker run ubuntu /bin/ls
$ docker run -v C:\downloads:/home/test ubuntu /bin/ls -l /home/test
```
{% endraw %}

## 六、讓 Container 在執行結束後自動移除
被啟動的 Container 停止後就會自我毀滅~  
```bash
$ docker run --rm -v C:\downloads:/home/test ubuntu /bin/ls -l /home/test
```

## 七、讓 Container 掛掉後自動重新啟動，或開機後自動啟動
{% raw %}
docker run 可以使用 \--restart 來決定要不要嘗試自動重新啟動 Container。  

\--restart 選項：no、always、unless-stopped、on-failure  

- no：預設值，不自動重新啟動。
- always：（exit code 必須是正常值=0）可以達到電腦開機就自動啟動 Container 的效果，因為這個 Container 會跟 Docker 本身的 Daemon 綁在一起，所以 Docker 只要一啟動，有 \--restart=always 的 Container 就會跟著啟動。  
- unless-stopped：（exit code 必須是正常值=0）Container 會自動啟動，但是不會在電腦開機時自動啟動。  
- on-failure：exit code 不等於 0 時自動啟動，因為若 exit code 不等於 0 代表可能是錯誤造成的退出或結束，通常會指定自動重啟的次數，以防造成無窮啟動迴圈。  

```bash
$docker run --restart=always nginx
$docker run --restart=unless-stopped nginx
$docker run --restart=on-failure:5 nginx
```
{% endraw %}

## 八、進入 Container 操作命令列
>> docker run -it \--name nginx-cmd nginx /bin/bash -> 用 docker run 也可以。

```bash
# -t：把 Container 開在"互動模式"
$ docker exec -it <Container 名稱或 ID> <指令完整檔案路徑與名稱及指令選項>

# 進入 nginx 操作命令列
$ docker run --name nginx-cmd -d nginx
$ docker exec -it nginx-cmd /bin/bash
```
![](/assets/images/2022-05-30-Advanced-Docker-Operations-19/5.JPG) 

## 九、Container 綁定指定的 IP 位址（正式的服務環境必用）
如果只有指定埠號而沒有指定 Container 要綁到哪個 IP 位址，Docker 會預設綁定到主機的 0.0.0.0，表示這個主機上的任一組 IP 都可以連到這個 Container。
```bash
# 指定 Container 綁定 IP位址
$ docker run -p <主機的 IP 位址>:<主機的連接埠號>:<Container 的連接埠號> -d <Docker Image 名稱>

# 假如主機上有兩個網站、兩組 IP 位址，分別是 192.168.1.1 和 10.0.1.1，如果想要用 192.168.1.1 來連到這個 nginx container：
$ docker run -p 192.168.1.1:80:80 -d nginx

# 如果只想讓主機自己可以連上：
$ docker run -p 127.0.0.1:80:80 -d nginx
```
## 十、建立多個 Container 專用的 Docker 網路
>> 除了 docker network 之外，docker run 有個 \--link 選項也可以把兩個 Container 串接在一起，不過限制多，官方也不建議繼續使用 Docker link 的功能，還是建議用 Docker 網路，不然就是用 Docker Compose 來達到。

```bash
# 建立 Docker 網路
$ docker network create <Docker 網路名稱>

# 啟動 Container 並加入網路
$ docker run --name=<Contianer 名稱> --net=<網路名稱>
```
很多時候都會需要運行兩個以上的軟體，例如使用部落格軟體，就會同時用到網頁伺服器和資料庫系統。  

在虛擬機時，會把網站伺服器跟資料庫系統這兩套都安裝同一個虛擬機上。  

但在 Container 上，就不建議這樣的作法，比較建議用 docker network 建立一個 Docker 網路，然後再把這兩套軟體分別啟動為獨立的 Container，同時在 docker run 的時候，利用 \--net 選項將這兩個 Container 加到同一個 Docker 網路之中。
```bash
# 建立 nginx-a 和 nginx-b 兩個 Container 然後加入到 nginx-net 的 Docker 網路，然後 nginx-a 用 ping 指令加上 nginx-b 的 Container 名稱就可以 ping 到 nignx-b，而不用知道 nginx-b 的 IP
$ docker network create  nignx-net
$ docker run --name nginx-a --net=nginx-net -p 8100:80 -d nignx
$ docker run --name nginx-b --net=nginx-net -p 8101:80 -d nignx
$ docker exec -it nginx-a /bin/bash 

root@e8b718be1d25:/# ping nginx-b
```
![](/assets/images/2022-05-30-Advanced-Docker-Operations-19/6.JPG) 

![](/assets/images/2022-05-30-Advanced-Docker-Operations-19/7.JPG) 


## 十一、查看及刪除 Docker 網路
```bash
# 可以在輸出結果的 [Containers] 項目查到已加入此 Docker 網路的 Container ID
$ docker network inspect <Docker 網路名稱>

# 刪除 Docker 網路
$ docker network rm <Docker 網路名稱>
```
![](/assets/images/2022-05-30-Advanced-Docker-Operations-19/8.JPG) 

Docker 網路被移除後，所有已加入該 Docker 網路的 Container 都會無法再被啟動，因為它們會找不到需要的 Docker 網路可用。  

如果需要再次啟動那些 Container，只能建立一個相同名稱的 Docker 網路。  

## 十二、docker kill v.s. docker stop?
建議使用 docker stop 來關閉 Container。  

docker stop 會讓 Container 進入標準的關機程序，也就是說會讓 Container 收到要關機的訊號，並通知各個程序進入各自的關機處理程序（像是資料同步或更新到檔案等工作）。

docker kill 是當遇到使用 docker stop 關閉不了，需要強制關閉的狀況時來使用。  
```bash
$ docker kill <Container 名稱或 ID>
```

## 十三、當運行中的軟體推出新版本時，如何升級？
更新的方式就是下載最新版的 Docker Image，然後再重新啟動新的 Container 就好。  

更新過程中會有 Container 停止和重新啟動的需要，所以在更新之前要先規劃及準備好所需資料。 

有些事項要特別注意，才可以做到只下載新板 docker image 和用 docker run 重新啟動 Container 就可以完成更新的做法：  
(1) 不要在 Container 內儲存任何資料和設定  
(2) 要記下啟動 Container 時所使用的 docker run 指令和選項及參數  
```bash
# 檢視或備份運行中的 Container 資訊
$ docker inspect <Container 名稱或 ID>

# 取回最新版的 Docker Image
$ docker pull <Container 名稱或 ID>

# 停止運行中的 Container
docker stop <Container 名稱或 ID>

# 移除 Container
docker rm <Container 名稱或 ID>

# 重新啟動 Container
docker run <原來的選項與參數>
```
## 十四、建立專用 Data Volume 儲存資料
>> 除了用 docker run -v 和 docker volume 來建立 Data volume，也可以使用 docker-compose 指令搭配 docker-compose.yml 檔案新增 Data volume。

用 docker run -v 可以直接從主機存取 Container 的檔案，但是容易發生權限的問題，無法順利存取。  
另一個方法是可以透過建立 Docker Data volume 永久性存放 Container 的資料，還可以"共用"或"回收"。  


### (1) docker run -v 自動建立的 Data volume
沒有指定外部資料夾的路徑，而是直接指定該 Container 的 /usr/share/nginx/html 資料夾要掛載外部的 Data volume。  

docker run 會在啟動 Container 時，立即建立一組 Data volume，並掛載此 Data volume 至 Container 的 /usr/share/nginx/html 資料夾路徑上，同時，還會自動把該資料夾內的檔案複製到此新建的 Data Volume 裡。  
```bash
$ docker run -v /usr/share/nginx/html --name=nginx-vol -p 8080:80 -d nginx
```

{% raw %}
```bash
# 查看該 Container 的 Data volume
$ docker volume ls 

# 使用 docker inspect 找出自動建立的 Data volume 的名稱（用 docker run -v 建立的 Data volume 無法命名）
$ docker inspect -f '{{json .Mounts}}' nginx-vol
```
{% endraw %}

![](/assets/images/2022-05-30-Advanced-Docker-Operations-19/9.JPG)  
Name: 新建的 Data Volume 名稱。  
Source: Data volume 在主機的實際位址和路徑。  

### (2) docker volume create 建立的 Data volume
```bash
$ docker volume create <volume 名稱>

# 建立名叫 nginx-html 的 data volume
$ docker volume create nginx-html
# 查看一下剛剛建立的 Data volume
$ docker volume ls 
# 將 docker volume create 建立的 Data volume 掛載到新啟動的 Container 的 /usr/share/nginx/html 上
$ docker run -v nginx-html:/usr/html/nginx/html --name nginx-vol -p 8080:80 -d nginx
```
可以將這個 Data volume 同時共用到其他 Container 上面，指令跟上面一樣。  

不過共用的話要考量到"共用讀寫"同一檔案的問題，如果多個 Container 共用一個 Data volume 最好還是以不會共同更新同一個檔案的前題為原則，以免資料混亂或錯誤。  
```bash
$ docker run -v nginx-html:/usr/html/nginx/html --name nginx-vol2 -p 8080:80 -d nginx
```

### (3) 查看 Docker Data volume 的詳細資訊
```bash
$ docker volume inspect <volume 名稱>

$ docker volume inspect nginx-html
```
![](/assets/images/2022-05-30-Advanced-Docker-Operations-19/10.JPG) 

### (4) 刪除 Docker Data volume
Data volume 不會自動被刪除，必須要手動操作刪除指令，  
且必須刪除綁著此 Data volume 的 Container 才能刪除該 Data volume。  
```bash
$ docker volume rm <Data volume 名稱>
```

## 參考資料
- Docker 這樣學才有趣：從入門，到玩直播、挖礦