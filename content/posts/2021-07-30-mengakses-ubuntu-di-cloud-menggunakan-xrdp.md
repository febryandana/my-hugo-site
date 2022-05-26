---
title: "Mengakses Ubuntu Di Cloud Menggunakan Xrdp"
date: 2021-07-30
author : "Feb"
description : "Kita bisa mengakses Ubuntu server yang ada di cloud seperti AWS, GCP, DigitalOcean, dll dengan menggunakan GUI daripada melalui SSH seperti yang biasa digunakan."
toc : true
---

Kita bisa mengakses Ubuntu server yang ada di cloud seperti AWS, GCP, DigitalOcean, dll dengan menggunakan GUI daripada melalui SSH seperti yang biasa digunakan. Manfaatnya? Tidak begitu banyak. Justru cenderung lebih banyak kekurangan yang perlu diperhatikan seperti tambahan beban kerja yang tidak diperlukan, tambahan aplikasi-aplikasi yang mungkin akan jarang dipakai, dan beban jaringan yang lebih berat karena harus mengirim lebih banyak data.

Tapi bukan berarti GUI untuk Ubuntu server tidak berguna. Kalian bisa memanfaatkan Ubuntu GUI di cloud tersebut sebagai pengganti desktop, untuk menyimpan data jauh lebih banyak, atau memanfaatkannya untuk mengunduh file-file besar menggunakan jaringan internet dari cloud provider yang lebih cepat dan stabil. Bisa juga digunakan untuk melakukan administrasi server dengan aplikasi versi GUI yang lebih user friendly.

Pada artikel ini, penulis menggunakan OS Ubuntu Server 20.04 LTS yang dipasang di AWS EC2. Tidak ada perhatian khusus selain layanan Security Group pada EC2 sehingga perintah-perintah ini dapat direplikasi untuk sistem lain yang berbasis Ubuntu. Desktop Environment yang digunakan adalah [LXDE](https://lxde.org/) dengan pertimbangan ukurannya yang kecil dan ringan tetapi memiliki layanan yang lengkap. Kalian juga bisa memasang Desktop Environment lainnya atau memilih jalur menggunakan Window Manager.

Untuk menghubungkan komputer dengan Ubuntu LXDE di cloud, penulis menggunakan [xrdp](https://github.com/neutrinolabs/xrdp). Aplikasi xrdp adalah implementasi open source dari layanan RDP (Remote Desktop Protocol) milik Microsoft. Fitur dari xrdp antara lain :

1. Terhubung dengan desktop Linux dari manapun
2. Reconnect ke sesi yang sudah ada
3. Mengubah ukuran session
4. Dapat berperan sebagai RDP/VNC proxy
5. Two-way clipboard (text, bitmap, file)
6. Audio & microphone redirection
7. Drive redirection

## Persiapan

1. Pastikan untuk membuka akses pada port TCP/3389. Hal ini dapat dilakukan pada Security Group di EC2 (kalian bisa menyesuaikan dengan cloud provider kalian sendiri). Pastikan juga untuk membuka akses port tersebut jika menggunakan firewall di dalam Ubuntu (ufw, iptables, dsb).
2. Pastikan sisa storage yang tersedia cukup untuk melakukan instalasi LXDE dan aplikasi bawaannya. Setidaknya siapkan ruang sebesar 2 GB.
3. Siapkan waktu dan teh karena proses download dan instalasinya dapat memakan waktu.
4. Untuk terhubung dengan Ubuntu Desktop di cloud, kita perlu memasang RDP/VNC client seperti Remmina di komputer client.

## Langkah-Langkah

1. Masuk ke Ubuntu Server melalui koneksi SSH
2. Pastikan user kalian memiliki password. Jika belum ada, berikan password dengan perintah :  

    ```bash
    sudo passwd $USER
    ```

3. Melakukan update untuk memperbarui software dan menambal keamanan Ubuntu  

    ```bash
    sudo apt-get update && sudo apt-get upgrade -y
    ```

4. Melakukan instalasi LXDE  

    ```bash
    sudo apt-get install lxde -y
    ```

5. Minum teh dan lakukan pemanasan singkat sembari menunggu instalasi berlangsung
6. Setelah instalasi LXDE selesai, lakukan instalasi xrdp.  

    ```bash
    sudo apt-get install xrdp -y
    ```

7. That's it. Setelah langkah-langkah di atas selesai dan sukses dijalankan, kita bisa langsung mengakses versi GUI-nya melalui RDP/VNC client.

## Mengakses Ubuntu Desktop di Cloud

1. Buka aplikasi Remmina atau RDP/VNC client lainnya.
2. Gunakan public IP Address atau public DNS milik Ubuntu Server untuk terhubung ke Remmina
![access remmina](/posts/images/2021-07-30-mengakses-ubuntu-di-cloud-menggunakan-xrdp/images_1.png)

3. Masukkan user dan password user yang digunakan oleh Ubuntu Server
![input user password](/posts/images/2021-07-30-mengakses-ubuntu-di-cloud-menggunakan-xrdp/images_2.png)

4. Selesai. Kalian telah masuk ke Ubuntu Server dengan mode GUI.
![access file manager via xrdp](/posts/images/2021-07-30-mengakses-ubuntu-di-cloud-menggunakan-xrdp/images_3.png)

5. Akses terminal dari dalam xRDP.
![access terminal via xrdp](/posts/images/2021-07-30-mengakses-ubuntu-di-cloud-menggunakan-xrdp/images_4.png)

## Kesimpulan

Memasang antarmuka grafis untuk Ubuntu Server di Cloud cukup sederhana dan mudah. Kita bisa memanfaatkan mode GUI di Ubuntu Server sebagai cloud PC. Jika ingin melakukan administrasi server dengan antarmuka grafis, penulis lebih menyarankan penggunaan layanan administrasi berbasis web seperti Webmin dan Cockpit.

## Referensi

- linuxize.com. How to Install Xrdp Server (Remote Desktop) on Ubuntu 20.04. Diakses 30 Juli 2021, dari <https://linuxize.com/post/how-to-install-xrdp-on-ubuntu-20-04/>
- Stas, C. (2021). How to Install GUI on Ubuntu Server [Beginner's Guide]. Diakses 30 Juli 2021, dari <https://itsfoss.com/install-gui-ubuntu-server/>
- www.australtect.net. How To Enable GUI On AWS EC2 Ubuntu server. Diakses 30 Juli 2021, dari <https://www.australtech.net/how-to-enable-gui-on-aws-ec2-ubuntu-server/>
