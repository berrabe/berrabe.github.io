---
title: "Openvpn Redirect Problem"
date: 2021-01-04T16:02:48+07:00
weight: 1
tags: ["OpenVPN", "VPN", "Linux", "Analysis"]
author: "berrabe"
showToc: true
TocOpen: True
hidemeta: false
disableShare: false
cover:
    image: "https://kb.kenceng-solusindo.net/wp-content/uploads/2018/10/openvpn.png"
    alt: "Loh Imange Nya Ilang"
    relative: false
comments: false
---

&nbsp;

&nbsp;

## 1) Masalah Besar pada OpenVPN
---
Saat saya menginstall openvpn dari script automasi bang [angristan](https://github.com/angristan/openvpn-install) (fork dari script [Nyr](https://github.com/Nyr/openvpn-install)), saya bisa setup server openvpn dengan cepat (hanya 5 menit) dan langsung work, tanpa harus setting cert, ip, routing dll secara manual

tapi ada satu masalah yang sangat simple tetepi membahayakan, dimana saya tidak bisa akses server client yang terkonek dengan jaringan OpenVPN!!!!, semua service tidak bisa di akses, mulai dari ping, ssh, sftp, webserver, dll ... oke oke, jangan tenang tetap panik  :stuck_out_tongue_closed_eyes: 

> masalah ini terjadi pada server yang melewatkan semua trafik nya melalui jaringan OpenVPN terlebih dahulu baru ke internet (redirect-gateway def1 bypass-dhcp)

&nbsp;

&nbsp;

&nbsp;

## 2) Masalah Itu di Selesaikan, Bukan Di Hindari
---
Oke pertama kita lihat dimana masalah nya, kita analisa atu-atu, pokoknya pantang tidur sebelum solved  :sleeping:, biar simple ... saya analisa trafik dari service webserver saja, untuk service lain (ssh, sftp, ftp dll) sama saja kok prinsip nya

&nbsp;
### 2.a) Alur Paket Web (Standar)
pertama kita harus tahu bagaimana sebuah trafik / lalu lintas paket itu bekerja ketika kita mengakses sebuah website dari sudut pandang client / end-user
```sh
+----------+             +------------+            +--------------+
|          |             |            |            |              |
|  Client  |    +---->   |  Internet  |   +---->   |  Web Server  |
|          |   <----+    |            |  <----+    |              |
+----------+             +------------+            +--------------+
```

wewww, grafik yang sangat simpel bukan? saya kasih log web server nih, biar agak detail dikit  :sweat_smile:

```sh
180.254.92.138 - - [04/Jan/2021:09:57:29 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:84.0) Gecko/20100101 Firefox/84.0" "-"
180.254.92.138 - - [04/Jan/2021:09:57:30 +0000] "GET /favicon.ico HTTP/1.1" 404 153 "http://36.80.241.251/" "Mozilla/5.0 (X11; Linux x86_64; rv:84.0) Gecko/20100101 Firefox/84.0" "-"
```

okehhh, perlu di inget dulu ye, ip public saya adalah **180.254.92.138** (sama seperti yang ada di log). Brati kalo saya simpulkan, cara kerja web server itu, dia akan mengembalikan request sesuai alamat ip client yang meminta.

Jadi semisal ip public saya 180.254.92.138 melakukan request ke google, ya google akan ngembaliin request ke ip public komputer saya, yaitu 180.254.92.138 ... bukannya ke ip 2.2.2.2 punyanya komputer pak RT  :joy:

&nbsp;
### 2.b) Alur Paket Web (OpenVPN)
okeh bosq, sekarang kita uda tau bagaimana paket web standar itu bekerja, trus sekarang kita lihat trafik yang menggunakan OpenVPN kenapa tidak bekerja  :cry:

mari kita lihat grafik nya
```sh
+----------+             +------------+            +--------------+
|          |             |            |            |              |
|  Client  |   +---->    |  Internet  |   +---->   |  Web Server  |
|          |  <----+     |            |            |              |
+----------+             +------+-----+            +--------------+
                                ^
                                |                         +
                       +--------+---------+               |
                       |                  |               |
                       |  OpenVPN Server  |  <------------+
                       |                  |
                       +------------------+
```

jika di lihat sekilas, seharusnya tidak ada masalah bukan? request dari client menuju web server, kemudian di kembalikan ke client, Normal and Simpel ... Hanya saja sebelum di kembalikan ke client, trafik di lewatkan ke jaringan OpenVPN terlebih dahulu

eits, tidak semudah itu ferguso ... kenyatannya trafik request dari client tidak sampai / balik lagi ke client alias **PUTUS**, trafik ini berlaku tidak hanya untuk service web, tapi semua service dari ssh, sftp, ftp dll

&nbsp;
### 2.c) Masalah nya tu disini ....
Intinya, saat client mengakses service (ping, ssh, web server, dll) di server yang semua traffik nya ter-redirect / dilewatkan ke OpenVPN, trafik tidak bisa kembali lagi ke client. Nah Loh  :scream:

Dari pada vusing dan ngawang knape bisa gitu, mari kita analisis secara mendalam menggunakan wireshark-cli / tshark. Untuk mempermudah pengecekan, kita akan menggunakan protokol **icmp / ping** ... klo pake http ribet cyinn (ngelesss) :grimacing:

tapi sebelum itu, kita harus tahu trafik ping yang normal itu seperti apa

```sh
> ping 1.1.1.1                                                                      
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=55 time=29.6 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=55 time=30.6 ms

# wireshark-cli
Capturing on 'wlp9s0'
    1 0.000000000    180.254.92.138 → 1.1.1.1      ICMP 98 Echo (ping) request  id=0x0016, seq=1/256, ttl=64
    2 0.028321186      1.1.1.1 → 180.254.92.138    ICMP 98 Echo (ping) reply    id=0x0016, seq=1/256, ttl=55 (request in 1)
    3 1.001599961    180.254.92.138 → 1.1.1.1      ICMP 98 Echo (ping) request  id=0x0016, seq=2/512, ttl=64
    4 1.045006664      1.1.1.1 → 180.254.92.138    ICMP 98 Echo (ping) reply    id=0x0016, seq=2/512, ttl=55 (request in 3)
```

okeh, traffik yang sangat sederhana, dimana saya sebagai client (180.254.92.138) melakukan ping (request) ke cloudflared (1.1.1.1) dan di balas dengan paket (reply)

sekarang mari kita lihat trafik saat kita ping server yang semua traffic nya ter-redirect melewati OpenVPN

```sh
> ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.

# wireshark-cli
Capturing on 'wlp9s0'
    1 0.000000000 180.254.92.138 → 114.xx.xx.21    ICMP 98 Echo (ping) request  id=0x663a, seq=1/256, ttl=54
    2 1.169998179 180.254.92.138 → 114.xx.xx.21    ICMP 98 Echo (ping) request  id=0x663a, seq=2/512, ttl=54
    3 2.036470879 180.254.92.138 → 114.xx.xx.21    ICMP 98 Echo (ping) request  id=0x663a, seq=3/768, ttl=54
```

loh loh, kok command ping nya stcuk dan paket nya request saja? tidak ada balasan / reply? seperti cinta bertepuk sebelah tangan bukan?  :stuck_out_tongue:

&nbsp;

&nbsp;

&nbsp;

## 3) Menyelesaikan Masalah Tanpa Solusi
---
setelah mencari wangsit dari sabang sampai merauke, akhirnya saya dapat akar masalahnya  :joy: .. sebelum itu mari kita lihat grafik trafik sekali lagi
```sh
+----------+             +------------+            +--------------+
|          |             |            |            |              |
|  Client  |   +---->    |  Internet  |   +---->   |  Web Server  |
|          |  <----+     |            |            |              |
+----------+             +------+-----+            +--------------+
                                ^
                                |                         +
                       +--------+---------+               |
                       |                  |               |
                       |  OpenVPN Server  |  <------------+
                       |                  |
                       +------------------+
```
jika kita amati di **Web Server**, kita bisa lihat bahwa jalur trafik masuk dan keluar itu berbeda bukan? katakanlah masuk lewat pintu depan dan keluar lewat jendela

perlu di ingat bahwa komputer hanya malayani trafik default, di mana trafik masuk dan keluar lewat pintu depan ... Tapi jangan putus asa dulu, mau tahu cara solved nya? jangan kasi kendor, baca terus sampai bawah  :blush:

&nbsp;
### 3.a) Skenario 1
sebelum mulai, kita lihat gambaran skema trafik flow dari skenario 1 ini
```sh
+----------+             +------------+            +--------------+
|          |             |            |            |              |
|  Client  |   +---->    |  Internet  |   +---->   |  Web Server  |
|          |  <----+     |            |  <----+    |              |
+----------+             +------+-----+            +--------------+
                                ^
                                |                         +
                       +--------+---------+               |
                       |                  |               |
                       |  OpenVPN Server  |  <------------+
                       |                  |
                       +------------------+

```
secara garis besar tidak ada perubahan, tetapiiii jika kita perhatikan di **Web Server**, ada panah bolak-balik yang menandakan, trafik client yang masuk pintu depan, akan *kita balikkan ke pintu depan lagi* dari pada kita redirect / alihkan melalui jendala (OpenVPN)

traffic di atas bisa di lakukan dengan routing ke arah ip public client yang mengakses server
```sh
> ip route add  < ip public client >  dev  < interface WAN server >

# Contoh
> ip route add 1.1.1.1 dev eth0
```

> kekurangan dari solusi ini adalah kita harus routing satu2 public ip client yang ingin terhubung / akses ke server

&nbsp;
### 3.b) Skenario 2
skenarion ke 2 ini akan tetap mempertahankan skema awal trafic, dimana trafik masuk dari pintu depan (eth0 WAN Internet) dan keluar melalui jendela (tun0 - OpenVPN) dengan menggunakan chain filter dan nat masqurade. Untuk skema trafik nya seperti di bawah ini
```sh
+----------+             +------------+            +--------------+
|          |             |            |            |              |
|  Client  |   +---->    |  Internet  |   +---->   |  Web Server  |
|          |  <----+     |            |            |              |
+----------+             +------+-----+            +--------------+
                                ^
                                |                         +
                       +--------+---------+               |
                       |                  |               |
                       |  OpenVPN Server  |  <------------+
                       |                  |
                       +------------------+
```
semua bisa di lakukan dengan cara 
```sh
# ipv4 forward
> sysctl -w net.ipv4.ip_forward=1

# accept forward chain
> iptables -A FORWARD --in-interface eth0 -j ACCEPT

# masking ip (NAT) after routing process
> iptables --table nat -A POSTROUTING --out-interface tun0 -j MASQUERADE
```

> kekurangan dari solusi ini adalah, tidak semua service akan berhasil. karena akan di anggap sebagai serangan MITM / spoofing. dimana kita meminta (request) ke server ip 1.1.1.1 tapi yang membalas (reply) adalah server ip 5.5.5.5

&nbsp;
### 3.c) Skenario 3
ada 2 skenario terakhir dan ini solusi yang recommended menurut saya, yaitu
1. Pertama, non-aktifkan redirecting all traffic
   - berguna agar tidak semua traffic di lewatkan OpenVPN, hanya network ip tertentu saja yang di lewatkan. Delete / Comment baris seperti di bawah
```sh
# redirect-gateway def1 bypass-dhcp
```
2. Kedua, jika ingin tetap melewatkan semua trafic melewati jaringan OpenVPN, gunakan fitur packet marking yang ada di iptables
   - rule ini berfungsi untuk menandai setiap packet yang masuk ke server. Yang mana nantinya packet response akan melewati jalan di mana packet tersebut masuk.

   sebagai contoh, jika kita masuk lewat pintu depan, maka saat kita keluar, server akan mengarahkan lewat pintu depan juga, tidak lewat jendela seperti sebelumnya

untuk point 2 explore sendiri yeee, ribet cyin soalnyeee ... nih [link nya](https://www.niftiestsoftware.com/2011/08/28/making-all-network-traffic-for-a-linux-user-use-a-specific-network-interface/)  :joy: