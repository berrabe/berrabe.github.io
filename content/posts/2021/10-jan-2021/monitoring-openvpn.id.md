---
title: "Monitoring OpenVPN"
date: 2021-01-24T05:34:24+07:00
weight: 1
tags: ["Docker", "Linux", "Tools", "OpenVPN"]
author: "berrabe"
showToc: true
TocOpen: True
hidemeta: false
disableShare: false
cover:
    image: "https://raw.githubusercontent.com/berrabe/monitoring-openvpn/master/docs/mon_poc.png"
    alt: "Loh Imange Nya Ilang"
    relative: false
comments: false
---

&nbsp;

&nbsp;

## 1) TLDR
---
wokai ... ketemu lagi sama saya, **berrabe**. Kali ini saya pengen sharing caranyoo monitoring openvpn yang simple  :heart_eyes:

kenapa harus OpenVPN? dan kenapa harus di Monitoring?  :confused:

- pertama, **OpenVPN** itu protokol private tunneling yang paling secure dan paling banyak di implementasikan saat ini
- kedua, **monitoring** itu sangat crucial sekali ... klo sebuah sistem / app tidak di monitoring, itu sama seperti orang yang menanam padi tapi tidak pernah di rawat / dibiarin aja gitu. Berharap padinya akan tetap sehat dan bisa di panen

oiya, untuk OpenVPN Server [saya setup otomatis menggunakan script nya pak angristan](https://github.com/angristan/openvpn-install), soalnya kalo harus manual setting dari awal seperti setup certs, key dkk per client, pusing pala saya  :cry:  :tired_face: ... 

&nbsp;

&nbsp;

&nbsp;

## 2) Simple Monitoring #1 (Bash Script)
---
cara monitoring OpenVPN yang pertama ini menggunakan [script bash yang sudah saya buat](https://github.com/berrabe/monitoring-openvpn). tinggal clone git nya saja di OpenVPN Server, dan langsung bisa di jalankan

```sh
> git clone https://github.com/berrabe/monitoring-openvpn.git
> cd monitoring-openvpn
> chmod +x openvpn_mon.brb
> ./openvpn_mon.brb mon
```

nanti output nya akan seperti ini

![bash openvpn](https://raw.githubusercontent.com/berrabe/monitoring-openvpn/master/docs/mon_poc.png)

untuk info lebih jelasnya, [silahkan baca di readme github nya](https://github.com/berrabe/monitoring-openvpn)

:warning: **Note** : pastikan OpenVPN Server sudah di setup status log nya ke file `/var/log/openvpn/status.log` menggunakan directive `status /var/log/openvpn/status.log`

&nbsp;

&nbsp;

&nbsp;

## 3) Simple Monitoring #2 (Docker Web Based)
---
monitoring kedua ini tampilan nya akan berupa web based, [bukan CLI seperti cara pertama](#2-simple-monitoring-1-bash-script) ... tapi syarat nya harus pakai docker, [baca tutor nya di sini](https://docs.docker.com/get-docker/) jika belum install

nah, setelah uda punya docker, OpenVPN Server harus di setup **management interface** supaya aplikasi monitoring nya bisa work ... cara kerja nya beda dari [cara pertama yang pakai shell script (parsing status log)](#2-simple-monitoring-1-bash-script)

&nbsp;

&nbsp;
### 3.a) Management Interface


cari file config OpenVPN Server nya, kalo saya ada di  `/etc/openvpn/server.conf`

terus, tambahkan directive config `management <ip listen> <port listen>` untuk mengaktifkan **management interface**, alhasil akan seperti ini

```sh
port 1194
management 172.17.0.1 5555
proto udp
dev tun
...
```

IP `172.17.0.1` merupakan ip default dari **network bridge docker0**

karna saya install OpenVPN Server nya di host, dan Monitoring nya pakai Docker di atas **network bridge default**

```sh
           HOST SERVER
+-------------------------------+
|                               |
|                               |
|    OpenVPN Server           <-----------+
|                               |         |
|                               |         |
|                               |         |
|                               |         |  Docker0 Bridge
|       DOCKER                  |         |  (172.17.0.1/16)
|  +---------------+            |         |
|  |               |            |         |
|  |   Docker      |            |         |
|  |   OpenVPN     +----------------------+
|  |   Monitoring  |            |
|  |               |            |
|  +---------------+            |
+-------------------------------+
```

:warning: **Note** : jangan set listen IP ke `0.0.0.0`  karna orang lain bisa akses **Management Interface OpenVPN tanpa credentials**


&nbsp;

&nbsp;
### 3.b) Deploy The Bad Guy
saat nya deploy aplikasi monitoring nya, pakai compose file saja biar enak, copy script ke `docker-compose.yml`

```yaml
version: "2.2"

services:
    monitoring-OpenVPN:
        image: ruimarinho/openvpn-monitor
        container_name: monitoring-OpenVPN
        ports:
      		- "80:80"
        environment:
            - OPENVPNMONITOR_DEFAULT_DATETIMEFORMAT="%%d/%%m/%%Y"
            - OPENVPNMONITOR_DEFAULT_MAPS=True
            - OPENVPNMONITOR_DEFAULT_SITE=berrabe
            - OPENVPNMONITOR_SITES_0_ALIAS=UDP
            - OPENVPNMONITOR_SITES_0_HOST=172.17.0.1
            - OPENVPNMONITOR_SITES_0_NAME=UDP Protocol
            - OPENVPNMONITOR_SITES_0_PORT=5555
            - OPENVPNMONITOR_SITES_0_SHOWDISCONNECT=True
        restart: unless-stopped
        cpus: 1
        mem_limit: 150M
        memswap_limit: 150M
        network_mode: bridge
```

fire up aplikasi pakai command `docker-compose up -d`

jangan lupa di lihat status container nya sudah up atau belum

```sh
> docker ps -a
CONTAINER ID   IMAGE                        COMMAND                  CREATED       STATUS        PORTS                 NAMES
fd4e484f326a   ruimarinho/openvpn-monitor   "/entrypoint.sh guni…"   2 hours ago   Up 2 hours    0.0.0.0:80->80/tcp    monitoring-OpenVPN
```

&nbsp;

&nbsp;
### 3.c) Access
setelah semua setup selesai, sekarang aplikasi sudah bisa di lihat di `<ip server>:80`, tampilannya akan seperti ini

![web based mon](https://raw.githubusercontent.com/furlongm/openvpn-monitor/gh-pages/screenshots/openvpn-monitor.png)


&nbsp;

&nbsp;
### 3.d) Secure The Access
monitoring Docker Web Based tadi bisa di akses dari mana saja (public) tanpa harus memasukkan credential login dsb. Untuk Mengamankan nya saya menaruh Web App tadi di bawah **reverse proxy**

untuk **reverse proxy** nya berbasis docker juga, artikel bisa di lihat di  [docker reverse proxy #1]({{< ref "docker-reverse-proxy-1.id.md" >}}) dan  [docker reverse proxy #2]({{< ref "docker-reverse-proxy-2.id.md" >}})

untuk memasukkan aplikasi tadi ke bawah **reverse proxy**, modif `docker-compose.yml` menjadi

```yaml
version: "2.2"

services:
    monitoring-OpenVPN:
        image: ruimarinho/openvpn-monitor
        container_name: monitoring-OpenVPN
        environment:
            - OPENVPNMONITOR_DEFAULT_DATETIMEFORMAT="%%d/%%m/%%Y"
            - OPENVPNMONITOR_DEFAULT_MAPS=True
            - OPENVPNMONITOR_DEFAULT_SITE=berrabe
            - OPENVPNMONITOR_SITES_0_ALIAS=UDP
            - OPENVPNMONITOR_SITES_0_HOST=172.17.0.1
            - OPENVPNMONITOR_SITES_0_NAME=UDP Protocol
            - OPENVPNMONITOR_SITES_0_PORT=5555
            - OPENVPNMONITOR_SITES_0_SHOWDISCONNECT=True
            - HTTPS_METHOD=redirect
            - VIRTUAL_HOST=vpn-mon.example.com
            - LETSENCRYPT_HOST=vpn-mon.example.com
            - NETWORK_ACCESS=internal
        restart: unless-stopped
        cpus: 1
        mem_limit: 150M
        memswap_limit: 150M
        network_mode: bridge
```

ganti domain `vpn-mon.example.com` ke domain yang telah di sediakan

bedanya dari [compose yang tidak secure](#3b-deploy-the-bad-guy) apa?

- **ports di hilangkan**, container tidak butuh expose port standalone nya, karna traffic akan di lewatkan **reverse proxy** dulu
- **reverse proxy Environment**, ada beberapa tambahan pada bagian `environment:`
  - `HTTPS_METHOD=redirect`, akan membuat aplikasi force redirect (302) ke protokol HTTPS
  - `VIRTUAL_HOST=vpn-mon.example.com`, aplikasi hanya bisa di akses pakai domain ini
  - `LETSENCRYPT_HOST=vpn-mon.example.com`, domain nya sudah auto-generated SSL Certs letsencrypt, tidak perlu manual validasi lagi  :relieved:
  - `NETWORK_ACCESS=internal`, domain hanya bisa di akses pakai ip private (OpenVPN), kalau di akses dari ip public akan Error Forbidden (403)
