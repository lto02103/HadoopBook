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

## IP設定

要先固定ip

1. 在win10固定IP \(cmd &gt;&gt; ipconfig\)
   * 為了建置Hadoop的叢集
   * Hadoop叢集會抓IP,IP每次都不固定會跑不了

| IP |  |  |
| :--- | :---: | :---: |
| IP位置 | 192.168.100.210\(個人IP\) | 座號x10 |
| 子網路遮罩 | 255.255.255.0 | C級網域 |
| 預設閘道 | 192.168.100.254 | 中華電信gateway |
|  | DNS |  |
| 慣用 | 168.95.1.1 | 中華電信 |
| 其他 | 101.101.101.101 | 國家網路中心\(twnic\) |

## 安裝ubuntu-server

### 下載ubuntu-server.iso

到[國網的鏡像站](https://free.nchc.org.tw/ubuntu-cd/bionic/)下載ubuntu-server   \(此為連結\)

> [ubuntu-18.04.4-live-server-amd64.iso](https://free.nchc.org.tw/ubuntu-cd/bionic/ubuntu-18.04.4-live-server-amd64.iso)  \( 點此下載即可\)

### 新增一台虛擬機

### 系統設定

{% tabs %}
{% tab title="First Tab" %}
d

```text

```

* [ ] 
{% endtab %}

{% tab title="Second Tab" %}

{% endtab %}

{% tab title="" %}

{% endtab %}
{% endtabs %}

**確認** : cmd-&gt;ipconfig

1. 安裝VMware
   1. 灌VMware

   * 預設 spilt,500G
     * 若是機房可選one disk
     * spilt 是考慮到拷貝及回家使用的需求
2. 設定
   * CPU 12
     * VM : 4,4
     * HOST : 4
   * RAM 64
     * VM : 24,24 \(24 x1024\)
     * HOST : 16
   * 
3. 新增VMs
   * 取名:ubuntu211
   * 放在\VMs\ubuntu211
   * 4個 process
   * RAM :24576MB \(24\*1024\)
   * 選bridged路徑
     * 橋接實際網卡,不能接在VM上的網卡\(intel開頭\)
   * DVD/CD:放入ubuntu\_server\_64.iso
     * 安裝完記得退片
4. 安裝Ubuntu Server 設定很重要
   * 選英文
   * 選擇ens33 \(定死IP\)
     1. Edit IPv4
     2. Manual
     3. 設定IP,如表格1
     4. 確認出現static



1. proxy不填
2. mirror address:[http://free.nchc.org.tw/ubuntu](http://free.nchc.org.tw/ubuntu)
   * 找最快的:國網
3. 硬碟切割
   * Use an entire disk and set up LVM
     * ubuntu-vg\(new\)&gt;LVM volume group &gt; create logical volume
       * SWAP \(一般8G就夠用\)
       * 12G
     * at /\(根目錄\)&gt;edit&gt;size改成max
   * done&gt; continue
4. 設定帳號名稱
   * 都一樣比較好
   * 帳號:ubuntu
   * 密碼:ubuntu \(自己編\)
   * your server’s name : ubuntu211 \(跟VMs名稱一致\)
5. SSH
   * 打開 \(有X\)
6. 選Done &gt; reboot

```text
                swap in
          <------------------↰
         |                   |
     ____|____           ____|____
    |         |         |        |
    |   RAM   |         |  SWAP  |   swap(硬碟割出來)
    |_________|         |________|
         |                   |
         |                   |
         ↳------------------>
                swap out
```

|   | ens33 IPv4 configuration |  |
| :--- | :---: | :---: |
| subnet | 192.168.100.0/24 | C級網段 |
| address | 192.168.100.211 | VM的IP |
| gatway | 192.168.100.254 | 中華電信gateway |
| Name servers | 168.95.1.1 , 101.101.101.101 | DNS |
| searh domains | example.org | 測試用,保證不侵權 |



{% hint style="info" %}

{% endhint %}

### 



