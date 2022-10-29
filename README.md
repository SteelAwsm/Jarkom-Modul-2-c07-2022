# Jarkom-Modul-2-c07-2022

## Soal 1

[Prefix IP] : 10.13 </br>

Membuat topologi dan konfigurasi mirip dengan modul GNS3, dengan catatan nama node yang digunakan sama dengan soal, setelah itu menambah (WISE) node baru serta mengkonfigurasinya dengan:

```
auto eth0
iface eth0 inet static
 address 10.13.3.2
 netmask 255.255.255.0
 gateway 10.13.3.1
```
langkah selanjutnya mengkoneksikan seluruh node dengan internet, dengan cara memasukkan kode di bawah kedalam terminal Ostania:
```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.13.0.0/16
```
setelah itu masukkan 
```
cat /etc/resolv.conf
```
[IP DNS] = 192.168.122.1 </br>
untuk mengetahui IP DNS, lalu masukkan kode ini didalam seluruh terminal node:
```
echo nameserver 192.168.122.1 > /etc/resolv.conf
```
untuk mengkoneksikan semua node kepada internet. </br>
semua command yang dilakukan untuk node-node ditulis kedalam file /root/.bashrc untuk menyimpan pengaturan dan menginisasi kembali secara otomatis jika proyek dimatikan </br>

## Soal 2
pertama, menjadikan WISE DNS Master </br>

Dalam Node `Wise` </br>
Install bind9 menggunakan `apt-get update` lalu `apt-get install bind9 -y`, setelah selesai install bind9, buka file `nano /etc/bind/named.conf.local`, setelah itu konfigurasikan file:

```
zone "wise.c07.com" {
 type master;
 file "/etc/bind/jarkom-c07/wise.c07.com";
};
```

setelah itu restart bind9 dengan `service bind9 restart` </br>

lalu, membuat website wise.c07.com </br>
Dalam terminal WISE buat folder dalam directory etc/bind dengan `mkdir /etc/bind/wise` </br>

copy dan rename db.local menjadi wise.c07.com dengan cara `cp /etc/bind/db.local /etc/bind/wise/wise.c07.com`

buka file dengan `nano /etc/bind/wise/wise.c07.com` </br>

lalu edit file menjadi seperti ini:
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     wise.c07.com. root.wise.c07.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      wise.c07.com.
@       IN      A       10.13.3.2       ; 
@       IN      AAAA    ::1
```
setelah itu restart bind9 dengan `service bind9 restart`

## Soal 3

Dalam node `Wise` </br>
Mengedit `file wise.c07.com` dengan membuka file `nano /etc/bind/wise/wise.c07.com` </br>

lalu edit file menjadi seperti ini:
```
;
; BIND data file for local loopback interface
;
\$TTL    604800
@       IN      SOA     wise.c07.com. root.wise.c07.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@               IN      NS      wise.c07.com.
@               IN      A       10.13.3.2 ; IP Wise
www             IN      CNAME   wise.c07.com.
eden           IN      A        10.13.2.3 ; IP Eden
www.eden       IN      CNAME   eden.wise.c07.com.
```

setelah itu restart bind9 dengan `service bind9 restart`

## Soal 4

Dalam node `Wise` </br>
mengedit file `named.conf.local` dengan membuka file `nano /etc/bind/db.local`  </br>

lalu edit file menjadi seperti ini:
```
zone "wise.c07.com" {
        type master;
        file "/etc/bind/wise/wise.c07.com";
};

zone "3.13.10.in-addr.arpa" {
        type master;
        file "/etc/bind/wise/3.13.10.in-addr.arpa";
};
```

lalu membuat file `3.13.10.in-addr.arpa` serta membukanya dengan `nano /etc/bind/wise/3.13.10.in-addr.arpa`

lalu tulis file menjadi seperti ini:
```
\$TTL    604800
@       IN      SOA     wise.c07.com. root.wise.c07.com. (
                                2       ; Serial
                           604800       ; Refresh
                            86400       ; Retry
                          2419200       ; Expire
                           604800 )     ; Negative Cache TTL
;
3.13.10.in-addr.arpa.   IN      NS      wise.c07.com.
3                       IN      PTR     wise.c07.com.
```

setelah itu restart bind9 dengan `service bind9 restart`

## Soal 5
Dalam node `Wise` </br>

mengedit file `named.conf.local` dengan membuka file `/etc/bind/db.local`  </br>

lalu edit file menjadi seperti ini:
```
zone "wise.c07.com" {
        type master;
        notify yes;
        also-notify {10.13.2.2;};  //IP Berlint tanpa tanda petik
        allow-transfer {10.13.2.2;}; //IP Berlint tanpa tanda petik
        file "/etc/bind/wise/wise.c07.com";
};

zone "3.13.10.in-addr.arpa" {
        type master;
        file "/etc/bind/wise/3.13.10.in-addr.arpa";
};
```
setelah itu restart bind9 dengan `service bind9 restart` </br>

Dalam node `berlint` </br>

Install bind9 menggunakan `apt-get update` lalu `apt-get install bind9 -y`, setelah selesai install bind9, buka file `nano /etc/bind/named.conf.local`, setelah itu konfigurasikan file:
```
zone "wise.c07.com" {
        type slave;
        masters { 10.13.3.2; }; //IP Wise tanpa tanda petik
        file "/var/lib/bind/wise.c07.com";
};
```
setelah itu restart bind9 dengan `service bind9 restart` </br>

Dalam node `SSS` </br>

Install dnsutils serta lynx menggunakan `apt-get update` lalu `apt-get install dnsutils -y` lalu `apt-get install lynx -y`

lalu membuka dan mengedit file `resolv.conf` dengan cara membuka file dengan command `nano /etc/resolv.conf` </br>

lalu edit file menjadi seperti ini:
```
nameserver 10.13.3.2 //Wise
nameserver 10.13.2.2 //berlint
nameserver 10.13.2.3 //eden
```

Dalam node `Garden` </br>

Install dnsutils serta lynx menggunakan `apt-get update` lalu `apt-get install dnsutils -y` lalu `apt-get install lynx -y`

lalu membuka dan mengedit file `resolv.conf` dengan cara membuka file dengan command `nano /etc/resolv.conf` </br>

lalu edit file menjadi seperti ini:
```
nameserver 10.13.3.2 //Wise
nameserver 10.13.2.2 //berlint
nameserver 10.13.2.3 //eden
```




## Soal 6
Dalam node `Wise` </br>
Mengedit `file wise.c07.com` dengan membuka file `nano /etc/bind/wise/wise.c07.com` </br>

lalu edit file menjadi seperti ini:
```
\$TTL    604800
@       IN      SOA     wise.c07.com. root.wise.c07.com. (
                                2       ; Serial
                           604800       ; Refresh
                            86400       ; Retry
                          2419200       ; Expire
                           604800 )     ; Negative Cache TTL
;
@               IN      NS      wise.c07.com.
@               IN      A       10.13.2.3 ; IP Eden
www             IN      CNAME   wise.c07.com.
eden           IN      A       10.13.2.3 ; IP Eden
www.eden       IN      CNAME   eden.wise.c07.com.
ns1             IN      A       10.13.2.2; IP Berlint
operation           IN      NS      ns1
```

setelah itu membuka file bernama `named.conf.options` dengan cara `nano /etc/bind/named.conf.options` </br>

lalu edit file menjadi seperti ini:
```
options {
        directory \"/var/cache/bind\";

        allow-query{any;};
        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};
```

setelah itu membuka file bernama `named.conf.local` dengan cara `nano /etc/bind/named.conf.local` </br>

lalu edit file menjadi seperti ini:
```
zone "wise.c07.com" {
        type master;
        //notify yes;
        //also-notify {10.13.2.2;};  // IP Berlint tanpa tanda petik
        file "/etc/bind/wise/wise.c07.com";
        allow-transfer {10.13.2.2;}; // IP Berlint tanpa tanda petik
};

zone "3.13.10.in-addr.arpa" {
        type master;
        file "/etc/bind/wise/3.13.10.in-addr.arpa";
};
```

setelah itu restart bind9 dengan `service bind9 restart`

Dalam node `Berlint` </br>
setelah itu membuka file bernama `named.conf.options` dengan cara `nano /etc/bind/named.conf.options` </br>

lalu edit file menjadi seperti ini:
```
options {
        directory \"/var/cache/bind\";
        allow-query{any;};
        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};
```

setelah itu membuka file bernama `named.conf.local` dengan cara `nano /etc/bind/named.conf.local` </br>

lalu edit file menjadi seperti ini:
```
zone "wise.c07.com" {
        type slave;
        masters { 10.13.3.2; }; //IP Wise tanpa tanda petik
        file "/var/lib/bind/wise.c07.com";
};

zone "operation.wise.c07.com"{
        type master;
        file "/etc/bind/operation/operation.wise.c07.com";
};
```

setelah itu membuat directory baru dengan cara `mkdir /etc/bind/operation`

setelah itu membuat file bernama `operation.wise.c07.com` dengan cara `nano /etc/bind/operation/operation.wise.c07.com` </br>


lalu buat file menjadi seperti ini:
```
\$TTL    604800
@       IN      SOA     operation.wise.c07.com. root.operation.wise.c07.com. (
                                2      ; Serial
                           604800      ; Refresh
                            86400      ; Retry
                          2419200      ; Expire
                           604800 )    ; Negative Cache TTL
;
@               IN      NS      operation.wise.c07.com.
@               IN      A       10.13.2.3       ; IP Eden
www             IN      CNAME   operation.wise.c07.com.
```

setelah itu restart bind9 dengan `service bind9 restart`


## Soal 7

Dalam node `Berlint` </br>

edit file bernama `operation.wise.c07.com` dengan cara `nano /etc/bind/operation/operation.wise.c07.com` </br>

lalu edit file menjadi seperti ini:
```
\$TTL    604800
@       IN      SOA     operation.wise.c07.com. root.operation.wise.c07.com. (
                                 2      ; Serial
                           604800      ; Refresh
                            86400      ; Retry
                          2419200      ; Expire
                           604800 )    ; Negative Cache TTL
;
@               IN      NS      operation.wise.c07.com.
@               IN      A       10.13.2.3       ; IP Eden
www             IN      CNAME   operation.wise.c07.com.
strix           IN      A       10.13.2.3       ; IP Eden
www.strix       IN      CNAME   strix.operation.wise.c07.com.
```

setelah itu restart bind9 dengan `service bind9 restart`

## Soal 8
Dalam node `Eden` </br>
Install apache2 serta memulai service nya menggunakan `apt-get update` lalu `apt-get install apache2 -y` lalu `service apache2 start`, setelah itu install php dan libapache2-mod-php7.0 dengan cara `apt-get install php -y` lalu `apt-get install libapache2-mod-php7.0 -y` lalu install ca-certificates openssl, unzip, serta git dengan cara `service apache2` lalu `apt-get install ca-certificates openssl -y` lalu `apt-get install unzip -y` lalu `apt-get install git -y` </br>

lalu clone git dan unzip dengan cara dengan cara 
`clone https://github.com/SteelAwsm/jarkomresourcesC07.git`
lalu `unzip -o /root/jarkomresourcesC07/\*.zip -d /root/jarkomresourcesC07`

lalu buka dan tulis di file `wise.c07.com.conf` dengan cara `nano /etc/apache2/sites-available/wise.c07.com.conf` </br>

lalu isi file menjadi seperti ini:
```
<VirtualHost *:80>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/wise.c07.com
        ServerName wise.c07.com
        ServerAlias www.wise.c07.com

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

lalu lakukakn `a2ensite wise.c07.com` lalu buat directory dengan cara `mkdir /var/www/wise.c07.com` setelah itu copy directory dengan cara `cp -r /root/jarkomresourcesC07/wise/. /var/www/wise.c07.com`

lalu restart service apache22 dengan cara `service apache2 restart`

## Soal 9
Dalam node `Eden` </br>

Gunakan command `a2enmod rewrite` lalu restart service apache2 dengan `service apache2 restart` setelah itu tulis di dalam file `.htaccess` dengan cara `nano /var/www/wise.c07.com/.htaccess`  </br>

lalu tulis file menjadi seperti ini:
```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule (.*) /index.php/\$1 [L]
```

setelah itu tulis di dalam file `wise.c07.com.conf` dengan cara `nano /etc/apache2/sites-available/wise.c07.com.conf` </br>

lalu tulis file menjadi seperti ini:
```
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/wise.c07.com
        ServerName wise.c07.com
        ServerAlias www.wise.c07.com

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined

        <Directory /var/www/wise.c07.com>
                Options +FollowSymLinks -Multiviews
                AllowOverride All
        </Directory>
</VirtualHost>
```

setelah itu restart service apache dengan cara `service apache restart`