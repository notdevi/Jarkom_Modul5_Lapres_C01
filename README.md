# Jarkom_Modul5_Lapres_C01
Repository Lapres Praktikum Jarkom Modul 5

## NAMA ANGGOTA KELOMPOK :

**1. Devi Hainun Pasya (05111840000014)**

**2. Kevin Christian Hadinata (05111840000066)**

## PEMBAHASAN SOAL

### Soal (A) 

Tugas pertama yaitu membuat topologi jaringan sesuai dengan rancangan yang diberikan seperti di bawah ini :

![](img/)

Keterangan : 
<li> SURABAYA diberikan IP TUNTAP
<li> MALANG merupakan DNS Server diberikan IP DMZ
<li> MOJOKERTO merupakan DHCP Server diberikan IP DMZ
<li> MADIUN dan PROBOLINGGO merupakan WEB Server
<li> Setiap Server diberikan memory sebesar 128M
<li> Client dan Router diberikan memori sebesar 96M
<li> Jumlah host pada subnet SIDOARJO 200 Host
<li> Jumlah host pada subnet GRESIK 210 Host

Untuk membuat topologi, perlu dibuat konfigurasi dalam file **topologi.sh** dengan menyesuaikan kriteria yang diminta oleh soal sebagai berikut :

```
# Switch
uml_switch -unix switch1 > /dev/null < /dev/null &
uml_switch -unix switch2 > /dev/null < /dev/null &
uml_switch -unix switch3 > /dev/null < /dev/null &
uml_switch -unix switch4 > /dev/null < /dev/null &
uml_switch -unix switch5 > /dev/null < /dev/null &
uml_switch -unix switch6 > /dev/null < /dev/null &

# Router
xterm -T SURABAYA -e linux ubd0=SURABAYA,jarkom umid=SURABAYA eth0=tuntap,,,10.151.76.9 eth1=daemon,,,switch0 eth2=daemon,,,switch3 mem=96M &
xterm -T KEDIRI -e linux ubd0=KEDIRI,jarkom umid=KEDIRI eth0=daemon,,,switch3 eth1=daemon,,,switch2 eth2=daemon,,,switch5 mem=96M &
xterm -T BATU -e linux ubd0=BATU,jarkom umid=BATU eth0=daemon,,,switch0 eth1=daemon,,,switch1 eth2=daemon,,,switch4 mem=96M &

# Server
xterm -T MALANG -e linux ubd0=MALANG,jarkom umid=MALANG eth0=daemon,,,switch2 mem=128M &
xterm -T MOJOKERTO -e linux ubd0=MOJOKERTO,jarkom umid=MOJOKERTO eth0=daemon,,,switch2 mem=128M &
xterm -T MADIUN -e linux ubd0=MADIUN,jarkom umid=MADIUN eth0=daemon,,,switch1 mem=128M &
xterm -T PROBOLINGGO -e linux ubd0=PROBOLINGGO,jarkom umid=PROBOLINGGO eth0=daemon,,,switch1 mem=128M &

# Klien
xterm -T SIDOARJO -e linux ubd0=SIDOARJO,jarkom umid=SIDOARJO eth0=daemon,,,switch5 mem=96M &
xterm -T GRESIK -e linux ubd0=GRESIK,jarkom umid=GRESIK eth0=daemon,,,switch4 mem=96M &
```

**HASIL :**

![](img/)

### Soal (B)

Diminta untuk membuat topologi tersebut menggunakan teknik **CIDR** atau **VLSM**.

Disini kami memilih untuk menggunakan teknik **VLSM**. 

**Langkah 1** - Menentukan jumlah dan memberi nama *subnet* yang terdapat dalam topologi.

![](img/)

**Langkah 2** - Menentukan jumlah alamat IP yang dibutuhkan oleh tiap-tiap *subnet* dan melakukan *labelling netmask* berdasarkan jumlah IP yang dibutuhkan dengan mengacu pada tabel *subnet*.

![](img/)

Berdasarkan total IP dan netmask yang dibutuhkan, maka dapat digunakan netmask **/22** untuk memberikan pengalamatan padaa subnet.

**Langkah 3** - Subnet besar yang dibentuk memiliki NID **192.168.0.0** serta netmask **/22**. Selanjutnya hitung pembagian IP berdasarkan NID dan netmask tersebut menggunakan pohon. Kemudian lakukan *subnetting* dengan pohon yang telah dibuat untuk membagi IP sesuai dengan kebutuhan masing-masing *subnet* yang ada.

![](img/)

**Langkah 4** - Dari pohon tersebut, akan didapatkan pembagian IP sebagai berikut :

![](img/)

Kemudian setelah didapatkan hasil perhitungan, kita dapat menghubungkan tiap-tiap *subnet* sesuai dengan hubungannya dengan *subnet-subnet* lainnya. Untuk melakukannya, kita tambahkan konfigurasi *interface* di tiap UML melalui ```etc/network/interfaces``` dengan mengambil referensi dari pengelompokkan dan perhitungan yang telah dibuat sebelumnya sebagai berikut :

**SURABAYA (Sebagai Router)**
```
SURABAYA
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 10.151.76.10
netmask 255.255.255.252
gateway 10.151.76.9

auto eth1
iface eth1 inet static
address 192.168.0.1
netmask 255.255.255.252

auto eth2
iface eth2 inet static
address 192.168.0.5
netmask 255.255.255.252
```

**KEDIRI (Sebagai Router)**
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 192.168.0.2
netmask 255.255.255.252
gateway 192.168.0.1

auto eth1
iface eth1 inet static
address 192.168.0.9
netmask 255.255.255.248

auto eth2
iface eth2 inet static
address 192.168.1.1
netmask 255.255.255.0
```

**BATU (Sebagai Router)**
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 192.168.0.6
netmask 255.255.255.252
gateway 192.168.0.5

auto eth1
iface eth1 inet static
address 10.151.77.17
netmask 255.255.255.248

auto eth2
iface eth2 inet static
address 192.168.2.1
netmask 255.255.255.0
```

**MALANG (Sebagai DNS Server)**
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 10.151.77.18
netmask 255.255.255.248
gateway 10.151.77.17
```

**MOJOKERTO (Sebagai DHCP Server)**
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 10.151.77.19
netmask 255.255.255.248
gateway 10.151.77.17
```

**MADIUN (Sebagai Web Server)**
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 192.168.0.10
netmask 255.255.255.248
gateway 192.168.0.9
```

**PROBOLINGGO (Sebagai Web Server)**
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 192.168.0.11
netmask 255.255.255.248
gateway 192.168.0.9
```

**SIDOARJO (Sebagai Client)**
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 192.168.2.2
netmask 255.255.255.0
gateway 192.168.2.1
```

**GRESIK (Sebagai Client)**
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 192.168.1.2
netmask 255.255.255.0
gateway 192.168.1.1
```

**HASIL :**

![](img/)

### Soal (C)

Setelah melakukkan *subnetting*, juga diminta untuk melakukan *routing* agar setiap perangkat pada jaringan tersebut dapat terhubung.

Untuk menambahkan *route*, digunakan *command* ```route add -net <NID subnet> netmask <netmask> gw <IP gateway>``` pada tiap UML. Dalam topologi ini, kita perlu menambahkan *routing* pada *router* :
<li> SURABAYA => SUBNET A1 (Server), A2, A3, dan A5.
<li> KEDIRI => A0 (default routing)
<li> BATU => A0 (default routing)
  
Dengan *setting* sebagai berikut :
#### SURABAYA
```
route add -net 192.168.1.0 netmask 255.255.255.0 gw 192.168.0.2
route add -net 192.168.2.0 netmask 255.255.255.0 gw 192.168.0.6
route add -net 192.168.0.8 netmask 255.255.255.248 gw 192.168.0.2
route add -net 10.151.77.16 netmask 255.255.255.248 gw 192.168.0.6
```

#### KEDIRI
```
route add -net 0.0.0.0 netmask 0.0.0.0 gw 192.168.0.1
```

#### BATU
```
route add -net 0.0.0.0 netmask 0.0.0.0 gw 192.168.0.5	
```

### Nomor 1
```
iptables -t nat -A POSTROUTING -s 192.168.0.0/16 -o eth0 -j SNAT --to-source 10.151.76.10
```
Penggunaan **SNAT** (*Source NAT*) dapat dijadikan alternatif ketimbang penggunaan **MASQUERADE**. SNAT difungsikan untuk pengambilan IP *source* yang mengarah ke TUNTAP.

### Nomor 2
```
iptables -A FORWARD -d 10.151.77.16/29 -i eth0 -p tcp --dport 22 -j DROP
```
Angka 22 di sini difungsikan sebagai port SSH dan diberi fungsi **DROP**.

### Nomor 3
```
iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 -j DROP
```
Untuk membatasi jumlah orang yang dapat melakukan koneksi, dilakukan perintah **DROP** apabila *connlimit (connection limit)* sudah lebih dari 3.

### Nomor 4
```
iptables -A INPUT -s 192.168.2.0/24 -m time --timestart 07:00 --timestop 17:00 --days Mon,Tue,Wed,Thu,Fri -j ACCEPT

iptables -A INPUT -s 192.168.2.0/24 -j REJECT
```
Untuk line yang pertama, menunjukkan hari untuk menerima koneksi yaitu hari Senin hingga Jumat pukul 7 hingga pukul 5 sore.

Sementara line yang kedua menunjukkan bahwa selain hari yang tertera di line pertama, akan di-*reject*.

### Nomor 5
```
iptables -A INPUT -s 192.168.1.0/24 -m time --timestart 07:00 --timestop 17:00 -j REJECT
```
Mirip dengan nomor 4, kita me-*reject* koneksi untuk IP tertera pada jam 7 hingga jam 5 sore, sehingga diluar jam tersebut, IP dapat diakses.

### Nomor 6
```
iptables -t nat -A PREROUTING -p tcp -d 10.151.77.18 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 192.168.1.3:80

iptables -t nat -A PREROUTING -p tcp -d 10.151.77.18 -j DNAT --to-destination 192.168.1.2:80
```
Kedua *line* di atas difungsikan untuk mendistribusikan request client ke DNS Server pada IP Probolinggo dan Madiun. Keduanya dilakukan secara bergantian melalui ```--mode nth --every 2``` dan pada port 80 (IP:80).

### Nomor 7
```
iptables -N LOGGING
   iptables -A INPUT -j LOGGING
   iptables -A OUTPUT -j LOGGING
   iptables -A LOGGING -j LOG --log-prefix "IPTables-Dropped: " --log-level 4
   iptables -A LOGGING -j DROP
```
*Line* di atas merupakan *line* yang digunakan untuk melakukan ***Logging*** atau pencatatan. Jadi, setiap **DROP** yang dilakukan oleh *firewall* dimasukkan ke dalam log.
