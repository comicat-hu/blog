---
title: Auto deploy from GitHub using AWS CodeDeploy
date: 2017-12-18 11:15:09
categories:
  - 資訊技術
  - AWS
tags:
  - AWS
  - Deploy
---
以下步驟會透過aws web console操作。

本篇使用個人的專案為實驗，僅供參考。

## 流程摘要

* 建立IAM user, role, policy
* 建立EC2, 安裝codedeploy-agent
* 建立專案部署設定檔`appspec.yml`
* 建立code deploy application
* 手動透過code deploy部署一個GitHub專案
* 建立自動從GitHub通知部署的服務

<!--more-->

## 建立IAM user, role, policy

* 參考 <http://docs.aws.amazon.com/codedeploy/latest/userguide/getting-started-create-service-role.html>
* 如果要規範特定region存取，可以edit Trust Relationship中的service設定

{% asset_img create-role.PNG create-role %}

## 建立EC2, 安裝codedeploy-agent

* 起一台amazon linux ami t2 micro，記得加個tag Name
* 參考 <http://docs.aws.amazon.com/zh_cn/codedeploy/latest/userguide/codedeploy-agent-operations-install-linux.html>
* 順便環境建置及設定
* 之後codedeploy會用root執行，所以環境設定時注意一下

## 建立專案部署設定檔appspec.yml

* 參考 <http://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file.html>，將檔案放在專案根目錄
* YAML格式非常嚴謹，必須確保縮排及空格都要符合規範，不需要用的區塊要整個註解掉或移除
* CodeDeploy會先將專案放置在某個暫存資料夾(source)再移到你所設定的目的地(destination)
* CodeDeploy會有幾個重要的階段 `ApplicationStop -> BeforeInstall -> AfterInstall ->  ApplicationStart`，可以在這些階段中設定要執行的script，install指的是複製專案到目的地的動作。
    + `ApplicationStop` : 在下載新的commit前會先跑stop，所以這時執行的是當前(舊版)的stop script，第一次部屬因為沒有舊版stop script所以沒有執行任何東西。
    + `BeforeInstall` : 在安裝前時期
    + `AfterInstall` : 在安裝後的時期
    + `ApplicationStart` : 啟動程序時期

## 建立code deploy application

* 到CodeDeploy點選，create application
* 參考 <http://docs.aws.amazon.com/zh_cn/codedeploy/latest/userguide/applications-create.html>
* 使用Tag Name選擇前面起的EC2
* 選擇前面設定的CodeDeployRole
* Advanced中可以設定部署失敗是否要rollback

{% asset_img create_application.png create_application %}

## 手動透過code deploy部署一個GitHub專案

* Create deployment
* Repository type選擇from !GitHub，輸入GitHub的代稱，如果登入的GitHub沒有連結aws過，會要求一個認證，點選即可。
* 輸入完整的Repository名稱(帳戶名稱/專案名稱)，這裡似乎只能用master branch
* 完整的commit ID

確認上面的流程都可以順利運作之後，再來加入自動通知部屬的服務，務必連續部署幾次查看，有時候在applicationStop時期會有exit 0的錯誤，目前尚不知道原因為何，但通常是跑script或初次部屬就失敗，這時候再推新版也無法成功，只能重建一個新的deploy application。(通常也會是因為applicationStop時其會執行舊版appspec和script有關，所以永遠離不開錯誤的script，進機器修改舊版暫存檔也許可以改善，但沒試過)

## 建立自動從GitHub通知部署的服務

* auto deploy 可以直接參考這篇後半，<https://crypt.codemancers.com/posts/2016-12-26-autodeploy-from-github-using-aws-codedeploy/>
* 建立IAM user的policy時，必須注意其中的`APPLICATION_NAME`和`DEPLOYMENT_GROUP`，要和我們建立的名稱一樣，不然github就存取不到了
* 產生一組IAM user的Access key，記得複製
* 到github帳號的`settings -> Developer settings -> Personal access tokens`產生一組token，並且勾選`repo:status`和`repo_deployment`，記得複製token
* 到專案的`settings -> Integrations & services -> Add service`，總共要加入兩個服務`AWS CodeDeploy EditDelete`和`GitHub Auto-Deployment`，各別填上相應的資訊就是了，剛剛的access key, github token都是在這裡使用

接下來嘗試push commit，如果兩個服務都有順利運行就會有綠色勾勾，反之則會有驚嘆號的標示

{% asset_img service.PNG service %}

github服務過了之後流程就會到codedeploy這邊，

登入後在CodeDeploy deployments頁面查看是否有部署動作中

* 點選`deployment ID`可以查看詳細的內容
* 在內容中點選`view event`可以查看部屬的各個流程情況，如果失敗可以知道是哪個流程，也會有一些logs可以看
  {% asset_img view_event.PNG view_event %}
* 如果logs仍然不夠清楚知道失敗的點，可能就必須要進ec2中查看了，可以參考這篇 <http://docs.aws.amazon.com/codedeploy/latest/userguide/deployments-view-logs.html#deployments-view-logs-instance>

以上就是一個簡單的從github串連aws codedeploy的部署流程，並只部到一台機器上，在撰寫script時要非常注意，在這個地方我出錯非常多次。

## issue

* 如果要搭配github branch可能需要用aws codepipeline
* 自動部屬似乎沒辦法設定overwrite模式(手動部屬可以)，所以會導致install流程時檔案重複而失敗，目前我是在install前先rm整個專案，留空給他重新部 [AWS CodeDeploy 的 File already exists at location 理解](https://shazi.info/aws-codedeploy-%E7%9A%84-file-already-exists-at-location-%E7%90%86%E8%A7%A3/)

## 參考資料

* [Autodeploy from github using AWS CodeDeploy](https://crypt.codemancers.com/posts/2016-12-26-autodeploy-from-github-using-aws-codedeploy/)
* [AWS CodeDeploy-Welcome](http://docs.aws.amazon.com/zh_cn/codedeploy/latest/userguide/welcome.html)
* [Codedeploy-agent-operations-install-linux](http://docs.aws.amazon.com/zh_cn/codedeploy/latest/userguide/codedeploy-agent-operations-install-linux.html)
* [AWS-CodeDeploy-Github-File-Already-Exist](https://stackoverflow.com/questions/34951797/aws-codedeploy-github-file-already-exist)
* [Add Support for "Overwrite" instruction in appspec.yml "Files" section](https://github.com/aws/aws-codedeploy-agent/issues/14)
* [AWS 的 CodeDeploy 是如何做 「部署」的](https://www.nosa.me/2014/11/13/aws-%E7%9A%84-codedeploy-%E6%98%AF%E5%A6%82%E4%BD%95%E5%81%9A-%E3%80%8C%E9%83%A8%E7%BD%B2%E3%80%8D%E7%9A%84/)
* [Integrating AWS CodeDeploy with GitHub](http://docs.aws.amazon.com/codedeploy/latest/userguide/integrations-partners-github.html)
* [AWS CodeDeploy 會使用舊的 appspec.yml 進行 deploy 的問題](https://shazi.info/aws-codedeploy-%E6%9C%83%E4%BD%BF%E7%94%A8%E8%88%8A%E7%9A%84-appspec-yml-%E9%80%B2%E8%A1%8C-deploy-%E7%9A%84%E5%95%8F%E9%A1%8C/)
