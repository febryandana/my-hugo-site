---
title: "Membuat EC2 Instance di AWS"
date: 2021-08-18
author : "Feb"
description : "Elastic Compute Cloud adalah salah satu layanan paling populer dari Amazon Web Service karena menawarkan solusi komputasi cloud yang sangat fleksibel dengan harga relatif murah. Kali ini kita akan belajar bagaimana cara membuat sebuah virtual machine atau instance pada AWS EC2."
toc : true
tags : ['aws', 'cloud', 'server']
---

![cover image](/posts/images/2021-08-18-membuat-ec2-instance-di-aws/image_1.png)

***AWS Elastic Compute Cloud*** (**EC2**) adalah salah satu service dari [Amazon Web Service](https://aws.amazon.com) yang paling populer digunakan. **EC2** adalah service yang menawarkan komponen *cloud computing* seperti *virtual machine* yang Amazon sebut sebagai ***Instance*** yang memiliki banyak pilihan spesifikasi, ***Amazon Machine Image*** (**AMI**) yang merupakan kumpulan OS image yang dapat dipasang untuk *Instance*, ***Elastic IP*** yang merupakan IPv4 Public statis cocok digunakan untuk DNS record, ***Amazon Elastic Block Store*** (**Amazon EBS**) sebagai *persistent storage*, dan lain-lain yang bisa dilihat sendiri [disini](https://docs.aws.amazon.com/ec2/index.html).

Membuat EC2 Instance di AWS cukup sederhana, hanya ada beberapa tahap yang perlu dilakukan.

## 0. Persiapan - Masuk ke layanan EC2 pada AWS Console

![AWS EC2 Dashboard.png](/posts/images/2021-08-18-membuat-ec2-instance-di-aws/image_2.png)
Sebelum membuat EC2 Instance, pertama-tama siapkan dulu kebutuhan sistem yang akan digunakan. Perkirakan seberapa berat beban komputasi yang akan ditanggung dan seberapa besar spesifikasi mesin yang cukup untuk menanggungnya. Setelah itu baru kita masuk ke tahap awal dari membuat EC2 instance, yaitu masuk ke dalam AWS Console. Pastikan akun yang kita gunakan memiliki akses untuk EC2, kemudian pilih layanan EC2 di bagian All Services. Gambar di atas adalah tampilan dashboard utama dari layanan EC2, untuk membuat Instance baru, kita bia memilih tombol "Launch instance".

## 1. Memilih AMI (Amazon Machine Image) dari OS yang akan digunakan

![AMI Selection.png](/posts/images/2021-08-18-membuat-ec2-instance-di-aws/image_3.png)
Selanjutnya kita akan disambut oleh menu pilihan AMI. Kita bisa memilih AMI yang direkomendasikan AWS di bagian Quick Start, menggunakan AMI milik kita sendiri jika ada, atau memilih untuk menggunakan AMI yang ada di AWS Marketplace atau AMI milik komunitas.

## 2. Memilih spesifikasi Instance

![Instance types.png](/posts/images/2021-08-18-membuat-ec2-instance-di-aws/image_4.png)
Tahap selanjutnya adalah memilih jenis dan spesifikasi Instance yang akan digunakan. AWS menyediakan banyak sekali spesifikasi Instance yang dapat dipilih, pilihan spesifikasi Instances dibedakan sebagai "family", mulai dari keluarga T, C, M, R dan sebagainya. Penjelasan lebih lanjut tentang pilihan Instance bisa dilihat [disini](https://aws.amazon.com/ec2/instance-types/).

## 3. Mengatur detail Instance

![Instance detail.png](/posts/images/2021-08-18-membuat-ec2-instance-di-aws/image_5.png)
Spesifikasi Instance yang kita pilih dapat diatur lebih lanjut di tahap ini. Disini kita bisa mengatur jumlah Instance yang akan dibuat, pilihan pembayaran, jaringan dan subnet yang digunakan, IAM Role, monitoring, dan lain sebagainya. Setelah dirasa cukup, kita bisa melanjutkan ke tahap berikutnya.

## 4. Memberi storage untuk Instance

![Instance storage.png](/posts/images/2021-08-18-membuat-ec2-instance-di-aws/image_6.png)
Tahap selanjutnya adalah mengatur ukuran dan tipe media penyimpanan yang akan digunakan. Minimal satu volume harus ada sebagai Root Volume. Kita bisa memilih kapasitas storage dengan bebas tetapi sebaiknya disesuaikan dengan kebutuhan karena setiap GB dari storage ini akan tetap dihitung biayanya (kecuali jika masih dalam batasan Free Tier). Kita juga bisa memilih tipe volume dan enkripsi yang akan digunakan untuk volume tersebut. Penjelasan lengkap untuk EC2 Storage ini dapat dibaca [disini](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Storage.html?icmpid=docs_ec2_console).

## 5. Memberi tag pada instance

![Instance tag.png](/posts/images/2021-08-18-membuat-ec2-instance-di-aws/image_7.png)
Langkah kelima ini sebenarnya opsional. Tanpa tag pun kita masih bisa membuat Instance. Tetapi best practice-nya memang sebaiknya kita memberikan tag untuk setiap komponen AWS yang dibuat agar lebih mudah dalam manajemen. Paling tidak berikan Name tag untuk membedakan nama setiap Instance yang akan dibuat.

## 6. Mengatur Security Group untuk keamanan Instance

![EC2 Security Group.png](/posts/images/2021-08-18-membuat-ec2-instance-di-aws/image_8.png)
Tentunya kita pasti tidak ingin Instance kita dapat dibobol dengan mudah oleh orang lain bukan? Oleh karena itu tahap keenam ini menjadi bagian yang cukup krusial karena berhubungan dengan keamanan Instance. Pastikan untuk hanya membuka protokol dan port yang memang benar-benar dibutuhkan saja. Untuk keamanan lebih, kita juga bisa membatasi hanya IP Address dari sumber tertentu saja yang dapat mengakses Instance. Penjelasan lebih lanjut tentang EC2 Security Group dapat dibaca [disini](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html?icmpid=docs_ec2_console).

## 7. Pengecekan terakhir dan meluncurkan Instance

![EC2 Instance review.png](/posts/images/2021-08-18-membuat-ec2-instance-di-aws/image_9.png)
Tahap terakhir dari pembuatan EC2 Instance adalah review. Di tahap ini kita disuguhkan detail dari konfigurasi Instance yang sudah kita buat sebelumnya. Pastikan bahwa pengaturan yang dibuat sudah sesuai dengan apa yang kita butuhkan agar tidak repot mengatur ulang nantinya. Jika dirasa sudah yakin, kita lanjutkan dengan menekan tombol "Launch" untuk meluncurkan Instance.

![EC2 SSH key pair.png](/posts/images/2021-08-18-membuat-ec2-instance-di-aws/image_10.png)
Sebelum meluncurkan Instance, kita akan diberikan jendela terakhir dimana kita harus memilih SSH key pair untuk mengakses Instance nantinya. Kita bisa memilih untuk membuat key baru, memilih dari yang sudah ada, atau melanjutkan tanpa key. Simpan SSH key yang dibuat dengan baik karena kita tidak bisa mengunduhnya ulang setelah Instance dibuat. Setelah itu lanjutkan dengan menekan tombol "Launch Instances" untuk menyelesaian pembuatan AWS EC2 Instance.

![EC2 Instance dashboard.png](/posts/images/2021-08-18-membuat-ec2-instance-di-aws/image_11.png)
Detail Instance yang telah dibuat dapat dilihat di menu "Instances" dari daftar menu sebelah kiri. Disini kita bisa melihat detail Instance, Security, Networking, dan lain-lain. Selain itu kita juga bisa menghentikan dan menghapus Instance. Ada juga pilihan Actions seperti untuk menghubungi Instance, mengganti tipe Instance, mengubah Security Group, dan lain-lain. AWS EC2 memberikan kemudahan dan kontrol yang sangat luas terhadap cloud computer yang kita buat.

## Mematikan EC2 Instance

![Terminate Instance.png](/posts/images/2021-08-18-membuat-ec2-instance-di-aws/image_12.png)
Untuk menghemat biaya, sebaiknya kita mematikan EC2 Instance yang sudah tidak digunakan. Cara mematikan dan menghapus EC2 Instance sangat sederhana. Pertama-tama masuk ke dalam EC2 Dashboard dan pilih menu Instances. Kemudian centang Instance yang ingin dimatikan atau ingin dihapus. Selanjutnya klik pada menu Instance State di sebelah kanan atas. Akan muncul pilihan menu status instance :

- Stop Instance : Untuk mematikan Instance tanpa menghapusnya. Biaya Instance masih akan tetap berjalan meskipun Instance berhenti
- Start Instance : Untuk menjalankan kembali Instance yang sudah di-stop
- Reboot Instance : Untuk memuat ulang Instance
- Hibernate Instance : Untuk menghibernasi Instance
- Terminate Instance : Untuk menghapus Instance

Item yang terhapus termasuk Instance, Network Interface, dan Volume storage beserta data-data yang ada di dalamnya. Instance yang telah terhapus tidak dapat dikembalikan, karena itu sebelum menghapus Instance pastikan bahwa sudah tidak ada data penting di dalamnya dan Instance benar-benar sudah tidak dibutuhkan lagi.
