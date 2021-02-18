# 在DigitalOcean vm建立基本系統的記錄 (GCE也用的上)

## 安裝 ubuntu-s-1vcpu-2gb-nyc1 when 2021-02-18

- ubuntu18.04
- 2G/50GB/2TB/1 CPU ($10/mo)
- 選 Password

## 初始化的文章 [link](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04?utm_source=local&utm_medium=Email_Internal&utm_campaign=Email_UbuntuDistroNginxWelcome&mkt_tok=eyJpIjoiTXpSak16aGlOekJsWXpZMSIsInQiOiJBRzJzTURwbXJHdGU5Q2E0cXhDMTZMeWZJSUg2bWlPblwvZ3drekF0dlRNUzU4d3FiOWNiSllrbGNPMHBaNFo2VUFqbnYxUWFmN3hXS011eVhPMEdwTncxXC9nK1h5OXhoUEY4K3p5ZUx4QTArbjhONStXbE1Vb0dZd2RhMXhVM0ZOIn0%3D)

- 先用root連進去 (從web console來連也可以)
- 建一個平時用的帳號:  
- adduser user1
- 可以把新user加入sudo權限
- usermod -aG sudo user1
- 用 `ufw status` 看防火牆是 inactive的--> ufw是沒有啟用的 → 很危險，因為root容易被try過
- `ufw allow OpenSSH`
- `ufw enable`
- 就可以啟用，並只讓ssh連進來console操作

### 設定root不能用putty ssh登入 (很重要!!!! 至少root要不可遠端登入，不然很快就會被攻破)

- 編輯 【/etc/ssh/sshd_config】
- 修改 PermitRootLogin 項目為 no
- 再以 service sshd restart 載入設定，登出後，再以root連線被拒

### 改時區
- sudo timedatectl set-timezone Asia/Taipei

### 設定swap
- sudo swapon --show
- 不會有輸出，因為沒有swap
- free -h
- 查ram, swap
- sudo fallocate -l 1G /swapfile
  - 一般建議swap是ram的2倍
- ls -lh /swapfile
- sudo chmod 600 /swapfile
- sudo mkswap /swapfile
- sudo swapon /swapfile
- 這時swap只是暫時啟用，電腦重開就不使用了，因此要設定 /etc/fstab檔
- 在檔案結尾加入一行 `/swapfile none swap sw 0 0`

- 用 `df -h` 看空間 / `free -h` 看ram, swap空間


## 安裝docker 
[ref1](https://medium.com/%E4%B8%80%E5%80%8B%E5%B0%8F%E5%B0%8F%E5%B7%A5%E7%A8%8B%E5%B8%AB%E7%9A%84%E9%9A%A8%E6%89%8B%E7%AD%86%E8%A8%98/docker-%E5%AD%B8%E7%BF%92%E7%AD%86%E8%A8%98-%E5%AE%89%E8%A3%9D-docker-8adb49a4c4ce)
[docker指令文章](https://docs.docker.com/config/containers/start-containers-automatically/)

- sudo apt update
- sudo apt install docker.io
- sudo service docker start  → 用這個指令在沒有登入時docker服務不會跑起來
- 改下 sudo systemctl enable docker 就可以在重開後不用登入就自動執行，所有有用--restart參數的docker指令就都生效了!!!!
- 每次都要使用 sudo 調用 docker 很麻煩，我們可以將自己加入 docker 的 group 中
- 建議這樣做，這樣要給別人可以弄docker服務時，就不用給它sudo權限!!!!
- `sudo usermod -aG docker malo`

### 補充一下docker指令

- 查log，以圖中的image name來說 `docker logs mariadb_13306`
- 停掉 docker container  --> 以 container id 來操作 `docker stop 932e9476ca85`
- 刪除: `docker rm 932e9476ca85`
- 把停掉的container跑起來，用start指令: `docker start mariadb_13306`
- 更新設定: `docker update --restart unless-stopped sql1`

## docker 跑 mariadb

- 手動配置port
- `docker run --name mariadb_13306 -e MYSQL_ROOT_PASSWORD=my-pwd -d -p 13306:3306 mariadb`
- 上面的方式，不會自動restart，改用 (記得 --restart always 要在 -d 正後方)，這樣在登入後就會跑起來
- `docker run --name mariadb_13306 -e MYSQL_ROOT_PASSWORD=my-pwd -d --restart always -p 13306:3306 mariadb`
- 注意!! 外部port寫在前面!
- 然後在打開防火牆對外port: 13306
- `sudo ufw allow 13306/tcp`
- 這樣就可以連線了!


## docker mariadb with yml file: 使用yml 設定檔，用docker-compose一次開啟相關需要的container

- 先在`~/mariadb_docker`下建立stack.yml，如下:
```
# Use root/example as user/password credentials
version: '3.1'

services:

  db:
    image: mariadb
    restart: always
    ports:
      - 13306:3306
    environment:
      MYSQL_ROOT_PASSWORD: my-pwd

  adminer:
    image: adminer
    restart: always
    ports:
      - 3380:8080
```

- 上面的yml檔意思是: 安裝mariaDB，對外13306 port, 然後安裝 adminer 為管理工具container中，對外port為3380
- 安裝 docker-compose  `sudo apt install docker-compose`
- 讓他背景執行，結果如下: `docker-compose -f stack.yml up -d`
- 如果是測試可以不加 -d 可以用 ctrl^c 就中斷
- 為服務開個port， `sudo ufw allow 3380/tcp`, `sudo ufw allow 3306/tcp`
- 這時開網頁 `http://161.35.123.157:3380/`，只要輸入DB的帳密就可以登入


## 用docker建立SQL Server2019

- `docker run -e "ACCEPT_EULA=Y" -e "1qazADMIN@123" -p 1433:1433 --name sql1 -h sql1 -d mcr.microsoft.com/mssql/server:2019-latest`
- 開port 11433: `sudo ufw allow 1433/tcp`
- 再重開後，sql server就沒服務了，要加上 restart 參數: `docker update --restart unless-stopped sql1`

### 安裝 postgreSQL

- TODO


### 安裝vsftp

[ref](https://www.alvinchen.club/2019/12/11/ubuntu-ftp-server-%E6%9E%B6%E8%A8%AD/)

- sudo apt install vsftpd
- sudo nano /etc/vsftpd.conf    修改內容如下
```
/*加入以下資訊*/
pasv_enable=Yes
pasv_min_port=20000
pasv_max_port=20100
#local_root=/home/ftp /*(這是登入的根目錄)*/
```
- local_root可以讓使用者的起始目錄都在 /home/ftp 中
- sudo service vsftpd restart
- 建立一個不可登入的使用者: ftpuser1
- `sudo useradd -m ftpuser1 -s /usr/sbin/nologin`
- 設定密碼: `sudo passwd ftpuser1`
- 重開: `sudo service vsftpd restart`
- 開啟防火牆讓port=21, 和pasv可以通過: 
  - `sudo ufw allow 20000:20100/tcp`
  - `sudo ufw allow 21/tcp`
- 編輯 /etc/shells檔案，在最後加入  `/user/sbin/nologin`
- 限制上線人數、同IP上限數
  - 編輯 `/etc/vsftpd/vsftpd.conf`，加入
  - max_clients=5
  - max_per_ip=2
- 設定哪個帳號可以連線 (vsftpd.conf)
  - userlist_enable=YES
  - userlist_deny=NO
  - userlist_file=/etc/vsftpd.user_list
  - allow_writeable_chroot=YES
- 設定白名單於 `/etc/vsftpd.user_list`，加入: `ftpuser1`


