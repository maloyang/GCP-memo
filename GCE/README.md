# 記錄一些 GCE 的經驗

GCE就是Google Compute Engine，也就是一個可以讓你看一台雲端虛擬機的平台，像AWS, Digital Ocean也都是有提供這樣的服務的

## 開啟一台ubuntu vm

因為很簡單所以先不詳細寫了… 

- 只說明一下，如果你只是要一台基本的配置，跑跑你的web server, SQL, FTP Server等服務，可以開一台f1-micro的機器就好
- 我開的f1-micro規格(額度內免費)：1 vCPU, 0.6G ram, 25G HHD, 附一個臨時的對外IP，google會有一個預設全封的防火牆設置
- Ubuntu 18.04.4 LTS(image: ubuntu-1804-bionic-v20200626)

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

- 打`psql`可以進去存取DB，然後`\q`跳出來，再以`exit`登出帳號
```
postgres@instance-1:~$ psql
psql (10.12 (Ubuntu 10.12-0ubuntu0.18.04.1))
Type "help" for help.

postgres=# \q
postgres@instance-1:~$ exit
logout
```

### 另外建立帳號
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

### 設定DB可以由外部連線

- 首先要在google的防火牆把相對應的port打開

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

- 參考這一篇：https://cloud.google.com/community/tutorials/setting-up-postgres
    - 先修改 `/etc/postgresql/9.3/main/postgresql.conf`中的 `listen_addresses = '*'`
    - 還要修改 `/etc/postgresql/9.3/main/pg_hba.conf` 末端加入 :
        `host    all             all           [YOUR_IPV4_ADDRESS]/32         md5`
    - 再 `sudo service postgresql restart`
    - 這邊有這個檔案的說明: https://docs.postgresql.tw/server-administration/client-authentication/the-pg_hba.conf-file
    - 不過 your_ip_address是指client端的IP，所以怪怪的，應該要有全開放的寫法才對


### 建立iot server安裝的venv環境

- 建立一個iot-server專案，需要的東西
    - python3 -m venv venv
    - source venv/bin/activate
    - pip install modbus_tk
    - pip install sqlalchemy
    - pip install requests
    - pip install pymssql
    - pip install psycopg2  --> 這個會出錯: Error: b'You need to install postgresql-server-dev-X.Y for building a server-side extension or libpq-dev for building a client-side application.\n'
    - 改成以apt安裝: sudo apt install python3-psycopg2 --> 也不行
    - sudo apt install libpq-dev python-dev  --> 先安裝要在本地端build套件的lib就可以用pip安裝了
    - pip install psycopg2  --> 結果：Successfully installed psycopg2-2.8.5
    - 最後還要在防火牆中把GCP的modbus port打開

----
## 安裝 MariaDB
參考這一篇：https://pcion123.github.io/2019/03/16/ubuntu-installmariadb/

- 下指令:
```
sudo apt update
sudo apt upgrade
sudo apt install mariadb-server
```
- 確認系統是否有正常跑起來 `sudo systemctl status mysql`
```
● mariadb.service - MariaDB 10.1.44 database server
   Loaded: loaded (/lib/systemd/system/mariadb.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2020-07-14 08:07:16 UTC; 44min ago
     Docs: man:mysqld(8)
           https://mariadb.com/kb/en/library/systemd/
 Main PID: 4690 (mysqld)
   Status: "Taking your SQL requests now..."
    Tasks: 27 (limit: 638)
   CGroup: /system.slice/mariadb.service
           └─4690 /usr/sbin/mysqld

Jul 14 08:07:16 db-server /etc/mysql/debian-start[4727]: Processing databases
Jul 14 08:07:16 db-server /etc/mysql/debian-start[4727]: information_schema
Jul 14 08:07:16 db-server /etc/mysql/debian-start[4727]: mysql
Jul 14 08:07:16 db-server /etc/mysql/debian-start[4727]: performance_schema
Jul 14 08:07:16 db-server /etc/mysql/debian-start[4727]: Phase 6/7: Checking and upgrading tables
Jul 14 08:07:16 db-server /etc/mysql/debian-start[4727]: Processing databases
Jul 14 08:07:16 db-server /etc/mysql/debian-start[4727]: information_schema
Jul 14 08:07:16 db-server /etc/mysql/debian-start[4727]: performance_schema
Jul 14 08:07:16 db-server /etc/mysql/debian-start[4727]: Phase 7/7: Running 'FLUSH PRIVILEGES'
Jul 14 08:07:16 db-server /etc/mysql/debian-start[4727]: OK
```

- 對mariadb初始安全性設定，下指令: `sudo mysql_secure_installation`，我使用時比下面多了一個是否使用目前root帳號的密碼，我是按enter直接跳過
```
# 是否要設置root權限密碼
- Set root password? [Y/n] y
# 是否要移除匿名登入
- Remove anonymous users? [Y/n] y
# 是否關閉遠端登入
- Disallow root login remotely? [Y/n] y
# 是否移除test預設庫
- Remove test database and access to it? [Y/n] y
# 是否重新載入資料表權限
- Reload privilege tables now? [Y/n] y
```

- 指令 `sudo mysql -u root -p` 確認是否可以用root登入
```
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 49                                                                                   
Server version: 10.1.44-MariaDB-0ubuntu0.18.04.1 Ubuntu 18.04                                                      
                                                                                                                   
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.                                               
                                                                                                                   
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.                                     
                                                                                                                   
MariaDB [(none)]> \q                                                                                               
Bye
```

### 設定遠端連線的使用者
- 先用root指入設定 `sudo mysql -u root -p`
```
# 建立使用者
$ CREATE USER 'my_account'@'%' IDENTIFIED BY 'my_password';
# 建立使用者權限
$ GRANT ALL PRIVILEGES ON *.* TO 'my_account'@'%' IDENTIFIED BY 'my_password' WITH GRANT OPTION;
# 刷新設置
$ FLUSH PRIVILEGES;
```
過程如下：
```
MariaDB [(none)]> CREATE USER 'user'@'%' IDENTIFIED BY 'userpassword';                          
Query OK, 0 rows affected (0.00 sec)
                                                                                                                   
MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO 'user'@'%' IDENTIFIED BY 'userpassword' WITH GRANT OPTION; 
Query OK, 0 rows affected (0.00 sec)
                                                                                                                   
MariaDB [(none)]> FLUSH PRIVILEGES;                                                                                
Query OK, 0 rows affected (0.00 sec)
```

- 確認建立的user
```
# 查看使用者
$ SELECT host,user,password FROM mysql.user;
# 查看使用者權限
$ SHOW GRANTS FOR 'my_account';
```

- 因為我們是在GCE的vm上建立服務的，因此要把port號 3306 打開

- 這時仍然是連不上，要再修改這一個檔案 `sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf`，把 `bind-address  = 127.0.0.1`註解掉就行了!

- 重開mariadb，讓設定值生效: `sudo systemctl restart mysql`


----
## 以下為測試後，好像沒什麼用，但暫時不刪掉，先留下來的資料


### 這邊依文章，開始安裝postgreSQL的工具 --> 應該不用裝也沒關係
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
