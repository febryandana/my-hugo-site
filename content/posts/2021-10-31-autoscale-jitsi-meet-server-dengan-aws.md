---
title: "Autoscale Jitsi Meet Server Dengan AWS"
date: 2021-10-31
author : "Feb"
description: "Pada artikel ini, kita akan belajar membuat autoscaling untuk Jitsi Videobridge, komponen XMPP server Jitsi yang bertanggung jawab untuk pembuatan multiuser video communication. Kita bisa menganggapnya sebagai backend dari Jitsi Meet Server yang bertugas untuk mengurus masalah video & audio di conference room."
tags: ['cloud', 'jitsi', 'server']
---

![Jitsi Diagram.png](/posts/images/2021-10-31-autoscale-jitsi-meet-server-dengan-aws/2c2a4d30688d45e089415a7283ee6e91.png)

Pada artikel ini, kita akan belajar membuat autoscaling untuk Jitsi Videobridge, komponen XMPP server Jitsi yang bertanggung jawab untuk pembuatan multiuser video communication. Kita bisa menganggapnya sebagai backend dari Jitsi Meet Server yang bertugas untuk mengurus masalah video & audio di conference room.

Langkah terbaik untuk membuat Jitsi Meet Server adalah dengan memisahkan antara Jitsi Meet, Jibri, dan Jitsi Videobridge ke dalam instance masing". Lebih baik lagi jika kita membuat autoscaling untuk videobridge (dan juga Jibri, tapi kita akan fokus pada videobridge saja sekarang), dengan begitu, resource yang perlu digunakan dapat menyesuaikan dengan kebutuhan, tidak ada yang menganggur atau kelebihan beban. Langkah-langkah pembuatan autoscaling untuk Jitsi Videobridge menggunakan Amazon Web Service adalah sebagai berikut.

## Persiapan Instance

Pertama kita perlu membuat instance baru yang akan menjadi template dari Jitsi Videobridge. Cara pembuatan instance bisa dilihat di artikel [Membuat EC2 di AWS](https://febryandana.xyz/posts/2021-08-18-membuat-ec2-instance-di-aws/). Port yang perlu dibuka di Security Group adalah port 22/TCP, 10000/UDP, 5347/TCP, dan 4443/TCP.

## Instalasi Jitsi Videobridge

Setelah instance terbuat dan kita bisa masuk ke dalam instance, selanjutnya kita perlu menambahkan repository dari Jitsi dan memasang paket instalasi jitsi-videobridge2.

```bash
# Install dependencies
sudo apt install -y apt-transport-https

# Adding jitsi repository
echo 'deb https://download.jitsi.org stable/' | sudo tee \ /etc/apt/sources.list.d/jitsi-stable.list
wget -qO - https://download.jitsi.org/jitsi-key.gpg.key | sudo apt-key add -

# Update system
sudo apt-get update

# Install jitsi-videobridge2
sudo apt install -y jitsi-videobridge2
```

## Konfigurasi Jitsi Videobridge

Tahap selanjutnya adalah melakukan sedikit tweaking pada Jitsi Meet Server dan beberapa file konfigurasi jitsi-videobridge agar dapat digunakan untuk autoscaling.

Yang pertama adalah menambahkan user baru khusus untuk videobridge melalui komponen prosodyctl pada instance Jitsi Meet Server. User ini akan digunakan sebagai autentikasi saat ada videobridge baru yang tersambung.

```bash
sudo prosodyctl register jvb auth.meet.yourdomain.com yourjvbpassword
```

Selanjutnya adalah file `sip-communicator.properties` yang ada di instance jitsi-videobridge. File ini digunakan untuk memberitahu videbridge tentang informasi-informasi dirinya sendiri dan Jitsi Meet Server yang akan dihubungi.

```bash
sudo nano /etc/jitsi/videobridge/sip-communicator.properties
```

Konten file `sip-communicator.properties` :

```bash
org.jitsi.videobridge.xmpp.user.shard.DISABLE_CERTIFICATE_VERIFICATION=true
org.ice4j.ice.harvest.DISABLE_AWS_HARVESTER=true
org.jitsi.videobridge.ENABLE_STATISTICS=true
org.jitsi.videobridge.STATISTICS_TRANSPORT=muc
org.jitsi.videobridge.xmpp.user.shard.HOSTNAME=meet.yourdomain.com
org.jitsi.videobridge.xmpp.user.shard.DOMAIN=auth.meet.yourdomain.com
org.jitsi.videobridge.xmpp.user.shard.USERNAME=jvb
org.jitsi.videobridge.xmpp.user.shard.PASSWORD=yourjvbpassword
org.jitsi.videobridge.xmpp.user.shard.MUC_JIDS=jvbbrewery@internal.auth.meet.yourdomain.com
org.jitsi.videobridge.xmpp.user.shard.MUC_NICKNAME=1cc60513-7064-4c47-afee-a838eb29a58c
org.jitsi.videobridge.DISABLE_TCP_HARVESTER=false
org.jitsi.videobridge.TCP_HARVESTER_PORT=4443
org.jitsi.videobridge.TCP_HARVESTER_MAPPED_PORT=443
org.jitsi.videobridge.TCP_HARVESTER_SSLTCP=true
org.ice4j.ice.harvest.NAT_HARVESTER_LOCAL_ADDRESS=172.0.0.1
org.ice4j.ice.harvest.NAT_HARVESTER_PUBLIC_ADDRESS=11.22.33.44
```

Berikutnya adalah file `jvb.conf` dan file `config` yang berisi konfigurasi dasar dari jitsi-videobridge.

```bash
sudo nano /etc/jitsi/videobridge/jvb.conf
```

Konten file `jvb.conf` :

```bash
videobridge {
 http-servers {
  public {
   port = 9090
  }
 }
 websockets {
  enabled = true
  domain = "meet.yourdomain.com:443"
  tls = true
 }
}
```

```bash
sudo nano /etc/jitsi/videobridge/config
```

Konten file `config` :

```bash
# Jitsi Videobridge settings

# sets the XMPP domain (default: none)
JVB_HOSTNAME=localhost

# sets the hostname of the XMPP server (default: domain if set, localhost otherwise)
JVB_HOST=

# sets the port of the XMPP server (default: 5275)
JVB_PORT=5347

# sets the shared secret used to authenticate to the XMPP server
JVB_SECRET=jvbpassword

# extra options to pass to the JVB daemon
JVB_OPTS="--apis=,"
```

Yang terakhir adalah konfigurasi file `jvb.sh` yang akan dijalankan saat jitsi-videobridge pertama kali aktif (pertama kali booting). Buka file `jvb.sh` dan tambahkan baris berikut di bagian atas file.

```bash
sudo nano /usr/share/jitsi-videobridge/jvb.sh
```

Konten file `jvb.sh` :

```bash
RAND=`openssl rand -hex 5`
USERNAME=jvb
BRIDGE=bridge$RAND
JVB_SECRET=yourjvbpassword
NICKNAME=`uuidgen`
LOCAL_ADDRESS=`hostname -I | awk '{print $1}'`
PUBLIC_ADDRESS=`curl ifconfig.me`
sed -i "s/\(JVB_SECRET=\).*\$/\1${JVB_SECRET}/" /etc/jitsi/videobridge/config
sed -i "s/\(org.jitsi.videobridge.xmpp.user.shard.USERNAME=\).*\$/\1${USERNAME}/" /etc/jitsi/videobridge/sip-communicator.properties
sed -i "s/\(org.jitsi.videobridge.xmpp.user.shard.PASSWORD=\).*\$/\1${JVB_SECRET}/" /etc/jitsi/videobridge/sip-communicator.properties
sed -i "s/\(org.jitsi.videobridge.xmpp.user.shard.MUC_NICKNAME=\).*\$/\1${NICKNAME}/" /etc/jitsi/videobridge/sip-communicator.properties
sed -i "s/\(org.ice4j.ice.harvest.NAT_HARVESTER_LOCAL_ADDRESS=\).*\$/\1${LOCAL_ADDRESS}/" /etc/jitsi/videobridge/sip-communicator.properties
sed -i "s/\(org.ice4j.ice.harvest.NAT_HARVESTER_PUBLIC_ADDRESS=\).*\$/\1${PUBLIC_ADDRESS}/" /etc/jitsi/videobridge/sip-communicator.properties
sed -i "s/\(org.jitsi.videobridge.octo.BIND_ADDRESS=\).*\$/\1${LOCAL_ADDRESS}/" /etc/jitsi/videobridge/sip-communicator.properties
sed -i "s/\(org.jitsi.videobridge.octo.PUBLIC_ADDRESS=\).*\$/\1${PUBLIC_ADDRESS}/" /etc/jitsi/videobridge/sip-communicator.properties
sed -i "s/\(org.jitsi.videobridge.REGION=\).*\$/\1${BRIDGE}/" /etc/jitsi/videobridge/sip-communicator.properties
```

Setelah selesai, kita bisa melakukan test dengan restart service jitsi-videobridge2 dan sekaligus mengaktifkan autostart agar service tersebut selalu dijalankan setiap sistem aktif.

```bash
sudo systemctl enable jitsi-videobridge2
sudo systemctl restart jitsi-videobridge2
```

Sampai sini, tahap pembuatan instance untuk jitsi-videobridge2 telah selesai. Selanjutnya kita bisa memanfaatkan instance tersebut untuk membuat template baru untuk autoscaling.

## Pembuatan Template

Untuk membuat instance template, pertama kita perlu mematikan instance dengan klik `Instance state -> Stop instance`. Selanjutnya kita buat template dengan klik `Action -> Image and templates -> Create template from instance`.

Beri nama dan deskripsi untuk template yang akan dibuat. Lalu centang checkbox ***Provide guidance to help me set up a template that I can use with EC2 Auto Scaling*** untuk mendapat bantuan konfigurasi template yang sesuai untuk autoscaling.

Pada bagian Network Interfaces, untuk pengaturan subnet, pilih ***Specify custom value...*** dan klik ***Save*** tanpa memberikan value.

Kemudian klik *Advanced Details* untuk membuka pengaturan lanjutan. Pada bagian ***Shutdown behavior*** dan ***Stop - Hibernate behavior***, pilih opsi ***Don't include in launch template***. Setelah pengaturan sudah cukup dan tidak ada tanda merah, klik ***Create launch template*** untuk mengonfirmasi pembuatan template.

## Pembuatan Auto Scaling Group

![AWS_Auto_Scaling_Group.png](/posts/images/2021-10-31-autoscale-jitsi-meet-server-dengan-aws/6a509a1f66d14b31a748f6c1f42789a9.png)

Tahap terakhir adalah pembuatan auto scaling group. Menu auto scaling group bisa dibuka dari **EC2 Services**. Buka EC2 dari menu **All Services** atau melalui **Search Bar**, lalu pilih opsi **Auto Scaling Group** dari kolom sebelah kiri, kemudian klik **Create Auto Scaling Group**.

Berikut adalah tampilan wizard dari Create Auto Scaling Group. AWS telah memudahkan pembuatan auto scaling group dengan menyediakan urutan langkah-langkah pembuatan. Kita cukup mengikutinya dan mengisikan data-data yang perlu diatur.

![Auto_Scaling_Step_1.png](/posts/images/2021-10-31-autoscale-jitsi-meet-server-dengan-aws/2dc57b6f75a64452bf3543e8a080d619.png)

Pertama kita perlu memberi nama group dan memilih launch template yang telah kita buat sebelumnya. Pemilihan nama harus unik dalam satu akun di Region tersebut, dan tidak boleh lebih dari 255 karakter. Setelah selesai memberi nama dan launch template, klik **Next** untuk melanjutkan.

![Auto_Scaling_Step_2.png](/posts/images/2021-10-31-autoscale-jitsi-meet-server-dengan-aws/645d557cb9614fdebc264a1d00098194.png)

Pada langkah kedua, Pada bagian **Instance purchase options**, kita dapat mengatur berapa persen yang akan dibayar secara on-demand dan secara spot instances. Selanjutnya pada bagian **Network**, kita dapat mengatur VPC yang akan digunakan beserta Availability Zone dan Subnet yang akan digunakan oleh instances dalam auto scaling group. Kemudian pada bagian **Instance type requirements** kita bisa memilih untuk menggunakan instance type sesuai dengan template yang sudah dibuat atau mengubahnya ke instance type yang lain. Setelah selesai, klik **Next** untuk melanjutkan.

![Auto_Scaling_Step_3.png](/posts/images/2021-10-31-autoscale-jitsi-meet-server-dengan-aws/43d175af438141f6990d9e0a30810efe.png)

Untuk langkah ke-3 hingga step ke-6 sifatnya adalah optional. Langkah ketiga berisi opsi untuk memasangkan auto scaling group ke **load balancer**, **health check**, dan **monitoring** melalui CloudWatch.

![Auto_Scaling_Step_4.png](/posts/images/2021-10-31-autoscale-jitsi-meet-server-dengan-aws/0b37f467686b4a5c832fe60f1aeb70be.png)

Langkah ke-4 berisi opsi **Group size** untuk mengatur ukuran group (kapasitas instance yang diinginkan, kapasitas minimal, dan kapasitas maksimal). **Scaling policies** untuk mengatur bagaimana grup akan melakukan scaling, berupa Target tracking dari Average CPU Utilization, Average Network In/Out, dan jumlah request dari Application Load Balancer. Opsi **Instance scale-in protection** digunakan untuk memproteksi instance yang terbuat dari aksi scale-in (mengurangi/menghapus instance yang tak terpakai).

![Auto_Scaling_Step_5.png](/posts/images/2021-10-31-autoscale-jitsi-meet-server-dengan-aws/39c2353e09dc4d078bfbf3c8e8aee09e.png)

Langkah ke-5 berisi opsi untuk memberikan notifikasi jika ada event yang terjadi pada auto scaling group. Kita bisa mengirimkan topik SNS ke administrator jika ada event berupa : launch instance, terminate instance, Instance fail to launch, dan instance fail to terminate.

![Auto_Scaling_Step_6.png](/posts/images/2021-10-31-autoscale-jitsi-meet-server-dengan-aws/ad0c9de923ee4d1db84d96fe9b4b754e.png)

Langkah ke-6 berisi opsi penambahan tag berupa key-value untuk auto scaling group.

![Auto_Scaling_Step_7.png](/posts/images/2021-10-31-autoscale-jitsi-meet-server-dengan-aws/522ddd5659e64dd3b3ef79ae227067e6.png)

Langkah terakhir berupa review dari kumpulan konfigurasi yang telah kita buat di langkah-langkah sebelumnya. Disini kita bisa mengecek apabila ada pengaturan yang salah atau terlewatkan. Jika dirasa sudah cukup, kita bisa mengonfirmasi pembuatan auto scaling group dengan klik tombol ***Create Auto Scaling Group***.

![Auto_Scaling_Dashboard.png](/posts/images/2021-10-31-autoscale-jitsi-meet-server-dengan-aws/0e8bc3433a74437f89216cfd12e95868.png)

Gambar di atas adalah tampilan Auto Scaling Group yang telah kita buat. Kita bisa melihat informasi detail tentang auto scaling group tersebut dengan klik tanda centang dan memperhatikan daftar kolom di bawahnya, atau langsung klik nama group tersebut. Sampai sini pembuatan auto scaling group Jitsi Videobridge telah selesai dan seharusnya instance baru khusus untuk videobridge telah terbuat dan terhubung dengan Jitsi Meet Server.

### Referensi

- Amazon EC2 Auto Scaling User Guide. Retrieved September 26, 2021, from <https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html>
- Garcia, A. (2020). Script for Autoscaling JVB. Community.Jitsi.Org. Retrieved September 26, 2021, from <https://community.jitsi.org/t/script-for-autoscaling-jvb/59157/11>
