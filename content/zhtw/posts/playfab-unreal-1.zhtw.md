---
title: "在PlayFab 上建立Unreal dedicated server (1)"
date: 2021-06-10T23:23:11+08:00
description: 使用微軟PlayFab在世界各地建立你的多人對戰伺服器，並透過內建的配對機制幫助玩家找到適合的對手及隊友。
draft: false
enableToc: true
enableTocContent: true
tags:
- unreal
- playfab
categories:
- 
series:
- "Unreal dedicated server & PlayFab"
image: images/feature2/mathbook.png
---

本文會著重在環境的建立及一些相關設定，不會涉及多人連線的設計和太多的程式。主要就是用來記錄在PlayFab 使用的過程中遇到的一些問題及處理。

* [在PlayFab 上建立Unreal dedicated server (1)](../playfab-unreal-1/)
* [在PlayFab 上建立Unreal dedicated server (2)](../playfab-unreal-2/)
<!--more-->

## 多人連線種類
這裡的多人連線指多個玩家共同加入一個房間的遊戲類型，連線架構大致分為dedicated server 及非dedicated server。
非dedicated server 的部分因為有很多細分，這邊叫他master client 並且簡單帶過因為這不是本文重點。

### Master client
由多個client 中挑出一位當master client，負責遊戲中主要的邏輯判斷及驗證，其他client 可能需要透過master client 才能和其他人溝通。

優點是相對容易實作，只要client 間互相知道IP即可連線；另一個優點是費用，因為沒有真正的**server** 所以應該會比較便宜。
缺點也因為只有client 端所以無法確保公平性(當然還是有一些方法例如client 之間互相檢驗等)，在需要高度競爭遊戲用這種方法要非常小心，但party game 等類型則很適合。

### Dedicated server
所有玩家連線進入一個獨立的server 負責處理遊戲中所有實際上發生的邏輯，而client 只負責把server 運算結果呈現在畫面上，以及告訴server 玩家的操作。

優點是你可以完全控制整個server 的環境以及執行的程式，client 端不容易作弊。在後續要給玩家金幣、升級等等獎勵也相對安全。
缺點是貴和費工(多一份server 專用的程式、server 端的環境架設等等)。正是因為這些麻煩的步驟催生出本文來記錄這些事情。
> 多寫一個dedicated server 專用的程式不一定比寫master client 費工。
> 以Unreal Engine 為例，因為dedicated server 通常與client 是同一個project 產生出來的，只要設定不同的build target 即可，所以即使server 是獨立的程式也不用完全重新寫一次server 專用的遊戲邏輯。

## Dedicated server 要架在哪？
一般遊戲server 需求通常包含auto scale、matchmaking 以及authentication。
目前市面上有兩家很大的服務商提供這些功能：
* Amazon GameLift
* Microsoft PlayFab

你可以透過他們上傳你的server 程式，在玩家登入並配對成功後動態產生新的server instance 讓玩家連入。

玩家連入dedicated server 大致的流程為：
1. 玩家在client 端登入並透過matchmaking 尋找隊友、對手
2. 動態產生新的server
3. 讓玩家知道連線位址並連入
4. Dedicated server 驗證連入client 的身份後，賦予其在資料庫上記錄的角色裝備、狀態
5. 遊戲結束後，dedicated server 上傳戰鬥結果給後端server 計算獎勵並寫入資料庫

### 為什麼選PlayFab？

其實我兩家都試過，相比之下PlayFab 整合得更大包，對於規模較小沒辦法花太多時間在server 端基礎建設的開發團隊來說其實不錯。
但GameLift 的文件比較完整，網路社群上的教學也比較多。PlayFab 的文件也是很多，但是真的用起來很難一步一步帶你把事情做好。本文就是紀錄我使用PlayFab 架起Unreal dedicated server 並讓玩家連入的過程。
                PlayFab | GameLift
------------------------|----------------------
每個月免費750 core hours[^1] | Free tier 每個月125小時[^2]
內建authentication | 可透過Amazon Cognito
內建player item 及currency | 可透過Amazon DynamoDB
內建matchmaking，但本文寫的時候還在preview[^3] | 內建FlexMatch

[^1]: PlayFab 每月免費750 core hours，但是最小的VM 也要2個 cores
[^2]: AWS free tier 只有註冊的前12個月才有
[^3]: PlayFab matchmaking 本文寫的時候還在preview
