---
title: "Jibri Recorder Untuk Jitsi"
date: 2021-10-29
author : "Feb"
description: "Setelah menyelesaikan panduan instalasi Jitsi Meet, kita dapat menggunakannya untuk rapat atau pertemuan secara online. Tetapi ada fitur yang tidak ikut terinstall, yaitu fitur rekaman. Fitur ini sangat berguna apabila kita perlu menyimpan hasil konferensi online kita untuk diarsipkan atau di-upload ke Youtube. Untungnya, Jitsi juga menyediakan layanan rekaman dengan nama Jibri. Pada artikel ini, kita akan mencoba melakukan instalasi Jibri untuk Jitsi Meet Sever sebelumnya."
tags: ['cloud', 'jitsi', 'server']
---

![cover image.png](/posts/images/2021-10-29-jibri-recorder-untuk-jitsi/b10576b3a4b14c93ba2bb943ebe7cf6b.png)

Setelah menyelesaikan panduan instalasi Jitsi Meet, kita dapat menggunakannya untuk rapat atau pertemuan secara online. Tetapi ada fitur yang tidak ikut terinstall, yaitu fitur rekaman. Fitur ini sangat berguna apabila kita perlu menyimpan hasil konferensi online kita untuk diarsipkan atau di-upload ke Youtube. Untungnya, Jitsi juga menyediakan layanan rekaman dengan nama Jibri. Pada artikel ini, kita akan mencoba melakukan instalasi Jibri untuk Jitsi Meet Sever sebelumnya.

Hal yang perlu diperhatikan sebelum melakukan instalasi Jibri antara lain :

- Jibri bekerja dengan membuat sebuah instance browser Chrome untuk masuk ke dalam room Jitsi lalu menangkap dan melakukan proses encoding video dan audion dengan FFMPEG. Oleh karena itu Jibri sebaiknya dipasang pada standalone system yang dibuat khusus untuk Jibri.
- Karena melakukan proses encoding video dan audio, Jibri membutuhkan resource yang cukup besar. Spesifikasi yang direkomendasikan adalah 4 core CPU dan 8 GB memory. Jibri juga perlu ruang storage yang cukup besar untuk menyimpan hasil rekamannya.
- Saat ini, satu Jibri instance hanya bisa melakukan 1 proses record. Jika ada banyak room dan semuanya perlu untuk merekam maka kita juga harus menyediakan instance Jibri sesuai jumlah room tersebut.
- Jibri membutuhkan modul ALSA loopback yang ada di generic kernel. Jika kita menggunakan AWS maka perlu untuk mengubah kernel-nya menjadi kernel generic.

## Mengubah kernel AWS menjadi generic-kernel pada Ubuntu Server 18.04 AWS

Pertama-tama kita perlu install image generic-kernel dengan perintah:

```bash
sudo apt-get update
sudo apt-get upgrade -y
sudo apt install linux-image-extra-virtual
```

Cek kernel yang saat ini digunakan dengan perintah:

```bash
uname -r
```

Jika masih ada "aws" berarti Ubuntu Server masih menggunakan kernel AWS.

Cek daftar kernel yang tersedia dengan perintah:

```bash
grep -A100 submenu /boot/grub/grub.cfg |grep menuentry
```

Contoh hasil keluarannya adalah seperti di bawah ini:

```bash
submenu 'Advanced options for Ubuntu' $menuentry_id_option 'gnulinux-advanced-db937f23-4ed7-4c4b-8058-b23a860fae08' {
        menuentry 'Ubuntu, with Linux 5.4.0-1054-aws' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-5.4.0-1054-aws-advanced-db937f23-4ed7-4c4b-8058-b23a860fae08' {             menuentry 'Ubuntu, with Linux 5.4.0-1054-aws (recovery mode)' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-5.4.0-1054-aws-recovery-db937f23-4ed7-4c4b-8058-b23a860fae08' {
        menuentry 'Ubuntu, with Linux 5.4.0-1045-aws' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-5.4.0-1045-aws-advanced-db937f23-4ed7-4c4b-8058-b23a860fae08' {             menuentry 'Ubuntu, with Linux 5.4.0-1045-aws (recovery mode)' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-5.4.0-1045-aws-recovery-db937f23-4ed7-4c4b-8058-b23a860fae08' {
        menuentry 'Ubuntu, with Linux 4.15.0-153-generic' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-4.15.0-153-generic-advanced-db937f23-4ed7-4c4b-8058-b23a860fae08' {
        menuentry 'Ubuntu, with Linux 4.15.0-153-generic (recovery mode)' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-4.15.0-153-generic-recovery-db937f23-4ed7-4c4b-8058-b23a860fae08' {
```

Salin id option dari baris pertama (Advanced options for Ubuntu). Contohnya seperti di bawah ini:

```bash
gnulinux-advanced-db937f23-4ed7-4c4b-8058-b23a860fae08
```

Selanjutnya salin id option dari bagian generic (Ubuntu, with Linux 4.15.0-153-generic). Contohnya seperti di bawah ini:

```bash
gnulinux-4.15.0-153-generic-advanced-db937f23-4ed7-4c4b-8058-b23a860fae08
```

Kombinasikan keduanya menggunakan tanda > menjadi seperti di bawah ini:

```bash
gnulinux-advanced-db937f23-4ed7-4c4b-8058-b23a860fae08>gnulinux-4.15.0-153-generic-advanced-db937f23-4ed7-4c4b-8058-b23a860fae08
```

Kemudian edit file grub.cfg dengan perintah:

```bash
sudo nano /etc/default/grub.cfg
```

Ubah value dari GRUB_DEFAULT=0 dengan value di atas, menjadi

```bash
GRUB_DEFAULT='gnulinux-advanced-db937f23-4ed7-4c4b-8058-b23a860fae08>gnulinux-4.15.0-153-generic-advanced-db937f23-4ed7-4c4b-8058-b23a860fae08'
```

Simpan dan keluar, lalu restart grub dan sistem:

```bash
sudo update-grub && sudo update-grub2
sudo restart
```

Kemudian cek apakah kernel telah berubah dengan perintah `uname -r`. Seharusnya hasilnya menjadi:

```bash
$ uname -r
4.15.0-153-generic
```

## Persiapan di Jitsi Meet Server Instance

Semua domain yang digunakan pada panduan ini adalah **meet.yourdomain.com**. Kalian perlu mengubahnya ke domain Jitsi kalian sendiri. Pertama-tama kita perlu menyiapkan environment Jibri pada Jitsi Meet Server, antara lain:

### Prosody

Kita perlu membuat entry untuk internal muc agar Jibri client dapat berkomunikasi dengan Jicofo. Edit file `/etc/prosody/conf.avail/meet.yourdomain.com.cfg.lua`, cari dan sesuaikan dengan baris berikut:

```bash
-- internal muc component, meant to enable pools of jibri and jigasi clients
Component "internal.auth.meet.yourdomain.com" "muc"
    modules_enabled = {
      "ping";
    }
    -- storage should be "none" for prosody 0.10 and "memory" for prosody 0.11
    storage = "memory"
    admins = { "focus@auth.meet.yourdomain.com", "jvb@auth.meet.yourdomain.com", "jibri@auth.meet.yourdomain.com" }
    muc_room_locking = false
    muc_room_default_public_jids = true
    muc_room_cache_size = 1000
    c2s_require_encryption = false
```

Selanjutnya kita perlu membuat virtual host entry untuk recorder untuk sebagai user akun untuk sesi Jibri. Hal ini diperlukan agar Jibri dapat menjadi hidden participant pada konferensi yang direkam. Edit dan tambahkan baris berikut ke file `/etc/prosody/conf.avail/meet.yourdomain.com.cfg.lua`:

```bash
VirtualHost "recorder.meet.yourdomain.com"
  modules_enabled = {
    "ping";
  }
  authentication = "internal_plain"
  c2s_require_encryption = false
  allow_empty_token = true
```

Kemudian kita perlu membuat akun untuk Jibri dengan perintah:

```bash
sudo prosodyctl register jibri auth.meet.yourdomain.com jibriauthpass
sudo prosodyctl register recorder recorder.meet.yourdomain.com jibrirecorderpass
```

Simpan username dan password dari prosodyctl di atas karena nantinya akan diperlukan lagi.

### Jicofo

Edit file `/etc/jitsi/jicofo/sip-communicator.properties` dan tambahkan baris berikut ini:

```bash
org.jitsi.jicofo.jibri.BREWERY=JibriBrewery@internal.auth.meet.yourdomain.com
org.jitsi.jicofo.jibri.PENDING_TIMEOUT=90
```

Baris tersebut diperlukan untuk membuat MUC yang sesuai untuk Jibri Controllers agar dapat berkomunikasi dengan Jicofo. PENDING_TIMEOUT juga sebaiknya di-set ke 90 detik untuk memberikan jeda waktu Jibri sebelum dicap failed dan dimatikan.

### Jitsi Meet

Edit file /etc/jitsi/meet/meet.yourdomain.com-config.js lalu cari dan tambahkan set konfigurasi berikut ini:

```bash
fileRecordingsEnabled: true, // Untuk mengaktifkan fitur file recording
liveStreamingEnabled: true, // Untuk mengaktifkan fitur live streaming
hiddenDomain: 'recorder.meet.yourdomain.com', // Untuk menyembunyikan user jibri dari room
```

## Proses Instalasi Jibri di Jibri Instance

Panduan selanjutnya dilakukan di instance Jibri. Pertama-tama pastikan modul ALSA loopback telah tersedia dengan mengikuti panduan mengubah kernel AWS menjadi kernel generic di awal.

### ALSA dan Loopback Device

Jalankan beberapa perintah di bawah ini untuk mengaktifkan modul ALSA loopback

- Set modul agar di-load saat boot : `sudo echo "snd-aloop" >> /etc/modules`
- Load modul ke kernel saat ini : `sudo modprobe snd-aloop`
- Cek jika modul telah di-load : `sudo lsmod | grep snd_aloop`
- Jika output-nya menampilkan modul snd-aloop, berati ALSA loopback telah selesai dikonfigurasi.

### Google Chrome dan Chromedriver

Selanjutnya kita perlu memasang Google Chrome stabil dan Chromedriver yang akan digunakan Jibri untuk mengakses room Jitsi dan melakukan rekaman. Instruksi instalasi manual Google Chrome stabil adalah sebagai berikut:

```bash
wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
sudo sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list'
apt-get -y update
apt-get -y install google-chrome-stable
```

Tambahkan file chrome managed policies dan set `CommandLineFlagSecurityWarningEnabled` ke `false` untuk menyembunyikan warning dari Chrome:

```bash
sudo mkdir -p /etc/opt/chrome/policies/managed
sudo sh -c 'echo '{ "CommandLineFlagSecurityWarningsEnabled": false }' >> /etc/opt/chrome/policies/managed/managed_policies.json'
```

Selanjutnya install Chromedriver dengan perintah dibawah ini:

```bash
CHROME_DRIVER_VERSION=`curl -sS chromedriver.storage.googleapis.com/LATEST_RELEASE`
wget -N http://chromedriver.storage.googleapis.com/$CHROME_DRIVER_VERSION/chromedriver_linux64.zip -P ~/
unzip ~/chromedriver_linux64.zip -d ~/
rm ~/chromedriver_linux64.zip
sudo mv -f ~/chromedriver /usr/local/bin/chromedriver
sudo chown root:root /usr/local/bin/chromedriver
sudo chmod 0755 /usr/local/bin/chromedriver
```

### Repository Jitsi dan Instalasi Jibri

Paket Jibri terdapat di repository Jitsi, jadi kita perlu menambahkan Jitsi ke dalam repository Ubuntu Server dengan perintah:

```bash
echo 'deb https://download.jitsi.org stable/' | sudo tee \ /etc/apt/sources.list.d/jitsi-stable.list
wget -qO - https://download.jitsi.org/jitsi-key.gpg.key | sudo \ apt-key add -
sudo apt-get update
```

Selanjutnya kita mulai instalasi Jibri terbaru dengan perintah:

```bash
sudo apt-get install jibri
```

Agar dapat bekerja dengan baik, sebaiknya kita juga memasukan user jibri ke dalam beberapa grup berikut:

```bash
sudo usermod -aG adm,audio,video,plugdev jibri
```

### File Konfigurasi Jibri

File konfigurasi Jibri terdapat di `/etc/jitsi/jibri/jibri.conf`. Contoh konfigurasinya dapat dilihat di [reference.conf](https://github.com/jitsi/jibri/blob/master/src/main/resources/reference.conf) untuk default values dan contoh bagaimana membuat jibri.conf. Di bawah ini adalah contoh file konfigurasi jibri.conf yang digunakan penulis:

```bash
jibri {
  // A unique identifier for this Jibri
  // TODO: eventually this will be required with no default
  id = ""
  // Whether or not Jibri should return to idle state after handling
  // (successfully or unsuccessfully) a request.  A value of 'true'
  // here means that a Jibri will NOT return back to the IDLE state
  // and will need to be restarted in order to be used again.
  single-use-mode = false
  api {
    http {
      external-api-port = 2222
      internal-api-port = 3333
    }
    xmpp {
      // See example_xmpp_envs.conf for an example of what is expected here
      environments = [
              {
                name = "prod environment"
                xmpp-server-hosts = ["meet.yourdomain.com"]
                xmpp-domain = "meet.yourdomain.com"

                control-muc {
                    domain = "internal.auth.meet.yourdomain.com"
                    room-name = "JibriBrewery"
                    nickname = "jibri"
                }

                control-login {
                    domain = "auth.meet.yourdomain.com"
                    username = "jibri"
                    password = "jibriauthpass"
                }

                call-login {
                    domain = "recorder.meet.yourdomain.com"
                    username = "recorder"
                    password = "jibrirecorderpass"
                }

                strip-from-room-domain = "conference."
                usage-timeout = 0
                trust-all-xmpp-certs = true
            }]
    }
  }
  recording {
    recordings-directory = "/srv/recordings"
    # TODO: make this an optional param and remove the default
    finalize-script = "/srv/finalize.sh"
  }
  streaming {
    // A list of regex patterns for allowed RTMP URLs.  The RTMP URL used
    // when starting a stream must match at least one of the patterns in
    // this list.
    rtmp-allow-list = [
      // By default, all services are allowed
      ".*"
    ]
  }
  chrome {
    // The flags which will be passed to chromium when launching
    flags = [
      "--use-fake-ui-for-media-stream",
      "--start-maximized",
      "--kiosk",
      "--enabled",
      "--disable-infobars",
      "--autoplay-policy=no-user-gesture-required"
    ]
  }
  stats {
    enable-stats-d = true
  }
  webhook {
    // A list of subscribers interested in receiving webhook events
    subscribers = []
  }
  jwt-info {
    // The path to a .pem file which will be used to sign JWT tokens used in webhook
    // requests.  If not set, no JWT will be added to webhook requests.
    # signing-key-path = "/path/to/key.pem"

    // The kid to use as part of the JWT
    # kid = "key-id"

    // The issuer of the JWT
    # issuer = "issuer"

    // The audience of the JWT
    # audience = "audience"

    // The TTL of each generated JWT.  Can't be less than 10 minutes.
    # ttl = 1 hour
  }
  call-status-checks {
    // If all clients have their audio and video muted and if Jibri does not
    // detect any data stream (audio or video) comming in, it will stop
    // recording after NO_MEDIA_TIMEOUT expires.
    no-media-timeout = 90 seconds

    // If all clients have their audio and video muted, Jibri consideres this
    // as an empty call and stops the recording after ALL_MUTED_TIMEOUT expires.
    all-muted-timeout = 10 minutes

    // When detecting if a call is empty, Jibri takes into consideration for how
    // long the call has been empty already. If it has been empty for more than
    // DEFAULT_CALL_EMPTY_TIMEOUT, it will consider it empty and stop the recording.
    default-call-empty-timeout = 30 seconds
  }
}
```

Selanjutnya, aktifkan jibri agar berjalan otomatis ketika sistem dinyalakan dengan perintah:

```bash
sudo systemctl enable jibri
sudo systemctl restart jibri
```

Setelah selesai melakukan rekaman, file rekaman Jibri akan tersimpan di direktori `/srv/recordings/`. Kita juga bisa menambahkan file `finalize.sh` di direktori `/srv/finalize.sh` yang akan dijalankan setiap jibri selesai merekam. File `finalize.sh` dapat diisi dengan script untuk mengunggah hasil rekaman ke cloud storage atau yang lain sesuai keinginan.

### Referensi

- Github Jitsi/Jibri. <https://github.com/jitsi/jibri>
- Coders, N. (2021, July 28). How to change default kernel in Ubuntu (AWS) (Jibri). NIMBLE CODERS. <https://nimblecoders.in/how-to-change-default-kernel-in-ubuntu-aws/>
- Saiflakhani, S. (2021, February 24). [Setup / Guide] Jitsi Meet Native + Multiple (6) Jibri Docker instances working on the same AWS server. Community.Jitsi.Org. <https://community.jitsi.org/t/setup-guide-jitsi-meet-native-multiple-6-jibri-docker-instances-working-on-the-same-aws-server/>
