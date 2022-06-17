---
title:  "Docker 基本指令"
slug: Docker-Command-18
date:   2022-05-28
excerpt: 都是些手指頭放在鍵盤上就會自己出來的實用指令。
categories:
  - Docker
---

## 使用的環境

| 系統與使用工具 | 
| ----- |  
| Centos 7.6 | 
| Docker 1.13.1 | 

## 一、找到要用的硬像（Image）檔
### (1) 尋找某個 Docker Image
```bash
$ docker search <軟體名稱>

# 尋找 nginx 的 Image 檔
$ docker search nginx
```
![](/assets/images/2022-05-28-Docker-Command-18/1.JPG)


### (2) 下載搜尋到的 Docker Image
\<Docker Image 名稱> 就是 docker search 指令執行結果所顯示的 [NAME]  
```bash
$ docker pull <Docker Image 名稱>

# 下載 nginx 的 Image 檔
$ docker pull nginx
```
![](/assets/images/2022-05-28-Docker-Command-18/2.JPG)

### (3) 查看已經下載過的 Docker image 檔
```bash
$ docker images
```
![](/assets/images/2022-05-28-Docker-Command-18/3.JPG)

## 二、啟動一個 Docker Container
```bash
# 以 Detach 的方式啟動一個 Container
$ docker run -p [對外的埠號]:[預設的埠號] -d

# 啟動 nginx 的 Docker Images
$ docker run -p 8080:80 -d nginx
```
上述例子是要用 nginx 的 docker images 以 Detach 的方式在主機的 8080 port 上提供服務，  
所以啟動這個 Container 後，可以在瀏覽器上用 http://127.0.0.1:8080 開啟網頁。

![](/assets/images/2022-05-28-Docker-Command-18/4.JPG)

## 三、解決 Container 啟動後就結束的問題
Container 在作業系統也是被視為一隻執行中的程序（Process），  
所以如果 Docker 裡面執行不是伺服器般的軟體，就會像跑一隻程式一樣，跑完就結束（像 Ubuntu 或 Node.js），  
如果 Container 一啟動就結束的話，可以在 Container 啟用一個持續執行中的程式，保持運行。  

## 四、看看啟動過哪些 Container
顯示出所有啟動過的 Container 的清單（包括正在執行與停止的）
```bash
# -a: 可以看到已經中斷運行和正在運行的 Container 
$ docker ps -a
```
STATUS: 顯示 Container 已經啟動多久了。  

![](/assets/images/2022-05-28-Docker-Command-18/5.JPG)

## 五、啟動和停止和重新啟動 Container 
### (1) 啟動 Container  
```bash
$ docker start <Container ID>

# <Container ID> 通常輸入前四碼就可以
$ docker start b87ea384818a
# 或
$ docker start b87e
```

### (2) 停止 Container 
```bash
$ docker stop <Container ID>

# <Container ID> 通常輸入前四碼就可以
$ docker stop b87ea384818a
# 或
$ docker stop b87e
```

### (3) 重新啟動 Container
```bash
$ docker restart <Container ID>

# <Container ID> 通常輸入前四碼就可以
$ docker restart b87ea384818a
# 或
$ docker restart b87e
```

## 六、移除不再使用的 Container 
Container 的狀態要在"停止"的狀態才可以移除，  
且移除掉後無法再使用 start 和 restart 再啟動，先前啟動後所產生的資料也會不見。  
```bash
$ docker rm <Container ID>

# <Container ID> 通常輸入前四碼就可以
$ docker remove b87ea384818a
# 或
$ docker remove b87e
```

## 七、刪掉一些不用的 Docker Image
Docker Image 會吃掉一部分的碟空間，所以沒用到的可以刪一刪，  
且 Docker Image 是一層一層疊出來的，不同的 Docker Images 可能有共用到同一個層的部分，    
所以移除掉某個 Docker Image 後所拿回的硬碟空間不一定會跟所想的一樣。  

這也是為什麼不論下載或移除 Docker Image 的時候，都會看到是很多檔案在下載或刪除的緣故。  
```bash
$ docker rmi <Image 名稱或 ID>

$ docker rmi nginx
```

## 八、查看 Container 資訊 
docker inspect 會傳回一個 JSON 格式的資訊，  
裡面包括 Container ID、Container 名稱、建立日期、網路配置、使用的 Docker Image、主機配置資訊、資料卷、各項配置參數。  
```bash
# 檢視 Container 所有資訊
$ docker inspect <Container 名稱或 ID>
```
因為 docker inspect 資訊很多，所以可以使用 --format 選項，來擷取部分所需的資訊，加上 json 代表傳回的格式是 JSON。

{% raw %}
```bash
# 使用 --format (-f) 選項來擷取 Container 部分資訊
# ._____  代表某個欄位區段，大小寫要和 JSON 資料裡的欄位值完全相同，譬如狀態資訊就是 .State

$ docker inspect --format='{{jason .____}}' <Container 名稱或 ID>
```
{% endraw %}

{% raw %}
```bash
# 擷取 Container 狀態資訊，相關資訊會放在 [State] 欄位區段
$ docker inspect --format='{{json .State}}' <Container 名稱或 ID> 

# 查看 [State] 欄位區段裡的 [Status] 欄位值
$ docker inspect --format='{{json .State.Status}}' <Container 名稱或 ID>
```
{% endraw %}

![](/assets/images/2022-05-28-Docker-Command-18/7.JPG)

![](/assets/images/2022-05-28-Docker-Command-18/8.JPG)




## 九、查看每個 Container 用掉多少硬碟空間
最後面的 SIZE 有寫大小。
```bash
$ docker ps -s 
```
![](/assets/images/2022-05-28-Docker-Command-18/6.JPG)

## 十、查看哪一個 Container 用掉最多資源
docker stats 會出現全畫面的"即時" Container 資源使用狀況，Ctrl + C 退出。
```bash
$ docker stats <Container 名稱或ID>

# 加入 -a 列出所有的 Container，可以知道目前機器共有多少個 Container、哪些有在執行、哪些沒有在執行
$ docker stats -a
```
![](/assets/images/2022-05-28-Docker-Command-18/9.JPG)

## 十一、查看 Container 的 Log
一次只能指定一個 Container，不支援同時查看多個 Container 的 Log 資訊。
```bash
$ docker logs <Container 名稱或 ID>

# 執行一個 nginx Container 後 curl 它，接著查看啟動後到目前為止的 log 
$ docker run -d -p 8080:80 nginx
$ curl http://127.0.0.1:8080
$ docker logs 81e3
```

```bash
# 使用 --tail 指定最後幾行的 log 行數，查看最後兩行的 log
$ docker logs --tail 2 <Container 名稱或 ID>
$ docker logs --tail 2 81e3

# 使用 --since 查詢倒數幾分鐘以前的 log，查看兩分鐘前的 log
$ docker logs --since 2m <Container 名稱或 ID>
$ docker logs --since 2m 81e3

# 加上 -f 可以查看"即時更新"的 Container log，Ctrl + C 退出
$ docker logs -f <Container 名稱或 ID>
$ docker logs -f 81e3
```
![](/assets/images/2022-05-28-Docker-Command-18/10.JPG)

## 參考資料
- Docker 這樣學才有趣：從入門，到玩直播、挖礦