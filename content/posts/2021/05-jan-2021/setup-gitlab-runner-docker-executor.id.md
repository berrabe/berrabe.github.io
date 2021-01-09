---
title: "Setup Gitlab-Runner + Docker Executor"
date: 2021-01-08T13:14:33+07:00
weight: 1
tags: ["Automation", "Linux", "Docker"]
author: "berrabe"
showToc: true
TocOpen: True
hidemeta: false
disableShare: false
cover:
    image: "https://camo.githubusercontent.com/62094a1ac392fd7cf26e6c5215708fd004c1ac8c2c4c996763343f1c134ee6a5/68747470733a2f2f61626f75742e6769746c61622e636f6d2f696d616765732f63692f63692d63642d746573742d6465706c6f792d696c6c757374726174696f6e5f32782e706e67"
    alt: "Loh Imange Nya Ilang"
    relative: false
comments: false
---

&nbsp;

&nbsp;

## 1) TLDR
---
tet tet tetttt ... balek lagi sama saya, **berrabe** ... dimana saya suka memecahkan masalah menjadi masalah lagiii, YEAY  :sunglasses:

okehhh ... saya pengen share lagi caranyooo lari dari kenyataan, ehh  :sweat_smile: ... 

fokusss ... maksudnya cara nge-install gitlab-runner, WUHUUU  :dizzy_face:

> :warning: **Note** : sebelum masuk ke sini, pastikan sudah paham apa itu kultur DevOps, dan istilah-istilah di dalamnya, seperti CI / CD, Pipeline, Build, Deploy, Automation, dll

&nbsp;

lanjottt , gitlab-runner itu ~~cemilan~~ apaan sih? ... 

bagi yang belum tau / belum pernah dengar, gitlab-runner merupakan ~~cemilan~~ program yang di gunakan untuk menghandle proses automasi kerja yang terjadi pada pipeline di gitlab CI / CD

dyarrrr ... mumet to ndase  :weary:  :weary:

&nbsp;

![Gitlab Runner](https://res.cloudinary.com/practicaldev/image/fetch/s--M_MmjEPL--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://docs.gitlab.com/ee/ci/img/cicd_pipeline_infograph.png)

untuk lebih jelasnya lihat gambar di atas, **CI PIPELINE dan CD PIPELINE** semua nya butuh gitlab-runner

dimana runner berfungsi sangat penting agar CI / CD pipeline pada gitlab bisa berjalan, jika tidak ada runner yang aktif / terset, maka job di pipeline status nya akan **pause dan stuck / tidak bisa jalan**

&nbsp;

&nbsp;

&nbsp;

## 2) Wuuzzzz ....
---
okeh, tapi bukannya Gitlab sudah menyediakan runner punya mereka sendiri (shared runner)? , trus kenapa harus install runner punya sendiri?

jawabannya **"tergantung orangnya"**, kalo saya install runner sendiri agar proses pipeline bisa lebih cepat di bandingkan shared runner gitlab (namanya juga berbagi) dan untuk alasan keamanan ... yap karna keamanan sangat penting

walaupunnnnn shared-runner akan menghapus container saat job selesai, tapi kan kita tidak bisa menjamin 100%, apalagi jika code di repo sangat private dan ada beberapa informasi sensitif di dalamnya (walau bukan best practice) 

hhmmmmm .... OK, setelah mulut berbusa karna terlalu banyak jelasin, langsung lanjot ae ke praktek setup nya dah  :sweat:

&nbsp;
### 2.a) ~~Siomay~~ Docker sebagai executor
sebelum masuk ke gitlab-runner nya, install executor dulu

executor itu merupakan ~~makanan~~ tools / metode yang di gunakan gitlab-runner untuk mengeksekusi job di pipeline CI / CD nya, untuk list executor bisa lihat [di sini](https://docs.gitlab.com/runner/executors/#i-am-not-sure) ... saya memilih docker biar isolated dan mudah maintenance

```sh
> curl -fsSL https://get.docker.com -o get-docker.sh
> sh get-docker.sh
> systemctl start docker
```

&nbsp;
### 2.b) Install Gitlab-Runner
kebetulan saya install gitlab-runner di Debian dengan spec server 48 Core CPU 128 GB RAM ... kenthirrr  :joy:  :joy:

biar gampang maintenance gitlab-runner nantinya, saya install sebagai package repo saja
```sh
> curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash

# Pengguna Debian buster harus disable skel untuk mencegah error No such file atau directory Job failures
> export GITLAB_RUNNER_DISABLE_SKEL=true;

> sudo -E apt install gitlab-runner
```
anddd ... donee, gimana? gampang to ... cuman 3 baris perintah doang  :thumbsup:  :sunglasses:

:warning: **Note** : spec yang di butuhkan menjalankan gitlab-runner + docker sangat kecil ... semakin tinggi semakin bagus

&nbsp;
### 2.c) Register Gitlab-Runner
Tahap selanjutnya, Gitlab-Runner harus di daftarkan ke Gitlab, agar nantinya Gitlab bisa kenalan dan menggunakan gitlab-runner di server ini

sebelum itu, dapetin token untuk `runner dan URL` nya di `settings --> CI / CD --> runners`

![Get Token](https://assets.digitalocean.com/articles/gitlab_pipeline1804/step2anew.png)

langsung masuk ke CLI, buat daftarin runner di server agar bisa nyambung ke gitlab ... 

jangan lupa kalo copas sambil di lihat scriptnya  :stuck_out_tongue_closed_eyes: , sesuaikan `url` dan `gitlab-runner token` nya

```sh
> gitlab-runner register -n \
--url < URL > \
--registration-token < GITLAB-RUNNER TOKEN > \
--executor docker \
--description "Mantan Jelek" \
--docker-image "docker:stable" \
--tag-list mantan-jelek \
--request-concurrency 30 \
--limit 30 \
--docker-privileged
```

kalau uda, nanti di gitlab akan muncul beginian, menandakan kalau runner di server sudah tersambung ke gitlab dan sudah siap di gunakan oleh CI / CD job

![runner success](https://assets.digitalocean.com/articles/gitlab_pipeline1804/step2b.png)

&nbsp;

&nbsp;

&nbsp;

## 3) Mencoba Gitlab-Runner
---
nah tadi kan uda proses install executor, gitlab-runner sama registrasi nya ... sekarang cara gunain runner nya begimaneeeee ? 

sabarrr ... kasih istirahat otak dulu nape  :sleepy:

&nbsp;
### 3.a) tagging

buat gunain gitlab-runner yang uda di setting tadi, bisa dengan ngasi atribut `tags` pada file `.gitlab-ci.yml` di bagian job atribut nya ... contoh nya seperti di bawah di mana tadi saya setting tag runner nya dengan nama `mantan-jelek`

```yaml
build:
  image: docker:latest
  stage: publish
  tags:
    - mantan-jelek
  services:
    - docker:dind
  script:
    - docker build -t $TAG_COMMIT -t $TAG_LATEST .

    # push to Gitlab container registry instead of docker hub
    - docker login -u gitlab-ci-token -p $CI_BUILD_TOKEN $CI_REGISTRY
    - docker push $TAG_COMMIT
    - docker push $TAG_LATEST
  only:
  	# regex for semantic version (vx.x.x)
    - /^v([0-9]+)\.([0-9]+)\.([0-9]+)$/
  except:
    - branches
```

&nbsp;
### 3.b) Untagged Jobs

cara yang kedua adalah dengan cara aktifkan mode untagged-jobs ... berguna untuk jobs yang tidak punya / tidak di set `tags`

kalau opsi ini tidak di aktifkan, maka status pipeline job akan `pause stuck` (jika shared runner tidak ada / di nonaktifkan)

untuk mengaktifkan, tinggal masuk ke settingan runner yang tadi dan centang seperti di gambar

![Untagged Jobs](https://images3.programmersought.com/728/97/97013646fb326db705cc74362f6abce0.png)

:warning: **Note** : `tags` di gitlab CI / CD berbeda dengan `git tag` ... `tags` di CI / CD di gunakan untuk memilih runner yang di gunakan untuk mengeksekusi job

&nbsp;

&nbsp;

&nbsp;

## 4) TroubleShoot `"Cannot Connect to Docker Daemon"`?
---
akhirnyaaa semua sudah berhasil, tetapi saat proses packaging dari source code menjadi docker image, ada error yang terjadi, dimana executor tidak bisa mengkontak socket dari docker-machine API   :scream:  :scream: ... 

cobaan apa lagi ini gustiiiii  :weary:  :weary: ....

setelah bertapaa selama 5 menit, dapat wangsit petunjuk jugaaa  :joy:

ternyata cara solve nya sangat mudah, tinggal binding `docker.sock volume` di runner yang telah di set tadi ... edit file `/etc/gitlab-runner/config.toml`

```toml
[runners.docker]
  xxx
  xxx
  volumes = ["/cache"]
  xxx
```

tambahkan docker socket di bagian volumes, menjadi
```toml
volumes = ["/var/run/docker.sock:/var/run/docker.sock", "/cache"]
```

![gitlab pipeline](https://res.cloudinary.com/www-talvbansal-me/image/upload/v1583495330/posts/gitlab-ci-pipelines.png)

dan PIPELINE pun berjalan lancarrrr, YEAY  :relieved:

:warning: **Note** : saat menjalankan command `gitlab-runner register`, konfigurasi akan otomatis di buat pada dir `/etc/gitlab-runner/config.toml`

