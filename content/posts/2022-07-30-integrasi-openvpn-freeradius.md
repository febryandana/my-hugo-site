---
title: "Integrasi Openvpn Freeradius"
date: 2022-07-30
author: "Feb"
description: "OpenVPN dapat memanfaatkan layanan RADIUS sebagai sumber autentikasi akun penggunanya. Pada artikel ini kita akan mencoba integrasi antara OpenVPN dengan FreeRADIUS plus memanfaatkan DaloRADIUS untuk layanan dashboard FreeRADIUS. Dengan begitu, administrator dapat mengatur manajemen pengguna OpenVPN dalam satu tempat secara mudah."
tags: ['vpn', 'cloud', 'server']
toc: true
---

## Overview

OpenVPN dapat memanfaatkan layanan RADIUS sebagai sumber autentikasi akun penggunanya. Pada artikel ini kita akan mencoba integrasi antara OpenVPN dengan FreeRADIUS plus memanfaatkan DaloRADIUS untuk layanan dashboard FreeRADIUS. Dengan begitu, administrator dapat mengatur manajemen pengguna OpenVPN dalam satu tempat secara mudah.

## 1. Konfigurasi FreeRADIUS & DaloRADIUS

### FreeRADIUS

1. Install Web Server

```bash
sudo -i
apt update && apt -y upgrade
apt -y install apache2
apt -y install php libapache2-mod-php php-{gd,common,mail,mail-mime,mysql,pear,db,mbstring,xml,curl}
php -v
```

2. Install DB server

```bash
apt -y install mariadb-server
mysql_secure_installation
```

3. Create radius database

```bash
mysql -u root -p

MariaDB [(none)]> CREATE DATABASE radius;
MariaDB [(none)]> GRANT ALL ON radius.* TO radius@localhost IDENTIFIED BY "StrongPassword";
MariaDB [(none)]> FLUSH PRIVILEGES;
MariaDB [(none)]> QUIT
```

4. Install & Configure FreeRADIUS

```bash
apt -y install freeradius freeradius-mysql freeradius-utils
```

Import freeRADIUS sql database

```bash
mysql -u root -p radius < /etc/freeradius/3.0/mods-config/sql/main/mysql/schema.sql
mysql -u root -p -e "use radius; show tables;"
```

Create softlink

```bash
ln -s /etc/freeradius/3.0/mods-available/sql /etc/freeradius/3.0/mods-enabled/
```

Configure sql module

Comment SSL sections in mysql

```bash
nano /etc/freeradius/3.0/mods-enabled/sql

---
sql {
driver = "rlm_sql_mysql"
dialect = "mysql"

# Connection info:
server = "localhost"
port = 3306
login = "radius"
password = "StrongPassword"

# Database table configuration for everything except Oracle
radius_db = "radius"
}

# Set to ‘yes’ to read radius clients from the database (‘nas’ table)
# Clients will ONLY be read on server startup.
read_clients = yes

# Table to keep radius client info
client_table = "nas"
---
```

Change Group

```bash
chgrp -h freerad /etc/freeradius/3.0/mods-available/sql
chown -R freerad:freerad /etc/freeradius/3.0/mods-enabled/sql
```

Restart freeradius service

```bash
sudo systemctl restart freeradius.service
```

### DaloRADIUS

1. Install daloRadius

```bash
apt -y install wget unzip
wget https://github.com/lirantal/daloradius/archive/master.zip
unzip master.zip
mv daloradius-master daloradius
cd daloradius
```

2. Configure daloRadius

Import daloRadius tables

```bash
mysql -u root -p radius < contrib/db/fr2-mysql-daloradius-and-freeradius.sql
mysql -u root -p radius < contrib/db/mysql-daloradius.sql
```

Move daloRadius to Web Server

```bash
cd ..
mv daloradius /var/www/html/
mv /var/www/html/daloradius/library/daloradius.conf.php.sample /var/www/html/daloradius/library/daloradius.conf.php
chown -R www-data:www-data /var/www/html/daloradius/
chmod 664 /var/www/html/daloradius/library/daloradius.conf.php
```

Configure daloRadius connection

```bash
nano /var/www/html/daloradius/library/daloradius.conf.php

---
$configValues['CONFIG_DB_HOST'] = 'localhost';
$configValues['CONFIG_DB_PORT'] = '3306';
$configValues['CONFIG_DB_USER'] = 'radius';
$configValues['CONFIG_DB_PASS'] = 'StrongPassword';
$configValues['CONFIG_DB_NAME'] = 'radius';
---

touch /tmp/daloradius.log
```

Restart services

```bash
systemctl restart freeradius.service apache2.service
```

3. Verify daloRadius

Akses via <http://IP_ADDRESS/daloradius/login.php>

Default User & Password:  
User = administrator  
Password = radius

![daloradius login page](/posts/images/2022-07-30-integrasi-openvpn-freeradius/daloradius-login.png)

![daloradius homepage](/posts/images/2022-07-30-integrasi-openvpn-freeradius/daloradius-homepage.png)

![radtest](/posts/images/2022-07-30-integrasi-openvpn-freeradius/radtest-radius.png)

## 2. Instalasi OpenVPN

OpenVPN diinstall dengan bantuan script eksternal.

```bash
apt install -y openvpn openvpn-auth-radius freeradius-utils
wget https://git.io/vpn -O openvpn-install.sh
bash openvpn-install.sh

---
Public IPv4 address / hostname [a.b.c.d]: IP_Server_OpenVPN
---

Sisanya pakai default
```

## 3. Integrasi OpenVPN dengan FreeRADIUS

### Sisi Server FreeRadius

1. Buat NAS untuk OpenVPN server
   - Via DaloRadius:
     - NAS IP/Host = 192.168.1.12/24 //IP ADDRESS OPENVPN SERVER
     - NAS Secret = fb-ovpn
     - NAS Type   = other
     - NA Shortname = fb-ovpn
  
   - Via clients.conf:

        ```bash
        nano clients.conf

        ---
        client fb-ovpn {
            ipaddr = 192.168.1.12 //IP ADDRESS OPENVPN SERVER
            netmask = 24
            secret = fb-ovpn
            shortname = fb-ovpn
            nastype = other
        }
        ---
        ```

Restart service freeradius setiap kali membuat NAS baru

```bash
systemctl restart freeradius.service
```

### Sisi Server OpenVPN

1. Test koneksi RADIUS via server OpenVPN

```bash
radtest {username} {password} {radius_hostname} 10 {radius_secret}

radtest febry febry 192.168.1.11 10 fb-ovpn
```

2. Konfigurasi file server.conf pada OpenVPN server agar bisa berkomunikasi dengan server FreeRadius

```bash
nano /etc/openvpn/server/server.conf

---
plugin /usr/lib/openvpn/radiusplugin.so /etc/openvpn/radiusplugin.cnf
verify-client-cert
key-direction 0
duplicate-cn
local 192.168.1.12
port 1194
proto udp
dev tun
ca ca.crt
cert server.crt
key server.key
dh dh.pem
auth SHA512
tls-crypt tc.key
topology subnet
server 10.8.0.0 255.255.255.0
push "redirect-gateway def1 bypass-dhcp"
ifconfig-pool-persist ipp.txt
push "dhcp-option DNS 8.8.8.8"
keepalive 10 120
cipher AES-256-CBC
user nobody
group nogroup
persist-key
persist-tun
verb 3
crl-verify crl.pem
explicit-exit-notify
---
```

3. Konfigurasi file ovpn yang akan digunakan oleh client

```bash
nano /root/client.ovpn

---
key-direction 1
auth-user-pass
;user nobody
;group nogroup

client
dev tun
proto udp
remote 192.168.1.12 1194
resolv-retry infinite
nobind
persist-key
persist-tun
remote-cert-tls server
auth SHA512
cipher AES-256-CBC
ignore-unknown-option block-outside-dns
block-outside-dns
verb 3
<ca>
...
---
```

## 4. Test koneksi OpenVPN dengan RADIUS

> Dilakukan di node fb-ubuntu (192.168.2.11 - lab6)

```bash
openvpn --config client.ovpn
```

![ovpn-radius test](/posts/images/2022-07-30-integrasi-openvpn-freeradius/ovpn-radius.png)

> Dilakukan di komputer pribadi

```bash
sudo openvpn --config client.ovpn
```

![ovpn-radius pc cli](/posts/images/2022-07-30-integrasi-openvpn-freeradius/ovpn-pc-cli.png)

![ovpn-radius pc gui 1](/posts/images/2022-07-30-integrasi-openvpn-freeradius/ovpn-pc-gui-1.png)

![ovpn-radius pc gui 2](/posts/images/2022-07-30-integrasi-openvpn-freeradius/ovpn-pc-gui-2.png)

## Known Issues

1. Unknown column 'acctupdatetime' in 'field list'

```bash
DaloRadius, RADIUS log :
ERROR: (21) sql: ERROR: rlm_sql_mysql: ERROR 1054 (Unknown column 'acctupdatetime' in 'field list'): 42S22 
```

Solve: Rebuild table radacct

```bash
mysql -u root -p radius

DROP TABLE radacct;

CREATE TABLE radacct (
radacctid bigint(21) NOT NULL auto_increment,
acctsessionid varchar(64) NOT NULL default '',
acctuniqueid varchar(32) NOT NULL default '',
username varchar(64) NOT NULL default '',
groupname varchar(64) NOT NULL default '',
realm varchar(64) default '',
nasipaddress varchar(15) NOT NULL default '',
nasportid varchar(15) default NULL,
nasporttype varchar(32) default NULL,
acctstarttime datetime NULL default NULL,
acctupdatetime datetime NULL default NULL,
acctstoptime datetime NULL default NULL,
acctinterval int(12) default NULL,
acctsessiontime int(12) unsigned default NULL,
acctauthentic varchar(32) default NULL,
connectinfo_start varchar(50) default NULL,
connectinfo_stop varchar(50) default NULL,
acctinputoctets bigint(20) default NULL,
acctoutputoctets bigint(20) default NULL,
calledstationid varchar(50) NOT NULL default '',
callingstationid varchar(50) NOT NULL default '',
acctterminatecause varchar(32) NOT NULL default '',
servicetype varchar(32) default NULL,
framedprotocol varchar(32) default NULL,
framedipv6address varchar(32) default NULL,
framedipv6prefix varchar(32) default NULL,
framedinterfaceid varchar(32) default NULL,
delegatedipv6prefix varchar(32) default NULL,
framedipaddress varchar(15) NOT NULL default '',
PRIMARY KEY (radacctid),
UNIQUE KEY acctuniqueid (acctuniqueid),
KEY username (username),
KEY framedipaddress (framedipaddress),
KEY acctsessionid (acctsessionid),
KEY acctsessiontime (acctsessiontime),
KEY acctstarttime (acctstarttime),
KEY acctinterval (acctinterval),
KEY acctstoptime (acctstoptime),
KEY nasipaddress (nasipaddress)
) ENGINE = INNODB;
```

1. Error reading log file: /tmp/daloradius.log

```bash
error reading log file: /tmp/daloradius.log
looked for log file in /tmp/daloradius.log but couldn't find it.
if you know where your daloradius log file is located, set it's location in your library/daloradius.conf file
```

Solve: Create new log file

```bash
touch /tmp/daloradius.log
chown www-data:www-data daloradius.log
chmod 644 /tmp/daloradius.log
```

3. Error reading log file: /var/log/syslog

```bash
error reading log file: /var/log/syslog
possible cause is file permissions or file does not exist.
```

Solve: Change file permission

```bash
chmod 644 /var/log/syslog
```

4. Tipe user RADIUS yang bisa digunakan OpenVPN

- [x] cleartext-password
- [ ] User-password (AUTH_FAILED)
- [x] Crypt-password
- [x] MD5-password
- [ ] SHA1-password (AUTH_FAILED)
- [ ] CHAP-password (AUTH_FAILED)

5. Beberapa sistem operasi tidak memiliki group "nogroup", melainkan "nobody" sementara yang lain sebaliknya. Workaround untuk isu ini ada 3 :

   - Membuat user `nobody` dan group `nogroup`.
   - Membuat user `nobody` dan group `nobody`. Lalu mengubah baris kode `group nogroup` menjadi `group nobody` pada file client.ovpn.
   - Memberi comment pada baris kode `user nobody` dan `group nogroup` pada file client.ovpn.

## References

- <https://book.btech.id/books/riset-freeradius>
- <https://bytexd.com/freeradius-ubuntu/>
- <https://computingforgeeks.com/how-to-install-freeradius-and-daloradius-on-ubuntu/>
- <https://github.com/Nyr/openvpn-install>
- <https://go.btech.id/ops/skk/-/wikis/Riset/OpenVPN-Dnsmasq-FreeRADIUS>
- <https://linux.die.net/man/5/clients.conf>
- <https://serverfault.com/questions/1095415/daloradius-error-3-sql-error-rlm-sql-mysql-error-1054-unknown-column-acc>
- <https://sourceforge.net/p/daloradius/support-requests/27/>
- <https://www.osradar.com/openvpn-authentication-with-freeradius/>
