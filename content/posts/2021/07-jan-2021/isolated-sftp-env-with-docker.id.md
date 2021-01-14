---
title: "Isolated SFTP Env With Docker"
date: 2021-01-14T21:17:42+07:00
weight: 1
tags: ["Linux", "Docker"]
author: "berrabe"
showToc: true
TocOpen: True
hidemeta: false
disableShare: false
cover:
    image: "https://guides.wp-bullet.com/wp-content/uploads/2018/08/sftp-download-linux-command-line.png"
    alt: "Loh Imange Nya Ilang"
    relative: false
comments: false
---

&nbsp;

&nbsp;

## 1) TLDR
---
tet tet tetttt ... balek lagi sama saya, **berrabe** ... dimana saya suka memecahkan masalah tanpa solusi ... YEAY  :sunglasses:

kali ini, saya pengen sharing caranya setup **SFTP (SSH File Transfer Protocol) yang isolated.**

Apa itu isolated dan kenapa harus isolated? ... mari saya critain dulu

jadi gini ... suatu ketika client request untuk di buatkan akun SFTP, yang mana di khususkan untuk download / upload file-file pada folder tertentu saja

Tapi saya berfikir, kalau saya setup SFTP nya pakai SSH yang di gunakan untuk remote dan config server (Port 22), takutnya jadi *berrabe*  :joy: :joy:

siapa tau tangan client jail .... pengen ngetest system, trus dia nyoba meng-eksploitasi system buat loncat dari chroot SSH, trus setelah keluar dia maintenance user ke root dengan privilege escalation, trus setelah dari root malah otak-atik naruh backdoor dsb, trus trus trus dan akhirnya nabrak   :weary: :weary:

hhmm, saya termasuk orang yang posesif dan overprotektif sekali sepertinya terhadap pacar, eh ... server maksudnya  :sleepy: :sleepy:
&nbsp;

&nbsp;

&nbsp;

## 2) The Real Helper ... Docker!!
---
saya suka memakai docker, karna nantinya, tools / app yang berjalan di dalamnya akan bersifat isolated tetapi tetap enteng, tidak seperti memakai Virtual Machine

dan saya akan setup SFTP di dalam docker, yang nantinya akan terpisah dari SSH utama yang di gunakan untuk remote dan config server ...  :sunglasses:

yahh walaupun sudah ada laporan bahwa ada security researcher yang bisa keluar dari isolated env docker ke host system, tapi setidaknya saya sudah menambah layer security dan mengurangi attack vector yang akan terjadi kedepannya  :thumbsup:

&nbsp;
### 2.a) Install Docker
pastikan docker sudah terinstall, jika belum ikutin langkah di bawah ini ... *please jangan nanya knapa harus install docker* :triumph:
```sh
> curl -fsSL https://get.docker.com -o get-docker.sh
> sh get-docker.sh
> systemctl start docker
```

&nbsp;
### 2.b) Setup Env
setelah docker terinstall, saya akan membuat folder `SFTP-Isolated-berrabe/berrabe` yang di gunakan untuk menampung file-file dan script SFTP
```sh
> mkdir -p SFTP-Isolated-berrabe/upload
> cd SFTP-Isolated-berrabe
> chmod -R 666 upload
```

&nbsp;

&nbsp;

&nbsp;

## 3) The Power Of Copas (Copy/Paste)
---
setelah docker ter install, saatnya setup SFTP nya agar berjalan di bawah docker dan bersifat isolated ... 

&nbsp;
### 3.a) Script
dri pada config manual, saya lebih suka config otomatis, untuk itu saya membuat script yang sangat sederhana ini 
```sh
#!/bin/bash

# -------------------------------------------------

_CONTAINER_NAME_="SFTP-Isolated-berrabe"
_CONTAINER_VOL_NAME="upload"

_SFTP_PORT_=2323
_SFTP_USER_="berrabe"
_SFTP_PASS_="iloveyou3000"
_SFTP_UID_="1000"
_SFTP_GUID_="1000"
_SFTP_FOLDER_BINDING_="$(pwd)/upload"

# -------------------------------------------------


clear
echo -ne " [+] Stopping $_CONTAINER_NAME_ Container ..... "
docker rm -f $_CONTAINER_NAME_ >/dev/null 2>&1

if [[ $? -eq 0 ]]; then
	echo "OK"
else
	echo "ERROR $?"
	exit
fi



echo -ne " [+] Starting New $_CONTAINER_NAME_ Container ..... "
docker run \
    --name $_CONTAINER_NAME_ \
    -v $_SFTP_FOLDER_BINDING_:/home/$_SFTP_USER_/$_CONTAINER_VOL_NAME \
    -p $_SFTP_PORT_:22 \
    -d atmoz/sftp:alpine \
    $_SFTP_USER_:$_SFTP_PASS_:$_SFTP_UID_:$_SFTP_GUID_ \
    >/dev/null 2>&1

if [[ $? -eq 0 ]]; then
	echo "OK"
else
	echo "ERROR $?"
	exit
fi


docker logs -f $_CONTAINER_NAME_

```

:warning: **Note** : copy script ke file `SFTP-Isolated-berrabe/run.sh`, jangan lupa di beri execute permission dengan cara `chmod +x run.sh` ... *ingat, beri execute permission, bukan harapan palsu*  :blush:

&nbsp;
### 3.a) Config The Script
jika di lihat di baris paling atas, ada beberapa konfigurasi default ... jika ingin diganti, tinggal sesuaikan saja
```sh
_CONTAINER_NAME_="SFTP-Isolated-berrabe"
_CONTAINER_VOL_NAME="upload"

_SFTP_PORT_=2323
_SFTP_USER_="berrabe"
_SFTP_PASS_="iloveyou3000"
_SFTP_UID_=""
_SFTP_GUID_=""
_SFTP_FOLDER_BINDING_="$(pwd)/upload"
```
Penejelasan :
- **Container Name** : nama SFTP container saat di lihat pada command `docker ps -a`
- **Container Volume** : nama folder yang ada pada saat konek ke dalam SFTP, biar gampang, samakan dengan `_SFTP_FOLDER_BINDING_`
- **SFTP Port** : port SFTP yang di gunakan client untuk konek
- **SFTP User** : username untuk masuk ke SFTP
- **SFTP Port** : password untuk masuk ke SFTP
- **SFTP UID / GUID** : di gunakan agar bisa read/write, isi sesuai UID / GUID user di host system, lihat di `/etc/passwd`, default kosong (hanya read)

:warning: **Note** : UID / GUID tidak di butuhkan jika folder upload sudah di set permission ke 666 / rw seperti [di sini](#2b-setup-env)

&nbsp;

&nbsp;

&nbsp;

## 4) Start The Show
---
setelah semua setup dan konfigurasi selesai, jalankan dengan command `./run.sh`
```sh
> ./run.sh

 [+] Stopping SFTP-Isolated-berrabe Container ..... OK
 [+] Starting New SFTP-Isolated-berrabe Container ..... OK

...
```

untuk melihat status container, apakah sudah jalan atau ada error, gunakan command `docker ps -a`
```sh
> docker ps -a

CONTAINER ID   IMAGE               COMMAND                  CREATED          STATUS          PORTS                  NAMES
931768b6ca16   atmoz/sftp:alpine   "/entrypoint berrabe…"   39 minutes ago   Up 39 minutes   0.0.0.0:2323->22/tcp   SFTP-Isolated-berrabe
```

:warning: **Note** : untuk run pertama kali, sedikit lama, karna harus mendownload image dari docker hub repo ... tekan `CTRL+C` jika ingin keluar


&nbsp;

&nbsp;

&nbsp;

## 5) Testing
---
setelah SFTP di jalankan, saat nya coba untuk konek, bisa pakai filezilla atau SFTP CLI ... sama aja konsepnya
```sh
> sftp -P 2323 berrabe@x.x.x.x
berrabe@x.x.x.x password: 
Connected to x.x.x.x.

sftp> ls
upload

sftp> cd upload/

sftp> put index.php .
Uploading index.php to /upload/./index.php
index.php                                  100% 1281   1.0MB/s   00:00

sftp> ls
index.php  
```

folder `upload` sesuai dengan [config default](#3a-config-the-script), dan di sini saya berhasil upload file `index.php` dari local ke server karna sudah di setting folder permission atau adanya UID / GUID