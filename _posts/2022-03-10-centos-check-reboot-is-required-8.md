---
title:  "檢查你的 CentOS 系統或服務是否需要重新啟動"
date:   2022-03-10
excerpt: 勇者沒在重開的啦(X)
categories:
  - Linux 
---

## 使用的環境

| 系統與使用工具 | 
| ----- |  
| Centos 7.6 | 

## 一、為什麼要關機或重啟服務？

Linux 伺服器的系統與軟體更新是定期需要做的工作，  
某些重要的系統套件在更新完之後，可能會需要重新啟動對應的服務，  
甚至若有更新到 Linux 核心的時候，還會需要重新開機。  

而如何判斷哪些服務要重新啟動，以及何時需要重新開機，就是更新後常會遇到的小問題。  

## 二、介紹 needs-restarting 工具
needs-restarting 是一個 yum-utils 套件中的一個小工具，  
它可以快速檢查目前的系統狀態，列出需要重新啟動的服務，
並且檢查 Linux 核心的版本，判斷是否需要重新開機。  

## 三、安裝 needs-restarting 工具  
### (1) 安裝 yum-utils 套件  
```bash
# 使用前先用 yum 安裝 yum-utils 套件  
$ yum install yum-utils -y
```

## 四、檢查作業系統是否需要重新啟動  
```bash
# 搭配 -r 參數可以檢查 Linux 核心版本  
$ needs-restarting -r
```
如果輸出的訊息中，有 Reboot is required ... 這樣的訊息， 
就代表目前運行的 Linux 核心版本過舊，需要重新開機。  

或是可以用另一種簡單明瞭的方式：  

```bash
$ needs-restarting -r ; echo $?
```
如果是 0 就代表不需要重新開機，而若是 1 則代表需要重新開機

### (1) 重開機指令
```bash
# 立即重新開機
$ reboot

# 指定時間重新開機，舉例 21:30 再重新關機
$ shutdown -r 21:30 & 
```

## 五、檢查服務是否需要重新啟動
```bash
# 搭配 -s 參數可以列出需要重新啟動的系統服務
$ needs-restarting -s
```

若要重新啟動指定的服務，可以使用 systemctl 搭配 restart 參數。  

例如，重新啟動 sshd：  

```bash
$ systemctl restart sshd
```

## 六、顯示由當前 UID 建立起的程序
```bash
$ needs-restarting -r
```


## 參考資料
- [CentOS Linux 判斷更新後是否要重新開機？](https://blog.gtwang.org/linux/centos-linux-how-to-check-if-reboot-is-required/)