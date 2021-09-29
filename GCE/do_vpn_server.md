# VPN Server for DigitalOcean or GCP

## Digital Ocean: pptp VPN

### initial a VM
- 選ubuntu18.04 / basic
- CPU option: Regular Intel with SSD / $5/mo (1CPU,25G SSD,1T transfer data)
- Authentication : Password

- 初始化先參考文章<do_db_nyc1.md>
- 建立好了，再來設定vpn的事

### setup
- ref: https://www.digitalocean.com/community/tutorials/how-to-setup-your-own-vpn-with-pptp
- sudo apt-get install pptpd
- sudo nano /etc/pptpd.conf
- 在檔案的最後加入，其中localip是我們server的IP, 然後remoteip是配發給連進來的client的ip:
```
localip 10.0.0.1
remoteip 10.0.0.100-200
```
- /etc/ppp/chap-secrets
- 在檔案的最後加入，三個欄位分別是：登入用的username, 使用的vpn加密方式為pptp, 密碼
```
username1 pptpd password1
```
- 修改 /etc/ppp/pptpd-options ，檔案中找到'ms-dns'的字眼，加入以下的文字
```
ms-dns 8.8.8.8
ms-dns 1.1.1.1
```
- sudo service pptpd restart 重開
- 用指令`netstat -alpn | grep :1723`確認程式是否有正常在服務，結果如下，代表有正常
```
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 0.0.0.0:1723            0.0.0.0:*               LISTEN  
```
- 讓封包可以從對外的ip出去
- `sudo nano /etc/sysctl.conf` 把裡面的`net.ipv4.ip_forward = 1`註解拿掉，讓它生效
- 並下指令讓它生效 `sudo sysctl -p`

### 防火牆
- `sudo ufw allow 1723` 因為pptpd要用到1723 port，所以要開放它
- 

