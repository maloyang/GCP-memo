# 記錄一些 GCE 的經驗

GCE就是Google Compute Engine，也就是一個可以讓你看一台雲端虛擬機的平台，像AWS, Digital Ocean也都是有提供這樣的服務的

## 開啟一台ubuntu vm

因為很簡單所以先不詳細寫了… 

- 只說明一下，如果你只是要一台基本的配置，跑跑你的web server, SQL, FTP Server等服務，可以開一台f1-micro的機器就好
- 我開的f1-micro規格(額度內免費)：1 vCPU, 0.6G ram, 25G HHD, 附一個臨時的對外IP，google會有一個預設全封的防火牆設置

## 設定時區

- 看一下預設時區 `timedatectl`
```
                      Local time: Wed 2020-07-08 07:08:08 UTC
                  Universal time: Wed 2020-07-08 07:08:08 UTC
                        RTC time: Wed 2020-07-08 07:08:09
                       Time zone: Etc/UTC (UTC, +0000)
       System clock synchronized: yes
systemd-timesyncd.service active: yes
                 RTC in local TZ: no
```
- 由於時區是由 /etc/localtime這一個軟連結決定的
```
malo_richart@instance-1:~$ ls -l /etc/localtime
lrwxrwxrwx 1 root root 27 Jun 26 18:08 /etc/localtime -> /usr/share/zoneinfo/Etc/UTC
```
- 因此下指令`sudo timedatectl set-timezone Asia/Taipei`，結果如下:
```
malo_richart@instance-1:~$ sudo timedatectl set-timezone Asia/Taipei
malo_richart@instance-1:~$ date
Wed Jul  8 15:18:17 CST 2020
```


## 安裝python3, postgreSQL

- 我使用的是 Ubuntu 18.04.4 LTS(image: ubuntu-1804-bionic-v20200626)，裡面預設就有python3

- Heroku中的postgreSQL是11.8版，所以是可以follow這個較新的版次的

- install postgreSQL:
```
sudo apt update
sudo apt install postgresql postgresql-contrib
```

- 這時postgreSQL會建利一個預設的user name `postgres`，我們可以下指令切換user
`sudo -i -u postgres` 然後提示列就會變成 `postgres@instance-1:~$`

- 打`psql`可以進去存取DB，然後`\q`跳出來，再以`exit`登出帳放
```
postgres@instance-1:~$ psql
psql (10.12 (Ubuntu 10.12-0ubuntu0.18.04.1))
Type "help" for help.

postgres=# \q
postgres@instance-1:~$ exit
logout
```

- 因為預設沒有pip3，所以`sudo apt install python3-pip`安裝一下 (套件比較多，大約270MB的東西要裝)

- `pip3 install virtualenv`
- 接著要依這篇文章建立virtual envirment就出問題了: [ref](https://medium.com/@akash16s/how-to-install-postgresql-and-pgadmin4-on-ubuntu-18-04-lts-c19895548df8)

```
$ mkdir pgadmin4
$ cd pgadmin4
$ virtualenv pgadmin4

Command 'virtualenv' not found, but can be installed with:
apt install virtualenv
Please ask your administrator.

```

- 由提示，再安裝一次 `sudo apt install virtualenv` --> 依然不行，變成是python2版本的指令

- 想改用 python3 -m venv 這一個套件來建立 virtualenv了...
    - 下指令 `python3 -m venv venv` 被指示要安裝 venv --> `sudo apt-get install python3-venv`
    - 這時再一次 `python3 -m venv venv` 就正常了!

- 然後進入virtual envirment後就可以開始安裝 pgadmin4了:
    - `wget https://ftp.postgresql.org/pub/pgadmin/pgadmin4/v4.8/pip/pgadmin4-4.8-py2.py3-none-any.whl`
    - `pip install pgadmin4–4.8-py2.py3-none-any.whl`
    - 跑了一堆東西後，終於安裝完成了…

- 來測試一下pgadmin4是否正常運作 `python venv/lib/python3.6/site-packages/pgadmin4/pgAdmin4.py`
    - 結果不能運作，說是少了module: url_encode
    `ImportError: cannot import name 'url_encode'`

    - 人家說這是在werkzeug==1.0.x版時會發生的，因為舊版才有url_encode
    ```
    Python 3.6.9 (default, Apr 18 2020, 01:56:04) 
    [GCC 8.4.0] on linux
    Type "help", "copyright", "credits" or "license" for more information.
    >>> import werkzeug
    >>> werkzeug.__version__
    '1.0.1'    
    ```
    - 因此我下指令改安裝舊版
    ```
    (venv) malo_richart@instance-1:~/pgadmin4$ pip install werkzeug==0.16.0
    Collecting werkzeug==0.16.0
    Downloading https://files.pythonhosted.org/packages/ce/42/3aeda98f96e85fd26180534d36570e4d18108d62ae36f87694b476b83d6f/Werkzeug-0.16.0-py2.py3-none-any.whl (327kB)
        100% |████████████████████████████████| 327kB 2.8MB/s 
    Installing collected packages: werkzeug
    Found existing installation: Werkzeug 1.0.1
        Uninstalling Werkzeug-1.0.1:
        Successfully uninstalled Werkzeug-1.0.1
    Successfully installed werkzeug-0.16.0    
    ```
    - 然後得到錯誤，依然卡關 `PermissionError: [Errno 13] Permission denied: '/var/lib/pgadmin'`
    - 看起來 pgAdmin4 是一個 GUI介面的 pg 管理套件，所以先不用也沒差… (不是我弄不起來喔)

- 改看[這一篇](https://blog.gtwang.org/linux/how-to-install-and-use-postgresql-ubuntu-18-04/)
    - 先來確認一下postgreSQL運作中 `systemctl status postgresql.service`
    ```
    ● postgresql.service - PostgreSQL RDBMS
    Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
    Active: active (exited) since Wed 2020-07-08 06:07:10 UTC; 58min ago
    Main PID: 4639 (code=exited, status=0/SUCCESS)
        Tasks: 0 (limit: 638)
    CGroup: /system.slice/postgresql.service

    Jul 08 06:07:10 instance-1 systemd[1]: Starting PostgreSQL RDBMS...
    Jul 08 06:07:10 instance-1 systemd[1]: Started PostgreSQL RDBMS.    
    ```

- 參考: https://cloud.tencent.com/developer/article/1436845
    - `sudo -u postgres psql postgres` 登入
    - 下指令建立帳號、資料庫
    ```
    pggeo=# create role pgtest1 with password 'ixnqjpostgresql' login;
    CREATE ROLE
    pggeo=# create database test1 owner 'pgtest1';
    CREATE DATABASE    
    ```
    - 修改設定，要可以遠端登錄
    
- 後來參考了這一篇：https://cloud.google.com/community/tutorials/setting-up-postgres
    - 原來不只是修改一個 `/etc/postgresql/9.3/main/postgresql.conf`中的 `listen_addresses = '*'`
    - 還要修改 `/etc/postgresql/9.3/main/pg_hba.conf` 末端加入 :
        `host    all             all           [YOUR_IPV4_ADDRESS]/32         md5`
    - 再 `sudo service postgresql restart`
    - 上一篇沒有說要改第二項，所以才不能動 --> 這邊有這個檔案的說明: https://docs.postgresql.tw/server-administration/client-authentication/the-pg_hba.conf-file
    - 不過 your_ip_address是指client端的IP，所以怪怪的，應該要有全開放的寫法才對

