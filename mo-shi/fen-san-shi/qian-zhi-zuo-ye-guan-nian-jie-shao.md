---
description: 架設hadoop前，確認Linux所需要的環境
---

# 事前準備

## Linux環境下一定要做的重要事項

### 大綱

* 系統升級

  * `apt-upgrade`  \( 在VM最乾淨的狀況下才能做此步驟 \)

* 確認時區
  * 時區錯誤叢集會無法運作
* 確認網路
  * 
* 安裝ssh-server
  * 會從putty 或是conemu 透過22 port 登入
* 更改倉庫 
  * 改成台灣的鏡像站
  * 國網速度較快
* 安裝open-jdk8
  * hadoop底層是java
* 停用ipv6 
  * hadoop 只支援ipv4，停用後效能會快一倍
* 創建hadoop 帳號
  * 專門給hadoop環境的帳號

#### 複製VM時所需步驟

* 更改VM ip
  * 才不會ip相衝
* 更改主機名稱
  * 若不改名，易造成辨識上的混淆

#### 上述步驟完成才可以做此步驟

* ssh 數位簽章登入 hadoop 帳號 
  * 無密碼登入
  * ip 跟主機改名才可以做
  * 若出現問題較好除錯

## 系統升級

## 更改倉庫

[鏡象站](https://launchpad.net/ubuntu/+cdmirrors)

## 2 . 

