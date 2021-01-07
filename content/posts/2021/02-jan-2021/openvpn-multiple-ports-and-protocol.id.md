---
title: "Openvpn Multiple Port and Protocol"
date: 2021-01-05T11:56:28+07:00
weight: 1
tags: ["OpenVPN", "VPN", "Linux"]
author: "berrabe"
showToc: true
TocOpen: True
hidemeta: false
disableShare: false
cover:
    image: "https://cdn.worldvectorlogo.com/logos/openvpn-logo-1.svg"
    alt: "Loh Imange Nya Ilang"
    relative: false
comments: false
---

&nbsp;

&nbsp;

## 1) TLDR
---
yo yo yo eee teammm ... welcome bek to my webseettt  :anguished: ... selamat pagi, siang atau malam buat temen2, om, tante, pakdhe, budhe, simbah kakung, simbah putri dan semuanya dah  :sleepy:

kali ini saya akan mendokumentasikan gimaneee caranyee setting banyak openvpn dalam satu server.

Mungkin kalian pernah berfikir untuk bypass limitasi yang di lakukan oleh pihak Vendor / ISP kan? semisal Vendor / ISP kalian tidak bisa buka reddit, netplik, skidipapap huahuahua, dsb. sudah coba pakai OpenVPN default (port 1194/UDP) malah ndak bisa connect juga OpenVPN nya ~~(Vendor Kampret)~~

atau kalian punya server di Data Center yang Vendor Internetnya mem-block semua port dan protokol default VPN?

oke oke, seperti biasaaa ... jangan tenang dan tetap panik  :kissing_smiling_eyes: karna kita akan bahas caranye, cekidot

&nbsp;

&nbsp;

&nbsp;

## 2) Setting Dan Install OpenVPN (Mau Ribet / Engga?)
---
pastinya ni ye, sebelum gunain OpenVPN Client, pasti harus punya OpenVPN Server nya dulu. Untuk itu saya nawarin nih.

mau yang cara install nya gampang ape susah, kalau gampang, silahkan gunakan script otomasi seperti dari bang [angristan](https://github.com/angristan/openvpn-install) atau dari bang [Nyr](https://github.com/Nyr/openvpn-install). Atau mau yang susah / install sendiri dari awal, bisa di cari sendiri tutor nya di google, ade bejibun, biasanya dari [DigitalOcean](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwjLuYPlhYTuAhX3gtgFHepUA3kQFjADegQIAxAC&url=https%3A%2F%2Fwww.digitalocean.com%2Fcommunity%2Ftutorials%2Fhow-to-set-up-an-openvpn-server-on-ubuntu-18-04&usg=AOvVaw16k6PvQCfYHz7gVl6G7j8N) ... sampe bilang kaga ada gw jitak lu  :innocent:

kebetulan saya pakai punya nya bang angristan (Fork dari Nyr), karna lebih mudah aje gitu, tinggal enter2, ikutin settingan default ... kalau internet server nya cepet, 5 menit selesai dan bisa langsung di pakai (lebih lengkap nya lihat [dokumentasi github angristan OpenVPN](https://github.com/angristan/openvpn-install))

> pastikan akfitkan port forwarding / DMZ jika OpenVPN server di bawah jaringan NAT

&nbsp;

&nbsp;

&nbsp;

## 3) Caranya Multiple Port / Protokol?
---
Wokehh, setelah installasi OpenVPN server yang saya yakin uda pada bisa, tinggal ikutin tutorial doang, copy / paste beres, tutorial pakai bahasa inggris? tinggal translate  :smiley:

kita akan lanjutkan bagaimana caranya buat multiple ports dan protokol, saya menggunakan script angristan, jika tidak sama bisa di sesuaikan karna tidak beda jauh

&nbsp;
### 3.a) Konfigurasi OpenVPN Server
sebagai contoh, saya akan buat server OpenVPN baru di port 443/TCP dari settingan default angristan (1194/UDP). Login ke OpenVPN Server menggunakaan SSH
1. copy / duplicate `server.conf` di folder `/etc/openvpn/`
```sh
> cd /etc/openvpn

> ls | grep -E "server.*.conf$" 
-rw-r--r-- 1 root root  696 Dec 17 07:27 server.conf

> cp server.conf server_tcp.conf
```

2. edit file `server_tcp.conf`, sesuaikan seperti di bawah
```sh
# setting port, protokol
port 443
proto tcp

# setting Network OpenVPN Server
server 172.16.111.0 255.255.255.0

# non-aktifkan static ip
# ifconfig-pool-persist ipp.txt

# setting log file untuk mode 443/TCP
status /var/log/openvpn/status_tcp.log
```

3. tambahkan rule masqurade agar network OpenVPN `(172.16.111.x/24)` dapat akses ke internet `(interface eth0 / WAN)`
```sh
iptables -t nat -A POSTROUTING -s 172.16.111.0/24 -o eth0 -j MASQUERADE
```

4. Buat Config untuk Client, terserah namanya apa, format `xxx.ovpn` dan aktifkan OpenVPN server TCP tadi
```sh
> ./openvpn-install.sh
> systemctl start openvpn@server_tcp
> systemctl enable openvpn@server_tcp
```

&nbsp;
### 3.a) Konfigurasi OpenVPN Client
karna multi ports dan protkol tidak di dukung secara default oleh script angristan, maka saya akan modifikasi config auto-generated dari file `openvpn-install.sh` yang sudah di buat pada point 4 di atas

1. pertama edit config file `xxx.ovpn`, ganti 4 baris teratas menjadi seperti di bawah
```sh
client
proto tcp
# explicit-exit-notify
remote <ip openvpn server> 443
```

2. pindah `xxx.ovpn` ke client dan jalankan filenya
```sh
> sudo openvpn xxx.ovpn
2021-01-05 12:54:48 OPTIONS IMPORT: adjusting link_mtu to 1626
2021-01-05 12:54:48 OPTIONS IMPORT: data channel crypto options modified
2021-01-05 12:54:48 Outgoing Data Channel: Cipher 'AES-128-GCM' initialized with 128 bit key
2021-01-05 12:54:48 Incoming Data Channel: Cipher 'AES-128-GCM' initialized with 128 bit key
2021-01-05 12:54:48 TUN/TAP device tun0 opened
2021-01-05 12:54:48 net_iface_mtu_set: mtu 1500 for tun0
2021-01-05 12:54:48 net_iface_up: set tun0 up
2021-01-05 12:54:48 net_addr_v4_add: 172.16.111.2/24 dev tun0
2021-01-05 12:54:48 Initialization Sequence Completed
```

> OpenVPN sudah berhasil connect ke OpenVPN server dengan protokol TCP port 443

3. jika sudah berhasil, bisa kita buat service agar jalan secara otomatis
```sh
> cp xxx.ovpn /etc/openvpn/client/xxx.conf
> sudo systemctl start openvpn-client@xxx
> sudo systemctl enable openvpn-client@xxx
```

&nbsp;

&nbsp;

&nbsp;

## 4) Terakhir
---
Gimane, simpel kan? cuman modifikasi beberapa baris aja dari config default bang angristan aja uda bisa setup OpenVPN server Multiple Port dan Protokol.

Ga cuman 2 OpenVPN Server doang, tapi bisa ratusan  :joy:, yang penting port nya cukup dan ga susah maintenance nantinya.

Untuk port TCP/443 merupakan port teraman, karna port ini di gunakan oleh sejuta umat untuk akses website **HTTPS** ... di jamin Vendor tidak akan blocking  :yum: bakal di amuk se RW entar ... kalaupun di block secara analisis packet data, kaga bisa, kan di encrypt  :joy: