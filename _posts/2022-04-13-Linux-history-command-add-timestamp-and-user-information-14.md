---
title:  "將 History 指令加上時間戳及使用者資訊"
date:   2022-04-13
excerpt: 必要時很方便！
categories:
  - Linux 
---

## 使用的環境

| 系統與使用工具 | 
| ----- |  
| Centos 7.6 | 

## 設定方式
先上結論，設定方式長這樣：  
```bash
$ export HISTTIMEFORMAT="%F %T `who am i` "
```
## 輸出結果
![](/assets/images/2022-04-13-Linux-history-command-add-timestamp-and-user-information-14/1.jpg)  

## 設定解說
### 日期及時間
將 history 指令加上日期及時間，可以用 HISTTIMEFORMAT 這個設定，  
%F %T 代表日期跟時間。
```bash
$ export HISTTIMEFORMAT="%F %T "
``` 

|  | 說明  | 範例  |   
| ----- | ----- | ----- |    
| %F | full date; same as %Y-%m-%d | 2022-04-11 |   
| %T | time; same as %H:%M:%S | 10:25:58 |      

### 使用者資訊
若想顯示當下使用者資訊可以加上 who am i 這個指令，    
who am i 會顯示使用者名稱、該使用者登入的時間、使用者登入時的 IP。
```bash
$ export HISTTIMEFORMAT="%F %T `who am i` "  
```
```
[user@wattt ~]# who am i
root     pts/0        2022-04-12 14:45 (x.x.x.x)
```
> who am i 指令跟 who -m 指令效果一樣。

## 永久設定
如果單純在 Shell 輸入 export 那行指令，當下 history 輸出的格式是會生效沒錯，但那只是臨時的，下次登入就沒了。    

想要永久設定，讓每次登入都有這種格式，那就把他寫進 profile 檔案裡面。
```bash
# 編輯 /etc/profile
$ vim /etc/profile

# 在最下面加入
export HISTTIMEFORMAT="%F %T `who am i` "

# 保存後吃一下設定
$ source /etc/profile

# 查看現在 history 指令輸出的樣子
$ history
```
大功告成！

## 參考資料
- [Bash history 加上 日期和時間](https://blog.longwin.com.tw/2017/05/linux-bash-history-date-time-display-2017/)
- [Linux 日期格式 – Shell Script 自訂格式](https://www.ltsplus.com/linux/linux-date-format-shell-script) 