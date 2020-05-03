---
description: 在hadoop系統裡架設spark，並且使用pyspark。
---

# Spark on yarn

## Spark介紹

* pyspark
  * python
  * 互動式
  * [使用dataframe，速度已經跟scala一樣快](https://databricks.com/session/a-tale-of-three-apache-spark-apis-rdds-dataframes-and-datasets)
  * 類似python3的互動式界面
* spark-submit
  * java
* spark-class
  * java
* spark-shell
  * scala
  * 互動式
* sparkR
  * R
  * 互動式
* spark-sql

  * SQL command
  * 互動式

  \*\*\*\*

### **參考資料 :** [**Spark performance for Scala vs Python**](https://stackoverflow.com/questions/32464122/spark-performance-for-scala-vs-python)\*\*\*\*

## 安裝spark

* 在進行此步驟前，要將叢集的所有節點停止
  1. mapred --daemon stop historyserver
  2. stop-yarn.sh
  3. stop-dfs.sh
* 在要跑spark上的worker執行就好
  * 主機：**bdse214.example.org** 
  * **帳號：**
    * 此範例用hadoop帳號執行spark
    * 也可以用egde node
* 參考資料 ： [Spark Configuration](https://spark.apache.org/docs/latest/configuration.html#yarn)

{% tabs %}
{% tab title="安裝spark" %}
* spark-2.4.5

```text
# root
sudo -i

# 下載spark-2.4.5版
# 請右鍵製貼上
# spark目前為2.4.5版本
# 若版本更新自行去官網搜尋 http://ftp.tc.edu.tw/pub/Apache/spark/
wget http://ftp.tc.edu.tw/pub/Apache/spark/spark-2.4.5/spark-2.4.5-bin-hadoop2.7.tgz

# 確認
ls -lh
# 222MB

# 查看
tar -tvf spark.... # tab自動完成

# 解開
# -C (change到 後面的目錄)
# /usr/local 裝第三方軟體的目錄
tar -xvf spark.... -C /usr/local  # tab自動完成

# 確認
ls -l /usr/local

# 查看檔案目錄真正大小
# 避免解縮縮過程檔案錯誤
# 259592 +-16k
du -s /usr/local/spark.... # tab

# 查看使用狀況
df -h /usr/local
# 2%

# 更名(官方用此方法)
mv /usr/local/spark-2.4.5-bin-hadoop2.7 /usr/local/spark

# 將/usr/local/spark 下的所有的檔案授權給hadoop帳號擁有
# -R :包含此目錄下所有的檔案
# 若terminal上有出現user:group=hadoop:hadoop，只是好巧。建議再做一次較好
chown -R hadoop:hadoop /usr/local/spark

# 登出
exit
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
 ​   # Set SPARK_HOME
    export SPARK_HOME=/usr/local/spark
    
    # ^+s > ^+x
    
# 重新載入設定檔
source ~/.bashrc # 方法一 (絕對路徑)
. .basrc # 方法二 (相對路徑:在目前目錄下重新載入.bashrc設定檔)

# 查看
env | grep 'spark'

# 確認版本以及找得到$SPARK_HOME
spark version
```
{% endtab %}

{% tab title="設定spark程式環境變數" %}
* spark-env.sh : 下spark相關指令，會執行此sh，會來找

  * HADOOP\_CONF\_DIR  : hadoop設定檔的路徑
  * PYSPARK\_PYTHON     : python的路徑
    * 用pyspark

  ```text
  # hadoop account
  su - hadoop

  # 從範本中複製一份spark-env.sh後編輯
  cd /usr/local/spark/conf
  cp spark-env.sh.template spark-env.sh 

  # 設定spark程式環境變數
  nano spark-env.sh
  	export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop
    export PYSPARK_PYTHON=/usr/bin/python3
  
    # ^+s > ^+x
  ```
{% endtab %}
{% endtabs %}

## 啟動spark

1. 先啟動叢集
   * 在bdse211.example.org上
     * start-dfs.sh
   * 在bdse212.example.org上
     * start-yarn.sh
   * 在bdse213.example.org上
     * mapred --daemon start historyserver
2. 開啟WebUI監控
   * NameNode,DataNode
     * http://bdse211.example.org:9870
   * YARN
     * http://bdse212.example.org:8088

{% hint style="info" %}
* 養成好習慣，jps確認一下

```text
# 確認
jps
```
{% endhint %}

## 測試spark

```text
# hadoop account
su - hadoop

# 語法
./bin/spark-submit --class path.to.your.Class \
--master yarn \
--deploy-mode cluster \
[options] \
<app jar> \
[app options]

# 測試spark
cd $SPARK_HOME
./bin/spark-submit --class org.apache.spark.examples.SparkPi \
--master yarn \
--deploy-mode cluster \
--driver-memory 1g \
--executor-memory 1g \
--executor-cores 1 \
--num-executors 3 \
--queue default \
examples/jars/spark-examples*.jar \
10

# WebUI查看狀況
http://bdse212.example.org:8088
```

## 處理jar檔

* 每次當你執行spark-submit，spark會在$SPARK\_HOME/jars下把所有的jar檔打包上傳到HDFS，並執行程式
  * 若不處理jars，會減少效能

{% tabs %}
{% tab title="查看jars目錄" %}
* 226個
* 235Mb

```text
# hadoop account
su - hadoop

# 所有jar檔數量
cat /usr/local/spark/jars | wc -l 

# 所有jar檔大小
du -s /usr/local/spark/jars
```
{% endtab %}

{% tab title="將jars目錄上傳到HDFS" %}
* 為了處理這樣的問題，要事先將jars目錄下的檔案丟到HDFS裡，之後在執行spark-submit的時候，請spark自己去HDFS裡找jars

```text
# hadoop account
su - hadoop

# -p : parent，若沒有父目錄，就一併建立
hdfs dfs -mkdir -p /user/spark/share/jars
   
# 將jars所有的檔案內容上傳到HDFS
hdfs dfs -put $SPARK_HOME/jars/* /user/spark/share/jars/

# 檢查
# hdfs dfs -ls /user/spark/share/jars 
```
{% endtab %}

{% tab title="設定jars目錄路徑" %}
* 將jars目錄路徑加入hdfs裡，請spark到此目錄下找jar檔

```text
# hadoop account
su - hadoop

# 複製spark-defaults.conf
cd /usr/local/spark/conf
cp spark-defaults.conf.template spark-defaults.conf

# 更改spark-defaults.conf
nano spark-defaults.conf
 ​   spark.yarn.jars hdfs://bdse211.example.org/user/spark/share/jars/*
 
#    if using NameNode HA
#    spark.yarn.jars hdfs://hacluster(名字)/user/spark/share/jars/* 
 
```
{% endtab %}
{% endtabs %}

