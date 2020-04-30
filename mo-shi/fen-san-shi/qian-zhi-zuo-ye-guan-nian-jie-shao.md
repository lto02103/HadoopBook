---
description: 架設hadoop前，確認Linux所需要的環境
---

# 事前準備

## Linux環境下一定要做的重要事項

### 大綱

1. 系統升級

   * `apt-upgrade`  \( 在VM最乾淨的狀況下才能做此步驟 \)

2. 確認時區
   * 時區錯誤叢集會無法運作
3. 確認網路
   * 
4. 確認ssh-server
   * 會從putty 或是conemu 透過22 port 登入
5. 確認倉庫 
   * 改成台灣的鏡像站
   * 國網速度較快
6. 安裝open-jdk8
   * hadoop底層是java
7. 停用ipv6 
   * hadoop 只支援ipv4，停用後效能會快一倍
8. 創建hadoop 帳號
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

#### 更新

```bash
# 檢查核心版本
uname -r

# 更新
sudo -i
apt update
apt upgrade
reboot

# 安裝必要套件
sudo -i
apt install dkms
apt install build-essential

# 檢查核心版本
uname -r    # 4.15.0-76-generic

# 安裝核心
apt install linux-headers-4.15.0-76-generic # 要升級的核心版本

# 安裝open VMware tools
apt install open-vm-tools
reboot

# 刪除不必要的套件
apt autoremove
```

## 確認時區

* 每一台的時區要一致,叢集才能跑

```text
# 切管理員
sudo -i

# 顯示時區
timedatectl list-timezones | grep -i 'taipei'

# 改變時區
timedatectl set-timezone Asia/Taipei

# 檢查時區
timedatectl
date
```

## 確認網路

* ip錯誤也是可以上網。**ip錯!=能上網**

```text
cat /etc/netplan/50-.... # tab

```

## 更改倉庫

[鏡象站](https://launchpad.net/ubuntu/+cdmirrors)

