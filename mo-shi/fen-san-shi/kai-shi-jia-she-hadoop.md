# 開始架設hadoop

## 所需環境



1. 改hosts
2. 創建hadoop 帳號
   * 專門給hadoop環境的帳號
3. ssh 數位簽章登入 hadoop 帳號 
   * 無密碼登入
   * ip 跟主機改名才可以做
   * 若出現問題較好除錯

{% tabs %}
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

