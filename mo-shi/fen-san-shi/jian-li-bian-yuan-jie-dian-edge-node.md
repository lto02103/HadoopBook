---
description: 也就是Client端
---

# 建立邊緣節點\(Edge Node\)

## 邊緣節點

* 管理資源的節點無法當Client端
  * NameNode
  * ResourceManager
  * 硬要也是可以啦，提早吃自己而已
* 其餘所有的機器\(workers\)都可以是邊緣節點，差別在於機房內或機房外  
  * 機房外 : 邊緣節點\(Edge Node\)
  * 機房內 : worker
* 結論 : **只有worker能拿來當作Edge Node**，但會消耗一部份的系統資源

## 設定步驟

{% tabs %}
{% tab title="在NameNode的主機上" %}
* NameNode : bdse211.example.org
* 若有HA，HA NameNodes 都要操作一次

```text
# root
sudo -i

# 建立user帳號
adduser user01

# 加到hadoop群組裡
gpasswd -a user01 hadoop

# 確認
groups user01
   
# 幫user在hdfs建目錄
su - hadoop
hdfs dfs -mkdir /user/user01

# 將權限全轉給user
hdfs dfs -chown user01 /user/user01

# 確認權限
hdfs dfs -ls /user
   
# 更新讓NameNode知道有新的user出現   
hdfs dfsadmin -refreshUserToGroupsMappings

```
{% endtab %}

{% tab title="要當作Egde Node的worker上" %}
* worker : bdse214.example.org 

```text
# root
sudo -i

# 建立user帳號(user名字要跟NameNode上的一樣)
adduser user01

# 加到hadoop群組裡
gpasswd -a user01 hadoop

# 確認
groups user01

# 切換成user
su - user01

# 設定環境變數
nano ~/.bashrc
    
    # hadoop
    
    # Set HADOOP_HOME
    export HADOOP_HOME=/usr/local/hadoop
	  # Set HADOOP_MAPRED_HOME
    export HADOOP_MAPRED_HOME=${HADOOP_HOME}
    # Add Hadoop bin and sbin directory to PATH
    export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
    
    # spark
    
    # Set SPARK_HOME
    export SPARK_HOME=/usr/local/spark
    # 在pyspark裡使用pandas
    # spark2.4才需要，spark3.0 koalas會完全移植上去，就不需要設定了
    export ARROW_PRE_0_15_IPC_FORMAT=1  
    
    # ^+s > ^+x
    
    
# 重新載入設定檔
source ~/.bashrc # 方法一 (絕對路徑)
. .basrc # 方法二 (相對路徑:在目前目錄下重新載入.bashrc設定檔)

# 查看
env 
    
# 測試map reduce 
# WebUI : http://bdse212.example.org:8088
hadoop jar  \
/usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.1.jar \
pi 30 10000 
   
```
{% endtab %}
{% endtabs %}



