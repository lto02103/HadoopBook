---
description: 架設hadoop前，確認Linux所需要的環境的重要事項
---

# 事前準備

## 目錄

#### 基本步驟

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
8. 安裝pip3
   * 後續寫code以python為主
   * 安裝python套件需要pip3

#### 複製VM後所需步驟

* 更改主機名稱
  * 若不改名，易造成辨識上的混淆
* 更改VM ip
  * 才不會ip相衝

### 基本步驟

{% tabs %}
{% tab title="更新" %}
{% hint style="warning" %}
最乾淨的時候才可以apt upgrade

```text
apt upgrade
```
{% endhint %}

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
{% endtab %}

{% tab title="確認時區" %}
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
{% endtab %}

{% tab title="確認網路" %}
* ip錯誤也是可以上網。**能上網!=ip正確**

```text
cat /etc/netplan/50-.... # tab
```
{% endtab %}

{% tab title="確認ssh-server" %}
* 確認能從putty 或是conemu透過22 port 登入
* 此**步驟**後全都用**putty**登入進行操作 \(詳見:從putty登入\)

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
{% endtab %}

{% tab title="確認倉庫" %}
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
{% endtab %}

{% tab title="安裝open-jdk8" %}
* 因為oracle java8 限制下載,故Debian無法透過透過apt install安裝oracle java8，因此使用openJDK8

```text
# root
sudo -i
aptupdate

# 看jdk8套件全名
apt search '^openJDK' |grep '8'

# 安裝openjdk-8-jdk
apt install openjdk-8-jdk

# 查詢版本
java -version
# 1.8.0_252

javac -version
# 1.8.0_252
```

1. 設定JAVA\_HOME的PATH

```text
# 檢查JAVA_HOME的 PATH
echo $JAVA_HOME

# 找JAVA_HOME的路徑
update-alternatives --display java
# /usr/lib/jvm/java-8-openjdk-amd64

# 開始設定
sudo -i
cd /etc/profile.d # JAVA_HOME的PATH設定檔放這裡
nano openjdk.sh

# in nano
export JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"
# 宣告環境變數
# ^+s > ^+x

# 立即載入
. /etc/profile.d/openjdk.sh
echo $JAVA_HOME

# 確認JAVA_HOME的內容是否正確
$JAVA_HOME/bin/javac -version
# 1.8.0_252 成功
```
{% endtab %}

{% tab title="停用ipv6" %}
* 2014年ubuntu-kenrnel 支援停用IPv6 ，速度會快一倍 。
* hadoop 目前也不支援IPv6

```text
# root
sudo -i

# 檢查有無出現ipv6
lsof -nPi | grep 'IPv6' 

# 直接進去核心停掉ipv6即可
nano /etc/default/grub
   ...
   GRUB_CMDLINE_LINUX_DEFAULT=...   (不能改這裡,系統升級有可能設定檔都會不見)
   ...
   GRUB_CMDLINE_LINUX="ipv6.disable=1"  (改這裡。停用ipv6,要重開機)
   
# 更新開機時的設定檔 
update-grub
# 要下這個指令，系統會自動去做設定。
# 若無下此指令，系統會還原成預設狀態 (用nano改沒有用)
# vmlinuz + initrd = 1組核心+工具  (4核就會有4組)

# 檢查ipv6有無寫信grub裡
grep 'ipv6' /boot/grub/grub.cfg

# 重開機生效
reboot  (從consle)

# 檢查有無ipv6
sudo -i
lsof -nPi
```
{% endtab %}

{% tab title="安裝pip3" %}
* Ubuntu18.04預設只有python3
* **沒有python2.7**

```text
# root
sudo -i

# 檢查python3版本
python3 -V

# 安裝python dev-tools (一堆的library,包含pip3)
apt update
apt install python3-dev 

# 下載pip3
wget https://bootstrap.pypa.io/get-pip.py
python3 get-pip.py

# 檢查pip3版本  
pip3 -V
pip -V

# 確認pip3 ,pip 是否有差異
which pip3
which pip

cd /usr/local/bin  # 到此目錄下
ls  # 查看檔案是否正確在此目錄下

# 確認檔案類型
file pip
file pip3
file pip3.6 

# 是腳本才可以cat，若是程式。直接cat，會呈現亂碼。會被老闆打
cat pip
cat pip3
cat pip3.6
# 都是一樣的
```
{% endtab %}
{% endtabs %}

### 複製VMs後的步驟

{% tabs %}
{% tab title="更改主機名稱" %}

{% endtab %}

{% tab title="更改IP" %}

{% endtab %}
{% endtabs %}

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

[putty連結](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) \(下載最新版即可 0.73版以後\)

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

## **參考資料**

* \*\*\*\*[TCP/IP](http://www.tsnien.idv.tw/Internet_WebBook/Internet.html)
* [sokcet/plug](http://www.tsnien.idv.tw/Internet_WebBook/chap8/8-1%20Socket%20%E7%B0%A1%E4%BB%8B.html)

## **連結**

* [在Linux建置hadoop](https://hackmd.io/s0AXY0tlQ9eYSXPB5v0jyA) 
* [IP 轉址觀念](https://hackmd.io/r4HlcZRJSNW1AeV7v3dTtA)

