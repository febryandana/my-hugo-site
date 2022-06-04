---
title: "Instalasi Jitsi Meet Sebagai Alternatif Open Source Video Conference"
date: 2021-10-27
author : "Feb"
description: "Di masa modern sekarang, terutama saat pandemi COVID-19 seperti ini, banyak instansi swasta, pemerintah, maupun pendidikan yang terpaksa untuk meniadakan interaksi tatap muka dan mengharuskan semuanya dilakukan dari rumah (Work from Home, School from Home). Salah satu tool/aplikasi yang paling populer digunakan adalah online video conference untuk menggantikan rapat atau pertemuan tatap muka secara online."
tags: ['cloud', 'jitsi', 'server']
---

![8783e16f83f7af4d1aea66721add09e7.png](/posts/images/2021-10-27-instalasi-jitsi-meet/c91a67c63b9c48f8aa01de9200ad8097.png)

Di masa modern sekarang, terutama saat pandemi COVID-19 seperti ini, banyak instansi swasta, pemerintah, maupun pendidikan yang terpaksa untuk meniadakan interaksi tatap muka dan mengharuskan semuanya dilakukan dari rumah (Work from Home, School from Home). Salah satu tool/aplikasi yang paling populer digunakan adalah online video conference untuk menggantikan rapat atau pertemuan tatap muka secara online. Ada banyak sekali layanan yang menyediakan fitur online video conference seperti Google Meet, Zoom, Skype, Microsoft Teams, Discord, dan lain-lain.

Sayangnya sebagian besar layanan tersebut sifatnya komersial dan closed source sehingga fitur-fiturnya terbatas atau mengharuskan untuk membayar biaya langganan. Oleh karena itu mulai banyak yang melirik alternatif lain yang lebih luwes dengan fitur melimpah tanpa harus membayar biaya langganan. Salah satunya alternatif tersebut adalah [Jitsi Meet](https://jitsi.org/jitsi-meet/), sebuah layanan Video Conference yang 100% gratis dan open source dari Jitsi. Jitsi Meet memiliki banyak sekali fitur yang menarik seperti autentikasi berbasis SIP dan LDAP, fitur livestreaming ke Youtube atau video streaming lainnya, end-to-end encryption, akses tanpa batas waktu, dan lain-lain. Karena merupakan produk open source, Jitsi Meet juga menyediakan paket instalasi yang dapat dipasang di server milik sendiri dengan custom domain. Salah satu contohnya adalah [meet.jit.si](https://meet.jit.si/) yang sudah disediakan oleh Jitsi sendiri.

Kali ini kita akan mencoba melakukan instalasi Jitsi Meet pada server milik sendiri dengan custom domain. Beberapa hal yang perlu diperhatikan antara lain :

- **Spesifikasi mesin**. Karena berfungsi sebagai video conference, Jitsi Meet membutuhkan spesifikasi server yang cukup kuat untuk dapat menampung beban audio dan video dari user. Semakin banyak user maka bebannya juga akan semakin berat sehingga butuh spesifikasi sistem yang semakin tinggi.

- **Custom domain**. Jitsi Meet dapat digunakan tanpa custom domain, tetapi ini menyebabkan kita tidak bisa memanfaatkan fitur SSL untuk HTTPS dan keamanan end-to-end encryption. Penulis sangat merekomendasikan untuk menyiapkan custom domain atau subdomain khusus untuk Jitsi Meet. Kita juga perlu mengalokasikan IP public static dengan Elastic IP agar tautan IP Address ke domain tidak berubah-ubah.

- **Firewall**. Berikut adalah port yang perlu dibuka atau diberi izin akses agar Jitsi Meet dapat bekerja dengan baik :
  ![1baebf24a92ffcc81cab0e4ee5831ffd.png](/posts/images/2021-10-27-instalasi-jitsi-meet/91988882a2c6475398270f1c42f702c2.png)

Pada artikel ini kita akan mencoba melakukan instalasi Jitsi Meet pada sistem operasi Ubuntu Server 18.04 dari AWS EC2 Instance. Panduan instalasi ini juga dapat digunakan untuk Ubuntu Server 20.04. Jika belum tahu bagaimana cara membuat EC2 Instance, kalian bisa membaca [Membuat EC2 Instance pada AWS](https://febryandana.xyz/posts/2021-08-18-membuat-ec2-instance-di-aws/).

## Instalasi Jitsi Meet pada Ubuntu Server 18.04 AWS

Untuk memudahkan proses, semua domain yang ada di bawah menggunakan meet.yourdomain.com. Kalian harus mengubahnya ke domain kalian sendiri.

Pertama-tama kita perlu memperbarui sistem Ubuntu kita dengan,

```bash
sudo apt-get update && sudo apt-get upgrade -y
```

Selanjutnya ubah hostname dari Ubuntu Server.

```bash
sudo hostnamectl set-hostname meet.yourdomain.com
```

Edit file /etc/hosts dengan menambahkan IP public server dan subdomain meet.yourdomain.com

```bash
sudo nano /etc/hosts
```

```bash
127.0.0.1 localhost
IP_PUBLIC_SERVER meet.yourdomain.com
```

Simpan dengan `Ctrl+S` dan keluar dengan `Ctrl+X`.

Install beberapa dependencies yang perlu dipersiapkan untuk menggunakan Jitsi Meet.

```bash
sudo apt install -y apt-transport-https
```

Selanjutnya tambahkan Jitsi ke repository Ubuntu Server dengan perintah :

```bash
echo 'deb https://download.jitsi.org stable/' | sudo tee \ /etc/apt/sources.list.d/jitsi-stable.list
wget -qO - https://download.jitsi.org/jitsi-key.gpg.key | sudo apt-key add -
sudo apt-get update
```

Setelah semuanya selesai selanjutnya kita akan mulai melakukan instalasi Jitsi Meet.

```bash
sudo apt install -y jitsi-meet
```

Proses instalasi akan berlangsung. Jitsi Meet akan diinstall beserta dengan dependencies yang diperlukan.

Jika muncul dialog masukan untuk hostname, masukkan meet.yourdomain.com (ganti dengan domain kalian). Pastikan untuk mengetik hostname dengan benar tanpa kesalahan.

Ketika muncul dialog tentang certification, pilih self-signed certification karena selanjutnya kita akan memanfaatkan Let's Encrypt untuk mendapatkan SSL certification.

Setelah proses instalasi selesai, selanjutnya kita akan membuat self-signed SSL certificate dengan memanfaatkan Let's Encrypt.

```bash
sudo /usr/share/jitsi-meet/scripts/install-letsencrypt-cert.sh
```

Saat diminta, masukkan hostname domain Jitsi Meet dan email kalian lalu tunggu proses sertifikasi berjalan hingga selesai.

Selanjutnya kita dapat meningkatkan system limit dengan mengedit file `/etc/systemd/system.conf` :

```bash
sudo nano /etc/systemd/system.conf
```

Pergi ke bagian bawah file lalu tambahkan baris berikut :

```bash
DefaultLimitNOFILE=65000
DefaultLimitNPROC=65000
DefaultTasksMax=65000
```

Restart service dengan perintah :

```bash
sudo systemctl daemon-reload
sudo systemctl restart {prosody,jicofo,jitsi-videobridge2,nginx}
```

Sampai tahap ini Jitsi Meet telah selesai di-install. Untuk mengujinya, buka browser dan pergi ke <https://meet.yourdomain.com>. Sekarang Jitsi Meet kita sudah siap digunakan, tetapi masih terdapat beberapa fitur yang belum tersedia seperti fitur record, livestreaming, dan autentikasi user. Panduan penambahan fitur record, autentikasi, dan kustomisasi home page akan dipecah ke dalam artikel selanjutnya.

### Referensi

- [Github Jitsi Meet](https://github.com/jitsi/jitsi-meet)
- [Jitsi Meet Website](https://jitsi.org/jitsi-meet/)
- [Jitsi Wikipedia entry](https://en.wikipedia.org/wiki/Jitsi)
- Saiflakhani, S. (2021, February 24). [Setup / Guide] Jitsi Meet Native + Multiple (6) Jibri Docker instances working on the same AWS server. Community.Jitsi.Org. <https://community.jitsi.org/t/setup-guide-jitsi-meet-native-multiple-6-jibri-docker-instances-working-on-the-same-aws-server/>
