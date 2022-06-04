---
title: "Kelola Server Lewat Website Dengan Cockpit"
date: 2021-08-20
author : "Feb"
description : "Tools ini bertujuan untuk memudahkan *sysadmin* mengelola sistem linux mereka tanpa ribet dengan urusan SSH key, salah ketik perintah, atau untuk mengurus banyak server sekaligus tanpa perlu berpindah-pindah terminal SSH."
tags : ['cloud', 'management', 'server']
---

![cockpit cover image.png](/posts/images/2021-08-20-kelola-server-lewat-website-dengan-cockpit/2c06837d29c94085b50b4a412092dd57.png)

Kebanyakan system administrator mengelola server mereka melalui koneksi SSH dengan antarmuka CLI, bukan GUI. Meskipun umum digunakan, tetap ada sebagian kecil orang yang merasa bahwa *command line interface* itu susah digunakan, dan merasa mengintimidasi. Karena salah satu karakter saja bisa menyebabkan server crash.

Tapi bukan berarti tidak ada jalan lain. Karena sekarang sudah banyak alat-alat manajemen server yang berbasis grafis dan mudah digunakan. Contohnya seperti [Webmin](https://webmin.com/) dan [Cockpit](https://cockpit-project.org/). Tools ini bertujuan untuk memudahkan *sysadmin* mengelola sistem linux mereka tanpa ribet dengan urusan SSH key, salah ketik perintah, atau untuk mengurus banyak server sekaligus tanpa perlu berpindah-pindah terminal SSH.

Pada artikel ini, kita akan mencoba untuk melakukan instalasi Cockpit di Ubuntu Server yang ada di AWS. Jika kalian belum tahu bagaimana cara membuat virtual machine Ubuntu Server di AWS, kalian dapat melihat artikel [Membuat EC2 Instance pada AWS](https://febryandana.xyz/posts/2021-08-18-membuat-ec2-instance-di-aws/) terlebih dahulu.

## Instalasi Paket Cockpit

Instalasi Cockpit sangat sederhana. Hanya ada satu package yang perlu dipasang dan untungnya, paket instalasi Cockpit sudah tersedia di hampir semua package manager. Yang perlu diperhatikan adalah bahwa Cockpit secara default memakai port TCP/9090 untuk akses ke website-nya sehingga kita perlu membuka firewall untuk port tersebut.

Jika menggunakan AWS EC2 Instance, kita bisa memodifikasi Security Group yang terhubung dengan Instance tempat Cockpit berada. Jika menggunakan aplikasi firewall seperti UFW, kita harus menambahkan aturan baru untuk mengizinkan akses masuk ke port TCP/9090 dengan perintah :

```bash
sudo ufw allow 9090
```

Paket instalasi Cockpit sudah tersedia dalam repositori Ubuntu sehingga untuk memasangnya kita cukup memberikan perintah :

```bash
sudo apt-get install -y cockpit
```

Jika kalian menggunakan sistem lain, Cockpit juga sudah menyediakan panduan instalasi untuk banyak sistem operasi yang bisa dilihat di [cockpit-project.org/running](https://cockpit-project.org/running.html). Cockpit sudah mendukung sebagian besar distro Linux seperti Debian, Red Hat Linux Enterprise, Fedora, Arch, dll.

Setelah sistem menyelesaikan instalasi, cockpit dapat langsung diakses di <http://SERVER_PUBLIC_IP:9090> atau di <http://YOUR_DOMAIN.COM:9090>.

> SERVER_PUBLIC_IP dan YOUR_DOMAIN.COM hanya contoh, gunakan IP Address atau domain yang ada di server Cockpit kalian sendiri.

### Tampilan Cockpit

![Cockpit login page.png](/posts/images/2021-08-20-kelola-server-lewat-website-dengan-cockpit//ab2df571a79d42ec83cf274558ecad8e.png)

Gambar di atas adalah tampilan login page Cockpit ketika pertama kali diakses. Untuk masuk ke dalam Cockpit kita perlu memberikan user name dan password sistem.
Dalam beberapa kasus seperti Instance di AWS, secara default kita tidak memiliki password untuk user sistem. Untuk itu kita bisa memberikan sendiri password-nya dengan perintah:

```bash
sudo passwd $USER
```

Perintah ini berguna untuk mengganti password dari user. Setelah password selesai dibuat, kita bisa masuk Cockpit menggunakan username dari `$USER` dan password yang baru kita buat.

![Cockpit Overview menu.png](/posts/images/2021-08-20-kelola-server-lewat-website-dengan-cockpit/50acd4ee0d8d42168cc253331ca996e1.png)

Ketika kita sudah masuk, kita akan disambut menu **Host** yang berisi layanan-layanan dari Cockpit seperti Overview, Logs, Storage, dan sebagainya yang akan dibahas lebih lanjut. Di bawah menu Host terdapat menu Dashboard yang berisi grafik penggunaan secara real time dari komponen CPU, Memory, Network, dan Disk I/O. Desain antarmuka pengguna dari Cockpit cukup sederhana dan intuitif sehingga mudah digunakan bahkan oleh pemula sekalipun.

![Cockpit Dashboard.png](/posts/images/2021-08-20-kelola-server-lewat-website-dengan-cockpit/1d26cba15ed044d98746b89af6c069c1.png)

Salah satu kelebihan Cockpit adalah kita bisa menambahkan server lain ke dalam Cockpit sehingga satu halaman Cockpit saja bisa digunaka untuk mengelola semua server yang kita miliki. Kita akan mencoba fitur ini di bagian akhir artikel.

## Daftar Layanan Cockpit

### Overview

![Cockpit Overview menu.png](/posts/images/2021-08-20-kelola-server-lewat-website-dengan-cockpit/50acd4ee0d8d42168cc253331ca996e1.png)

Pada bagian Overview kita bisa melihat ringkasan dari sistem seperti kesehatan sistem, penggunaan CPU dan Memory, informasi sistem, dan konfigurasi sistem

### Logs

![Cockpit Logs menu.png](/posts/images/2021-08-20-kelola-server-lewat-website-dengan-cockpit/39daa581332444b4b7eb01f82801a304.png)

Pada bagian Logs ditampilkan daftar error, peringatan, dan informasi log penting lainnya dari sistem. Kita dapat melihat logs dari range waktu tertentu

### Storage

![Cockpit Storage menu.png](/posts/images/2021-08-20-kelola-server-lewat-website-dengan-cockpit/9c749319449c4ce2af9e8dc560092891.png)

Pada bagian Storage ditampilkan daftar perangkat dan drive storage yang terhubung, detail filesystem dan sisa penggunaan storage, informasi logs dari storage, serta grafik real time usage dari aksi Baca/Tulis perangkat storage.

### Networking

![Cockpit Networking menu.png](/posts/images/2021-08-20-kelola-server-lewat-website-dengan-cockpit/11aa2311b1924b7ab3206e88221e30c6.png)

Di bagian ini, kita bisa melihat daftar perangkat kartu jaringan dan grafik dari trafik penggunaan jaringan (send/receive). Kita juga bisa melihat informasi logs tentang status jaringan dari sistem.

### Accounts

![Cockpit Accounts menu.png](/posts/images/2021-08-20-kelola-server-lewat-website-dengan-cockpit/4abbc7f9df364e52ada4b42e373c0d41.png)

Di bagian Account, terdapat daftar akun yang ada di dalam sistem. Di dalamnya kita bisa memodifikasi beberapa pengaturan akun seperti Role administrator, mengunci akses akun, mengganti password, dan lain-lain.

### Services

![Cockpit Services menu.png](/posts/images/2021-08-20-kelola-server-lewat-website-dengan-cockpit/a84a101024db4a588ad0b610565ad759.png)

Bagian Services menampilkan daftar services yang aktif maupun non-aktif yang ada di dalam sistem. Disini kita bisa mengatur status services, mengatur automatic startup, menambahkan/menghapus path, dan lain-lain.

### Applications

![Cockpit Applications menu.png](/posts/images/2021-08-20-kelola-server-lewat-website-dengan-cockpit/1036151cd284426eba04dd2fe5bdfd25.png)

Bagian Applications menampilkan daftar plugin aplikasi yang dapat digunakan oleh Cockpit. Kita bisa menambahkan atau mematikan aplikasi Cockpit melalui menu ini. Daftar aplikasi Cockpit yang tersedia dapat dilihat di [cockpit-project.org/applications](https://cockpit-project.org/applications.html).

### Software Updates

![Cockpit Software Updates menu.png](/posts/images/2021-08-20-kelola-server-lewat-website-dengan-cockpit/ee5db2bf08194c97ac1d4ed7637f8cad.png)

Kita bisa melakukan update sistem secara keseluruhan maupun per software melalui menu Software Updates.

### Terminal

![Cockpit Terminal menu.png](/posts/images/2021-08-20-kelola-server-lewat-website-dengan-cockpit/e8dc117870a84831bc89c81dec677a65.png)

Selain kontrol menggunakan GUI, Cockpit juga memiliki built-in terminal untuk mengelola sistem melalui command line. Dengan adanya built-in terminal, kita tidak perlu menggunakan SSH atau tool komunikasi lain untuk mengakses server, semuanya cukup menggunakan satu halaman Cockpit. Dengan terminal ini kita bisa melakukan apa saja ke dalam sistem seperti memasang aplikasi baru, mengganti file konfigurasi, manipulasi teks atau file, dan lain-lain.

## Mengelola Server Lain Melalui Cockpit

Selanjutnya kita akan mencoba menambahkan server baru untuk dikelola dalam satu Cockpit. Untuk mengakses server lain, Cockpit memanfaatkan layanan SSH dengan menggunakan proses `cockpit-bridge`. Cockpit mengakses server lain melalui *password based authentication* sehingga kita harus mengaktifkan opsi tersebut terlebih dahulu. Untuk mengaktifkan password based authentication, kita cukup memodifikasi satu baris di `/etc/ssh/sshd_config` :

```bash
sudo nano /etc/sshd/sshd_config
```

Cari baris "`PasswordAuthentication no`" dan ubah menjadi "`PasswordAuthentication yes`" (tanpa tanda kutip). Simpan file dengan `Ctrl+S` dan keluar dengan `Ctrl+X` . Kemudian restart service SSHD dengan perintah :

```bash
sudo systemctl restart sshd
```

Tahap selanjutnya akan dilakukan melalui website Cockpit di server utama kita. Pertama kita masuk ke dalam Dashboard Cockpit, lalu klik tombol "+" biru di sebelah kanan atas dari tabel Servers.

![Cockpit Dashboard.png](/posts/images/2021-08-20-kelola-server-lewat-website-dengan-cockpit/1d26cba15ed044d98746b89af6c069c1.png)

Akan muncul dialog *Add Machine to Dashboard*. Masukkan IP Address atau hostname dari mesin yang akan kita kelola,pilih warna identitasnya, lalu klik *Add*. Jika semua server yang akan dikelola berada dalam satu jaringan yang sama, lebih baik gunakan hostname atau IP Address lokal untuk mengurangi latency dan beban jaringan ke luar (internet).

![Add machine to dashboard.png](/posts/images/2021-08-20-kelola-server-lewat-website-dengan-cockpit/d9fbf8fd379743a187c32012abd590dc.png)

Selanjutnya akan muncul dialog *Log in to server*. Masukkan username server lain, ubah autentikasinya menggunakan password, lalu masukkan password user server lain tersebut dan klik *Log in*.

![Log in to server.png](/posts/images/2021-08-20-kelola-server-lewat-website-dengan-cockpit/596cbb941e004cfbac6e0e6c8e0bcf59.png)

Jika proses *Log in* berhasil, seharusnya akan muncul item berupa hostname dari server baru tersebut di tabel Servers. Grafik penggunaannya juga akan muncul di bagian grafik dengan warna yang sudah ditentukan sebelumnya.

![Servers table.png](/posts/images/2021-08-20-kelola-server-lewat-website-dengan-cockpit/67d2fa89a9a4453c9060f03f3a291cf6.png)

Kita bisa memulai mengelola server lain dengan menekan hostname di menu Host lalu memilih hostname server lain.

![Cockpit host list.png](/posts/images/2021-08-20-kelola-server-lewat-website-dengan-cockpit/65f3d1fc0610456fafaef1453c9f1a16.png)

Kita bisa menambahkan sistem server lain sebanyak yang kita mau selama semua server tersebut berhasil dipasang paket Cockpit tanpa ada masalah perbedaan sistem operasi. Cockpit cukup dapat diandalkan baik oleh pemula maupun sysadmin yang sudah berpengelaman untuk mengelola sistem server. Dengan Cockpit, kita bisa mengelola banyak remote server secara sekaligus dalam satu tempat tanpa perlu berganti-ganti perangkat atau SSH. Kita bisa mengelola banyak hal dengan memanfaatkan Cockpit seperti memasang aplikasi baru, mengecek load penggunaan CPU dan Memory, mengubah status services, reboot dan shutdown server, dan lain sebagainya.

## Referensi

- [Halaman Web Cockpit Project](https://cockpit-project.org)
- [Repositori Github Cockpit](https://github.com/cockpit-project/cockpit)
