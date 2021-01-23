---
title: "Docker Reverse Proxy #1"
date: 2021-01-23T11:36:29+07:00
weight: 1
tags: ["Docker", "Linux", "Automation"]
author: "berrabe"
showToc: true
TocOpen: True
hidemeta: false
disableShare: false
cover:
    image: "https://hackernoon.com/images/ass3ya2.jpg"
    alt: "Loh Imange Nya Ilang"
    relative: false
comments: false
---

&nbsp;

&nbsp;

## 1) TLDR
---
tet tet tetttt ... balek lagi sama saya, **berrabe** ... dimana saya suka memecahkan masalah tanpa masalah ... YEAY  :sunglasses:

kali ini, saya pengen curhat dikit tentang jiwa kere hore saya ....  :sweat_smile:

Jadi saya punya beberapa **WEB-APP** yang mau di jalanin, katakanlah ada Wordpress, Grafana dll ... tapi saya ga mau jalanin **WEB-APP** tadi di server-server terpisah kaya gini nih

```sh
                                                       +-------------------------+
                                                       |                         |
                                          +----------> |  SERVER 1 (WordPress)   |
                                          |            |                         |
                      +--------------+    |            +-------------------------+
+--------+            |              |    |
|  USER  |  +------>  |   INTERNET   |  +-+
+--------+            |              |    |            +-------------------------+
                      +--------------+    |            |                         |
                                          +----------> |    SERVER 2 (GRAFANA)   |
                                                       |                         |
                                                       +-------------------------+
```

saya mau nya yangggg ... paket hematttt kere hore ala anak kostan setiap akhir bulan, yang kalo makan nasi nya satu ember tapi lauknya cuman tempe goreng secuil  :weary:

pokoknya saya mau yang gimana caranya, semua **WEB-APP** yang buanyak tadi muat dalam satu server, dan **AKTIF SEMUA**

```sh
                                                  +-------------------------+
                                                  |        SERVER 1         |
                      +--------------+            |                         |
+--------+            |              |            |     = WordPress         |
|  USER  |  +------>  |   INTERNET   | +------->  |     = PHPMyAdmin        |
+--------+            |              |            |     = Grafana           |
                      +--------------+            |     = dst               |
                                                  |                         |
                                                  +-------------------------+
```
dan jugaaaa, per aplikasi ini di bedakan per domain, bukan per port .... **ingat ya, per DOMAIN**, jadi wordpress di `wp.berrabe.com` dan grafana di `dash.berrabe.com`

Uda kere, banyak mau nya pulakk  :expressionless: ... huufttt :sleepy:

tapi seperti biasaaa, tetap panik dan jangan tenang, karna **berrabe** selalu tidak punya solusi YEAY :sunglasses:


&nbsp;

&nbsp;

&nbsp;

## 2) Reverse Proxy ... Gebetan Yang Terlupakan
---
Setelah bertapa selama 1 jam di depan laptop, akhirnya saya ketemu dan kenalan ama ~~gebetan baru saya~~ reverse proxy :sweat_smile:  ... ni saya kasih fotonya

![reverse proxy](https://miro.medium.com/max/2800/1*rnzxfcy2N_ffJPnBundJQw.jpeg)

hhmm ... hhmm ... hhmm ...  :scream:

sepertinya saya mulai jatuh cinta ... eh, paham maksudnya  :sweat_smile: ... kalau di lihat dari gambar, dalam satu server bisa di letakkan 2 website yang berbeda dengan di kasih **reverse-proxy** di depannya

btw, gambarnya kalo di lihat malah kayak cinta segitiga  :joy:

&nbsp;

&nbsp;

&nbsp;

## 3) PDKT Sama Gebetan Baruuu
---
kayaknya bakal seru nih mainan ama ~~gebetan~~ reverse-proxy, hehe  :heart_eyes:

perlu di ingat, semua teknologi ini berbasis docker ... baik reverse-proxy maupun webapps / upstream server nya, jadi pastikan uda paham apa itu docker dan istilah2 di dalamnya

![docker reverse proxy](https://miro.medium.com/max/875/1*UDJaoKU3Foiz6KhfWI1eOA.png)

nah, gambaran jelasnya akan seperti ini


&nbsp;
### 3.a) Persiapan ~~Nembak Gebetan~~
sebelum menggunakan reverse proxy, pastikan sudah ada otak nya dulu .. yaitu **docker**, cara install nya sangattttt mudahhh

```sh
> curl -fsSL https://get.docker.com -o get-docker.sh
> sh get-docker.sh
> systemctl start docker
```

&nbsp;
### 3.b) Setup Env
okeh, supaya mudah dalam maintenance kedepannya ... maka lebih baik di buat folder hirarki berstruktur seperti di bawah

```sh
Reverse_Proxy/
   ├──  auto_net.brb  
   ├──  docker-compose.yml
```

tidak semua file akan di isi langsung, bertahap ... karna artikel ini akan puanjang  :scream:

&nbsp;

&nbsp;

&nbsp;

## 4) Compose File != Surat Cinta
---
setelah selesai membuat struktur folder, saat nya mengisi file-file yang telah di buat ... copy config di bawah ke file `docker-compose.yml`

```yaml
version: "2.2"

services:
    reverse-proxy:
        image: jwilder/nginx-proxy:alpine
        container_name: reverse-proxy
        volumes:
            - /var/run/docker.sock:/tmp/docker.sock:ro
            - ./reverse-proxy/cert:/etc/nginx/certs
            - ./reverse-proxy/vhost.d:/etc/nginx/vhost.d
            - ./reverse-proxy/html:/usr/share/nginx/html
            - ./reverse-proxy/dhparam:/etc/nginx/dhparam
            - ./conf/my_proxy.conf:/etc/nginx/conf.d/my_proxy.conf:ro
        ports:
            - "80:80"
            - "443:443"
        environment:
            - HTTPS_METHOD=noredirect
            - DEFAULT_HOST=example.com
        restart: unless-stopped
        cpus: 1
        mem_limit: 100M
        memswap_limit: 100M
        network_mode: bridge

    reverse-proxy-letencrypt:
        image: jrcs/letsencrypt-nginx-proxy-companion
        container_name: reverse-proxy-letencrypt
        volumes_from:
            - reverse-proxy
        volumes:
            - /var/run/docker.sock:/var/run/docker.sock:ro
            - ./reverse-proxy/acme:/etc/acme.sh
        environment:        
            - DEFAULT_EMAIL=xxx@example.com
        depends_on:
            - reverse-proxy
        restart: unless-stopped
        cpus: 1
        mem_limit: 100M
        memswap_limit: 100M
        network_mode: bridge
```

:warning: **Note** : Oiya, jangan lupa ganti beberapa value yang ada kata `example.com` ke domain yang sudah di sediakan

&nbsp;

&nbsp;

&nbsp;

## 5) Jalanin Aja Dulu
---
menjalankan docker-compose reverse-proxy ini sangat mudah ... tidak seperti menjalankan hubungan yang tidak pasti  :disappointed:

```sh
> docker-compose up -d

Starting reverse-proxy ... done
Starting reverse-proxy-letencrypt ... done
```

untuk melihat status container nya, apakah sudah up / restart gegara ada error
```sh
> docker ps -a

CONTAINER ID   IMAGE                                    COMMAND                  CREATED          STATUS         PORTS                                      NAMES
67627ced6c7e   jrcs/letsencrypt-nginx-proxy-companion   "/bin/bash /app/entr…"   3 seconds ago    Up 2 seconds                                              reverse-proxy-letencrypt
59f8fc51bfbc   jwilder/nginx-proxy:alpine               "/app/docker-entrypo…"   21 seconds ago   Up 2 seconds   0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   reverse-proxy
```

&nbsp;

&nbsp;

&nbsp;

## 6) Testing
---
untuk testing, pastikan sudah punya domain sendiri ... contoh nya di sini `berrabe.com` yang di pointing ke ip server `1.2.3.4` ... jangan lupa, server nya yang sudah ada reverse-proxy nya

```sh
1. berrabe.com ----> 1.2.3.4
2. a.berrabe.com ----> 1.2.3.4
3. b.berrabe.com ----> 1.2.3.4
```

setelah di pastikan domain sudah ada dan resolved dengan benar ke ip server, jalankan web server sederhana untuk mengecek apakah reverse-proxy sudah berjalan dengan semestinya

```sh
> docker run -d \              
        --name web \
        -e VIRTUAL_HOST=a.berrabe.com \
        -e LETSENCRYPT_HOST=a.berrabe.com \
        nginx:alpine                        
```

untuk melihat apakah ada error saat proses auto-generated conf dan ssl certs berjalan, bisa di lihat pakai command
```sh
> docker-compose logs -f
```

seharusnya docker-reverse-proxy yang tadi di jalankan menggunakan compose mendeteksi adanya container `nginx:alpine` diatas dan otomatis men-generated conf untuk nginx dan validasi ssl certs

dannn ... tadaaa, reverse-proxy bisa di gunakan

```sh
> curl -s -I http://a.berrabe.com | head -n 1
HTTP/1.1 200 OK

> curl -s -I https://a.berrabe.com | head -n 1
HTTP/2 302
```

:warning: **Note** : agar reverse-proxy berhasil, pastikan antara container reverse-proxy dan container upstream harus dalam satu network (default bridge) ... [lihat di sini](#8-peringatan)

&nbsp;

&nbsp;

&nbsp;

## 7) Untuk Yang Kepo
---
untuk yang kepo bagaimana cara kerja docker ini

- pertama, kita menjalankan docker reverse-proxy menggunakan docker-compose ... yang akan menciptakan 2 container, yaitu `reverse-proxy` dan `reverse-proxy-letencrypt`
  - `reverse-proxy` untuk auto-gen conf dan handle nginx reverse proxy lewat proxy pass
  - `reverse-proxy-letencrypt` untuk auto-gen dan validasi SSL Certs
- setelah 2 container RP tadi berjalan, saat nya menjalankan **WEB-APP** / Upstream Server (contoh: nginx:alpine)
- saat **WEB-APP** berjalan dan status nya UP, maka 2 container RP tadi tau kalau ada container baru, gimana bisa tahu? karna dia komunikasi dengan `docker.sock`
- jika container baru yang sudah UP tadi terdapat 2 environment variable seperti `VIRTUAL_HOST` dan `LETSENCRYPT_HOST`, maka 2 container RP akan men-generated conf sesuai dengan :
  - `VIRTUAL_HOST` untuk generated domain mapping ke nginx conf, terdapat di `/etc/nginx/conf.d/default.conf` pada container `reverse-proxy`
  - `LETSENCRYPT_HOST` untuk generated dan validasi SSL Certs domain menggunakan letsencrypt


&nbsp;

&nbsp;

&nbsp;

## 8) Peringatan
---
Agar reverse-proxy ini bekerja, pastikan antara container reverse-proxy dan container upstream (contoh: nginx-alpine) berada dalam satu network (default bridge) ... jika semisal upstream di jalan kan sebagai `docker-compose up` yang biasanya akan otomatis auto generated nama network sendiri, maka container reverse-proxy tadi harus di masukkan ke network tersebut ... [artikel nya ada di docker reverse proxy #2]({{< ref "docker-reverse-proxy-2.id.md" >}})

jika yang ndak mau repot-repot copy paste script di atas, tinggal clone [github saya di sini](https://github.com/berrabe/docker-reverse-proxy)