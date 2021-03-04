# GCE ubuntu docker DB
- demo for KHPY 2021-03-04 lighting talk

## Create ubuntu 18.04 when 2021-03-04

- area: us-central1-a
- N1, f1-micro
- ubuntu18.04, 20G

## initial

### 改時區
- sudo timedatectl set-timezone Asia/Taipei

### 設定swap
- sudo swapon --show
- 不會有輸出，因為沒有swap
- free -h
- 查ram, swap
- sudo fallocate -l 1G /swapfile
  - 一般建議swap是ram的2倍
- sudo chmod 600 /swapfile
- sudo mkswap /swapfile
- sudo swapon /swapfile
- 這時swap只是暫時啟用，電腦重開就不使用了，因此要設定 /etc/fstab檔 --> `sudo nano /etc/fstab`
- 在檔案結尾加入一行 `/swapfile none swap sw 0 0`

- 用 `df -h` 看空間 / `free -h` 看ram, swap空間


## 安裝docker 
[docker指令文章](https://docs.docker.com/config/containers/start-containers-automatically/)

- sudo apt update
- sudo apt install docker.io
- `sudo apt install docker-compose` -> 如果要用yml file要安裝
- 下 `sudo systemctl enable docker` 就可以在重開後不用登入就自動執行，所有有用--restart參數的docker指令就都生效了!!!!
  - `sudo service docker start`  → 用這個指令在沒有登入時docker服務不會跑起來


## docker 跑 mariadb

- 手動配置port
- `sudo docker run --name mariadb_13306 -e MYSQL_ROOT_PASSWORD=my-pwd -d --restart always -p 13306:3306 mariadb`
- `--restart always` 要在 -d 正後方，這樣在登入後就會跑起來
- 注意!! 外部port寫在前面!
- 看有沒有跑起來 `sudo docker ps`
- 然後在打開防火牆對外port: 13306

## docker 刪除

- `sudo docker ps` 列出清單
- `sudo docker stop 932e9476ca85` 先停止
- `sudo docker rm 932e9476ca85` 再刪除

## db test

- 用 https://www.adminer.org/ 簡單測試一下
- IP:Port


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
      - 8080:8080
```

- 上面的yml檔意思是: 安裝mariaDB，對外13306 port, 然後安裝 adminer 為管理工具container中，對外port為8080
- 安裝 docker-compose  `sudo apt install docker-compose`
- 讓他背景執行，結果如下: `sudo docker-compose -f stack.yml up -d`
- 如果是測試可以不加 -d 可以用 ctrl^c 就中斷

