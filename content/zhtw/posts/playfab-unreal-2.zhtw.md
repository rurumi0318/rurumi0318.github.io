---
title: "在PlayFab 上建立Unreal dedicated server (2)"
date: 2021-06-17T22:28:19+08:00
draft: true
description: "建立必要的開發環境"
enableToc: true
enableTocContent: true
tags:
- unreal
- playfab
series:
- "Unreal dedicated server & PlayFab"
image: images/feature2/mathbook.png
---

* [在PlayFab 上建立Unreal dedicated server (1)](../playfab-unreal-1/)
* [在PlayFab 上建立Unreal dedicated server (2)](../playfab-unreal-2/)

本文使用的開發環境是：
* Unreal Engine 4.26
* Visual Studio 2019

## 開啟PlayFab 多人連線功能
以下來自[PlayFab 官方教學](https://docs.microsoft.com/en-us/gaming/playfab/features/multiplayer/servers/enable-playfab-multiplayer-servers)
* 在[playfab.com](https://playfab.com/)註冊新帳號
* 建立新title
* 進入title 頁面後，選擇Multiplayer > Servers page > Enable
* 接著需要填入一些付款資料，完成後即可啟用multiplayer 的相關功能

啟用後要等待一段時間，可以先繼續做下個步驟。

## 建立Unreal Engine dedicated server 開發環境

以下來自[Unreal 官方教學](https://docs.unrealengine.com/4.26/en-US/InteractiveExperiences/Networking/HowTo/DedicatedServers/)


### 下載Unreal source code
<mark>想要在Unreal 上寫dedicated server 必需使用Unreal source build</mark>。
已經有Unreal source build 的話直接跳過這個和下一個步驟。

* [加入GitHub 的Epic Games群組](https://www.unrealengine.com/en-US/ue4-on-github)，否則沒辦法下載source code
* 到[Unreal Engine repository](https://github.com/EpicGames/UnrealEngine)下載source code，沒特別需求的話應該是從`Release branch`下載即可。

官方說build 整個Unreal Engine 大約40分鐘，但我的電腦第一次build 大概花了4個小時。如果電腦有SSD的話可以考慮讓Unreal source code 放在那個槽，但完整build 完之後大概會佔用硬碟150GB。


### 建立Unreal source build

官方說明提到需要Visual Studio 2017，但其實我用Visual Studio 2019也可以。

* 在下載好的Unreal source code 資料夾中執行`Setup.bat`，會下載一些東西到電腦上需要等一小段時間。
* 同一個資料夾中執行`GenerateProjectFiles.bat`
* 打開UE4.sln，把solution configuration 改成`Development Editor`平台則是`Win64`，然後在`UE4`上按右鍵點`Build`

![Build UE4](/images/playfab-unreal/build-ue4.gif "Build UE4")

* Build 完以後在`UE4`上按右鍵 > `Set as Startup Project`，然後按下`F5`啟動Unreal Engine

![UE4 as startup project](/images/playfab-unreal/startup-project-ue4.gif "UE4 as startup project")

* 用source build 開啟Unreal 之後，建立新的<mark>c++專案</mark>。

### 設定build target
ss
