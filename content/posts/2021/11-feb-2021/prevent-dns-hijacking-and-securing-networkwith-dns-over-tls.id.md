---
title: "How To Prevent DNS Hijacking and Securing Local Network"
date: 2021-02-08T15:28:38+07:00
weight: 1
tags: ["Analysis", "Docker", "Linux", "Security"]
author: "berrabe"
showToc: true
TocOpen: True
hidemeta: false
disableShare: false
cover:
    image: "https://www.imperva.com/learn/wp-content/uploads/sites/13/2019/01/DNS-spoofing-768x435.jpg.webp"
    alt: "Loh Imange Nya Ilang"
    relative: false
comments: false
---

&nbsp;

&nbsp;

## 1) Intro
---
okeh ... selamat pagi, siang, ataupun malam ... balek lagi ama saya, **berrabe** ... orang yang selalu menyelesaikan masalah tanpa solusi  :sunglasses:

kali ini saya pengen sharing lagi, sharing tentang keresahan saya menggunakan internet :sob:

jadi gini, saya punya beberapa masalah, seperti

- pengen buka web yang di blokir pemerintah, khusunya reddit, netflix, dkk tanpa vpn
- pengen mengamankan privasi saya, karna banyak banget tracking system di tanamkan pada app, web dll
- pengen memblock iklan di semua gadget saya, baik iklan di web, app dll tanpa install aplikasi di setiap gadget

:warning: **Note** : masalah utama ada pada point 1 ... point 2 dan 3 merupakan bonus

&nbsp;

&nbsp;

&nbsp;

## 2) The Konspirasi
---
okeh ... untuk yang belum tahu, cara kerja pemerintah dalam memblokir website / domain adalah menggunakan teknik **DNS Hijacking / DNS Redirection**  :scream: :scream:

untuk lebih jelasnya, mari lihat cuplikan dari [wikipedia ini](https://en.wikipedia.org/wiki/DNS_hijacking)

> A number of consumer ISPs such as AT&T ..... and Telkom Indonesia use DNS hijacking for their own purposes,
such as displaying advertisements or collecting statistics, they were ordered to block access to The Pirate Bay and display a warning page instead.

&nbsp;

inti dari cuplikan wikipedia di atas adalah ... bahwa di dunia ini, banyak ISP yang menerapkan teknik **DNS Hijacking / DNS Redirection**, contohnya ada AT&T di Amerika, dan Telkom Indonesia.

gunanya buat apa? untuk menampilkan iklan, mengkoleksi statistik penggunanya, dan memblock akses ke beberapa website yang di anggap buruk dengan memberikan halaman peringatan, kalau di indonesia ada [internet positif](http://internetpositif.uzone.id) dari telkom  :pensive:

:warning: **Note** : DNS Hijack juga sering di gunakan untuk kejahatan oleh para hacker. Biasanya target akan di redirect ke halaman phishing untuk di curi data sensitif nya  :scream:

&nbsp;

&nbsp;

&nbsp;

## 3) Bagaimana Cara Kerjanya
---
setelah tau teknik apa yang di gunakan ISP beserta alasannya, saat nya ngulik lebih dalam tentang bagaimana cara kerjanya  :blush:

&nbsp;
### 3.a) Normal DNS
cara kerja DNS sangat sederhana, di mana DNS berguna untuk men-translate domain ke ip

what ?? kenapa harus kek gitu, ribet amat dah ....  :triumph:  :weary:

gampangnya, komputer harus bekerja dengan alamat IP, sedangkan manusia susah untuk menghapal alamat IP, makanya manusia pakai domain name kaya google.com, facebook.com, dll

```sh
> dig google.com

;; ANSWER SECTION:
google.com.		68	IN	A	74.125.200.139
google.com.		68	IN	A	74.125.200.138
google.com.		68	IN	A	74.125.200.101
google.com.		68	IN	A	74.125.200.113
google.com.		68	IN	A	74.125.200.102
google.com.		68	IN	A	74.125.200.100
```

nah tuh, bayangin manusia harus ngapalin ip 74.125.200.139 - dst untuk ngakses google ... ribet kan?  :cry: :cry:

belum lagi hafalin IP untuk akses web / app yang lain kaya facebook, instagram, twitter dkk  :scream:

&nbsp;
### 3.b) Hijacked / Redirected DNS
oke, sekarang cara kerja DNS hijack, intinya sama saja seperti normal DNS .... tapiii, ada tapinya nih

tapi, sebelum DNS Trafik sampe ke DNS Server tujuan `( contoh : google DNS 8.8.8.8 )`, Trafik DNS tadi di cegat oleh ISP dan di belokin paksa / force-redirect ke DNS Server ISP

```sh
                                                                 +--------------+
+----------+          +----------+          +---------+          |              |
|          |          |          |          |         |          |  GOOGLE DNS  |
|  CLIENT  |  +--->   |  ROUTER  |   +--->  |   ISP   |   +----> |  8.8.8.8     |
|          |          |          |          |         |          |  8.8.4.4     |
+----------+          +----------+          +---------+          |              |
                                                                 +--------------+
                                               +  ^
                                               |  |
                                               v  +

                                            +---------+
                                            |         |
                                            | DNS ISP |
                                            |         |
                                            +---------+
```

masih belum paham? 

coba lihat hasilnya jika saya menggunakan DNS dari google `8.8.8.8 / 8.8.4.4`

```sh
> dig @8.8.8.8 reddit.com

;; ANSWER SECTION:
reddit.com.		897	IN	CNAME	internetpositif.uzone.id.
internetpositif.uzone.id. 27	IN	A	36.86.63.185
```

nah loh ... terlihat dengan jelas, bahwa domain reddit.com **ter-redirect** ke internet positif ... padahal saya menggunakan DNS Google, kok aneh?  :sob: :sob:

&nbsp;

lalu, bagaimana seharusnya domain yang benar / yang tidak **ter-redirect** itu?  :no_mouth: :no_mouth:

```sh
> dig @172.16.100.1 reddit.com

;; ANSWER SECTION:
reddit.com.		177	IN	A	151.101.65.140
reddit.com.		177	IN	A	151.101.1.140
reddit.com.		177	IN	A	151.101.193.140
reddit.com.		177	IN	A	151.101.129.140
```

di atas, adalah hasil dari IP-IP domain reddit.com yang valid ... *ini menggunakan DNS Server di local saya, jadi tidak ter-hijack oleh ISP*

jika saya ambil contoh 1 IP `( 151.101.65.140 )`, lalu saya teliti, akan seperti ini

```sh
> curl ipinfo.io/151.101.65.140

{
  "ip": "151.101.65.140",
  "anycast": true,
  "city": "San Francisco",
  "region": "California",
  "country": "US",
  "loc": "37.7621,-122.3971",
  "org": "AS54113 Fastly",
  "postal": "94107",
  "timezone": "America/Los_Angeles",
  "readme": "https://ipinfo.io/missingauth"
}
```

oke, terlihat kan

bahwa reddit.com dengan salah satu IP-nya yaitu `151.101.65.140` bukan `36.86.63.185`

setelah saya cek bahwa IP berasal dari San Francisco, California, US dengan organisasi bernama Fastly (CDN) ..... bukan **internet positif**  :joy: :joy: :joy:


&nbsp;
### 3.c) Test With Web / Script
cara alternatif lain untuk mengecek apakah ISP menerapkan DNS Hijack, bisa dengan menggunakan teknik [DNS leak test](https://bash.ws/dnsleak)

contoh nya ada di :
- Website
  - https://www.dnsleaktest.com/
  - https://bash.ws/dnsleak
- Terminal
  - https://github.com/macvk/dnsleaktest
- WordPress Integration
  - https://github.com/macvk/vpn-leaks-test

&nbsp;

&nbsp;

&nbsp;

## 4) DNS Hijacking vs DNS Spoofing
---
oke, sedikit tambahan aja ... di dunia IT Security, teknik-teknik pada peretasan menggunakan DNS itu banyak sekali ... yang paling terkenal ada **DNS Hijacking / DNS Redirection** dan **DNS Spoofing / Cache Poisoning**

intinya, kedua teknik di atas sama-sama mengubah IP dari domain name yang betul / valid menjadi IP yang salah

seperti reddit.com di translate ke internet positif oleh DNS Telkom

tapi, biasanya **DNS Spoofing / Cache Poisoning** terjadi pada cache DNS di local .... sedangkan **DNS Hijacking / DNS Redirection** lebih kompleks, bisa terjadi karna trafik DNS di force redirect seperti yang di gunakan ISP, atau bisa terjadi karna terkena serangan malware

&nbsp;

&nbsp;

&nbsp;

## 5) Mencegah DNS Hijack
---
wokai ... sebenarnya cara meng-atasi **DNS Hijacking / DNS Redirection** itu gampang-gampang susah  :joy:

ini saya kasih tau beberapa caranya

- Hosts File
  - cara ini merupakan cara yang paling sederhana tapi ampuh, tinggal modifikasi file hosts, kalau di linux ada di `/etc/hosts`

  ```sh
  > cat /etc/hosts

  127.0.0.1        localhost
  151.101.65.140   reddit.com
  ```

- Pakai VPN
  - ini adalah cara sejuta umat untuk buka web yang di block pemerintah, tapi kekurangannya satu,
  yaitu semua trafik akan di bungkus VPN, jadi akan ada overhead paket yang mengakibatkan latency bisa naik
  dan bandwidth cepet abis
  - keunggulan dari cara ini, trafik akan sangat aman, tapi tergantung protokol VPN yang dipakai ya  :sweat_smile:

- The Modern Way
  - oke, cara terakhir ini menurut saya cara yang paling modern, kenapa? .... karna teknologi nya baru
  di implementasikan baru-baru ini  :sunglasses:
  - Teknologi itu bernama DoT (DNS Over TLS) dan DoH (DNS Over HTTPS)

&nbsp;

&nbsp;

&nbsp;

## 6) DNS biasa vs DoT vs DoH vs DNSSEC
---
pada DNS biasa, trafik akan di lewatkan port 53 dengan protokol UDP ... makanya trafik DNS tadi bisa dengan mudah di manipulasi / tampering oleh attacker / ISP dalam kasus ini

lalu perbedaan dengan DNS biasa, DoT, DoH apa?

Metric | DNS Biasa | DoT | DoH |
-----------| -----------| ----------- | ----------- |
Port |  53 |  853 | 443 |
Protocol |  UDP | TCP | TCP |
Encyption | Plain UDP | UDP + TLS / SSL | HTTP / HTTP2 + TLS / SSL |

lalu mana yang lebih baik? DoT atau DoH?

jujur saja, semua ini masih dalam perdebatan karna masing-masing ada plus-minus sendiri

- DoT
  - ini lebih baik jika di lihat dari sisi security endpoint, ini memudahkan seorang security administrator untuk memonitor, indentifikasi trafik yang lewat jalur 853/tcp
- DoH
  - dari sudut pandang privasi, DoH lebih baik karna DoH menyatu dengan protokol yang di pakai semua orang untuk mengakses website, yaitu HTTPS ... yang membuat seorang system administrator kesusahan dalam menganalisa / memonitor paket

lalu, apa itu DNSSEC?

gampangnya, DNSSEC di rancang untuk mencegah serangan **DNS Spoofing / Cache Poisoning** ...  di mana DNSSEC ini merupakan kumpulan dari ekstensi security tambahan yang di gunakan untuk mengidentifikasi `DNS root server` dan `authoritative nameservers` dalam komunikasi dengan `DNS Resolver`

untuk lebih jelasnya, silahkan baca [di sini](https://www.cloudflare.com/learning/dns/dns-over-tls/)

&nbsp;

&nbsp;

&nbsp;

## 7) Menggunakan / Membuat DoT dan DoH
---
oke ... ini bab terakhir, yaitu gimana caranya gunain / membuat sendiri (self-hosted) DoT maupun DoH


&nbsp;
### 7.a) Menggunakan DoT / DoH
ada beberapa cara untuk menggunakan layanan DoT / DoH yang sudah ada, tanpa harus membuat sendiri

- Android
  - mulai dari versi 9, ada pengaturan Private DNS di Connection Setting ... ini memungkinkan android menggunakan DoT
  - untuk versi 8 kebawah, bisa menggunakan aplikasi semacam 1.1.1.1 punya cloudflare
  - untuk DoT Server, saya biasanya menggunakan adguard `tls://dns.adguard.com` atau `tls://dns-family.adguard.com`

  ![Private DNS Cloudflare](https://blog.cloudflare.com/content/images/2018/08/Screenshot_20180807-102253-1.png)
- Linux / Windows / Mac
  - untuk Linux / Windows / Mac, bisa menggunakan aplikasi DoH punya cloudflare, bisa di lihat [di sini](https://developers.cloudflare.com/1.1.1.1/dns-over-https/cloudflared-proxy)
  untuk dokumentasi nya dan untuk binary nya bisa di download [di sini](https://developers.cloudflare.com/argo-tunnel/downloads)
  - atau bisa juga menggunakan aplikasi [dnscrypt-proxy](https://dnscrypt.info/)


:warning: **Note** : provider DoT / DoH tidak selalu cloudflare, ada yang lain seperti adguard, Quad9, Google DNS dkk ... tapi yang saya tahu, cloudflare lah yang paling cepat dan reliable, tapi kalau mau ada block iklan + basic security, bisa pakai adguard. list lengkap bisa di lihat [di sini](https://kb.adguard.com/en/general/dns-providers)

&nbsp;

&nbsp;
### 7.b) Membuat Server DoT / DoH
cara membuat server DoT / DoH sangat sederhana dan mudah, yang paling terkenal dan yang saya tahu ada 2 cara, yaitu pakai PiHole atau Adguardhome

saya lebih memilih Adguardhome karna sangat simple, mudah dan fitur bawaannya banyak, sedangkan PiHole sudah ada fiturnya, tapi harus setup sendiri dengan tools yang berbeda (tidak include jadi satu)

:warning: **Note** : oiya, saya akan menginstall pakai docker biar gampang dan untuk tempat install nya, bisa di VPS, Cloud atau server sendiri ... kalau saya pribadi, pakai server DIY dari bekas STB Indihome B860H V1 yang saya install linux / bisa juga RaspberryPi

&nbsp;
#### 7.b.1) Install Docker
pastikan docker sudah terinstall, jika belum ikutin langkah di bawah ini ...
```sh
> curl -fsSL https://get.docker.com -o get-docker.sh
> sh get-docker.sh
> systemctl start docker
```

&nbsp;
#### 7.b.2) Setup Adguardhome
setelah docker terinstall, langkah selanjut tinggal setup adguardhome nya

:warning: **Note** : pastikan port 53/udp, 53/tcp, 80/tcp, 443/tcp, 853/tcp dan 3000/tcp tidak di gunakan oleh program lain

```sh
> mkdir DNS-Server && cd DNS-Server
> docker run --name DNS-Server \
    -v $(pwd):/opt/adguardhome/work \
    -v $(pwd):/opt/adguardhome/conf \
    -p 53:53/tcp -p 53:53/udp \
    -p 80:80/tcp -p 443:443/tcp \
    -p 3000:3000/tcp \
    -p 853:853/tcp \
    -d adguard/adguardhome
```

jika sudah menjalankan command diatas dan tidak ada error ... selanjutnya setting adguard nya dengan masuk ke `ip-server:3000`

nanti akan muncul tampilan seperti ini

![setup adguardhome](https://cdn.adguard.com/public/Adguard/Blog/AGHome/wizard.png)


&nbsp;
#### 7.b.3) Config Adguardhome
setelah setup selesai, adguardhome sudah bisa di gunakan sebagai DNS server pada port default 53, baik pakai TCP / UDP

tapi sebelum itu, saya sarankan aktifkan filter DNS Blocklist agar domain untuk penyedia iklan seperti google ads, tracking system seperti google analytics dan domain-domain jahat lain seperti penyedia malware, phishing dkk di block

masuk ke `Filters -> DNS Blocklist`

![adguardhome blocklist](https://res.cloudinary.com/canonical/image/fetch/f_auto,q_auto,fl_sanitize,w_819,h_503/https://dashboard.snapcraft.io/site_media/appmedia/2020/04/agh3_2Zqhv3A.png)


&nbsp;
#### 7.b.4) Beautiful Adguardhome Dashboard
hal yang paling saya suka dari adguardhome adalah ... dashboard nya

yap, bagi saya, dashboard nya itu clean, simple tapi sangat informatif

![adguardhome dashboard](https://cdn.adguard.com/public/Adguard/Blog/AGHome/dashboard.jpg)


&nbsp;
#### 7.b.5) Setup SSL
adguardhome secara otomatis listen pada port default DNS port 53, baik TCP maupun UDP ... ini seharusnya tidak masalah jika DNS Server berada di lokal

tapi jika ingin di akses global, harus pakai DoT / DoH, karna jika pakai plain DNS port 53, akan kena DNS Hijack ISP  :sleepy: :sleepy:

untuk membuat nya, step nya agak rumit ..

- pertama, pastikan sudah punya domain FQDN, contoh nya seperti dns.skidipapap.com
- kedua, buat SSL Cert untuk domain tadi, bisa pakai Lets Encypt kalau mau yang gratis, bisa di lihat [di sini](https://letsencrypt.org/getting-started/)
atau [di sini jika ingin pakai docker](https://certbot.eff.org/docs/install.html#running-with-docker)
- ketiga, copy isi dari file cerficate dan private key ke `Settings -> Encryption Settings`

![encryption setting](https://cdn.adguard.com/public/Adguard/Blog/AGHome/in-depth-review/home-encryption.png)


&nbsp;
#### 7.b.6) Saran Config Adguardhome
saya akan beri beberapa saran agar adguardhome dapat optimal, cepat dan reliable

- Tab **Settings**
  - non-antifkan *adguard browsing security web service* maupun *adguard parental control web service*  pada tab *Settings -> General Settings* karna ini akan membebani kerja dari server dan memperlambat resolving DNS, untuk lebih jelasnya bisa di lihat [di sini](https://kb.adguard.com/en/general/how-malware-protection-works)
  - ubah upstream DNS Server ke DoT Cloudflare `tls://1.1.1.1` pada tab *Settings -> DNS Settings*, ini akan mempercepat resolving DNS karna Adguardhome tidak perlu menggunakan **Bootstrap DNS Servers**
  - ubah *rate limit* menjadi 0 pada tab *Settings -> DNS Settings*, untuk mencegah adanya resolv DNS queue
  - naikkan nilai *DNS Cache Configuration*, agar buffer untuk DNS Caching lebih besar, membuat Resolv DNS lebih cepat karna tinggal load dari buffer memory tanpa harus resolv ke upstream server

- Tab **Filters**
  - jangan beri terlalu banyak rules pada *DNS Blocklist*, karna proses filtering memakan waktu yang cukup lama dan cpu intensive
  - buat rules yang simple (menggunakan regex), adguardhome mendukung banyak filter rule scheme, bisa di lihat [di sini](https://github.com/AdguardTeam/AdGuardHome/wiki/Hosts-Blocklists)

&nbsp;

&nbsp;

&nbsp;

## 8) Video By berrabe
---
**Bypass DNS Hijack ISP with AdBlock on Your Local Network**
{{< youtube rHvPWvnt_zI >}}