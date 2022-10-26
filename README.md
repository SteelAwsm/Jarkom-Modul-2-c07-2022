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

## Menjadikan WISE DNS Master
Install bind9 menggunakan `apt- get update` lalu `apt get install bind9 -y`, setelah selesai install bind9, buka file `nano /etc/bind/named.conf.local`, setelah itu konfigurasikan file:

```
zone "wise.c07.com" {
 type master;
 file "/etc/bind/jarkom-c07/wise.c07.com";
};
```
setelah itu restart bind9 dengan `service bind9 restart`

## membuat berlint jadi DNS Slave

Dalam terminal WISE, buka file `/etc/bind/named.conf.local` dan tambahkan kode:
```
zone "wise.c07.com" {
        type master;
        notify yes;
        also-notify { 10.13.2.2; };
        allow-transfer { 10.13.2.2; };
        file "/etc/bind/jarkom-c07/wise.c07.com";
};
```
setelah itu restart bind9 dengan `service bind9 restart` </br>

Dalam terminal Berlint, install bind9 dengan menggunakan `apt-get update` setelah itu `apt-get install bind9` </br>

lalu didalam file `/etc/bind/named.conf.local`, tambahkan kode:
```
zone "wise.c07.com" {
 type slave;
 masters { 10.13.3.2; };                   
 file "/var/lib/bind/wise.c07.com";
};
```
lalu restart bind9 pada Berlint dengan `service bind9 restart`

## membuat Website utama www.c07.com

Dalam terminal WISE buat folder dalam directory etc/bind dengan `mkdir /etc/bind/jarkom-c07` </br>

copy dan rename db.local menjadi wise.c07.com dengan cara `cp /etc/bind/db.local /etc/bind/jarkom-c07/wise.c07.com`

buka file dengan `nano /etc/bind/jarkom-c07/wise.c07.com` </br>

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


## Menjadikan SSS dan Garden menjadi Client

SSS dan Garden dijadikan client dengan mengedit file resolv.conf dengan menambahkan atau mengedit di dalam file menggunakan `nano etc/resolv.conf` dalam domain SSS serta Garden, lalu diisi dengan:

```
nameserver 10.13.3.2
```
untuk memastikan bahwa DNS sudah terkoneksi, dalam domain SSS dan Garden, ping domain wise.c07.com dengan command:

```
ping wise.c07.com -c 5
```
