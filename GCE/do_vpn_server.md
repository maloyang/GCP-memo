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
- 修改 /etc/ppp/pptpd-options ，檔案最後加入
```
ms-dns 8.8.8.8
ms-dns 1.1.1.1
```
- 
