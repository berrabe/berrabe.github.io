---
title: "Telegram Notification on SSH Login"
date: 2021-01-09T13:40:31+07:00
weight: 1
tags: ["Linux", "Automation", "Notif"]
author: "berrabe"
showToc: true
TocOpen: True
hidemeta: false
disableShare: false
cover:
    image: "https://www.howtogeek.com/wp-content/uploads/2019/07/img_5d28e05a4901f.png"
    alt: "Loh Imange Nya Ilang"
    relative: false
comments: false
---

&nbsp;

&nbsp;

## 1) TLDR
---
tet tet tetttt ... balek lagi sama saya, **berrabe** ... dimana saya suka memecahkan masalah menjadi masalah lagiii, YEAY  :sunglasses:

kali ini saya pengen sharing dikit tentang gimane caranya kasih notif ke tele kalau ada yang login ke server lewat SSH atau [interactive shell](https://askubuntu.com/questions/438150/why-are-scripts-in-etc-profile-d-being-ignored-system-wide-bash-aliases)

- **knapa harus ke telegram?** 

ya karna biar simpel aja gitu sembari chat ama ~~pacar~~ temen, bisa lihat notif server + bisa di buka di manapun (HP, Website, Desktop) dengan cepat tanpa buka WebApp, login dst kayak pake ELK, Grafana Loki dkk

- **dan knapa harus di kasih notif saat login SSH?**

buat info siapa aja yang lagi akses server, kan klo server nya bermasalah ada yang bisa di salahin  :joy: :joy:, tugas saya sebagai Administrator berkurang sikit :sleeping:

wokai tanpa banyak basa-basi langsung gassss ...

&nbsp;

&nbsp;

&nbsp;

## 2) Script !!!
---
oke di sini saya kasih shell script nya, ini script yang tugasnya detect ssh login dan kasih notif + logging ... *bukan mendeteksi doi lagi chat an ama siapa aja lho ya* :relieved:

copy script yang ada di bawah ini ke file namanya `login-ssh.sh` pada server yang ingin di montoring

```sh
#!/usr/bin/env bash

# ===============================================================================

_EXCLUDE_IP_=( "127.0.0.1" "::1" )
_USER_ID_=( -10xxxx -11xxxx )
_TOKEN_TELEGRAM_="1xxxxx:xxxxxxxxxxxxxxx"
_LOG_FILE="/var/log/ssh-login.log"

# ===============================================================================

TelegramNotif () {
	echo -ne "[+] Welcome To $(hostname -s) Server "
	curl -s -X POST \
	https://api.telegram.org/bot$_TOKEN_TELEGRAM_/sendMessage \
	-d chat_id=$1 \
	-d text="$2" \
	-d parse_mode=html \
	-d disable_web_page_preview=true \
	> /dev/null

	if [[ $? -eq 0 ]]; then
		echo "[ OK ]"
		Logging "$ClientIP_ as $USER ... Notif Success"
	else
		error_=$?
		echo "[ ERROR $error_ | Contact Server Administrator ]"
		Logging "$ClientIP_ as $USER ... Notif ERROR $error_"
	fi
}



Logging () {
	date_=$(date "+%d-%b-%Y")
	time_=$(date "+%H:%M:%S")

	echo -e "[ SSH ] [ $date_ ] [ $time_ ] >> $1" >> $_LOG_FILE
}



Filter () {
	for i in "${_USER_ID_[@]}"
	do
		ClientIP_=$1
		dateTime_=$(date "+%d %b %Y %H:%M:%S")
		ServerHostname_=$(hostname -s)
		IPInfoAPI_="https://ipinfo.io/${ClientIP_}"
		IPVPN=$(ip a | grep -E "inet.*tun" | awk '{sub(/\/.*/,"");print $2}')


		NotifText_="
		#ssh_login
		#$(echo $ServerHostname_ | sed -E 's/-|\./_/g')_login

		[#]   <b>$dateTime_</b>

		[=]   <b>$ClientIP_</b> <i>connect as</i> <b>$USER</b>
		       <i>on</i> <b>$ServerHostname_ [ $IPVPN ]</b>
		
		[?]   <i>More Info on</i> <b>$IPInfoAPI_</b>"


		TelegramNotif $i "$NotifText_"
	done
}




if [ -n "$SSH_CLIENT" ]; then

	ClientIP_=$(echo $SSH_CLIENT | awk '{print $1}')
	Start_=1

	for i in "${_EXCLUDE_IP_[@]}"
	do
		if [[ $i == $ClientIP_ ]]; then
			Logging "Excluded IP $USER @ $i"
			Start_=0
		fi
	done
fi

if [[ $Start_ -eq 1 ]]; then
	Filter $ClientIP_
fi
```

&nbsp;

&nbsp;

&nbsp;

## 3) How To
---
trus abis di copy ke file `login-ssh.sh`, cara jalanin nya gimane? ... 

waittt, sebelum di jalanin, script nya harus di setup agar berfungsi sesuai keingingan

&nbsp;
### 3.a) Konfigurasi Script
di bagian paling atas script, sesuaikan variabel dengan setup / credential yang ada
```sh
_EXCLUDE_IP_=( "127.0.0.1" "::1" )
_USER_ID_=( -10xxxx -11xxxx )
_TOKEN_TELEGRAM_="1xxxxx:xxxxxxxxxxxxxxx"
_LOG_FILE="/var/log/ssh-login.log"
```

sesuaikan `user id` dan `token telegram` ... 

untuk mendapatkan token telegram bisa buat di [@botFather](https://t.me/botfather) dan untuk user id bisa di lihat dengan API `https://api.telegram.org/bot<BOTID>/getUpdates` ... btw untuk user id bisa lebih dari 1, karna script ini bisa send notif ke beberapa group **sesuai user id group** yang di berikan

dan jika script mendeteksi adanya `exluded ip`, maka script tidak akan mengirimkan notif untuk ip tersebut ... **tetapi untuk log tetap tercatat di masing-masing server**

:warning: **Note** : jangan tanya `user id` atau `token telegram` ke abang bakso kalau tidak mau di lempar kuah panas ... sukur-sukur di lempar ama bakso / gerobaknya  :joy: :joy:

&nbsp;
### 3.b) Setup SSH
okehh, supaya script ini otomatis mendeteksi ada user login dari SSH / Interactive Shell, script nya harus di pindahin ke folder `/etc/profile.d`

```sh
> cp login-ssh.sh /etc/profile.d/
```

&nbsp;

&nbsp;

&nbsp;

## 4) hasilnya?
---
nah ini nanti tampilan notif nya, sudah generated hashtag per server secara otomatis juga, jadi akan mudah kalau mau liat history

![telegram](/2021/06-telegram.png)

dan yang ini log di setiap server yang sudah di pasang script monitoring SSH

```sh
> tail -f /var/log/ssh-login.log

[ SSH ] [ 09-Jan-2021 ] [ 15:41:37 ] >> 172.16.100.5 as root ... Notif ERROR 1
[ SSH ] [ 11-Jan-2021 ] [ 09:16:16 ] >> Excluded IP root @ 127.0.0.1
[ SSH ] [ 11-Jan-2021 ] [ 09:16:30 ] >> Excluded IP root @ ::1
[ SSH ] [ 12-Jan-2021 ] [ 11:26:51 ] >> 116.206.40.56 as root ... Notif Success
[ SSH ] [ 12-Jan-2021 ] [ 11:29:08 ] >> 116.206.40.56 as root ... Notif Success
```