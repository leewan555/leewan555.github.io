---
title:  "Tmux－視窗分割的好幫手"
date:   2022-03-29
excerpt: 但是每次都搞混指令。
categories:
  - Linux 
---

## 使用的環境

| 系統與使用工具 | 
| ----- |  
| Centos 7.6 | 

## Tmux 介紹
Tmux 是一個終端機管理工具，它的概念很簡單，就是在一個終端機下開啟多個 session (會話)，而 session 底下又可以開啟多個 window (視窗)或是 pane (視窗區塊)。    

| 架構 | 說明 |  
| ----- | ----- |  
| session | 執行 Tmux 時都會開啟一個新的 session，每個 session 都是獨立的 | 
| window | 就是可以看到的畫面，一個 session 裡面可以有多個 window | 
| pane | 一個 window 切成多個區塊，每個區塊就是一個 pane，通常用來同時觀察多個程式 | 

   
Tmux 會在同一個 session 下保存 window 和 pane，若暫時離開這個 session，這個動作叫做`detaching`，若重新連線到這個 session，則叫做`attaching`。  

Tmux 會一直維持你上次離開 session 時的狀態，除非主機重開機，或是你自己把 Tmux 或 session 刪掉，才會不見。


## Tmux 功能
 - 分割視窗
 - 同時開啟多個視窗
 - 當遇到 ssh 斷線時，session 會在背景執行，重新連回該 session 就可以回到之前的使用環境

## Session & Window & Pane 圖片解說
晚點放上


## 安裝及使用 Tmux

```bash
# 安裝 tumx
$ yum install tmux -y

# 使用 tmux
$ tmux
```

## Tmux 組合鍵注意事項
注意！  
Tmux 的組合鍵要先按 Ctrl 跟 b 再按其它鍵    
不是同時按 Ctrl 跟 b，是先按 Ctrl 再按 b  
我一開始被這個組合鍵雷了好幾次，還以為是 Tmux 壞掉  


## Session 組合鍵及指令

| Session 組合鍵 | 說明 |  
| ----- | ----- |  
| `<Ctrl+b>` + d | 把 session 放到背景並離開 tmux 環境 |  
| `<Ctrl+b>` + $ | 重新命名目前的 session |  
| `<Ctrl+b>` + s | 以視覺化選單切換 session |  
| `<Ctrl+b>` + L | 切換至上一個使用過的 session |  
| `<Ctrl+b>` + ( | 切換至上一個 session |  
| `<Ctrl+b>` + ) | 切換至下一個 session |  


### Session 指令

| Session 指令 | 說明 | 
| ----- | ----- |   
| $ tmux ls | 列出目前開啟的 session |  
| $ tmux at -t {Session name} | 連接到指定的 session |  
| $ tmux kill-session -t {Session name} | 連接到指定的 session |  


## Window 組合鍵

| Window 組合鍵 | 說明 |  
| ----- | ----- |  
| `<Ctrl+b>` + c | 建立新 window 視窗 |  
| `<Ctrl+b>` + w | 以視覺化選單切換 window 視窗 |  
| `<Ctrl+b>` + n | 切換至下一個 window 視窗 |  
| `<Ctrl+b>` + p | 切換至上一個 window 視窗 |  
| `<Ctrl+b>` + 數字鍵 | 切換至指定的 window 視窗 |  
| `<Ctrl+b>` + x | 關閉目前的 window 視窗 |  
| `<Ctrl+b>` + f | 在所有 window 視窗中搜尋關鍵字 |  


## Pane 組合鍵及指令

| Pane 組合鍵 | 說明 |  
| ----- | ----- |  
| `<Ctrl+b>` + " | 進行水平分割 (上下分割畫面) |  
| `<Ctrl+b>` + % | 進行垂直分割 (左右分割畫面) |  
| `<Ctrl+b>` + <方向鍵> | 移動到其他 pane |  
| `<Ctrl+b>` + \<Space> | 切換佈局 |  
| `<Ctrl+b>` + \<Alt+方向鍵> | 一格為單位的調整 pane 大小 |  
| `<Ctrl+b>` + \<Alt+o> | 逆時針旋轉當前 pane |  
| `<Ctrl+b>` + \<Ctrl+o> | 順時針旋轉當前pane |  
| `<Ctrl+b>` + x | 關閉目前的 pane |  
| `<Ctrl+b>` + z | 全螢幕目前的 pane |  


## 參考資料
- [終端機 session 管理神器 — tmux](https://larrylu.blog/tmux-33a24e595fbc) 
- [從0開始的 Tmux 教學 (二)](https://laudaihe.medium.com/%E5%BE%9E0%E9%96%8B%E5%A7%8B%E7%9A%84-tmux-%E6%95%99%E5%AD%B8-%E4%BA%8C-42b57056b9b0) 
- [Linux tmux 終端機管理工具使用教學](https://blog.gtwang.org/linux/linux-tmux-terminal-multiplexer-tutorial/) 
- [終端機管理工具：tmux](https://mropengate.blogspot.com/2017/12/tmux.html) 
- [tpm：tmux 套件管理員](https://ithelp.ithome.com.tw/articles/10241450) 