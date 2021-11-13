# Jarkom-Modul-3-E11-2021
## Anggota Kelompok :
| Nama | NRP |
|------|-----|
|Rizqi Rifaldi|05111940000068|
|Yusuf Anfasya|05111940000077|
|Muhammad Subhan|05111940000204|

Pada praktikum Modul 3 ini kami diperintahkan untuk menjawab 13 soal mengenai DHCP dan Proxy.

Luffy yang sudah menjadi Raja Bajak Laut ingin mengembangkan daerah kekuasaannya dengan membuat peta seperti berikut:

[![Whats-App-Image-2021-11-08-at-16-06-44.jpg](https://i.postimg.cc/Wz4Y8Tmt/Whats-App-Image-2021-11-08-at-16-06-44.jpg)](https://postimg.cc/mz0S2x5G)

### Soal 1
Luffy bersama Zoro berencana membuat peta tersebut dengan kriteria EniesLobby sebagai DNS Server, Jipangu sebagai DHCP Server, Water7 sebagai Proxy Server.

Untuk membuat EniesLobby sebagai DNS Server, kita perlu menginstall bind9 dengan menggunakan command :

```apt-get install bind9```

Untuk membuat Jipangu menjadi DHCP server, kita perlu menginstall isc-dhcp-server pada server Jipangu dengan menggunakan command :

```apt-get install isc-dhcp-server```

Kemudian untuk langkah selanjutnya akan dilanjutkan pada nomor yang akan datang.

Untuk membuat Water7 menjadi Proxy Server, maka kita perlu menginstall squid3 pada server Water7 menggunakan command :

```apt-get install squid```

Untuk kelanjutannya akan ada pada nomor selanjutnya.

### Soal 2
Foosha sebagai DHCP Relay.

Untuk membuat Foosha sebagai DHCP relay, kita perlu menginstall DHCP relay dengan menggunakan command :

```apt-get install isc-dhcp-relay```

kemudian pada foosha kita akan membuka file ```/etc/default/isc-dhcp-relay``` dan menambahkan menjadi :

```
# What servers should the DHCP relay forward requests to?
SERVERS="192.205.2.4"

# On what interfaces should the DHCP relay (dhrelay) serve DHCP requests?
INTERFACES="eth1 eth2 eth3"

# Additional options that are passed to the DHCP relay daemon?
OPTIONS=""
```

SERVERS diatas akan menunjuk ke IP dari Jipangu yaitu ```192.205.2.4``` sebagai DHCP Server, kemudian INTERFACES akan menunjuk pada interface yang akan dilayani oleh DHCP relay, yaitu eth1 eth2 dan eth3.

jika diterapkan pada script akan menjadi :

[![Whats-App-Image-2021-11-11-at-19-39-53.jpg](https://i.postimg.cc/B6CVvm9v/Whats-App-Image-2021-11-11-at-19-39-53.jpg)](https://postimg.cc/r0KJnCp6)

setelah itu kita akan melakukan restart pada DHCP relay dengan menggunakan command berikut :

```/etc/init.d/isc-dhcp-relay restart```

### Soal 3-6 
Ada beberapa kriteria yang ingin dibuat oleh Luffy dan Zoro, yaitu:
1. Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server.
2. Client yang melalui Switch1 mendapatkan range IP dari 192.205.1.20 - 192.205.1.99 dan 192.205.1.150 - 192.205.1.169 (3)
3. Client yang melalui Switch3 mendapatkan range IP dari 192.205.3.30 - 192.205.3.50 (4)
4. Client mendapatkan DNS dari EniesLobby dan client dapat terhubung dengan internet melalui DNS tersebut. (5)
5. Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 6 menit sedangkan pada client yang melalui Switch3 selama 12 menit. Dengan    waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 120 menit. (6)

Untuk meminjamkan konfigurasi IP ke client, kita menggunakan DHCP. 
Pertama-tama kita akan melakukan konfigurasi pada server Jipangu. Kita akan membuka file ```/etc/default/isc-dhcp-server``` dan menambahkan eth0 pada ```INTERFACES=""```

Setelah itu kita akan membuka file ```/etc/dhcp/dhcpd.conf``` dan menambahkan konfigurasi untuk setiap subnet.

Konfigurasi pada subnet 192.205.1.0 (Switch 1) :
```
subnet 192.205.1.0 netmask 255.255.255.0 {
    range 192.205.1.20 192.205.1.99;
    range 192.205.1.150 192.205.1.169;
    option routers 192.205.1.1;
    option broadcast-address 192.205.1.255;
    option domain-name-servers 192.205.2.2;
    default-lease-time 360;
    max-lease-time 7200;
}
```

Konfigurasi pada subnet 192.205.3.0 (Switch 3) :
```
subnet 192.205.3.0 netmask 255.255.255.0 {
    range 192.205.3.30 192.205.3.50;
    option routers 192.205.3.1;
    option broadcast-address 192.205.3.255;
    option domain-name-servers 192.205.2.2;
    default-lease-time 720;
    max-lease-time 7200;
}
```
Kemudian, kita akan merestart isc-dhcp menggunakan command ```service isc-dhcp-server restart```.

Setelah itu, pada setiap client kita akan membuka file ```/etc/network/interfaces``` dan menambahkan :

```
auto eth0
iface eth0 inet dhcp
```

Untuk mengecek apakah sudah benar, kita bisa melihat ip pada salah satu Client, contoh disini adalah Loguetown yang terhubung pada Switch 1 :

[![Whats-App-Image-2021-11-11-at-19-32-33.jpg](https://i.postimg.cc/ZKvS5hRs/Whats-App-Image-2021-11-11-at-19-32-33.jpg)](https://postimg.cc/vcyj0k55)

[![Whats-App-Image-2021-11-11-at-19-34-25.jpg](https://i.postimg.cc/mr8qSdkF/Whats-App-Image-2021-11-11-at-19-34-25.jpg)](https://postimg.cc/kVRfJvxn)

Pada gambar tersebut, dapat dilihat bahwa Loguetown dipinjami IP ```192.205.1.22``` , ini masih sesuai dengan range IP pada konfigurasi Switch 1 yaitu 192.205.1.20 - 192.205.1.99 dan 192.205.1.150 - 192.205.1.169, kemudian dapat dilihat juga bahwa ```default-lease-time``` nya adalah 360 second atau 6 menit.

Kemudian kita juga bisa melihat pada client di Switch 3, contoh disini adalah TottoLand :

[![Whats-App-Image-2021-11-11-at-19-37-29.jpg](https://i.postimg.cc/zBhQ1qpT/Whats-App-Image-2021-11-11-at-19-37-29.jpg)](https://postimg.cc/ftwCcnGb)

[![Whats-App-Image-2021-11-11-at-19-38-26.jpg](https://i.postimg.cc/CMj2PLRg/Whats-App-Image-2021-11-11-at-19-38-26.jpg)](https://postimg.cc/bdwLdP0C)

Pada gambar tersebut, dapat dilihat bahwa TottoLand dipinjami IP ```192.205.3.32```, ini masih sesuai dengan range IP pada konfigurasi Switch 3 yaitu 192.205.3.30 - 192.205.3.50, kemudian dengan ```default-time-lease``` nya adalah 720 second atau 12 menit.

Untuk lebih jelasnya bagian mana saja yang menjadi jawaban setiap nomornya, akan dijabarkan dibawah.

#### Soal 3
Client yang melalui Switch1 mendapatkan range IP dari 192.205.1.20 - 192.205.1.99 dan 192.205.1.150 - 192.205.1.169

Untuk menjawab soal nomor 3, pada konfigurasi DHCP server di Jipangu, ada pada subnet 192.205.1.0 (Switch 1) pada bagian :

```
 range 192.205.1.20 192.205.1.99;
 range 192.205.1.150 192.205.1.169;
```
Ini akan mengeset IP yang akan diberikan dari DHCP server ke Client pada Switch 1 akan ada pada range 192.205.1.20 - 192.205.1.99 dan 192.205.1.150 - 192.205.1.169

#### Soal 4
Client yang melalui Switch3 mendapatkan range IP dari 192.205.3.30 - 192.205.3.50

Untuk menjawab nomor 4, pada konfigurasi DHCP server di Jipangu, ada pada subnet 192.205.3.0 (Switch 3) pada bagian :

```
range 192.205.3.30 192.205.3.50;
```

Ini akan mengeset IP yang akan diberikan dari DHCP server ke Client pada Switch 3 akan ada pada range 192.205.3.30 - 192.205.3.50

#### Soal 5
Client mendapatkan DNS dari EniesLobby dan client dapat terhubung dengan internet melalui DNS tersebut.

Untuk menjawab nomor 5, pada konfigurasi DHCP server di Jipangu, ada pada subnet 192.205.1.0 (Switch 1) dan 192.205.3.0 (Switch 3) pada bagian :

```
option domain-name-servers 192.205.2.2;
```
Kemudian kita menambahkan DNS Forwarder seperti pada gambar berikut :

[![Whats-App-Image-2021-11-13-at-17-19-03.jpg](https://i.postimg.cc/s29Pd0Fn/Whats-App-Image-2021-11-13-at-17-19-03.jpg)](https://postimg.cc/940wybQT)

Potongan source code tersebut akan menyebabkan client mendapat DNS pada EniesLobby dengan IP 192.205.2.2 dan dapat terhubung dengan internet melalui DNS tersebut.

#### Soal 6
Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 6 menit sedangkan pada client yang melalui Switch3 selama 12 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 120 menit.

Untuk menjawab nomor 6, kita akan mengeset ```default-lease-time``` dan ```max-lease-time``` pada konfigurasi DHCP server pada Jipangu untuk subnet 192.205.1.0 (Switch 1) dan 192.205.3.0 (Switch 3), potongan konfigurasinya adalah sebagai berikut :

```
#Switch 1
default-lease-time 360;
max-lease-time 7200;

#Switch 3
default-lease-time 720;
max-lease-time 7200;
```

### Soal 7
Luffy dan Zoro berencana menjadikan Skypie sebagai server untuk jual beli kapal yang dimilikinya dengan alamat IP yang tetap dengan IP 192.205.3.69 

Untuk membuat Fixed Address pada Skypie, pertama kita akan melakukan konfigurasi pada DHCP server, yaitu Jipangu dengan cara membuka file ```/etc/dhcp/dhcpd.conf``` dan menambahkan : 

```
host Skypie {
    hardware ethernet be:98:0a:07:47:46;
    fixed-address 192.205.3.69 ;
} 
``` 
dan melakukan restart pada isc-dhcp dengan command ```service isc-dhcp-server restart```

Kemudian, kita akan melakukan konfigurasi pada Skypie dengan membuka file ```/etc/network/interfaces``` dan menambahkan :

```
hwaddress ether be:98:0a:07:47:46
```

Untuk mengecek apakah sudah benar, kita bisa melihat pada Skypie apakah IP nya sekarang adalah ```192.205.3.69```, bisa dilihat pada gambar berikut :

[![Whats-App-Image-2021-11-11-at-19-59-40.jpg](https://i.postimg.cc/YSqDj1R2/Whats-App-Image-2021-11-11-at-19-59-40.jpg)](https://postimg.cc/0zF0BKhh)

[![Whats-App-Image-2021-11-11-at-20-01-26.jpg](https://i.postimg.cc/bYTTSGGs/Whats-App-Image-2021-11-11-at-20-01-26.jpg)](https://postimg.cc/K41Ld8D2)

Pada gambar tersebut, dapat dilihat bahwa Skypie selalu dipinjami IP yang sama oleh DHCP server, yaitu ```192.205.3.69```, maka konfigurasinya sudah benar.


### Soal 8
Pada Loguetown, proxy harus bisa diakses dengan nama jualbelikapal.e11.com dengan port yang digunakan adalah 5000:

Untuk Solusi dalam Hal diatas maka kita akan melakukan beberapa config sebagai berikut:

#### Water7 

Command di `.bashrc` sebagi berikut:
```
echo 'nameserver 192.168.122.1' > /etc/resolv.conf
apt-get update
apt-get install squid -y
apt-get install apache2-utils -y
```

Sebelumnya kita harus melakukan backup konfigurasi default squid dengan command

```
mv /etc/squid/squid.conf /etc/squid/squid.conf.bak
```

Selanjutnya yaitu dengan melakukan konfigurasi `squid` pada file `etc/squid/squid.conf` dengan sebagai berikut:
```
http_port 5000
visible_hostname jualbelikapal.e11.com
http_access allow all
```
Untuk dapat menjalankan semua command tersebut maka dilakukan command

`service squid restart`

#### Enieslobby

kita mengatur zone dengan command
```
 zone "jualbelikapal.e11.com" {
        type master;
        file "/etc/bind/kaizoku/jualbelikapal.e11.com";
};
```
Kemudian pada melakukan command pada file `"/etc/bind/kaizoku/jualbelikapal.e11.com"` sebagai berikut
```
$TTL    604800
@       IN      SOA     jualbelikapal.e11.com. root.jualbelikapal.e11.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      jualbelikapal.e11.com.
@       IN      A       192.205.2.3
www     IN      CNAME   jualbelikapal.e11.com.
```

Selanjutnya agar command dapat berjalan maka lakukan command `service bind9 restart`

#### Loguetown

Pada `script.sh` kita menambah command untuk agar dapat berjalan di port 5000
```
auto eth0
iface eth0 inet dhcp '>/etc/network/interfaces

export http_proxy="http://jualbelikapal.e11.com:5000"
```

lalu untuk mengaktifkan proxy maka jalankan command `export http_proxy="http://jualbelikapal.e11.com:5000"`

kemudian untuk mengetahui status proxy kita jalankan command `env | grep -i proxy` untuk dapat memastikan proxy sudah berjalan di port 5000

[![image.png](https://i.postimg.cc/pX5XG67t/image.png)](https://postimg.cc/561VjnbP)


### Soal 9
Agar transaksi jual beli lebih aman dan pengguna website ada dua orang, proxy dipasang autentikasi user proxy dengan enkripsi MD5 dengan dua username, yaitu luffybelikapale11 dengan password luffy_e11 dan zorobelikapale11 dengan password zoro_e11 

Solusinya yaitu:

#### Water7
 
Kita lakukan `apt-get update` dan `apt-get install apache2-utils`

Selanjutnya lakukan command `htpasswd -cm /etc/squid/passwd luffybelikapale11`, `-c` untuk membuat file baru dan `m` yaitu melakukan enkripsi file dengan `md5`. 

selanjutnya input password `luffye11`

Untuk selanjutnya yaitu command `htpasswd -cm /etc/squid/passwd zorobelikapale11`. dan passwordnya `zoroe11`

kemudian pada file `/etc/squid/squid.conf` dengan command berikut: 

```
http_port 5000
visible_hostname jualbelikapal.e11.com
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Proxy
auth_param basic credentialsttl 2 hours
auth_param basic casesensitive on
acl USERS proxy_auth REQUIRED
```

Selanjutnya Kita lakukan `service squid restart`



### Soal 10
Transaksi jual beli tidak dilakukan setiap hari, oleh karena itu akses internet dibatasi hanya dapat diakses setiap hari Senin-Kamis pukul 07.00-11.00 dan setiap hari Selasa-Jum’at pukul 17.00-03.00 keesokan harinya (sampai Sabtu pukul 03.00)

Untuk Solusi Soal ini maka:

#### Water7

Kita melakukan command pada `script.sh` yaitu:

```
echo '

acl HARI_SATU time MTWH 07:00-11:00
acl HARI_DUA time TWHF 17:00-24:00
acl HARI_KETIGA time WHFA 00:00-03:00

acl google dstdomain google.com
http_access deny google
deny_info 301:http://super.franky.e11.com%R google'>/etc/squid/acl.conf

echo 'include /etc/squid/acl.conf
http_port 5000
```
Sehingga File acl.conf terbentuk.

Selanjutnya pada `/etc/squid/squid.conf` kita tambahkan command agar jadwal berjalan semestinya :

```
http_access allow HARI_SATU USERS
http_access allow HARI_DUA USERS
http_access allow HARI_KETIGA USERS
```

Kemudian lakukan `service squid restart`


### Soal 11
Agar transaksi bisa lebih fokus berjalan, maka dilakukan redirect website agar mudah mengingat website transaksi jual beli kapal. Setiap mengakses google.com, akan diredirect menuju super.franky.e11.com dengan website yang sama pada soal shift modul 2. Web server super.franky.e11.com berada pada node Skypie

Solusi untuk hal ini maka:

#### Enieslobby

pada  File `/etc/bind/named.conf.local` kita menambah command sebagai berikut:

```
zone "franky.e11.com" {
        type master;
        file "/etc/bind/kaizoku/franky.e11.com";
};
```
Kemudian pada file  `/etc/bind/kaizoku/franky.e11.com` kita menambahkan command agar dapat melakukan redirect
```
$TTL    604800
@       IN      SOA     franky.e11.com. root.franky.e11.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      franky.e11.com.
@       IN      A       192.205.3.69
www     IN      CNAME   franky.e11.com.
super   IN      A       192.205.3.69
www.super       IN      A   192.205.3.69
```

Selajutnya lakukan `service bind9 restart`

#### Skypie

Kita lakukan Command pada `script.sh` agar dapat berjalan otomatis

```
echo 'auto eth0
iface eth0 inet dhcp
hwaddress ether 16:15:db:d5:a1:6a'>/etc/network/interfaces

apt-get update
apt-get install apache2 -y
service apache2 start
apt-get install wget
apt-get install unzip


cd /etc/apache2/sites-available
cp 000-default.conf super.franky.e11.com.conf

echo ' <VirtualHost *:80>
       
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/super.franky.e11.com
        ServerName super.franky.e11.com
        serverAlias www.super.franky.e11.com 


        <Directory /var/www/super.franky.e11.com>
                Options +Indexes
                AllowOverride All
        </Directory>

        <Directory /var/www/super.franky.e11.com/error>
                Options -Indexes

        </Directory>

        <Directory /var/www/super.franky.e11.com/public/js>
        Options +Indexes
         </Directory>

        Alias "/js" "/var/www/super.franky.e11.com/public/js"
        ErrorDocument 404 /error/404.html

</VirtualHost>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

'>/etc/apache2/sites-available/super.franky.e11.com.conf
cd
cd super.franky
cp -r error /var/www/super.franky.e11.com
cp -r public /var/www/super.franky.e11.com
cd
cd /etc/apache2/sites-available
a2ensite super.franky.e11.com.conf
a2enmod rewrite
cd
service apache2 restart

```


### Soal 12
Saatnya berlayar! Luffy dan Zoro akhirnya memutuskan untuk berlayar untuk mencari harta karun di super.franky.e11.com. Tugas pencarian dibagi menjadi dua misi, Luffy bertugas untuk mendapatkan gambar (.png, .jpg), sedangkan Zoro mendapatkan sisanya. Karena Luffy orangnya sangat teliti untuk mencari harta karun, ketika ia berhasil mendapatkan gambar, ia mendapatkan gambar dan melihatnya dengan kecepatan 10 kbps

pada soal nomor 12 kita mendeklarasikan luffy menggunakan regex untuk mendownload file .png dan .jpg lalu kita menambahkan beberapa konfigurasi didalam ```/etc/squid/bandwidth.conf``` untuk membuat delay pool sebanyak dua yaitu luffy dan zoro , lalu kita menggunakan delay parameter untuk membatasi kecepatan download file .jpg dan .png menjadi 10000/10000 dimana dibatasi menjadi 10 kbps,sedangkan untuk zoro tidak dilakukan pembatasan kecepatan sehingga isi konfigurasi tersebut seperti berikut:

![image](https://user-images.githubusercontent.com/77099292/141444302-f314bd16-2ba7-4127-9229-3e0a4bfc9996.png)

dan hasilnya sebagai berikut :

![image](https://user-images.githubusercontent.com/77099292/141444622-d3dce7e9-cb78-4100-8adf-cc919f03d043.png)



### Soal 13
Sedangkan, Zoro yang sangat bersemangat untuk mencari harta karun, sehingga kecepatan kapal Zoro tidak dibatasi ketika sudah mendapatkan harta yang diinginkannya.

sedangkan untuk zoro tidak dilakukan pembatasan kecepatan sehingga file akan langsung terdownload tanpa ada pembatasan kecepatan sama sekali untuk konfigurasi dilakukan bersamaan dengan luffy dikarenakan menggunakan delay pool sebanyak 2, untuk konfigurasinya sebagai berikut :

![image](https://user-images.githubusercontent.com/77099292/141444873-b99e450d-a358-40ab-87bb-a9ad34cab082.png)

dan untuk hasilnya sebagai berikut :

![image](https://user-images.githubusercontent.com/77099292/141445172-3243cbdb-d3bf-4981-95fb-68ee0a173c71.png)

