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
   * ip錯誤也是可以上網。**能上網!=ip正確**
4. 確認ssh-server
   * 確認能從putty 或是conemu 透過22 port 登入
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

* ip錯誤也是可以上網。**能上網!=ip正確**

```text
cat /etc/netplan/50-.... # tab

```

## 確認ssh-server

* 確認能從putty 或是conemu透過22 port 登入
* 此步驟後全都用putty登入進行操作 \(詳見:從putty登入\)

```text

# 檢查socket有無接上22/tcp(ssh的port number)
sudo -i
lsof -nPi | grep ':22' 

# 安裝openssh-server   (無22 port 才需要安裝)
sudo -i
apt update
apt install openssh-server

#顯示IPv4的ip  
ip addr show | grep 'inet '   # 192.168.100.211
```

{% hint style="warning" %}
console只有雲端管理員可以登入,我們只能透過putty打開terminal進行操作
{% endhint %}

## 確認倉庫

* [鏡象站](https://launchpad.net/ubuntu/+cdmirrors)
* 更改倉庫設定檔

```text
# 確認倉庫的設定檔 
ls -l /etc/apt/source.list 
# 出現 /etc/apt/source.list  : 主設定檔(透過apt下載,docker放這)
# 出現 /etc/apt/source.list.d/*.list :其他第三方的倉庫設定
# e.g. MariaDB

# 進nano編輯
nano /etc/apt/sources.list
# ^+\  
tw.archive.ubuntu.com 改成 free.nchc.org.tw
# ^+s > ^+x

# 檢查
apt update
# 順利執行就是成功
```

## 重要知識 <a id="&#x9806;&#x5E8F;"></a>

**tags: 小心離職**

1. 開server,不需要login
2. 開putty接上後login
3. 需要安裝或更新,再切換成root

{% hint style="warning" %}
root權限最大，只有在需要安裝或更新時才會切換
{% endhint %}

```text
sudo -i  
apt update
apt install ...
```

{% hint style="danger" %}
**注意事項**

1. Server相當於在機房,使用putty相當於從遠端登入,**只能exit,不能poweroff跟reboot**,否則會離職。
2. 只能從sever的console的poweroff跟reboot
{% endhint %}

## 從putty登入

設定plug

* 從putty開啟terminal進入server
  1. 連上server ip : 192.168.100.211,port:22
  2. keyboard : Linux
  3. translation : UTF8
  4. 字體 :16~18 , bolder
  5. connection : 30 \(30s戳一次，不會逾時斷線\)
  6. save
* 以後就從putty開啟tetmianl去操作sever

{% hint style="info" %}
console

* 不能更改視窗大小
* 不能更改字體
* sever 本身的畫面

terminal

* 可以更改視窗大小
* 可以更改字體
* putty,conemu
{% endhint %}

