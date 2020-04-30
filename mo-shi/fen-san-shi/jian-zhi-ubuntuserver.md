---
description: '版本 : Ubuntu-server 18.04.04'
---

# 建置Ubuntu-server

## 所需要的工具

* [x] VMware workstation player 15
* [x] ubuntu18.04.04-server-amd64.iso
* [x] 一顆耐心

## 安裝VMware workstation player

### 下載VMware

[VMware workstation player](https://www.vmware.com/tw/products/workstation-player/workstation-player-evaluation.html)  \(下載最新版即可\)

### 安裝方式

一鍵確認到底。

## VMs檔案放置位置

在C槽新增VMs的目錄，VMs\(虛擬機\)的資料一律放置在**C:\VMs**底下 \(管理上較好管理\) 😀 

## IP設定

### 確認目前ip

* 開啟cmd輸入`ipconfig` 確認目前ip並記錄下來

### 固定ip

* 在win10固定IP
  * 為了建置Hadoop的叢集
  * Hadoop叢集會抓IP,IP每次都不固定會跑不了

| 名稱 | IP | 描述 |
| :--- | :---: | :---: |
| IP位置 | 192.168.100.210\(個人IP\) | 座號x10 |
| 子網路遮罩 | 255.255.255.0 | C級網域 |
| 預設閘道 | 192.168.100.254 | 中華電信gateway |
|  | DNS |  |
| 慣用 | 168.95.1.1 | 中華電信 |
| 其他 | 101.101.101.101 | 國家網路中心\(twnic\) |

* 開啟cmd輸入`ipconfig` 確認ip是否正確

## 安裝ubuntu-server

### 下載ubuntu-server.iso

到[國網的鏡像站](https://free.nchc.org.tw/ubuntu-cd/bionic/)下載ubuntu-server   \(此為連結\)

> [ubuntu-18.04.4-live-server-amd64.iso](https://free.nchc.org.tw/ubuntu-cd/bionic/ubuntu-18.04.4-live-server-amd64.iso)  \( 點此下載即可\)

### 步驟說明

{% tabs %}
{% tab title="新增VM" %}


新增VMs

* 取名:bdse211
* 放在\VMs\bdse211



1. * 預設 spilt,500G
     * 若是機房可選one disk
     * spilt 是考慮到拷貝及回家使用的需求  
{% endtab %}

{% tab title="VM資源設定" %}


1. 設定
   * CPU 12
     * VM : 4,4
     * HOST : 4
   * RAM 64G
     * VM : 24,24 \(24 x1024\)
     * HOST : 16G
2. 4個 process
   * RAM :24576MB \(24\*1024\)
   * 選bridged路徑
     * 橋接實際網卡,不能接在VM上的網卡\(intel開頭\)
   * DVD/CD:放入ubuntu\_server\_64.iso
     * 安裝完記得退片
{% endtab %}

{% tab title="UbuntuServer設定" %}


1. 安裝Ubuntu Server 設定很重要
   * 選英文
   * 選擇ens33 \(定死IP\)
     1. Edit IPv4
     2. Manual
     3. 設定IP,如表格1
     4. 確認出現static
2. proxy不填
3. mirror address:[http://free.nchc.org.tw/ubuntu](http://free.nchc.org.tw/ubuntu)
   * 找最快的:國網
4. 硬碟切割
   * Use an entire disk and set up LVM
     * ubuntu-vg\(new\)&gt;LVM volume group &gt; create logical volume
       * SWAP \(一般8G就夠用\)
       * 12G\(設定12G也可以\)
     * at /\(根目錄\)&gt;edit&gt;size改成max
   * done&gt; continue
5. 設定帳號名稱
   * 都一樣比較好
   * 帳號:ubuntu
   * 密碼:ubuntu \(自己編\)
   * your server’s name : bdse211 \(跟VMs名稱一致\)
6. SSH
   * 打開 \(有X代表打開\)
7. 選Done &gt; reboot
{% endtab %}

{% tab title="ubuntuserverIP\_table" %}


| ens33 IPv4 configuration |  |  |
| :--- | :---: | :---: |
| subnet | 192.168.100.0/24 | C級網段 |
| address | 192.168.100.211 | VM的IP |
| gatway | 192.168.100.254 | 中華電信gateway |
| Name servers | 168.95.1.1 , 101.101.101.101 | DNS |
| searh domains | example.org | 測試用,保證不侵權 |
{% endtab %}
{% endtabs %}

### SWAP觀念

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



### 



