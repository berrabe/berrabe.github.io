---
title: "Automation For Changing Server Password and Setup SSH-Key to Dozens Of Servers"
date: 2021-01-07T10:49:33+07:00
weight: 1
tags: ["Automation", "Ansible", "Linux", "SSH"]
author: "berrabe"
showToc: true
TocOpen: True
hidemeta: false
disableShare: false
cover:
    image: "https://gmedia.net.id/upload/foto_artikel/20200615AapDmbl6Pa.jpg"
    alt: "Loh Imange Nya Ilang"
    relative: false
comments: false
---

&nbsp;

&nbsp;

## 1) TLDR
---
tet tet tetttt ... balek lagi sama saya, **berrabe** ... dimana saya suka memecahkan masalah menjadi masalah lagiii, YEAY  :sunglasses:

kali ini saya pengen sharing dikit ... suatu hari, saya di suruh ganti password server + setting SSH-Key yang akan di lakukan rutin setiap bulan, untuk alasan keamanan

masih oke lahhhh, ganti gitu doanggg  :kissing_smiling_eyes: ... 

eits, tapi masih belum selesai, ternyata server nya kagak cuman atuu, tapi puluhan server ... **WHAT!?**, lebih tepat nya 40 server per bulan ini yang akan bertambah lagi seiring pertambahan client

FYI, beberapa block server di tempatkan pada DC (Data Center) yang berbeda dengan kebijkan firewall berbeda pula, dan tidak semua server punya IP PUBLIC ... **MAMPOSSSHHHH**  :scream:

mulai mikir nih, klo ganti manual tangan saya tekor kaga yak, dan butuh berapa lama? belum lagi klo kagak berhasil karna beberapa faktor, seperti human error

**dan di tambah lagi, semua ini di lakukan perbulan!!**  :sob:

&nbsp;

&nbsp;

&nbsp;

## 2) Mencari Solusi Tanpa Solusi  :sweat_smile:
---
okeh .. singkat cerita akhirnya saya nemu makanan namanya **Ansible** ... eh bukan makanan deng, tapi tools  :sweat_smile:

karna tadi ngobrol ama ansible nya cuman bentar, jadi saya crita tentang doi dikit aje ye, selebihnya bisa cari di dokumentasi, artikel, dll ... 

singkatnya, doi merupakan tools yang di gunakan untuk meng-automasi kerjaan kita terkait maintenance hardware di IT (Server, Router, Switch, dll)

automasi nya kek mana? yaaaa seperti ganti password, konfigurasi server / router, deploy script dll ... pokoknya semua hal yang kita lakukan secara manual menggunakan `CLI (Command Line Interface)` lewat `SSH (Secure Shell)` bisa di lakukan ama doi secara otomatis ... aji gile, gokil benerrr  :dizzy_face:

&nbsp;

&nbsp;

&nbsp;

## 3) Syarat Untuk ~~nikah~~ Memakai?
---
Syarat untuk memakai si **Ansible** sangat mudah, yang penting punya:
- Laptop / Komputer
- SSH + SSH-Key
- Python v2.x / v3.x
- VPN
- ~~**PACAR**~~

laptop / komputer itu sebagai "OTAK" yang akan mengatur semua server-server tadi .. kebetulan server-server yang akan saya remote basis nya CentOS dan Debian, jadi semua sudah otomatis terinstall SSH dan Python v2.x secara default

soooo, saya hanya butuh install **Ansible** aja sebagai "OTAK" di LAPTOP Saya

FYI, di karenakan server-server ini berbeda tempat + berbeda password, maka saya login atu-atu dulu ke semua server buat nyetting SSH-Key punya saya, biar klo login kaga usah ngetik password + saya juga setting VPN, tepatnya OpenVPN di setiap server, agar konek nya gampang, dan maintenance list server nya mudah (satu network)

&nbsp;

&nbsp;

&nbsp;

## 4) Caranya?
---
okehh, lanjottt ke cara make si **Ansible** nya kek manaaa? ... sabar sodara-sodara, mikir kalimat nya dulu ini, emang gampang apa nulis beginian :expressionless:

&nbsp;
### 4.a) Install si Otak
Karna Laptop saya menggunakan Linux (Arch-Based), maka cara install nya tinggal pakai paket manager saja ... untuk yang tidak sama, bisa ikutin caranyee di [dokumentasi resminya](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-arch-linux)
```sh
> pacman -S ansible 
```

&nbsp;
### 4.b) Konfigurasi
Masuk ke tahap konfig, mari kita buat simple
```sh
> mkdir skidipapap
> cd skidipapap
> touch hosts.yml && touch 01-passwd.yml
```

&nbsp;
#### 4.b.I) Konfigurasi - hosts
Nah, kan saya mau automasi ke server-server nih, saya harus ngasih tau dulu si **Ansible**, server nya yang mane ajee ... kan doi belum pernah kenalan, kesian ntar malah salah masuk server, kan jadi berrabe

isi ke file `hosts.yml` seperti di bawah, ganti x.x.x.x menjadi ip server
```yaml
---
all:
  hosts:
    x.x.x.x
    x.x.x.x
    x.x.x.x
    # dst

  vars:
    ansible_user: root
    ansible_python_interpreter: /usr/bin/python
```

&nbsp;
#### 4.b.I) Konfigurasi - Otak
Oke, sekarang bagian terpenting, saya akan setting logic automasi nya ... isi ke file `01-passwd.yml`
```yaml
---
- name: CHANGE PASSWORD ALL SERVER
  hosts: 'all'
  gather_facts: no

  tasks:
    - name: PING
      ping:

    - name: SET / DELETE SSH-KEY WITH BLOCK
      blockinfile:
        path: /root/.ssh/authorized_keys
        state: present
        marker: "# {mark} SSH Public Key, reset every month | DO NOT DELETE THIS BLOCK !!!"
        block: "{{ lookup('file', '$HOME/.ssh/authorized_keys') }}"
        # block: "ssh-rsa xxxxxxxxx ..."

    - name: CHANGE ROOT PASSWORD
      user:
        name: root
        update_password: always
        password: "{{ password|password_hash('sha512') }}"
```

FYI, jika setting manual biasanya menggunakan command CLI Linux, di **Ansible** berbeda ... 

**Ansible** menggunakan `module` yang isinya berupa script python (makanya server butuh terinstall python), yang akan di kirimkan dari si OTAK (Client) ke server-server ... jika sudah di kirimkan, script python tadi akan di eksekusi

tiap module berbeda cara penggunaan, tinggal baca dokumentasi terkait module apa yang akan di gunakan (misal modul user untuk membuat / mengganti / menghapus informasi suatu user di Linux)

&nbsp;
### 4.c) Ngengggg ...
wokai, gampang kan konfigurasi nya? langkah terakhir yaitu jalanin si ansible ini ... cukup 1 baris perintah
```sh
> ansible-playbook 01-passwd.yml -e 'password=berrabe_ganteng' -i hosts-yml -f 40
```

&nbsp;

&nbsp;

&nbsp;

## 4) Kesimpulan
---
huahhhh ... setelah 5 menit konfig ansible + 3 jam setting ssh-key, setting vpn, tshoot di puluhan server ... selesai juga ni kerjaan  :muscle:

hasilnya akan seperti ini
```sh

PLAY [CHANGE PASSWORD ALL SERVER]

TASK [PING]
ok: [x.x.x.x]
ok: [x.x.x.x]
ok: [x.x.x.x]

TASK [SET / DELETE SSH-KEY WITH BLOCK]
changed: [x.x.x.x]
changed: [x.x.x.x]
changed: [x.x.x.x]

TASK [CHANGE ROOT PASSWORD]
changed: [x.x.x.x]
changed: [x.x.x.x]
changed: [x.x.x.x]

PLAY RECAP
x.x.x.x          : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
x.x.x.x          : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
x.x.x.x          : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

hhmmm, gampang kan? 

BTW, saya konfigurasi manual untuk ganti password, setup ssh-key dan troubleshoot nya **BUTUH 3 JAM AN** ... belum ditambah ama kepala pusing, mata merah, laper, ngantuk, leher pegel, ga punya pacar, eh ... dan ini akan terulang setiap bulan, pfttt  :sleepy:

dan coba bandingkan menggunakan **Ansible**, dengan kemudahan yang doi punya, hanya butuh 5 - 30 menit untuk konfigurasi, tinggal search module nya apa (misalkan user dan blockinfile), saya bisa ganti password + setting SSH-Key dalam waktu **5 menit ke puluhan server**

jadi saya tidak perlu lagi meluangkan waktu 3 jam an setiap bulan hanya untuk konfigurasi server perihal pergantian password dan setup SSH-Key. :sunglasses: