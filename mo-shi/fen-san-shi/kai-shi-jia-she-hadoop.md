---
description: 複製VMs後才可以繼續下列步驟，有兩台VMs就要重複做兩次。
---

# 開始架設hadoop

## 事前準備

* 更改hosts
  * 讓nodes之間認得彼此的電腦是誰
* 創建hadoop 帳號
  * 專門管理hadoop系統的帳號
* ssh 數位簽章登入 hadoop 帳號 
  * 無密碼登入
  * ip 跟主機改名才可以做
  * 若出現問題較好除錯

{% tabs %}
{% tab title="更改hosts" %}
* 更改hosts檔，彼此nodes才認得對方的電腦是誰
* 除了所有的nodes的hosts之外，還要加上localhost

```text
# root
sudo -i

# 編輯hosts檔
# 格式 : ip FQDN alias
# FQDN = Fully Qualifed Domain Name 
# FQDN = hostname + domainname 

nano /etc/hosts

    # 本機一定要加上去   
    # 確認所有TCP/IP socket 程式(服務) 能正常運作
    127.0.0.1 localhost.localdomain localhost
    
    # 有多少個nodes就加多少個nodes上去
    #      ip             FQDN            alias
    192.168.100.211 bdse211.example.org bdse211
    192.168.100.212 bdse212.example.org bdse212
    192.168.100.213 bdse213.example.org bdse213
    192.168.100.214 bdse214.example.org bdse214
    192.168.100.215 bdse215.example.org bdse215
    
#   192.168.56.90 master1.example.org master1
#   192.168.56.89 master2.example.org master2
#   192.168.56.88 master3.example.org master3
#   192.168.56.91 worker1.example.org worker1
#   192.168.56.92 worker2.example.oeg worker2
#   192.168.56.85 client.example.org  client
    
    # ^+s 存檔 > ^+x 退出    

# 檢查換行是否正確
cat -A /etc/hosts

# 若有出現微軟換行^M$，要置換掉 (用filezilla傳送檔案)
sed -i 's/\r//g' hosts 

# 再檢查一次
cat -A /etc/hosts   # 出現$結尾才是正確

# 確認hosts內容是否正確(確認連線，所有的hosts都要確認)
ping -c4 bdse211.example.org
ping -c4 bdse212.example.org
ping -c4 bdse213.example.org
ping -c4 bdse214.example.org
ping -c4 bdse215.example.org
```
{% endtab %}

{% tab title="新增hosts至win10" %}
* 要讓win10知道我們自定義的hosts是誰，只做一次
* 將hosts內容貼到C:\Windows\System32\drivers\etc\下的hosts 裡面
  * 不能override，要append上去

{% code title="hosts" %}
```text
192.168.100.211 bdse211.example.org bdse211
192.168.100.212 bdse212.example.org bdse212
192.168.100.213 bdse213.example.org bdse213
192.168.100.214 bdse214.example.org bdse214
192.168.100.215 bdse215.example.org bdse215
```
{% endcode %}
{% endtab %}

{% tab title="創建hadoop 帳號" %}
* 創建hadoop帳號 

```text
# 創建hadoop帳號
sudo -i
adduser hadoop
password:hadoop (自行設定)
# 接著duble check
# 接著6次 enter

# 檢查帳號是否建立
grep '^hadoop' /etc/passwd /etc/shadow /etc/group 

# 檢查有沒有hadoop的家
ls -l /home 

# 退出
exit
```
{% endtab %}

{% tab title="數位簽章登入" %}
* 透過數位簽章登入所有主機的hadoop帳號

```text
# hadoop account
su - hadoop
cd
pwd

# 確認open-ssh版本
dpkg -l | grep 'openssh'  # 7.6版

# client端要比server端要新，才會認得key
# putty 要0.73後
# in git 
ssh -V # 8.2版

# 設定數位簽章
# RHEL派的一定要給密碼，會將密碼放在cache裡

ssh-keygen -t rsa 
    Enter file in which to save the key: [Return]
    Enter passphrase (empty for no passphrase): [Return]
    Enter same passphrase again: [Return]
    
ls -l .ssh
    id_rsa     # 私鑰
    id_rsa.pub # 公鑰
    
# 將公鑰灑給其他電腦
# 當對方透過ssh要連進來時，會去下載公鑰，並產生自己的私鑰連進來。
# 此步驟非正規作法，工作上這樣做要回家吃自己
# 有N個主機就灑N次
# 格式ssh-copy id hadoop@FQDN
ssh-copy id hadoop@localhost            # 灑給自己(方法一)
ssh-copy id hadoop@bdse211.example.org  # 灑給自己(方法二)
ssh-copy id hadoop@bdse212.example.org  # 灑給其他主機
ssh-copy id hadoop@bdse213.example.org  # 灑給其他主機
ssh-copy id hadoop@bdse214.example.org  # 灑給其他主機
ssh-copy id hadoop@bdse215.example.org  # 灑給其他主機

# 檢查收到幾把公鑰
# .ssh/authorized_keys : 此目錄只有hadoop本人可以read/write
grep 'bdse' .ssh/authorized_keys | wc -l # 灑5把，就有5把

# 測試是否授權成功
# 以hadoop@bdse211為主機，登入對方主機
ssh hadoop@bdse212.example.org
exit
ssh hadoop@bdse213.example.org
exit
ssh hadoop@bdse214.example.org
exit
ssh hadoop@bdse215.example.org
```

{% hint style="danger" %}
#### 授權數位簽章正規步驟

將id\_rsa.pub寄email給server管理員，管理員會在server裡的對方的家建立.ssh/authorized\_keys，將公鑰放進此目錄下，才算建立數位簽章。

對方才可以透過數位簽章的方式\(無密碼登入\)，登入你的家

* 無密碼登入非同小可，需層層回報讓上面主管簽名授權才可以。
{% endhint %}
{% endtab %}
{% endtabs %}

## 安裝hadoop

{% tabs %}
{% tab title="安裝hadoop" %}
* hadoop-3.2.1

```text
# root
sudo -i

# 下載hadoop-3.2.1版
# 請右鍵製貼上
# hadoop目前為3.2.1版本
# 若版本更新自行去官網搜尋 https://www.apache.org/dyn/closer.cgi/hadoop/common
wget http://ftp.tc.edu.tw/pub/Apache/hadoop/common/hadoop-3.2.1/hadoop-3.2.1.tar.gz 

# 確認
ls -lh
# 343MB

# 查看
tar -tvf hadoop.... # tab自動完成

# 解開
# -C (change到 後面的目錄)
# /usr/local 裝第三方軟體的目錄
tar -xvf hadoop.... -C /usr/local  # tab自動完成

# 確認
ls -l /usr/local

# 查看檔案目錄真正大小
# 避免解縮縮過程檔案錯誤
# 919332k +-16k
du -s /usr/local/ha.... # tab

# 查看使用狀況
df -h /usr/local
# 2%

# 更名(官方用此方法)
mv /usr/local/hadoop-3.2.1 /usr/local/hadoop

# 將/usr/local/hadoop 下的所有的檔案授權給hadoop帳號擁有
# -R :包含此目錄下所有的檔案
# 若terminal上有出現user:group=hadoop:hadoop，只是好巧。建議再做一次較好
chown -R hadoop:hadoop /usr/local/hadoop

```
{% endtab %}

{% tab title="設定個人環境變數" %}
* ~/.bashrc  : 個人設定檔

```text
# hadoop account
su - hadoop

# 設定個人環境變數
# 應用程式路徑要放在.bashrc
nano ~/.bashrc
 ​   # Set HADOOP_HOME
    export HADOOP_HOME=/usr/local/hadoop
	  # Set HADOOP_MAPRED_HOME
    export HADOOP_MAPRED_HOME=${HADOOP_HOME}
    # Add Hadoop bin and sbin directory to PATH
    export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
    # ^+s > ^+x
    
# 重新載入設定檔
source ~/.bashrc # 方法一 (絕對路徑)
. .basrc # 方法二 (相對路徑:在目前目錄下重新載入.bashrc設定檔)

# 查看
env 

# 確認版本以及找得到$JAVA_HOME
hadoop version
```

{% hint style="info" %}
* /etc/profile.d : 共用設定檔\($JAVA\_HOME放這裡\)
* ~/.bashrc : 個人設定檔\($HADOOP\_HOME放這裡\)
{% endhint %}
{% endtab %}

{% tab title="設定hadoop程式環境變數" %}
* hadoop-env.sh : 下hadoop相關指令，會執行此sh，會來找
  * JAVA\_HOME                  :  java的路徑
  * HADOOP\_HOME          : hadoop的路徑
  * HADOOP\_CONF\_DIR  : hadoop設定檔的路徑

```text
# hadoop account
su - hadoop

# 設定hadoop程式環境變數
nano /usr/local/hadoop/etc/hadoop/hadoop-env.sh
	export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
	export HADOOP_HOME=/usr/local/hadoop
	export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop
```
{% endtab %}
{% endtabs %}

## 更改hadoop系統設定檔

* 4 CPU, 24GB RAM per Node的設定

### 各節點設定

1. NameNode : bdse211.example.org
2. ResourceManager : bdse212.example.org
3. JobhostoryServer : bdse213.example.org \(**可以放在worker裡\)**
4. workers :
   * bdse213.example.org
   * bdse214.example.org
   * bdse215.example.org

### Port的對應

<table>
  <thead>
    <tr>
      <th style="text-align:left">Nodes</th>
      <th style="text-align:left">WebUI</th>
      <th style="text-align:left">Host:nodes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">NameNode</td>
      <td style="text-align:left">
        <p>9870(3.x)</p>
        <p>50070(2.x)</p>
      </td>
      <td style="text-align:left">
        <p>8020</p>
        <p>&#x53EA;&#x8981;&#x52D5;HDFS,&#x5C31;&#x6703;&#x4F7F;&#x7528;8020port</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">ResourceManager</td>
      <td style="text-align:left">8088</td>
      <td style="text-align:left">
        <p>8032</p>
        <p>&#x53EA;&#x8981;&#x52D5;YARN,&#x5C31;&#x6703;&#x4F7F;&#x7528;8020port</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">JobHistotryServer</td>
      <td style="text-align:left">19888</td>
      <td style="text-align:left">
        <p>10020</p>
        <p>&#x8DDF;NodeManger&#x6E9D;&#x901A;</p>
      </td>
    </tr>
  </tbody>
</table>* 參考資料
  1. [Apache hadoop 3.2.1](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/ClusterSetup.html)
  2. [cloudera](https://docs.cloudera.com/HDPDocuments/HDP2/HDP-2.6.5/bk_reference/content/hdfs-ports.html)

### 修改site檔

{% tabs %}
{% tab title="core-site.xml" %}
* core-site的設定檔
  * hdfs的目錄

```text
# hadoop account
su - hadoop

# 更改core-site.xml
nano /usr/local/hadoop/etc/hadoop/core-site.xml

<!-- 將hdfs系統的目錄設定在/home/hadoop/data下 -->
<!-- 此為全域變數，NameNode跟DataNode會自己偵測並產生資料夾-->
<!-- 
不用在hdfs-stie.xml設定 dfs.namenode.name.dir 和dfs.datanode.name.dir這兩種屬性
-->
<!--此為採用雲端硬碟的切割方式 (cloud HD partition)。
根目錄大(/)，家的空間就大(/home)
用df -h 或 mount | grep -E 'ext[234] | xfs' 確認。
xfs 為RHEL7後才有的檔案系統。
-->
<!-- 雲端硬碟：home很大，伺服器：home很小(最多5G)-->
<property>
    <name>hadoop.tmp.dir</name>         
    <value>/home/hadoop/data</value>    
    <description>Temporary Directory.</description>
</property>

<!--設定用hdfs系統當作檔案系統-->
<!--bdse211.example.org 當作NameNode-->
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://bdse211.example.org</value>   
    <description>Use HDFS as file storage engine</description>
</property>

# hadoop 3.1 後才有 檢查xml語法有無正確
hadoop conftest

```
{% endtab %}

{% tab title="hdfs-site.xml" %}
* hdfs-site的設定檔

```text
# hadoop accont 
su - hadoop

# 更改hdfs-site.xml 
nano /usr/local/hadoop/etc/hadoop/hdfs-site.xml

<!-- 副本份數、block大小都由hdfs-stie.xml 做決定-->
<!-- 跟NameNode,DataNode相關也在此-->
<!-- 設定群組-->
<property>
    <name>dfs.permissions.superusergroup</name>
    <value>hadoop</value>
    <description>The name of the group of super-users. The value should be a single group name.</description>
</property>

<!--
<property>
    <name>dfs.blocksize</name>
    <value>134217728</value>
    <description>Block size</description>
</property>
-->
<!--
<property>
    <name>dfs.replication	</name>
    <value>3</value>
    <description>Number of replications</description>
</property>
-->

# check xml syntax
hadoop conftest
```
{% endtab %}

{% tab title="yarn-site.xml" %}
* yarn-site的設定檔
  * ResourceManager
  * NodeManager
* [白名單](https://issues.apache.org/jira/browse/MAPREDUCE-6704)

```text
# hadoop account
su - hadoop

# 更改yarn-site.xml
nano /usr/local/hadoop/etc/hadoop/yarn-site.xml

<!--bdse212.example.org當作ResourceManager-->
<!-- 不用設定port，預設用8088-->
<!-- 若要知道預設的port，請詳閱官方文件或是查看官方的source programing -->
<!-- hadoop 是jar檔，看不到source-->
<property>
	   <name>yarn.resourcemanager.hostname</name>
	   <value>bdse212.example.org</value>
</property>	
    
<!--告訴NodeManager要跑shuffle-->    
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>

<!--白名單-->
<!--放在此名單裡。NodeManager才會認得-->
<property>
    <name>yarn.nodemanager.env-whitelist</name>
    <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>     
</property>

<!--NodeManager的設定-->
<property>
    <name>yarn.nodemanager.resource.cpu-vcores</name>
    <value>3</value>
</property>
<property>
    <name>yarn.nodemanager.resource.memory-mb</name>
    <value>20480</value>
</property>
<property>
    <name>yarn.scheduler.maximum-allocation-vcores</name>
    <value>3</value>
</property>


<!--YARN工作排程器的設定-->
<!--平衡負載-->
<property>
    <name>yarn.resourcemanager.scheduler.class</name>
    <value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FairScheduler</value>
</property>
<property>
    <name>yarn.scheduler.minimum-allocation-mb</name>
    <value>1024</value>
</property>
<property>
    <name>yarn.scheduler.maximum-allocation-mb</name>
    <value>20480</value>
</property>
<property>
    <name>yarn.scheduler.increment-allocation-mb</name>
    <value>512</value>
</property>

# check xml syntax
hadoop conftest
```
{% endtab %}

{% tab title="mapred-site.xml" %}
* mapreduce的設定檔
  * map
  * reduce
  * mrappmaster

```text
# hadoop account
su - hadoop

# 更改mapred-site.xml
nano /usr/local/hadoop/etc/hadoop/mapred-site.xml

<!--mapreduce的資料層是yarn-->
<property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
</property>

<!--設定jobhistoryserver的主機-->
<!--jobhistoryserver利用10020 port跟nodemanager互相溝通-->
<property>
   <name>mapreduce.jobhistory.address</name>
   <value>bdse213.example.org:10020</value>
</property>

<!--設定jobhistoryserver的webUI-->
<!--bdse213.example.org 當作jobhistoryserver-->
<!--只能追蹤mapreduce，無法追蹤spark(不會使用mapreduce)-->
<property>
   <name>mapreduce.jobhistory.webapp.address</name>
	 <value>bdse213.example.org:19888</value>      
</property>

<!--mrappmaster的設定-->
<property>
    <name>yarn.app.mapreduce.am.resource.cpu-vcores</name>
    <value>3</value>
</property>
<property>
    <name>yarn.app.mapreduce.am.resource.mb</name>
    <value>6656</value>
</property>
<property>
    <name>yarn.app.mapreduce.am.command-opts</name>
    <value>-Xmx5324m</value>
</property>

<!--map的設定-->
<property>
    <name>mapreduce.map.cpu.vcores</name>
    <value>3</value>
</property>
<property>
    <name>mapreduce.map.memory.mb</name>
    <value>6656</value>
</property>
<property>
    <name>mapreduce.map.java.opts</name>
    <value>-Xmx5324m</value>
</property>
<property>
    <name>mapreduce.task.io.sort.mb</name>
    <value>1331</value>
</property>

<!--reduce的設定-->
<property>
    <name>mapreduce.reduce.cpu.vcores</name>
    <value>3</value>
</property>
<property>
    <name>mapreduce.reduce.memory.mb</name>
    <value>6656</value>
</property>
<property>
    <name>mapreduce.reduce.java.opts</name>
    <value>-Xmx5324m</value>
</property>


# 檢查xml語法有無正確
hadoop conftest
```
{% endtab %}

{% tab title="wokers" %}
* 要增加workers名單，NameNode,ResourceManager才知道workers是誰

```text
# hadoop account
su - hadoop

# 有N台worker就要放N台
# 給FQDN
# 新增workers名單
nano /usr/local/hadoop/etc/hadoop/workers
    bdse213.example.org
    bdse214.example.org
    bdse215.example.org
    
    # ^+s > ^+x
    
# 檢查換行是否正確 ，避免程式抓不到最後一行
# nano 2.9.3 存檔時會自動換行
cat -A /usr/local/hadoop/etc/hadoop/workers
```
{% endtab %}
{% endtabs %}

## 啟動hadoop

### 目錄

#### 啟動順序

1. start-dfs.sh
2. start-yarn.sh
3. mapred --daemon start historyserver

#### 關機順序

1. mapred --daemon stop historyserver
2. stop-yarn.sh
3. stop-dfs.sh

### 詳細過程

{% tabs %}
{% tab title="啟動HDFS" %}
* 在bdse211.example.org啟動HDFS
* WebUI
  * http://bdse211.example.org:9870

```text
# hadoop account 
su - hadoop

# 格式化
# 只需要一次
hdfs namenode -format

# 啟動hdfs
start-dfs.sh

# 查看有無啟動
jps
```

{% hint style="info" %}
jps指令

* bdse211主機會出現**NameNode & SecondaryNameNode**
* bdse212主機不會出現任何東西\(ResourceManager\)
* workers會出現**DataNode**
{% endhint %}
{% endtab %}

{% tab title=" 啟動YARN" %}
* 在bdse212.example.org啟動YRAN
* WebUI
  * http://bdse212.example.org:8080

```text
# hadoop account 
su - hadoop

# 啟動yarn
start-yarn.sh

# 查看有無啟動
jps
```

{% hint style="info" %}
jps指令

* bdse211主機會出現NameNode & SecondaryNameNode
* bdse212主機會出現**ResourceManager**
* workers會出現DataNode跟**NodeManager**
{% endhint %}
{% endtab %}

{% tab title="啟動Jobhistoryserver" %}
* 在bdse213.example.org啟動Jobhistoryserver

```text
# hadoop account 
su - hadoop

# 啟動

# 查看有無啟動
jps
```
{% endtab %}
{% endtabs %}



{% hint style="info" %}
錯誤要看logs檔

```text
# hadoop account 
su - hadoop

# logs
cd /usr/local/hadoop
less -N logs
```
{% endhint %}

## 節點重啟

## 使用Hadoop上的注意事項



1. 要檢查沒有其他使用者才可以使用poweroff或是shutdown

```text
# root
sudo -i 

# 檢查使用者 
who
lsof -nPi | grep -i 'esta' 
```

who

1. 0 : 圖形介面\(desktop\)的terminal
2. pts/1 : 遠端terminal\(putty\)
3. tty/3  : 3號的console \(ctrl+alt+3\) 



{% hint style="info" %}
若不檢查下列這些nodes，直接poweroff。hadoop上的資料會損毀

* namenode
* resourcemanager
* jobhistorysever
* spark master
{% endhint %}



|  | poweroff | shutdown |
| :--- | :--- | :--- |
| 功用 | 直接關閉 | 預設等待1分鐘,會通知其他使用者 |

1. 檢查時區

```text
timedatectl
```

若時間不對,叢集會跑不起來。

1. 正規步驟\(玩Linux時\)

**tags: 小心離職 資安**

* server走putty
* desktop直接走使用者

```text


win10(Host)        Linux(使用者)         管理系統(root)
 _________   帳密①   _________ sudo -i②   _________
|         |------->|         |--------->|         |
|  putty  |        |         |          |   GOD   | 安裝軟體才會進來
|_________|<-------|_________|<---------|_________|
             exit⑥    ↑   |      exit③      |    ↑
                      |   |                 |    |
                      |   | su - hadoop④    |    |
               exit⑤  |   |                 |    |
                      |   |                 |    |
                    __|___↓__   su - hadoop |    |
                   |         |<----x--------     |   
                   |   man   |   不准(會離職)      |
                   |         |___________________|
                   |_________|    sudo -i (失敗)
                   管理應用程式     


```

* 帳密:管理員使用的login帳號\(e.g :ubuntu\)
* Debian,ubuntu的root帳號是被停用的,只能由使用者login,再切root
* 在root裡,只能exit,不能su
* 要正常exit,否則檔案會損毀
* 一個使用者只開一個terminal

切一般使用者不用密碼就是錯誤的 會離職

```text
ps -f #檢查(process fullist)
```

1. 正規步驟\(正式上班時\)
   * 只能走putty

```text
win10(Host)  
 _________   
|                      |
|    putty       |
|_________|
   |  ↑    
   |    |    
   |    |    
   |    |    
   |    |    
   |    |   exit②   _________ 
   |    |__________|         |
   |               |   man   |
    -------------->|         |
     帳密(hadoop)①  |_________|
                    管理應用程式     

```

## 透過filezilla 傳送檔案的注意事項

* 要用ubuntu身份\(管理員\)
* 不能用hadoop身份\(一般使用者\)
* RHEL派
* Ubuntu派

