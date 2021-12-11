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
