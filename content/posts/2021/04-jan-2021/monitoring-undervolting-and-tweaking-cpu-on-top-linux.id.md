---
title: "Monitoring, Undervolting and Tweaking CPU on Top Linux"
date: 2021-01-07T13:54:07+07:00
weight: 1
tags: ["Linux", "Analysis", "Tools"]
author: "berrabe"
showToc: true
TocOpen: True
hidemeta: false
disableShare: false
cover:
    image: "https://candid.technology/wp-content/uploads/2020/01/UNDERVOLTING-1024x576.jpg"
    alt: "Loh Imange Nya Ilang"
    relative: false
comments: false
---

&nbsp;

&nbsp;

## 1) TLDR
---
tet tet tetttt ... balek lagi sama saya, **berrabe** ... dimana saya suka memecahkan masalah menjadi masalah lagiii, YEAY  :sunglasses:

kali ini saya pengen sharing lagi bagaimana cara memasak indomie, eh ... maap, efek laperrr  :sleepy:

maksud saya cara Monitoring, Undervolting dan Tweaking CPU di Linux. Untuk info saja, topik utamanya adalah **UNDERVOLTING** ... dimana dari undervolting itu nyambung ke tweaking lalu monitoring

sebelumnya uda tau belom ama `undervolt`? adeknya `overclock` itu lhooo, anak nya pak RT  :relieved: ... klo blom, saya kasih tauu ... 

&nbsp;

&nbsp;

&nbsp;

## 2) Adek Kakak?
---
Penjelasan OverClock dan UnderVolt secara garis besar, biar ga salah paham ... beda kaya jelasin ke doi, selaluuuuu aja salah, hiks hiks  :weary:

&nbsp;
### 2.a) OverClocking
jadi gini, di dunia per-hardware an ini, ada istilah yang sering kali di ucapkan, baik oleh para gamers, Antusias PC, Editor, dll ... yaitu **OVERCLOCKING**, dimana overclocking ini akan menaikkan clock dari CPU, misal dari 2 ke 5 GHz dengan syarat dan ketentuan berlaku  :sunglasses:

jika clock CPU naik, kecepatan komputasi dari sebuah CPU akan menjadi lebih cepat, jadi semisal, ada editor rendering video 4k 30 detik di clock 2 GHz membutuhkan waktu 15 menit lebih, maka kalau di 5 GHz hanya 5 menit atau kurang

sangat lumayan bukan? ... brati saya sekarang bisa beli CPU harga 3-4 jutaan yang akan menyaingi CPU harga 6 jutaan ke atas, YEAYY  :grinning:

&nbsp;
### 2.b) UnderVolting
tadi saya sudah menjelaskan OverClocking secara garis besar, sekarang apa itu UndverVolting?

gampangnya, undervolting itu kebalikan dari overclocking ... jadi, klo overclocking itu Voltase CPU nya di naikkan agar Clock CPU nya bisa naik

nah klo undervolt, Voltase CPU nya di turunkan, supaya apa? ... bisa banyak, coba lanjut baca dulu, nanti saya suapin, eh ... jelasinnn  :yum:

&nbsp;
### 2.c) Kelebihan dan Kekurangan
kelebihan dan kekurangan secara garis besar antara OverClocking dan UnderVolting ... bukan antara kamu dan dia, yhaaa  :stuck_out_tongue_winking_eye:

Metric | OverClocking | UnderVolting |
-----------| -----------| ----------- |
CPU Voltase / Vcore |  Naik |  Turun |
CPU Clock |  Naik | Tetap / di Turunkan |
Suhu CPU | Semakin Tinggi | Stabil / Semakin Rendah |
Performa CPU | Naik | Tetap / Turun |
Listrik | Boros | Stabil / Hemat |
Baterai | Boros | Irit |
Umur CPU | Berkurang | Awet |
Biaya | Mahal / Ada Biaya | Tanpa Biaya |

sudah paham kan gambaran antara OverClocking dan UnderVolting secara umum ... ingat secara umum tujuan dari 2 metode diatas seperti itu, tapi balik lagi ke Tweaker / OverClocker nya

&nbsp;

&nbsp;

&nbsp;

## 3) UnderVolting!!!
---
kalau di windows, banyak sekali tools untuk overclocking / undervolting, contoh nya untuk CPU Intel ada ThrottleStop, Intel Xtreme Utility dsb. Untuk AMD Ryzen ada AMD Ryzen Master dsb

di linux, hanya ada beberapa yang terkenal dan sering di gunakan, yaitu `Python Undervolt` ... di buat menggunakan bahasa python, dokumentasi lengkap nya bisa di lihat [di sini](https://github.com/georgewhewell/undervolt)


&nbsp;
### 3.a) Persiapan UnderVolting
cara install dan makai nya sangat-sangat gampang, yang di butuhkan adalah
- Python v3.x
- Python3-pip
- **CPU Intel >= Gen 4 (Haswell)**
- ~~Gebetan~~  :joy:

```sh
# install undervolt
> sudo pip3 install undervolt

# testing pembacaan CPU Voltase / Vcore
> sudo undervolt --read
temperature target: -1 (99C)
core: 0.0 mV
gpu: 0.0 mV
cache: 0.0 mV
uncore: 0.0 mV
analogio: 0.0 mV
powerlimit: 46.25W (short: 0.00244140625s - enabled) / 37.0W (long: 28.0s - enabled) [locked]
```

:warning: **Note** : Jika proses read gagal, di karenakan tidak semua CPU bisa di undervolt, ada beberapa CPU yang tidak di dukung karna driver CPU nya atau faktor lainnya

&nbsp;
### 3.b) Core of The Core
Cara **UnderVolting** itu sangat mudah sekali, **seng penting yaqqinnnn**  :joy:

tapi serius, tidak ada rumus pasti untuk menentukan angka batas / offset, karna setiap CPU itu bisa berbeda **walaupun versi / seri nya sama**, dan karna saya sudah pernah coba-coba di windows menggunakan aplikasi ThrottleStop dan Intel Extreme Utility, maka angka offset nya tinggal saya aplikasikan di `Python Undervolt` linux ini

```sh
> sudo undervolt \
--core -165 \
--cache -165 \
--gpu -135 \
--uncore -165 \
--analogio -165

> sudo undervolt --read
temperature target: -1 (99C)
core: -165.04 mV
gpu: -134.77 mV
cache: -165.04 mV
uncore: -165.04 mV
analogio: -165.04 mV
powerlimit: 46.25W (short: 0.00244140625s - enabled) / 37.0W (long: 28.0s - enabled) [locked]
``` 

sudah selesaiii ... gampang kann??? kalo ada yang bilang susah, harus di ruqyah tu kayaknyaaa :triumph:

&nbsp;

&nbsp;

&nbsp;

## 4) Monitoring ~~Mantan~~
---
woke sekarang saatnya monitoring ~~Mantan~~ CPU nya, di sini saya membuat tools nya sendiri ... bisa di lihat [di sini](https://github.com/berrabe/cpu-util)

&nbsp;
### 4.a) Installing ~~Mantan~~
sekali lagi, cara install dan pakai nya sangat mudah, pastikan package `lm_sensors` dan `git` sudah terinstall

```sh
# Install Package Pendukung
> sudo pacman -S lm_sensors git

# Install CPU-UTIL
> git clone https://github.com/berrabe/cpu-util.git
> cd cpu-util
> sudo cp cpu-util /usr/bin/
```
:warning: **Note** : FYI untuk cpu-util ini bukan hanya untuk monitoring, tapi bisa untuk scaling frequency dan non-aktifkan multi-threading + booster ... 

&nbsp;
### 4.b) CPU Freq Scaling
Scaling CPU Frequency ini tidak wajib, tapi saya selalu menggunakannya agar hasil UnderVolt lebih maksimal
```sh
> cpu-util -sf 1000000
Core 0 	=> 1,000,000 MHz
Core 1 	=> 1,000,000 MHz
Core 2 	=> 1,000,000 MHz
Core 3 	=> 1,000,000 MHz
Core 4 	=> 1,000,000 MHz
Core 5 	=> 1,000,000 MHz
Core 6 	=> 1,000,000 MHz
Core 7 	=> 1,000,000 MHz
```
:warning: **Note** : Semakin Rendah CPU Frequency, maka nilai offset undervolt juga bisa di rendahkan ... semisal di angka 2.5 GHz sebesar -100 mV, maka setelah di **UnderClock** ke 800 MHz bisa di set ke -150 mV


&nbsp;
### 4.c) Keadaan ~~Mantan~~ CPU?
oke, saatnya menjalankan mode monitoring
```sh
> cpu-util -mo

[ Monitoring ] [ 15:16:18 ]

[+] OS Info
 |
 |-[+] Proc 	Intel(R) Core(TM) i7-4702MQ CPU @ 2.20GHz
 |-[+] Uptime 	5 hours, 1 minute
 |-[+] Host 	Lenovo IdeaPad Z410
 |-[+] Kernel 	5.4.85-1-MANJARO
 |-[+] Arch 	Manjaro Linux x86-64

[+] Sys Info
 |
 |-[+] cpu 		6.11 %
 |-[+] Driver 	intel_pstate
 |-[+] Gov 		performance - powersave
 |-[+] Mem 		2197 MB / 5701 MB
 |-[+] Core 	8 / 8 Cores
 |-[+] Stats 	976.873 MHz     39.0 °C
				984.844 MHz     39.0 °C
				997.397 MHz     37.0 °C
				997.473 MHz     39.0 °C
				997.428 MHz      
				985.895 MHz
				997.461 MHz
				989.919 MHz

```
sudah terlihat, informasi basic dari linux saya dan prosesor nya ... dan bisa di lihat juga suhu prosesor saya, **rerata hanya 39.0 °C !!** ... 

ini posisi intel i7 yang haus power, dan posisi lagi buka banyak tab firefox bejibun (lihat penggunaan ram), Ngetik, Coding, Dengerin Spotify dsb

**UnderVolting** itu Worth It Bukan?

&nbsp;

&nbsp;

&nbsp;

## 5) CPU Jadul / Non-Intel?
---
oke, semua hasil sudah terlihat, dan menurut saya **UnderVolt** itu sangat worth it ... karna banyak maanfaatnya, paling kerasa di suhu CPU, penggunaan baterai dan nantinya berimbas ke keawetan CPU itu sendiri  :sunglasses:

tapi masalahnya metode ini hanya berlaku untuk intel generasi terbaru `>= Gen 4 / Haswell`, tapi bagaimana jika intel di bawah `Gen 4 atau CPU AMD`?

- **Intel Di Bawah Gen 4**

saya dulu pernah mencoba pada prosesor Intel Core 2 Duo di laptop VAIO VGN-Z46GD tahun 2008 menggunakan [Intel PHC, bisa di lihat di sini](https://wiki.archlinux.org/index.php/PHC). dengan driver CPU lama `acpi_cpufreq` di bandingkan yang sekarang `Intel P-states`

tapi menurut saya, [Intel PHC](https://wiki.archlinux.org/index.php/PHC) tidak cukup powerful karna tidak bisa melakukan UnderVolting di bawah 0 mV (offset tidak bisa minus), sehingga hasil nya terasa kurang maksimal


- **Processor AMD**

untuk processor AMD jujur saja saya belum pernah coba, tapi bila membaca dokumentasi dari tools [Python3 Undervolt](https://github.com/georgewhewell/undervolt) dan [Intel PHC](https://wiki.archlinux.org/index.php/PHC). tools-tools ini bisa melakukan nya dengan lancar tanpa ada kendala, *syarat dan ketentuan berlaku* :blush: