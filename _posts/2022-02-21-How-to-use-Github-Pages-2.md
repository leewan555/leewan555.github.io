---
title: "用 GitHub Pages 建立個人部落格吧！(1)－Github Pages 設定"
slug: How-to-use-Github-Pages-2
date: 2022-02-21
excerpt: 一起來建立自己的部落格！
categories:
  - GitHub Pages
---
## Github Pages 介紹
GitHub Pages 是 GitHub 提供的一個網頁代管服務，可以讓使用者放靜態網站，做出個人部落格。  

- 優點  
1. 穩定  
2. 安全  
3. 免費  

- 缺點  
1. 僅能呈現靜態頁面內容  
2. 僅能使用 Git 上傳  
3. GitHub Pages 的 Repo 都是公開的  

## 一、Github Pages 設定
### (1) 建立一個想作為 Github Pages 的 Repo
按下最右邊綠色的 New  

![](/assets/images/2022-02-21-How-to-use-Github-Pages-2/1.png)  

`Repository name` 打上你想要的專案名  
專案名後面加上 .github.io，方便辨識這是 Github Pages 的專案  

  
`Description (optional)` 是描述你的專案，可以選擇寫或不寫  

其他的設定都不太用調整

好了後就按最下面的`Create Repository`  

![](/assets/images/2022-02-21-How-to-use-Github-Pages-2/2.png)  

### (2) Repo 建立成功
看到這個畫面表示你已經成功建立你的 Repo 了  

![](/assets/images/2022-02-21-How-to-use-Github-Pages-2/3.png)  

可以先試著新增一個檔案，按下`creating a new file`  

![](/assets/images/2022-02-21-How-to-use-Github-Pages-2/4.png)  

名稱跟內容可以先隨便打，反正可以刪掉  

![](/assets/images/2022-02-21-How-to-use-Github-Pages-2/5.png)  

比較需要注意的是底下的 `Commit new file`，要描述一下你這個檔案是什麼

都用好後按下`Commit new file`  

![](/assets/images/2022-02-21-How-to-use-Github-Pages-2/6.png)  


### (3) 開始設定 Github Pages
按最右邊的`Settings`  

![](/assets/images/2022-02-21-How-to-use-Github-Pages-2/7.png)  

再按左側選單的`Pages`  

![](/assets/images/2022-02-21-How-to-use-Github-Pages-2/8.png)  

Branch 可以先選`None`，也可以選擇其他分支（範例選 main）  

再按下`Save`  

![](/assets/images/2022-02-21-How-to-use-Github-Pages-2/9.png)  

如果要使用 Github 內建的主題，就按下`Choose a theme`  

![](/assets/images/2022-02-21-How-to-use-Github-Pages-2/10.png)  

我選 Dinky 主題當範例，然後按下`Select theme`  

（如果想學習用更方便的主題工具，請按 [用 GitHub Pages 建立個人部落格吧！(2)－在 MacOS 上安裝 Ruby 與 Jekyll](https://notes.lookfred.com/jekyll/ruby/github%20pages/macos-install-ruby-and-jekyll-to-create-Github-Pages-3/ "用 GitHub Pages 建立個人部落格吧！(2)－在 MacOS 上安裝 Ruby 與 Jekyll")）  

![](/assets/images/2022-02-21-How-to-use-Github-Pages-2/11.png)  

接著會跳出一個 index.md，寫一下描述後按下`Commit Change`  

就完成了！  

![](/assets/images/2022-02-21-How-to-use-Github-Pages-2/12.png) 

![](/assets/images/2022-02-21-How-to-use-Github-Pages-2/13.png)  

### (4) 訪問你的 Github Pages
Github Pages 的 網址是 **[USERNAME].github.io**  

像這樣`leewan555.github.io`  

平常只要這樣打就可以進入你的 Github Pages 頁面  

但由於現在是示範的狀態，所以我會有兩個 Github Pages 的 Repo

所以我得在後面再加個 Repo 名稱  

**[USERNAME].github.io/[REPO_NAME]**  

因此我的網址就會是`leewan555.github.io/test.github.io`  

進去後會發現網頁自動轉了 https   

![](/assets/images/2022-02-21-How-to-use-Github-Pages-2/14.png)  

自己的部落格就出現了！  

![](/assets/images/2022-02-21-How-to-use-Github-Pages-2/15.png)  


## 參考資料
- [如何用 GitHub Pages 建立部落格？ - GitHub Pages x Jekyll x Blog](https://ktinglee.github.io/install-github-pages-blog-1/) 
- [透過 Jekyll 與 GitHub Pages 建立自己的部落格(1)](https://ktinglee.github.io/install-my-blog(1)/) 
- [用 Jekyll 和 Github Page 來架設靜態 Markdown 部落格](https://medium.com/@starshunter/%E7%94%A8-jekyll-%E5%92%8C-github-page-%E4%BE%86%E6%9E%B6%E8%A8%AD%E9%9D%9C%E6%85%8B-markdown-%E9%83%A8%E8%90%BD%E6%A0%BC-fcaa288d4dd7) 