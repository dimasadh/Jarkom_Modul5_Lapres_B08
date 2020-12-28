# Jarkom_Modul5_Lapres

# Lapres Modul 5 Jarkom - Kelompok B08

Dimas Adhitya 05111840000015

Salim Bin Usman 05111840000104


Konfigurasi Topologi

![alt text](https://lh3.googleusercontent.com/x2coAvyUEeErA9ZfJtbqWCGSddAHvO63Ql99Z-yWwP0NErhShkoluJnQhL7PRjBbU-d1DNgj-6fSkQ9BHgs3DTx1C4x8IwCYMU8sgyIbUZTx08dIQQd0fkzQ_KbcRGMozmNc3Sxu) 

```
A1 = 2 host
A2 = 3 host
A3 = 211 host
A4 - 2 host
A5 = 201 host
```
### Pembagian subnet menggunakan VLSM

![alt text](https://lh5.googleusercontent.com/Ib6EPyOU79chFVRspuZK80nbbMrVXPvn1Kh8_PDZMXeElOX4lmPfe-O66yOuXX_K6FkwPc504vn0p8u2m-YnnG4WGX5KsnWwOc8cL3-cSNGlM8CGESdcZ0NwySxZ3ZPMhzEi7XaD)

dhcp pada server Mojokerto
```
// subnet A5 untuk SIDOARJO
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.2 192.168.1.254;
    option routers 192.168.1.1;
    option broadcast-address 192.168.1.255;
    option domain-name-servers 202.46.129.2;
    default-lease-time 600;
    max-lease-time 7200;
}

// subnet A3 untuk GRESIK
subnet 192.168.0.0 netmask 255.255.255.0 {
    range 192.168.0.2 192.168.0.254;
    option routers 192.168.0.1;
    option broadcast-address 192.168.1.255;
    option domain-name-servers 202.46.129.2;
    default-lease-time 600;
    max-lease-time 7200;
}
```

### Soal

1. Agar topologi yang kalian buat dapat mengakses keluar, kalian diminta untuk mengkonfigurasi
SURABAYA menggunakan iptables, namun Bibah tidak ingin kalian menggunakan
MASQUERADE. 

Pada UML Surabaya, tambahkan firewall :
```
iptables –t nat –A POSTROUTING –o eth0 –j SNAT --to 10.151.74.38 –s 192.168.0.0/16
```

2. Kalian diminta untuk mendrop semua akses SSH dari luar Topologi (UML) Kalian pada server
yang memiliki ip DMZ (DHCP dan DNS SERVER) pada SURABAYA demi menjaga keamanan.

Pada UML Surabaya, tambahkan firewall :
```
# menerima ssh dari dalam topologi
iptables -t mangle -A PREROUTING -s 192.168.0.0/22 -d 10.151.83.72/29 -p tcp --dport 22 -j ACCEPT
# drop ssh dari luar topologi
iptables -t mangle -A PREROUTING -d 10.151.83.72/29 -p tcp --dport 22 -j DROP
```

3. Karena tim kalian maksimal terdiri dari 3 orang, Bibah meminta kalian untuk membatasi DHCP
dan DNS server hanya boleh menerima maksimal 3 koneksi ICMP secara bersamaan yang berasal dari
mana saja menggunakan iptables pada masing masing server, selebihnya akan di DROP.

Pada UML Mojokerto (DHCP) dan Malang (DNS Server), tambahkan :
```
iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j DROP
```

kemudian kalian diminta untuk membatasi akses ke MALANG yang berasal dari SUBNET
SIDOARJO dan SUBNET GRESIK dengan peraturan sebagai berikut:
(4) Akses dari subnet SIDOARJO hanya diperbolehkan pada pukul 07.00 - 17.00 pada hari Senin
sampai Jumat.
(5) Akses dari subnet GRESIK hanya diperbolehkan pada pukul 17.00 hingga pukul 07.00 setiap
harinya.
Selain itu paket akan di REJECT.

Pada UML Malang
```
# a. untuk Sidarjo
iptables -I INPUT -s 192.168.1.0/24 -m time --weekdays Mo,Tu,We,Th,Fr --timestart 01:00 --timestop 17:00 -j ACCEPT
# b. untuk Gresik
iptables -I INPUT -s 192.168.0.0/24 -m time --timestart 17:00 --timestop 07:00 -j ACCEPT

# Selain rule diatas, maka subnet Sidoarjo dan Gresik akan direject
iptables -A INPUT -s 192.168.1.0/24 -j REJECT
iptables -A INPUT -s 192.168.0.0/24 -j REJECT
```

Karena kita memiliki 2 buah WEB Server, (6) Bibah ingin SURABAYA disetting sehingga setiap
request dari client yang mengakses DNS Server akan didistribusikan secara bergantian pada
PROBOLINGGO port 80 dan MADIUN port 80.

Pada UML Surabaya, tambahkan :
```
iptables -t nat -A PREROUTING -p tcp --dport 80 -d 10.151.83.74 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 192.168.4.3:80
iptables -t nat -A PREROUTING -p tcp --dport 80 -d 10.151.83.74 -m statistic --mode nth --every 1 --packet 0 -j DNAT --to-destination 192.168.4.2:80
```

7. Bibah ingin agar semua paket didrop oleh firewall (dalam topologi) tercatat dalam log pada setiap
UML yang memiliki aturan drop.

```
# Untuk di Surabaya, menggunakan table mangle
iptables -t mangle -N LOGGING
iptables -t mangle -A LOGGING -m limit --limit 2/min -j LOG --log-prefix "IPTables-Dropped: " --log-level 4
iptables -t mangle -A LOGGING -j DROP

#Untuk di Malang dan Mojokerto, menggunakan table  Filter
iptables -N LOGGING
iptables -A LOGGING -m limit --limit 2/min -j LOG --log-prefix "IPTables-Dropped: " --log-level 4
iptables -A LOGGING -j DROP
```
LOGGING diurutkan pada awal rule, kemudian untuk setiap rule DROP, diubah menjadi LOGGING.

