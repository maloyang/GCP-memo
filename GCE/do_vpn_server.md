# VPN Server for DigitalOcean or GCP
Digital Ocean: pptp VPN

## initial a VM
- 選Ubuntu 20.04 (LTS) x64
- CPU option: Regular Intel with SSD / $5/mo (1CPU,25G SSD,1T transfer data)
- Authentication : Password

## setup
- ref: 參考文章：https://www.linuxbabe.com/linux-server/setup-your-own-pptp-vpn-server-on-debian-ubuntu-centos

- sudo apt-get install pptpd -y
- 修改 /etc/ppp/pptpd-options, 這邊用google的，也可以用open dns的208.67.222.222
```
ms-dns 8.8.8.8
ms-dns 8.8.4.4
```

- 修改/etc/ppp/chap-secrets，最後面加入，這邊vpn client要登入的帳密
```
your_username pptpd your_password *
```

- 修改 /etc/pptpd.conf 在最後加入如下，localip是自己的vpn server ip, remoteip 是要配發的ip範圍
```
localip 10.0.0.1
remoteip 10.0.0.100-200
```

### 啟用 IP forwarding
- 編輯 /etc/sysctl.conf，把 net.ipv4.ip_forward = 1 取消註解
- 再使用 sysctl -p 讓剛剛的設定生效

### 規劃 firewall 讓vpn的private IP可以由對外的etho出去
- iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

- 另外，可以在 /etc/rc.local中加入這行，讓此設定開機後重新生效 --> 先不做

### 啟動 pptpd 的功能
- sudo systemctl start pptpd   or   sudo service pptpd start
- 再執行`systemctl enable pptpd` 這個指令可以讓 pptpd 開機時就執行

- 這時就可以用VPN Client去連線了，而vpn server的狀態查詢用 `service pptpd status` 結果如下
```
● pptpd.service - PoPToP Point to Point Tunneling Server
     Loaded: loaded (/lib/systemd/system/pptpd.service; disabled; vendor prese>
     Active: active (running) since Wed 2021-09-29 11:47:32 UTC; 23min ago
       Docs: man:pptpd(8)
             man:pptpctrl(8)
             man:pptpd.conf(5)
   Main PID: 8643 (pptpd)
      Tasks: 1 (limit: 1136)
     Memory: 636.0K
     CGroup: /system.slice/pptpd.service
             └─8643 /usr/sbin/pptpd --fg
```

### 查詢有哪一個帳號連線上來
- `last |grep ppp` 結果如下
```
farmiot2 ppp0         27.51.33.132     Fri Dec 17 17:25    gone - no logout
jerry    ppp0         114.33.247.201   Fri Dec 17 16:55 - 16:57  (00:01)
jerry    ppp1         111.184.233.170  Wed Dec 15 00:03 - 00:14  (00:10)
```


### ssh的部份
- 我是有再增加一個帳號，並且把root禁用遠端登入

