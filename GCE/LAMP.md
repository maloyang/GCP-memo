# LAMP: 在Linux(ubuntu)下安裝Apache, MariaDB(MySQL), PHP 的紀錄

- 可以參考[這一篇](https://medium.com/@rommelhong/ubuntu-18-04-lts-apache-mysql-php-lamp-%E5%AE%89%E8%A3%9D%E8%A8%AD%E5%AE%9A-c46e6f54f254)

## 安裝 MariaDB or MySQL

### 安裝
- sudo apt update
- sudo apt install mariadb-server
- 確認系統是否有正常跑起來: `sudo systemctl status mysql` or `systemctl status mariadb`
  - 啟動: systemctl start mariadb
  - 關閉: systemctl stop mariadb
  - 重啟: systemctl restart mariadb

### 設定root and db

- sudo mysql_secure_installation

```
- Set root password? [Y/n] y
- Remove anonymous users? [Y/n] y
- Disallow root login remotely? [Y/n] y
- Remove test database and access to it? [Y/n] y
- Reload privilege tables now? [Y/n] y
```

### 設定遠端連線的使用者

- 先用 root 登入設定 `sudo mysql -u root -p`，這樣設定後，會新增一個 user 帳號，其密碼是 userpassword

```
MariaDB [(none)]> CREATE USER 'user'@'%' IDENTIFIED BY 'userpassword';                          
Query OK, 0 rows affected (0.00 sec)
                                                                                                                   
MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO 'user'@'%' IDENTIFIED BY 'userpassword' WITH GRANT OPTION; 
Query OK, 0 rows affected (0.00 sec)
                                                                                                                   
MariaDB [(none)]> FLUSH PRIVILEGES;                                                                                
Query OK, 0 rows affected (0.00 sec)
```

- 還是連不上DB --> 編輯 `sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf` ，把 `bind-address = 127.0.0.1` 註解掉就行了!

- 重啟DB `sudo systemctl restart mysql`

## 安裝Apache, PHP

- 安裝 Apache: `sudo apt install apache2`
- 安裝 Apache for php的套件:  `sudo apt install php libapache2-mod-php php-mysql`
- 重開 Apache 讓新設定啟用: `sudo systemctl status apache2`



### 因為Apache這預設網頁目錄在 /var/www/html下 (預設有一個 index.html 在)

- 先建立一個 /var/www/html/info.php 測試php功能
`sudo nano /var/www/html/info.php`
