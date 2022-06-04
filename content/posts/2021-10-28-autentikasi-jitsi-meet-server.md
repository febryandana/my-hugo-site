---
title: "Autentikasi Jitsi Meet Server"
date: 2021-10-28
author : "Feb"
description : "Jitsi mendukung beberapa metode autentikasi seperti SIP, LDAP, dan plain account. Pada artikel ini, kita akan mencoba membuat autentikasi menggunakan plain account yang dibuat langsung menggunakan Prosody."
tags : ['cloud', 'jitsi', 'server']
---

![ca8ac2c38e70c45d9170a44fd2822d69.png](/posts/images/2021-10-28-autentikasi-jitsi-meet-server/b1a552e3ccba43699f6b945f4446b22a.png)

Sumber: <https://www.brring.com/2020/04/04/setting-up-a-jitsi-server-in-less-than-15-minutes/>

Jitsi mendukung beberapa metode autentikasi seperti SIP, LDAP, dan plain account. Pada artikel ini, kita akan mencoba membuat autentikasi menggunakan plain account yang dibuat langsung menggunakan Prosody.

## Konfigurasi Prosody

Buka file konfigurasi domain di prosody dengan menggunakan `nano` atau text editor lainnya.

```bash
nano /etc/prosody/conf.d/meet.yourdomain.com.cfg.lua
```

Cari VirtualHost yang berisi domain Jitsi Meet kita, lalu ubah value `authentication` menjadi `internal_hashed`

```bash
VirtualHost "meet.yourdomain.com"
        -- enabled = false -- Remove this line to enable this host
        -- authentication = "anonymous"
        authentication = "internal_hashed"
...
```

Kemudian, tambahkan VirtualHost baru dalam file yang sama untuk menampung guest. Domain guest ini bersifat internal only sehingga tidak memerlukan DNS record atau sertifikat SSL baru.

```bash
...
VirtualHost "guest.meet.yourdomain.com"
    authentication = "anonymous"
    c2s_require_encryption = false
```

Simpan file untuk mengonfirmasi perubahan.

## Menambahkan guest domain ke Jitsi Meet

Setelah mengatur Prosody XMPP Server untuk mengaktifkan fitur autentikasi, selanjutnya kita perlu memberi tahu Jitsi Meet tentang VirtualHost guest yang baru kita buat. Caranya adalah dengan membuka file berikut menggunakan text editor.

```bash
nano /etc/jitsi/meet/meet.yourdomain.com-config.js
```

Cari variabel `anonymousdomain` pada blok `hosts`, lalu tambahkan domain guest.yourdomain.com sebagai value-nya.

```bash
hosts: {
    // XMPP domain.
    domain: 'meet.yourdomain.com',

    // When using authentication, domain for guest users.
    // anonymousdomain: 'guest.example.com',
    anonymousdomain: 'guest.meet.yourdomain.com',
    ...
}
...
```

Simpan dan tutup file tersebut untuk mengonfirmasi perubahan.

## Konfigurasi Jicofo

Selanjutnya kita perlu mengatur komponen Jitsi Conference Focus (Jicofo) untuk mengizinkan request hanya dari authenticated domain saja. Buka file berikut dengan text editor.

```bash
nano /etc/jitsi/jicofo/sip-communicator.properties
```

Tambahkan key-value baru berikut ke dalam file dan simpan.

```bash
...
org.jitsi.jicofo.auth.URL=XMPP:meet.yourdomain.com
```

## Membuat akun moderator

Tahap terakhir adalah membuat akun moderator yang memiliki izin untuk membuat conference room di Jitsi Meet Server. Kita bisa menggunakan layanan prosodyctl untuk membuat akun ini.

```bash
sudo prosodyctl register USERNAME meet.yourdomain.com PASSWORD
```

Pastikan untuk membuat *username* dan *password* yang unik agar tidak mudah diretas. Pastikan juga untuk menyimpan kredensial *username* dan *password* tersebut di tempat yang aman dan tidak mudah lupa.

## Restart semua Jitsi service

Setelah selesai melalukan konfigurasi di atas, restart layanan Jitsi dengan perintah berikut.

```bash
sudo systemctl restart prosody
sudo systemctl restart jicofo
sudo systemctl restart nginx

# Jika Jitsi Meet diatur untuk menggunakan videobridge dalam satu server yang sama:
sudo systemctl restart jitsi-videobridge2
```

## Cek file log

Untuk melihat log aktifitas dari layanan Jitsi, kita bisa menggunakan perintah berikut.

```bash
# Log Prosody
tail -f /var/log/prosody/prosody.log

# Log Jicofo
tail -f /var/log/jitsi/jicofo.log

# Log Nginx
tail -f /var/log/nginx/error.log

# Log Jitsi Video Bridge
tail -f /var/log/jitsi/jvb.log
```

## Selesai

Kita bisa mengetesnya langsung dengan membuka domain Jitsi Meet dan mencoba membuat conference room. Seharusnya Jitsi akan memunculkan notifikasi "Waiting for the host". Pilih "I am the host" lalu masukkan kredensial akun yang telah dibuat sebelumnya. Baru lah setelah moderator masuk, peserta conference lainnya dapat saling berinteraksi.

### Referensi

- Kirstätter, J. (2020, May 3). Enable authentication in your instance of jitsi meet. Get Blogged by JoKi. Retrieved August 28, 2021, from <https://jochen.kirstaetter.name/authentication-jitsi-meet/>.
