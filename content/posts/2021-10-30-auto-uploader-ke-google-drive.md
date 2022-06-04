---
title: "Auto Uploader File Record Jibri ke Google Drive"
date: 2021-10-30
author : "Feb"
description: "Ada banyak cara untuk bisa mengunggah file rekaman ke cloud storage mulai dari sync folder, FTP/File server, memanfaatkan layanan AWS S3 jika server Jibri berada di AWS, atau menggunakan rclone untuk menyambungkan server kita dengan layanan cloud storage lain. Pada artikel ini, kita akan memanfaatkan rclone untuk mengunggah file rekaman Jibri ke layanan Google Drive."
tags: ['cloud', 'jitsi', 'server']
---

Setelah Jibri berhasil dipasang, tentunya kita ingin untuk bisa mengakses file-file hasil rekaman tersebut. Secara default, Jibri menyimpan hasil rekamannya di direktori `/srv/recordings` tanpa ada fitur untuk mengunggahnya ke layanan cloud storage.
Ada banyak cara untuk bisa mengunggah file rekaman ke cloud storage mulai dari sync folder, FTP/File server, memanfaatkan layanan AWS S3 jika server Jibri berada di AWS, atau menggunakan rclone untuk menyambungkan server kita dengan layanan cloud storage lain. Pada artikel ini, kita akan memanfaatkan rclone untuk mengunggah file rekaman Jibri ke layanan Google Drive.

## 1. Membuat file konfigurasi rclone di host machine

Pada artikel ini, penulis menggunakan PC pribadi sebagai host machine dengan sistem operasi Fedora 34. Kalian bisa menyesuaikan cara instalasi rclone sesuai dengan sistem operasi kalian sendiri.
Paket rclone sudah tersedia di repository Fedora 34 sehingga kita hanya perlu memberi perintah:

```bash
sudo dnf upgrade -y
sudo dnf install -y rclone
```

Setelah proses instalasi selesai, kita lanjutkan dengan proses membuat file konfigurasi, prosesnya adalah sebagai berikut:

```bash
rclone config
```

Cara konfigurasi rclone untuk Google Drive bisa kalian baca langsung [disini](https://rclone.org/drive/).

## 2. Mengirimkan file konfigurasi tersebut ke server Jibri

Di server Jibri, kita juga perlu memasang paket rclone. Karena sebelumnya kita menggunakan Ubuntu Server 18.04 sebagai server Jibri, paket rclone sudah tersedia di repositori:

```bash
sudo apt-get install rclone
```

Untuk mengirimkan file rclone.conf yang sudah dibuat sebelumnya, kita bisa menggunakan layanan scp atau sftp. Misalnya menggunakan scp maka perintahnya adalah:

```bash
scp ~/.config/rclone/rclone.conf jibri@IP_PUBLIC_JIBRI_SERVER:/home/jibri/.config/rclone/rclone.conf
```

Dapat diperhatikan bahwa tujuan pengiriman kita adalah ke home direktori dari user jibri. Ini karena nanti rclone akan dijalankan oleh user jibri melalui file `finalize.sh`.

Selanjutnya untuk memeriksa apakah config file kita sudah siap, kita bisa menggunakan perintah berikut dari dalam server Jibri:

```bash
// Masuk ke user jibri yang akan menjalankan rclone:
su - jibri

// Ubah ownership dari file rclone.conf
sudo chown jibri:jibri /home/jibri/.config/rclone.rclone.conf

// Cek ketersediaan config file:
rclone config file
```

Jika tidak ada error sampai sini berarti rclone sudah siap digunakan. Kita bisa mengujinya dengan memeriksa daftar file yang ada di Google Drive:

> Parameter `googledrive:` yang digunakan adalah nama config yang kita buat di rclone sebelumnya. Jika Anda membuat config dengan nama yang berbeda, ganti `googledrive:` sesuai dengan nama config yang Anda miliki.

```bash
rclone ls googledrive:
```

Untuk mengirim file dari server ke Google Drive, menggunakan perintah:

```bash
rclone copy /FILE/PATH googledrive:
```

Untuk mengirim file ke folder tertentu di Google Drive, bisa menggunakan perintah:

```bash
rclone copy /FILE/PATH googledrive:FOLDER/PATH/
```

## 3. Membuat file finalize.sh untuk menjalankan rclone secara otomatis

Setelah rclone selesai dikonfigurasi, selanjutnya kita perlu menyiapkan file finalize.sh untuk dijalankan Jibri setiap selesai rekaman. Path dari file ini sesuai dengan apa yang sudah kita tentukan di file `/etc/jitsi/jibri/jibri.conf`. Contohnya pada server Jibri sebelumnya, path file finalize.sh berada di `/srv/finalize.sh`.
Kita perlu membuat sekaligus membuka file `/srv/finalize.sh`. Kita bisa menggunakan nano, vim, atau text editor lainnya. Contoh:

```bash
sudo nano /srv/finalize.sh
```

Selanjutnya masukkan baris program di bawah ini ke dalam file finalize.sh:

```bash
#!/bin/bash

RECORDINGS_DIR=$1

VIDEO_FILE_PATH=$(find $RECORDINGS_DIR -name *.mp4)

/usr/bin/rclone copy $VIDEO_FILE_PATH googledrive:meet.yourdomain.com/ -v --log-file=/var/log/jitsi/jibri/googledrive_upload.log
```

Cara kerja program tersebut adalah dengan mengambil path direktori hasil rekaman yang baru saja dilakukan Jibri, kemudian mencari file dengan ekstensi mp4, lalu menyalin file tersebut ke dalam folder meet.yourdomain.com di Google Drive dengan bantuan aplikasi rclone. Opsi -v akan membuat rclone menampilkan output yang lebih banyak & lengkap. Selain itu, dengan opsi --log-file, rclone juga akan menyimpan informasi penggunaan dengan membuat log file  di `/var/log/jitsi/jibri/googledrive_upload.log`.

Setelah selesai, simpan dan keluar dari file finalize.sh. Selanjutnya kita perlu mengatur permission dan ownership dari file tersebut agar dapat dieksekusi oleh Jibri:

```bash
sudo chown jibri:jibri /srv/finalize.sh
sudo chmod +x /srv/finalize.sh
```

Sampai sini fitur auto uploader file rekaman Jibri ke Google Drive telah selesai. Kita bisa mengujinya dengan melakukan rekaman di Jitsi Meet. Setelah selesai, file rekamannya seharusnya diunggah ke folder meet.yourdomain.com di akun Google Drive yang sudah dipilih sebelumnya.
Pemanfaatan rclone tidak berhenti hanya sampai disini saja. Kita juga bisa menggunakan layanan cloud storage atau layanan penyimpanan file lain selain Google Drive untuk mengirim file rekaman Jibri.

### Referensi

- [Rclone](https://rclone.org/drive/)
- [How-to to get a WORKING setup of Google Drive, One Drive or other cloud services in Jibri, my comprehensive tutorial for the beginner](https://community.jitsi.org/t/how-to-to-get-a-working-setup-of-google-drive-one-drive-or-other-cloud-services-in-jibri-my-comprehensive-tutorial-for-the-beginner/42228)
