---
title: Cloudflare+GitHub-Pages自訂網域連線設定
date: 2018-09-28 13:18:27
categories:
  - 資訊技術
  - Blog
tags:
  - Blog
  - Deploy
  - GitHub-Pages
---

以此站為例，我使用了[hexo](https://github.com/hexojs/hexo)搭配Github-Pages來放置這個靜態網站。

建立Github-Pages的步驟很簡單，只要在專案設定中啟用並且將網站程式碼push上來就可以了。

首先我建了一個blog專案來使用，產生出的網址是[http://comicat-hu.github.io/blog](http://comicat-hu.github.io/blog)

接著在專案設定中填入custom domain並儲存，這時你的專案會自動新增一個commit來創建`CNAME`這個檔案，內容包含了你自訂的domain，若是要讓hexo在每次deploy時不會洗掉這個檔案，可以將其加入hexo專案的source資料夾下，_config.yml的url設定值也要記得一併修改。

再來因為我的domain先前已經給cloudflare代管(需要到網域管理商那邊將cloudflare NameServer設定進去)，並且啟用了免費的SSL服務，所以這邊要設定一下：

* 新增A record分別指到github，參考[troubleshooting-custom-domains/#dns-configuration-errors](https://help.github.com/articles/troubleshooting-custom-domains/#dns-configuration-errors)
* 新增CNAME綁入自訂的subdomain，並指到yourdomain.github.io上
* 接者要等待一段時間才會生效哦

{% asset_img use_cloudflare_to_github_page.PNG cloudflare DNS 設定 %}