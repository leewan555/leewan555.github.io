---
title:  "使用 Fish Shell 讓你的 Linux CLI 比別人更炫炮！"
date:   2022-03-04
excerpt: 而且也方便多了。
categories:
  - Linux 

---


## 使用的環境

| 系統與使用工具 | 
| ----- |  
| Centos 7.6 | 
| fish, version 2.7.1 | 
| git version 2.31.1 | 

## Fish Shell 介紹
Fish Shell 是個互動式 Shell，可以讓指令介面 (CLI, Command Line Interface) 變得更好看，又有指令自動補全功能的一個工具，非常方便。

## oh-my-fish 介紹
oh-my-fish 是 Fish Shell 的框架，允許安裝擴充套件或是更改 Fish Shell 的主題，簡單使用又快速。

## 開始！

### 檢查 Git 版本

```bash
# 查看原本的 git 版本，如果低於 1.9.5 就得升級
$ git --version
# 升級 git 
$ yum remove git    
$ yum install http://opensource.wandisco.com/centos/7/git/x86_64/wandisco-git-release-7-2.noarch.rpm
$ yum install git   
$ git --version
```

### 安裝 Fish Shell
```bash
$ cd /etc/yum.repos.d/
$ wget https://download.opensuse.org/repositories/shells:/fish:/release:/2/CentOS_7/shells:fish:release:2.repo
$ yum install fish
```

### 安裝 oh-my-fish

```bash
$ curl https://raw.githubusercontent.com/oh-my-fish/oh-my-fish/master/bin/install | fish
```

### 進入 Fish Shell 後使用 oh-my-fish
```bash
$ fish
$ omf
```

### 暫定先用 agnoster 這個主題
```bash
# 下載 agnoster 主題
$ omf install agnoster
# 正常下載後就會自動套用，若沒有套用就輸入下面這行
$ omf theme agnoster
```

### 更改主題顏色
```bash
$ vim /root/.local/share/omf/themes/agnoster/functions/fish_prompt.fish
```
找到 `Color setting` 這個區塊，裡面有一些東西可以調  

資料夾背景色是這個  
`set -q color_dir_bg; or set color_dir_bg green`

預設可以支援的顏色有這些  
Valid colors include:  
black, red, green, yellow, blue, magenta, cyan, white
brblack, brred, brgreen, bryellow, brblue, brmagenta, brcyan, brwhite

設定完後要 reload omf 
```bash
$ omf reload
```
完成！可以開始用美美德 fish shell 惹。

## 參考資料
- [fish-shell Github](https://github.com/fish-shell/fish-shell) 
- [oh-my-fish Github](https://github.com/oh-my-fish/oh-my-fish) 
- [Fish shell：讓指令更接近懶人使用](https://noob.tw/fish-shell/) 
