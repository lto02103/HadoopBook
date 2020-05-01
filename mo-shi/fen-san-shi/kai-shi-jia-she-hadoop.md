# 開始架設hadoop

## 所需環境



* 改hosts
  * 讓nodes之間認得彼此的電腦是誰
* 創建hadoop 帳號
  * 專門給hadoop環境的帳號
* ssh 數位簽章登入 hadoop 帳號 
  * 無密碼登入
  * ip 跟主機改名才可以做
  * 若出現問題較好除錯
* 
{% tabs %}
{% tab title="更改hosts" %}
* 更改hosts檔，彼此nodes才認得對方的電腦是誰
* 除了所有的nodes的hosts之外，還要加上localhost

```text
# root
sudo -i

# 編輯hosts檔
# 格式 : ip FQDN alias
# FQDN = hostname + domainname 
nano /etc/hosts

    # 本機一定要加上去   
    # 確認所有TCP/IP socket 程式(服務) 能正常運作
    127.0.0.1 localhost.localdomain localhost
    
    # 有多少個nodes就加多少個nodes上去
    #      ip             FQDN            alias
    192.168.100.211 bdse211.example.org bdse211
    192.168.100.212 bdse212.example.org bdse212
    
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
```
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

{% endtab %}
{% endtabs %}

## 使用Hadoop上的注意事項



1. 要檢查沒有其他使用者才可以使用poweroff或是shutdown

```text
sudo -i  # 切成#
who
lsof -nPi | grep -i 'esta' 
```

```text
Who
1. 0 :圖形介面(desktop)的terminal
2. pts/1 :遠端terminal(putty) (配合最後一欄的IP:VMnet8的IP)
3. tty/3 :3號的console (ctrl+alt+3)
```

若不檢查,直接poweroff。hadoop上的這些資料會損毀

* namenode
* nodemanager
* spark master

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
|         |
|  putty  |
|_________|
   |    ↑    
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

