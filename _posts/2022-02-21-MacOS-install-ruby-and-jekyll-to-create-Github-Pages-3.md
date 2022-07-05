---
title:  "用 GitHub Pages 建立個人部落格吧！(2)－在 MacOS 上安裝 Ruby 與 Jekyll"
slug: MacOS-install-ruby-and-jekyll-to-create-Github-Pages-3
date:   2022-02-21
excerpt: 其實其他 Linux 版本也都適用啦。
categories:
  - GitHub Pages
tags:
  - macos
  - github pages
  - ruby
  - jekyll
---

## 使用的環境

| 系統與使用工具 | 
| ----- |  
| MacOS Mojave 10.14.6 | 
| Ruby 2.7.5 | 

## 一、Jekyll 介紹
Jekyll 是用 Ruby 撰寫的靜態部落格框架，可以將靜態網站的 Markdown 語法轉成 HTML 語法。  

靜態網頁就是不包含資料庫，也不能和使用者互動的單純網頁，通常內容也不會時常變動。  

因此，Jekyll 對於要發展部落格的使用者來說非常方便。  

### (1) MacOS Install Homebrew
Homebrew 是軟體套件管理系統，支援 MacOS 和 Linux，非常好用，大推！  
如果是使用 Ubuntu 或 CentOS 或其他版本的 Linux，可以不用裝 Homebrew  
```bash
# MacOS Install Homebrew
$ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 若已經安裝過 Homebrew 則需要更新
$ brew update
```
### (2) Install Ruby
若使用 `brew install ruby` 安裝 ruby 會安裝到最新版 ruby 3  
但測試好幾次使用 ruby 3 的話，啟動 jekyll 會有錯誤  
所以我用另外的方式安裝 ruby 2.7.5  

```bash
$ curl -fsSL https://github.com/rbenv/rbenv-installer/raw/HEAD/bin/rbenv-installer | bash

# 因為我是使用 oh-my-zsh，若是使用 bash/dash 要將 ~/.zshrc 改成 ~/.bashrc
$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.zshrc  
$ echo 'eval "$(rbenv init -)"' >> ~/.zshrc
$ source ~/.zshrc
$ exec $SHELL
$ rbenv install -l

# ruby 版本我選 2.7.5
$ rbenv install [ruby-version] 
$ rbenv global [ruby-version] 
```
安裝完後檢查一下有沒有安裝正常 
```bash
# 安裝成功後，查看 ruby 跟 gem 的版本
$ ruby -v
$ gem -v
```

### (3) Install jekyll 和 bundler
```bash
$ gem update
$ gem install jekyll bundler

# 安裝成功後，查看 jekyll 和 bundle 的版本
$ jekyll -v
$ bundle -v
```

### (4) 使用 minimal-mistakes 主題
也可以去下載其他主題，現在先以 minimal-mistakes 作為示範
```bash
$ git clone https://github.com/mmistakes/minimal-mistakes.git

# 進入資料夾
$ cd minimal-mistakes
$ bundle install
```
### (5) 執行 Jekyll
```bash
# 啟動 Jekyll 有兩種指令方式，兩種都可以用！
$ jekyll serve
# 或是
$ bundle exec jekyll serve
```
看到 Server running... press ctrl-c to stop... 就表示成功！
![](/assets/images/2022-02-21-Macos-install-ruby-and-jekyll-to-create-Github-Pages-3/1.png)



### (6) 在瀏覽器上打 http://127.0.0.1:4000
完成！就可以看到目前主題的樣子了  
這時候就可以參考 [官方文件](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/ "官方文件") 上的教學去更改主題的設定  
可以新增主選單、主題或是更改樣式
自由發揮！

## 二、Github Page 
Giuhub Page 教學請看 [此篇文章](https://notes.lookfred.com/posts/How-to-use-Github-Pages-2/)  
若是 clone 別人的主題，記得在 push 時不能直接 push 喔，因為那是別人的 Repo！  
如果是使用 Github Desktop，記得按 Repository Setting -> Remote -> Primary remote repositoey (origin)  
在欄內輸入自己的 Github Pages 的 Repo 的網址，成功在 Github Desktop 上顯示後再 push 上去喔～  


## 參考資料
- [Minimal Mistakes Quick-Start Guide](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/) 
- [如何用 Jekyll 建立一個靜態網站？ - GitHub Pages x Jekyll x Blog](https://ktinglee.github.io/install-github-pages-blog-2/) 