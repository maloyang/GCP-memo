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
sudo timedatectl set-timezone Asia/Taipei

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



