---
title: "Docker Reverse Proxy #2"
date: 2021-01-23T17:49:50+07:00
weight: 1
tags: ["Docker", "Linux", "Automation"]
author: "berrabe"
showToc: true
TocOpen: True
hidemeta: false
disableShare: false
cover:
    image: "https://miro.medium.com/max/2800/1*rnzxfcy2N_ffJPnBundJQw.jpeg"
    alt: "Loh Imange Nya Ilang"
    relative: false
comments: false
---

&nbsp;

&nbsp;

## 1) TLDR
---
oke ... artikel kali ini akan singkat saja karna cuman lanjutan dari [docker reverse proxy #1]({{< ref "docker-reverse-proxy-1.id.md" >}}) 

pada [docker reverse proxy #1]({{< ref "docker-reverse-proxy-1.id.md" >}}), saya sudah berhasil memisahkan website berdasarkan domain yang di simpan pada satu server  :innocent:

```sh
                                                  +-----------------------------+
                                                  |        SERVER 1             |
                      +--------------+            |                             |
+--------+            |              |            |     = WordPress             |
|  USER  |  +------>  |   INTERNET   | +------->  |     = PHPMyAdmin            |
+--------+            |              |            |     = Grafana               |
                      +--------------+            |     = test (a.berrabe.com)  |
                                                  |                             |
                                                  +-----------------------------+
```

domain tester pada [docker reverse proxy #1]({{< ref "docker-reverse-proxy-1.id.md" >}}) bernama `a.berrabe.com`, dimana container upstream nya berupa web server dari `nginx:alpine`

:warning: **Note** : domain tester hanya di gunakan untuk ngetest auto-gen mapping domain reverse-proxy nya dan auto-gen SSL Certs letsencrypt domain

**terus artikel lanjutan ini di buat untuk apa?**  :worried:

jadi gini ... kemarin di [docker reverse proxy #1]({{< ref "docker-reverse-proxy-1.id.md" >}}) ada peringatan, bahwa reverse-proxy ini secara default hanya **bisa di gunakan pada container yang berjalan di atas network bridge**, selain itu?
... gatau dahh mwehehehe  :joy:
&nbsp;

&nbsp;

&nbsp;

## 2) Selalu Punya Yang Baru
---
katakanlah saya ingin menjalankan aplikasi [php framework laravel / CI](https://github.com/berrabe/docker-php-framework), dengan infranya berupa stack nginx, php-fpm, dan mariaDB ... di docker, stack infra tadi biasanya di jalankan melalui docker-compose

masalahnya, secara default docker-compose akan membuat network baru untuk stack infra tadi, biasanya di ambil dari nama folder tempat file `docker-compose.yml` di simpan

```sh
docker-php-framework/
   ├──  docs/  
   ├──  nginx/
   ├──  Dockerfile  
   ├──  docker-compose.yml
```

jika file `docker-compose.yml` di atas di up dengan `docker-compose up -d`, maka secara default network `dockerphpframework_default` akan dibuat

yang mana nantinya, **docker reverse proxy** tidak bisa memforward packet dari reverse-proxy ke aplikasi upstream `(docker-php-framework)` karna beda network
  - **docker reverse proxy** di atas network `bridge`
  - sedangkan **docker-php-framework / upstream** di atas network `dockerphpframework_default`

&nbsp;

&nbsp;

&nbsp;

## 3) Menyelesaikan Masalah Tanpa Solusi
---
sebetulnya cara resolve nya sangat sederhana, ada 2 cara:
  1. ubah network di `docker-compose.yml` stack infra upstream ke network `bridge`, dan menggunakan fitur legacy `links` jika container di dalam stack ingin saling terhubung 
  2. atau setup container `docker reverse-proxy` untuk join ke semua network yang tersedia, kecuali network `host` dan `none`

saya lebih memilih cara ke 2, karna lebih mudah menyesuaikan satu container agar bisa work dari pada sebaliknya ... dan juga cara 1 menggunakan fitur legacy `links` yang mana sudah tidak di rekomendasi oleh docker

&nbsp;

&nbsp;

&nbsp;

## 4) Semua Serba Otomatis
---
okeh .. sekarang sudah jelas, yang perlu di lakukan adalah memasukkan container **docker reverse proxy** ke semua network agar proses proxy pass dapat bekerja  :dizzy_face:

hhmm, tpi semisal saya jalanin 10 aplikasi, dan setiap aplikasi pakai compose, dan setiap compose punya network sendiri-sendiri, repot juga yak klo harus `docker network connect` atu-atu  :weary:

untuk itulah saya membuat script automasi nya  :sunglasses:

```sh
#!/bin/bash

# -------------------------------------------------

_CONTAINER_NAME_=$1

# -------------------------------------------------


clear
echo -e "\t ./ Docker Auto Join Network \.\n\n"


function check() {
	if [[ $? -eq 0 ]]; then
		echo "OK"
	else
		echo "ERROR $?"
		exit
	fi
}


echo -e " [+] LIST ALL DOCKER NETWORKS "
_ALL_NET_=$(docker network list --format "{{.ID}}={{.Name}}" | grep -vE "host|none") 
echo -e "$_ALL_NET_" | awk -F '=' '{printf "  |--[-] %s >>> %s\n", $1, $2}'



echo -e "\n\n [+] LIST [ $_CONTAINER_NAME_ ] CONTAINER CONNECTED NETWORKS"
_ALL_CONN_=$(docker inspect $_CONTAINER_NAME_ | grep "NetworkID" | awk -F '"' '{print $4}' | cut -b -12)
echo -e "$_ALL_CONN_" |  awk '{if($1=="") $1="EMPTY"; printf "  |--[-] %s\n", $1}'

if [[ $_ALL_CONN_ == "" ]]; then
	exit
fi



echo -e "\n\n [+] DISCONNECT ALL NETWORKS ATTACHED TO [ $_CONTAINER_NAME_ ] CONTAINER"
for i in $_ALL_CONN_; do
	printf "  |--[-] Disconecting %s ... " "$i"
	docker network disconnect $i $_CONTAINER_NAME_ >/dev/null 2>&1

	check
done



echo -e "\n\n [+] CONNECTING [ $_CONTAINER_NAME_ ] TO ALL NETWORKS"
for i in $(echo -e "$_ALL_NET_" | awk -F '=' '{print $1}'); do
	printf "  |--[-] Connecting to %s ... " "$i"
	docker network connect $i $_CONTAINER_NAME_ >/dev/null 2>&1

	check
done


echo -ne "\n\n [+] RESTART [ $_CONTAINER_NAME_ ] CONTAINER ... "
docker restart $_CONTAINER_NAME_ >/dev/null 2>&1
check
```

masih ingat dengan direktori struktur di [docker reverse proxy #1]({{< ref "docker-reverse-proxy-1.id.md" >}}) ?

di sana di buat file bernama `auto_net.brb` ... nah copy script di atas ke file itu, jangan lupa di beri exec permissions `chmod +x auto_net.brb`

&nbsp;

&nbsp;

&nbsp;

## 5) TESS ... SSSTTTTT
---
agar container **docker reverse proxy** masuk ke semua network, caranya sangat simple  :+1:

```sh
> ./auto_net.brb <nama container>
```

di karenakan nama container default nya **docker reverse proxy** adalah `reverse-proxy` ... maka command nya jadi `./auto_net reverse-proxy`

```sh
   ./ Docker Auto Join Network \.


 [+] LIST ALL DOCKER NETWORKS 
  |--[-] 329d3a2e32d8 >>> 05nginxdirlist_default
  |--[-] cefc731a02d6 >>> 06monitoringstack_default
  |--[-] a72740956861 >>> berrabe_default
  |--[-] 7f23f3060488 >>> bridge


 [+] LIST [ reverse-proxy ] CONTAINER CONNECTED NETWORKS
  |--[-] 7f23f3060488
  

 [+] DISCONNECT ALL NETWORKS ATTACHED TO [ reverse-proxy ] CONTAINER
  |--[-] Disconecting 7f23f3060488 ... OK


 [+] CONNECTING [ reverse-proxy ] TO ALL NETWORKS
  |--[-] Connecting to 329d3a2e32d8 ... OK
  |--[-] Connecting to cefc731a02d6 ... OK
  |--[-] Connecting to a72740956861 ... OK
  |--[-] Connecting to 7f23f3060488 ... OK


 [+] RESTART [ reverse-proxy ] CONTAINER ... OK
```

dann tadaaaa ... container reverse-proxy sudah masuk ke semua network, jadi sekarang sudah bisa reverse-proxy ke semua container  :clap: :clap: :clap:

untuk lebih detail nya, bisa pakai command inspect

```sh
> docker inspect reverse-proxy | grep "NetworkID"

"NetworkID": "329d3a2e32d80d5da8accfc71fdc1142f8ca06d12ae7ed328950136aba6c27c9",
"NetworkID": "cefc731a02d65e46b8085675b978d4abcf60c7a112ef0b829cf137870bc65424",
"NetworkID": "a727409568618db52186fdd2c2ed8bddc36bd2379c098c2b1eabaabf6c08c317",
"NetworkID": "7f23f306048847b82a36cacb517a6121c395ed48a9f3b2f784921915b78501f7",
```

**Penjelasan :**

  - jika di lihat, awalnya container `reverse-proxy` hanya connect ke network `bridge` dengan hash `7f23f3060488`
  - kemudian script akan disconnect container `reverse-proxy` dari semua network untuk mencegah error dan collision
  - baru deh container bakal di konekin ulang ke semua network, list network bisa di lihat pakai command `docker network list`