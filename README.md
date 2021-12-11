# Jarkom Modul 5 - Firewall

## Topologi
![image](https://user-images.githubusercontent.com/49693862/145671710-5c220dd4-d845-4fe0-904a-bdff4bd546dc.png)

## CIDR Tree
![image](https://user-images.githubusercontent.com/49693862/145671719-86cb4f79-3c9b-4f82-a19d-0d827b12b432.png)


## Kalian juga diharuskan melakukan Routing agar setiap perangkat pada jaringan tersebut dapat terhubung.
Untuk itu kita harus menambahkan aturan routing di Foosha agar seluruh host dapat terhubung satu sama lain.
```
route add -net 192.208.4.0/22 gw 192.208.0.2
route add -net 192.208.16.0/21 gw 192.208.24.2
```

Sesuai dengan CIDR Tree, untuk B1, B2, B3 akan diarahkan ke 192.208.0.2 (Guanhao) dan B4, B5, B6 akan diarahkan ke 192.208.24.2 (Water7).

## Tugas berikutnya adalah memberikan ip pada subnet Blueno, Cipher, Fukurou, dan Elena secara dinamis menggunakan bantuan DHCP server. Kemudian kalian ingat bahwa kalian harus setting DHCP Relay pada router yang menghubungkannya.
Jadikan Jipangu sebagai DHCP Server, Water7 dan Guanhao sebagai DHCP Relay. Untuk config nya adalah sebagai berikut
- Jipangu (DHCP Server)
File `/etc/default/isc-dhcp-server` akan berisi:
```
INTERFACES="eth0"
```
Maka, DHCP Server akan melayani request dari interface eth0.

File `/etc/dhcp/dhcpd.conf` akan ditambahkan dengan:
```
# BLUENO
subnet 192.208.16.0 netmask 255.255.255.128 {
    range 192.208.16.2 192.208.16.127;
    option routers 192.208.16.1;
    option broadcast-address 192.208.16.128;
    option domain-name-servers 192.208.16.130;
    default-lease-time 360;
    max-lease-time 7200;
}

# CIPHER
subnet 192.208.20.0 netmask 255.255.252.0 {
    range 192.208.20.2 192.208.23.254;
    option routers 192.208.20.1;
    option broadcast-address 192.208.23.255;
    option domain-name-servers 192.208.16.130;
    default-lease-time 360;
    max-lease-time 7200;
}

# ELENA
subnet 192.208.4.0 netmask 255.255.254.0 {
    range 192.208.4.2 192.208.5.254;
    option routers 192.208.4.1;
    option broadcast-address 192.208.5.255;
    option domain-name-servers 192.208.16.130;
    default-lease-time 360;
    max-lease-time 7200;
}


# FUKUROU
subnet 192.208.6.0 netmask 255.255.255.0 {
    range 192.208.6.2 192.208.6.254;
    option routers 192.208.6.1;
    option broadcast-address 192.208.6.255;
    option domain-name-servers 192.208.16.130;
    default-lease-time 360;
    max-lease-time 7200;
}

subnet 192.208.16.128 netmask 255.255.255.248 {
    option routers 192.208.16.129;
}
```
Masing-masing subnet ditentukan akan dilayani oleh DHCP server beserta informasi lengkapnya seperti dns dan router.

- Water7 
File `/etc/default/isc-dhcp-relay` akan berisi:
```
# Defaults for isc-dhcp-relay initscript
# sourced by /etc/init.d/isc-dhcp-relay
# installed at /etc/default/isc-dhcp-relay by the maintainer scripts

#
# This is a POSIX shell fragment
#

# What servers should the DHCP relay forward requests to?
SERVERS="192.208.16.131"

# On what interfaces should the DHCP relay (dhrelay) serve DHCP requests?
INTERFACES="eth0 eth1 eth2 eth3"

# Additional options that are passed to the DHCP relay daemon?
OPTIONS=""
```
Masukkan ip Jipangu sebagai DHCP Server dan seluruh interface yang akan dilayani DHCP Request nya oleh DHCP Server. `eth0` dimasukkan karena akan mendapatkan request dari Guanhao.

- Guanhao
File `/etc/default/isc-dhcp-relay` akan berisi:
```
# Defaults for isc-dhcp-relay initscript
# sourced by /etc/init.d/isc-dhcp-relay
# installed at /etc/default/isc-dhcp-relay by the maintainer scripts

#
# This is a POSIX shell fragment
#

# What servers should the DHCP relay forward requests to?
SERVERS="192.208.24.2"

# On what interfaces should the DHCP relay (dhrelay) serve DHCP requests?
INTERFACES="eth0 eth1 eth2"

# Additional options that are passed to the DHCP relay daemon?
OPTIONS=""
```
Server diset ke Water7 yang nantinya akan diteruskan ke DHCP Server dan seluruh interface dimasukkan kecuali eth3 karena tidak akan menggunakan layanan DHCP.

## Agar topologi yang kalian buat dapat mengakses keluar, kalian diminta untuk mengkonfigurasi Foosha menggunakan iptables, tetapi Luffy tidak ingin menggunakan MASQUERADE.
Tambahkan rule iptable berikut di Foosha <br>
```iptables -t nat -A POSTROUTING ! -d 192.208.0.0/19 -o eth0 -j SNAT --to-source 192.168.122.63``` <br>
Rule diatas akan menggunakan chain *POSTROUTING* dengan destinasi yang bukan dari subnet di topologi kita maka akan keluar ke eth0 dengan source NAT (mengganti source IP pada header) ke 192.168.122.63 (IP Foosha di subnet NAT)

## Kalian diminta untuk mendrop semua akses HTTP dari luar Topologi kalian pada server yang merupakan DHCP Server dan DNS Server demi menjaga keamanan.
Tambahkan rule iptable berikut di Water7 (router terdekat dari DHCP dan DNS Server) <br>
```iptables -t mangle -A PREROUTING -p tcp -d 192.208.16.128/29 ! -s 192.208.0.0/19 --dport 80 -j DROP```<br>
Rule diatas akan menggunakan chain *PREROUTING* di *mangle* table bila protocol nya adalah tcp dan port 80 (http) dan bukan berasal dari subnet root topologi kita serta menuju ke subnet DHCP dan DNS Server, maka akan di drop.

## Karena kelompok kalian maksimal terdiri dari 3 orang. Luffy meminta kalian untuk membatasi DHCP dan DNS Server hanya boleh menerima maksimal 3 koneksi ICMP secara bersamaan menggunakan iptables, selebihnya didrop.
Tambahkan rule iptable berikut di Water7 (router terdakt dari DHCP dan DNS Server) <br>
```iptables -t mangle -A PREROUTING -p icmp -d 192.208.16.128/29 -m connlimit --connlimit-above 3 --connlimit-mask 0 -j DROP```<br>
Rule diatas akan menggunakan chain *PREROUTING* di *mangle* table yang akan melimit koneksi diatas 3 koneksi dan mask yang digunakan adalah /0. Maksud dari mask tersebut adalah berarti seluruh IP akan dimasukkan ke satu grup subnet /0. Koneksi lebih dari 3 akan di drop.

## Akses dari subnet Blueno dan Cipher ke Doriki hanya diperbolehkan pada pukul 07.00 - 15.00 pada hari Senin sampai Kamis.
Tambahkan rule iptable berikut di Water7
```
iptables -A PREROUTING -t mangle -d 192.208.16.130 -s 192.208.16.0/25,192.208.20.0/22 -m time --weekdays Mon,Tue,Wed,Thu --timestart 07:00 --timestop 15:00  -j ACCEPT
iptables -A PREROUTING -t mangle -d 192.208.16.130 -s 192.208.16.0/25,192.208.20.0/22 -m time --weekdays Mon,Tue,Wed,Thu -j DROP
```
Seluruh akses ke Doriki dari subnet Blueno dan Cipher pada hari Sen-Kam dari jam 07-15 akan di allow, bila diluar waktu tersebut maka akan di drop.

## Akses dari subnet Elena dan Fukuro hanya diperbolehkan pada pukul 15.01 hingga pukul 06.59 setiap harinya.
Tambahkan rule iptable berikut di Water7

> iptables -A PREROUTING -t mangle -d 192.208.16.130 -s 192.208.4.0/23,192.208.6.0/24 -m time --timestart 07:00 --timestop 15:00  -j DROP

Seluruh akses ke Doriki dari subnet Elena dan Fukuro dari jam 07-15 akan di drop maka request di jam 15.01 hingga 06.59 akan di allow.

## Karena kita memiliki 2 Web Server, Luffy ingin Guanhao disetting sehingga setiap request dari client yang mengakses DNS Server akan didistribusikan secara bergantian pada Jorge dan Maingate
Tambahkan rule iptable berikut di Guanhao

```
iptables -A PREROUTING -t nat -p tcp -d 192.208.7.3 --dport 80 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 192.208.7.2:80`
```

Seluruh akses http ke Maingate setiap 2 request akan dilakukan DNAT yaitu mengganti tujuan ke Jorge, sehingga respon yang didapatkan akan bergantian sesuai dari Maingate - Jorge - Maingate - Jorge.
