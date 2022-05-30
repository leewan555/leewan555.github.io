---
title:  "CentOS 7 安裝 Docker"
date:   2022-05-26
excerpt: 英文小教室：Docker 的英文是碼頭。
categories:
  - Docker
---

## 使用的環境

| 系統與使用工具 | 
| ----- |  
| Centos 7.6 | 
| Docker | 


## 一、Docker 介紹

Docker 是 Container 管理工具。    

Container 是指利用 Linux 核心提供的技術達到把程式或軟體"包裹在可隔離且獨立執行的環境空間"。  

Container 比虛擬機器輕巧且具可攜性，因為 Container 可共用 Linux 核心的功能，而虛擬機是完全模擬硬體來達到讓虛擬機器可以獨立執行各種作業系統和軟體。  

- Docker CE（Docker 社群版)：免費  
如果只是要在區網內執行 Container，只需要使用 Docker Engine、Docker Compose 者很適合。  

- Docker EE（Docker 企業版）：付費  
提供更多整合性的基礎架構及雲端服務，如果需要代管 Docker Images、提供私有 Docker Registry 儲存庫、合併各家雲端服務和自動部署服務，可考慮採用。  


## 二、CentOS 安裝 Docker CE
### (1) 安裝必要套件
```bash
$ yum install -y yum-utils \
  device-mapper-persistent-data lvm2
```

### (2) 新增Docker官方的stable套件庫(repository)
```bash
$ yum-config-manager \
  --add-repo \
  https://download.docker.com/linux/centos/docker-ce.repo
```

### (3) 更新 yum 的套件索引
```bash
$ yum makecache fast
```

### (4) 安裝 Docker CE 版
```bash
$ yum install docker-ce
```

### (5) 安裝好之後，啟動系統的 Docker 服務
```bash
$ systemctl start docker
```

### (6) 確認 Docker 版本
```bash
$ docker version
```

### (7) 執行 hello world 程式測試
```bash
$ docker run hello-world
```

如果 hello-world 有成功顯示，表示已經成功安裝好 Docker！  

## 參考資料
- Docker 這樣學才有趣：從入門，到玩直播、挖礦