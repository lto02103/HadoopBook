---
description: 在yarn資料層上，安裝spark，必且用jupyterlab來連結yarn，使用koalas來進行資料分析
---

# Jupyter lab on yarn

## 介紹

* 實務上使用jupyterlab 來import pyspark，並且在yarn上用koalas來分析資料

## 所需工具

* pip3 install jupyterlab 
  * 包含jupyternotebook
    * 包含Ipython
* pip3 install koalas
  * 在pyspark上的pandas
  * 90%與pandas相同，spark3.0會內建koalas。spark-2.4.5目前移植中
* pip3 install pyspark

  * pyhton的套件，jupyter 要用的\(import pyspark\)
  * spark上的pyspark 類似python3，不一樣

```text
# root
sudo -i

# 安裝套件
pip3 install jupyterlab
pip3 install koalas
pip3 install pyspark
```

## 在pyspark中使用pandas\(koalas\)

* 在要跑spark上的worker執行就好
  * 主機：**bdse214.example.org** 
  * **帳號：**
    * 此範例用hadoop帳號執行spark
    * 也可以用egde node

{% tabs %}
{% tab title="設定個人環境變數" %}
* ~/.bashrc  : 個人設定檔
  * 啟用ARROW ：在pyspark上用pandas
    * spark2.4需要將arrow降級
    * spark3.0後koalas會完全移植上去，就不需要設定

```text
# hadoop account
su - hadoop

# 設定個人環境變數
# 應用程式路徑要放在.bashrc
nano ~/.bashrc
 ​   # Set ARROW
    export ARROW_PRE_0_15_IPC_FORMAT=1
    
    # ^+s > ^+x
    
# 重新載入設定檔
source ~/.bashrc # 方法一 (絕對路徑)
. .basrc # 方法二 (相對路徑:在目前目錄下重新載入.bashrc設定檔)
```
{% endtab %}

{% tab title="設定spark程式環境變數" %}
* spark-env.sh : 下spark相關指令，會執行此sh

  * 啟用ARROW ：在pyspark上用pandas
    * spark2.4需要將arrow降級
    * spark3.0後koalas會完全移植上去，就不需要設定

  ```text
  # hadoop account
  su - hadoop 

  # 先前已有複製，直接修改即可
  # 設定spark程式環境變數
  nano /usr/local/spark/conf/spark-env.sh
      # 在pyspark裡使用pandas
      # spark2.4才需要，spark3.0 koalas會完全移植上去，就不需要設定了
      # 將arrow降級​   
      # set ARROW
      export ARROW_PRE_0_15_IPC_FORMAT=1
  
    # ^+s > ^+x
  ```
{% endtab %}
{% endtabs %}

## 設定Jupyterlab遠端連線

* 在要跑jupyterlab上的worker執行就好
  * 主機：**bdse214.example.org** 
  * **帳號：**
    * 此範例用hadoop帳號執行jupyterlab
    * 也可以用egde node

{% tabs %}
{% tab title="設定遠端連線" %}
* 透過遠端登入jupyterlab，達到能用webUI界面操作server及分析資料

```text
# hadoop account
su - hadoop

# 設定遠端連線
# 產生設定檔
jupyter notebook --generate-config

# 確認
cd ~/.jupyter
ls
 
# 編輯設定檔
nano jupyter_notebook_config.py
 ​   # 在205行的地方加上
 ​   c.NotebookApp.ip = '*'
    # ^+s > ^+x
    
# 設定使用密碼，不使用token
# 密碼：hadoop (自行設定)
jupyter notebook password 
```
{% endtab %}

{% tab title="登入" %}
* 在win10上開啟web，在網址列輸入
  * bdse214:8888
  * 輸入密碼 : hadoop 即可登入
{% endtab %}
{% endtabs %}

## 參考資料

* Koalas

1. [10 minutes to Koalas](https://koalas.readthedocs.io/en/latest/getting_started/10min.html)
2. [Koalas: pandas API on Apache Spark](https://koalas.readthedocs.io/en/latest/)
3. [Koalas: Easy Transition from pandas to Apache Spark](https://databricks.com/blog/2019/04/24/koalas-easy-transition-from-pandas-to-apache-spark.html)

* **Apache Arrow**

1. [PySpark Usage Guide for Pandas with Apache Arrow](https://spark.apache.org/docs/latest/sql-pyspark-pandas-with-arrow.html#apache-arrow-in-spark)

* Jupyterlab

1. [Running a notebook server](https://jupyter-notebook.readthedocs.io/en/stable/public_server.html)

  




