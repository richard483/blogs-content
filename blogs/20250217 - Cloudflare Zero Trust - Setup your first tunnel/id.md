Kalian punya _device_ tak terpakai dan ingin di jadikan _server_? Atau ada yang sedang kebingungan bagaimana cara men-_sharing_ aplikasi yang di host secara _local_ ke internet agar bisa di akses oleh semua orang tanpa memerlukan _IP Public_? Mungkin _artikel_ ini dapat membantu kalian. Dengan menggunakan layanan **gratis** **Coludflare Tunnel**, kita dapat melakukan hal tersebut.

## Prerequisite

- Mesin apa pun yang bisa berjalan dan ter-koneksi dengan internet
- Domain

Untuk dapat memulai, tentu kita memerlukan sebuah mesin yang bisa kita anggap sebagai calon _server_ yang nantinya akan meng-_serve_ aplikasi kita. Dan kemudian _domain_, karena Cloudflare perlu untuk melakukan _routing_ _traffic_ kepada _server_ kita nantinya, karena dari awal kita tidak menggunakan _IP Public_. Walaupun sebenarnya jika kita hanya ingin meng-_expose_ aplikasi kita ke internet tanpa menggunakan domain, kalian bisa langsung membaca bagian **Cloudflare Quick Tunnel** pada artikel ini. Tapi perlu di catat, Quick tunnel ini tidak disarankan untuk _server production_ karena URL nya akan di _randomize_ setiap _server_ kita _restart_.

## How it works

Secara garis besar, cara kerja dari Cloudflare Tunnel ini cukup sederhana. Yaitu nantinya akan ada proses _daemon_ (baca: [daemon](https://id.wikipedia.org/wiki/Daemon)) ringan yang membuat koneksi _outbound-only_ yang bernama `cloudflared` ke jaringan global Cloudflare melalui _data center_ Cloudflare terdekat. Cloudflare Tunnel sendiri mendukung beberapa koneksi seperti HTTP _web server_, _SSH Server_, _remote desktop_, dan beberapa protokol lainnya.

![Gambaran cara kerja Cloudflare Tunnel](https://media.githubusercontent.com/media/richard483/blogs-content/refs/heads/master/assets/20250217/1.png)

Namun, karena Cloudflare Tunnel hanya membuat koneksi keluar (_outbound only_), Cloudflare Tunnel tidak mendukung protokol berbasis UDP secara langsung seperti _server_ untuk game. Sehingga jika ada beberapa dari kalian yang ingin meng-_serve server_ Minecraft, mungkin kalian tidak akan bisa melakukannya dengan Coudflare Tunnel. Namun tenang saja, kalian tetap bisa meng-_serve server game_ seperti Minecraft walau dengan _step_ tambahan dengan membaca artikel lain [mengenai _private network_](https://nephren.xyz/blogs/20250203%20-%20Cloudflare%20Zero%20Trust%20-%20Accessing%20Cloudflare%20Private%20Networks).

## Set up env

Yang perlu disiapkan:

- Server: komputer/laptop apa pun yang berjalan di OS Linux, Windows, ataupun Macintosh
- Konten: projek/aplikasi apa pun yang dapat berjalan di "server" (untuk tujuan demonstrasi)
- Domain: sebagai penghubung antara konten yang di _serve_ kepada internet
- Akun Cloudflare: dapat _register_ atau _login_ jika sudah punya sebelumnya [di sini](https://one.dash.cloudflare.com/)

Pertama-tama siapkan "mesin" yang akan kita gunakan. Pada artikel ini saya akan menggunakan _environment_ Ubuntu yang dijalankan ada WSL pada laptop windows. Untuk konten nya sendiri pada artikel ini saya akan menggunakan aplikasi web React JS, sehingga memerlukan node js pada "server" saya. Namun untuk konten apa pun yang ingin ditampilkan nanti sebagai demonstrasi sebenarnya bebas saja.

Untuk urusan domain sendiri, kita dapat mencari _provider_ yang sekiranya menyediakan _domain_ yang cocok dengan kita. Sudah banyak domain yang bisa didapatkan dengan harga cukup murah, atau jika kita cukup teliti dalam mencari sebenarnya ada beberapa penyedia domain gratis.

## Add existing domain

Pertama-tama kita perlu untuk mendaftarkan domain yang sudah kita miliki kepada Cloudflare dengan cara mengubah pengaturan DNS nya.

Untuk melakukan pengaturannya dapat dengan mengikuti langkah berikut:

1. Masukkan nama domain pada halaman utama Cloudflare [berikut](https://dash.cloudflare.com/)
\
![Halaman utama Cloudflare](https://media.githubusercontent.com/media/richard483/blogs-content/refs/heads/master/assets/20250217/3.png)
2. Pada _radio button_ di bawah pilih saja **Quick scan for DNS records**, kemudian klik **Continue**
3. Pilih plan **Free** kemudian klik **Continue**
4. _Review DNS records_ dari domain kita, kemudian klik **Continue to activation**
5. Ikuti _step-step_ yang tertera, nantinya kita perlu untuk melakukan beberapa _setting_ pada _provider_ domain kita dan mengatur nameservers agar domain kita dapat di _manage_ oleh Cloudflare.
6. Klik **Continue**

## Create a tunnel

Setelah membuat akun CLoudflare, dapat mengikuti langkah berikut

1. Pada halaman home dari [Cloudflare Zero trust](https://one.dash.cloudflare.com/) pilih tab **Networks** > **Tunnels** kemudian klik "Create a tunnel"
2. Pilih _tunnel type_ yang ingin digunakan, pada artikel ini kita akan menggunakan Cloudflared.
3. Tentukan nama tunnel, kemudian klik **Save tunnel**
4. Kita akan diarahkan ke halaman **Configure**, kemudian ikut instruksi yang ada untuk meng-_install_ _connector_ ke "server" kita. Karena pada artikel ini saya menggunakan Ubuntu, maka saya hanya perlu untuk men-_paste_ dan menjalankan _command_ yang tersedia untuk meng-_install connector_ nya
\
![Command untuk menginstall connector pada Linux berbasis Debian](https://media.githubusercontent.com/media/richard483/blogs-content/refs/heads/master/assets/20250217/2.png)
5. Kemudian _select_ **Next**

## Connect application

Sekarang, setelah tunnel yang selesai di _setup_, kita dapat menghubungkan aplikasi yang berjalan pada "server" kita ke tunnel. Pada artikel ini saya menggunakan aplikasi React JS yang berjalan pada localhost port 7000.

Untuk menambahkan aplikasi kita kepada tunnel, dapat dilakukan dengan step berikut:

1. Pada _dashboard_ Zero Trust, klik tab **Networks** > **Tunnels**
2. Klik **titik tiga** pada tunnel yang ingin di hubungkan kemudian klik **Configure**
3. Pilih tab **Public Hostname** kemudian klik tombol **Add a public hostname**
\
![Halaman pengaturan Public Hostname](https://media.githubusercontent.com/media/richard483/blogs-content/refs/heads/master/assets/20250217/4.png)
4. Isi informasi domain sesuai dengan kebutuhan kita, untuk _subdomain_ kita tidak perlu menambahkannya apabila kita ingin mengarahkan aplikasi kita ke _root domain_ kita
5. Untuk informasi aplikasi, cukup mengikuti apa yang sudah berjalan di "server" kita. Apabila saya mempunya aplikasi React JS yang berjalan di http://localhost:7000, maka saya cukup mengisi kolom **Type** dengan **HTTP**, dan **URL** dengan **localhost:7000**, mengikuti aplikasi yang ingin di _expose_
6. Klik **save hostname**
7. Tunggu beberapa saat sampai aplikasi dapat di akses melalui domain yang sudah di atur

## Quick tunnel

Solusi bagi kalian yang tidak memiliki domain tetapi tetap ingin meng-_expose_ aplikasi yang di _serve_ pada "server" kita adalah dengan menggunakan Cloudflare Quick Tunnel. Dengan Quick tunnel, cloudflare memberikan domain _random_ untuk dapat mengakses aplikasi kita di internet.

Untuk dapat menggunakan Quick tunnel, kita dapat melakukan step berikut:

- Download dan _authorize_ `cloudflared` pada "server", atau dapat dengan mengikuti point 1-5 pada bagian **Create a tunnel** di artikel ini
- Siapkan aplikasi yang ingin di sharing, misalkan React JS yang berjalan di http://localhost:7000
- Jalankan command berikut:

  ```bash
  cloudflared tunnel --url <local_address aplikasi_kita>
  ```

Kemudian cloudflared akan memberikan URL aplikasi kita yang dapat di akses melalui internet.
![Contoh keluaran cloudflared](https://media.githubusercontent.com/media/richard483/blogs-content/refs/heads/master/assets/20250217/5.png)

---

**References**

- [https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/)
- [https://id.wikipedia.org/wiki/Daemon](https://id.wikipedia.org/wiki/Daemon)
- [https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/create-remote-tunnel/](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/create-remote-tunnel/)
- [https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/do-more-with-tunnels/trycloudflare/](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/do-more-with-tunnels/trycloudflare/)
