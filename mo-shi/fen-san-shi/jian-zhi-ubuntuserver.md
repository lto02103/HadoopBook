---
description: '版本 : Ubuntu-server 18.04.04'
---

# 建置Ubuntu-server

## 所需要的工具

* VMware workstation player 15
* ubuntu18.04.04-server-amd64.iso
* 一顆耐心

## 安裝VMware workstation player

### 下載VMware

[VMware workstation player](https://www.vmware.com/tw/products/workstation-player/workstation-player-evaluation.html)  \(下載最新版即可\)

### 安裝方式

一鍵確認到底。

### VMs檔案放置

在C槽新增VMs的目錄，VMs\(虛擬機\)的資料一律放置在**C:\VMs**底下 \(管理上較好管理\) 😀 

## 安裝ubuntu-server

### 下載ubuntu-server.iso

到[國網的鏡像站](https://free.nchc.org.tw/ubuntu-cd/bionic/)下載ubuntu-server   \(此為連結\)

> [ubuntu-18.04.4-live-server-amd64.iso](https://free.nchc.org.tw/ubuntu-cd/bionic/ubuntu-18.04.4-live-server-amd64.iso)  \( 點此下載即可\)

### 新增一台虛擬機

### 系統設定

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

#### 確認時區



{% hint style="info" %}

{% endhint %}

### 



