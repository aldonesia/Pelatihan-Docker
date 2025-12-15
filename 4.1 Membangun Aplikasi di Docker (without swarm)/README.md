# **Membangun Aplikasi di Docker (Tanpa Swarm)**
- [**Glosarium**](#glosarium)
- [**Materi**](#materi)
  - [Microservice](#microservice)
    - [Pengertian Microservice](#pengertian-microservice)
    - [Karakteristik Microservice](#karakteristik-microservice)
    - [Inter-container communication](#inter-container-communication)
      - [Sockets](#sockets)
      - [Filesystem](#filesystem)
      - [Database Records](#database-records)
      - [HTTP](#http)
      - [MQTT](#mqtt)
  - [Membuat Aplikasi Microservice dengan Docker Compose](#membuat-aplikasi-microservice-dengan-docker-compose)
  - [Tambahan](#tambahan)
    - [Pengantar Aplikasi Web](#pengantar-aplikasi-web)
- [**Sumber Referensi**](#sumber-referensi)


## Glosarium

## Materi
### Microservice
#### Pengertian Microservice
Microservice adalah sebuah pendekatan arsitektur perangkat lunak untuk membangun sebuah aplikasi dengan memecahnya menjadi beberapa komponen kecil yang berdiri sendiri (independen) dan saling berkomunikasi melalui antarmuka yang terstandarisasi. Dalam arsitektur microservice, setiap komponen atau service memiliki tanggung jawab yang spesifik dan dijalankan secara independen dari komponen lainnya.

![Aplikasi microservice](../img/microservice.png)

Secara umum, arsitektur microservice menawarkan beberapa keuntungan, seperti memudahkan skalabilitas, mempercepat waktu deployment dan pengembangan, serta memudahkan untuk memperbaiki atau meng-upgrade aplikasi. Hal ini disebabkan karena setiap service dapat dikembangkan, di-deploy, dan di-maintain secara terpisah tanpa mempengaruhi service lainnya.

Namun, di sisi lain, menggunakan arsitektur microservice juga memiliki beberapa tantangan, seperti pengelolaan service yang lebih kompleks dan koordinasi antar service yang harus dilakukan secara hati-hati. Oleh karena itu, pemilihan arsitektur yang tepat harus dipertimbangkan berdasarkan kebutuhan bisnis dan teknologi yang digunakan.

#### Karakteristik Microservice

- Autonomous
Salah satu karakteristik penting dari aplikasi microservice adalah keberadaan setiap komponen atau layanan yang dapat beroperasi secara independen. Dengan arsitektur microservice, setiap komponen dapat dikembangkan, di-deploy, dioperasikan, dan diskalakan tanpa mempengaruhi komponen atau layanan lain. Dengan kata lain, setiap komponen dapat berjalan seperti sistem yang mandiri, dan jika terjadi masalah pada salah satu komponen, hanya layanan tersebut yang akan terpengaruh.

- Service-oriented
Aplikasi microservice dirancang dengan orientasi pada layanan. Setiap komponen aplikasi dirancang untuk melakukan satu tugas atau layanan khusus. Hal ini membuat aplikasi lebih modular dan memudahkan pengembangan dan pemeliharaannya.

- Terdistribusi
Aplikasi microservice terdiri dari beberapa komponen yang terdistribusi dan dapat berjalan di lingkungan yang berbeda-beda. Hal ini memungkinkan aplikasi untuk dapat dijalankan secara horizontal dan meningkatkan skalabilitasnya.

#### Inter-container communication
Inter-container communication di aplikasi microservice merujuk pada cara komunikasi antar service atau container yang berbeda dalam suatu aplikasi yang menggunakan arsitektur microservice. Karena setiap service dijalankan pada container yang terpisah dan terisolasi, maka inter-container communication harus dilakukan melalui mekanisme tertentu.

##### Sockets
Sockets adalah salah satu mekanisme yang dapat digunakan dalam inter-container communication pada aplikasi microservice. Socket dapat diartikan sebagai titik akhir dari suatu koneksi yang digunakan untuk berkomunikasi antar service atau container yang berbeda.

Dalam konteks aplikasi microservice, socket biasanya digunakan untuk melakukan komunikasi antar service pada jaringan lokal atau antar container pada mesin yang sama. Socket dapat digunakan untuk melakukan komunikasi secara sinkron atau asinkron.

Pada umumnya, penggunaan socket dalam inter-container communication pada aplikasi microservice dapat dilakukan dengan beberapa cara, seperti:

- TCP Socket: digunakan untuk melakukan komunikasi yang reliable dan ordered antar service atau container pada jaringan lokal.

- Unix Domain Socket: digunakan untuk melakukan komunikasi antar container pada mesin yang sama. Unix domain socket dapat lebih cepat daripada TCP socket karena tidak perlu melalui jaringan.

- Datagram Socket: digunakan untuk melakukan komunikasi yang tidak reliable dan unordered antar service atau container pada jaringan lokal. Datagram socket dapat digunakan untuk transfer data yang relatif kecil dan tidak memerlukan protokol seperti TCP.

Pemilihan jenis socket yang tepat harus dipertimbangkan berdasarkan kebutuhan bisnis dan teknologi yang digunakan dalam aplikasi microservice. Penggunaan socket dalam inter-container communication dapat membantu meningkatkan efisiensi dan performa dari aplikasi microservice.

##### Filesystem
Filesystem pada inter-container communication mengacu pada mekanisme berbagi file antara container pada sistem operasi host yang sama. Dalam aplikasi microservice, mekanisme ini dapat digunakan untuk berbagi data atau file konfigurasi antar service atau container yang berjalan pada mesin yang sama.

Dalam mekanisme filesystem, container yang berbagi data dapat diatur untuk menggunakan volume yang sama untuk mengakses file yang sama. Volume ini dapat diatur untuk berada di dalam atau di luar container. Jika volume berada di dalam container, maka setiap container yang berbagi volume dapat melihat isi volume secara bersamaan. Sedangkan jika volume berada di luar container, maka setiap container akan mengakses file atau data pada volume melalui jaringan file, sehingga memungkinkan container untuk berbagi data atau file konfigurasi dengan mudah.

Beberapa keunggulan dari mekanisme filesystem pada inter-container communication pada aplikasi microservice adalah sebagai berikut:

- Efisien: Mekanisme filesystem pada inter-container communication sangat efisien karena container tidak perlu mengirim atau menerima data melalui jaringan.
- Mudah diatur: Filesystem pada inter-container communication mudah diatur dan dikonfigurasi untuk berbagi data atau file konfigurasi antar container.
- Menyediakan akses bersama: Dengan mekanisme filesystem, setiap container yang berbagi volume dapat melihat isi volume secara bersamaan, sehingga memungkinkan container untuk berbagi data atau file konfigurasi dengan mudah.

Namun, penggunaan mekanisme filesystem juga memiliki beberapa kelemahan, seperti:

- Tidak cocok untuk container yang berjalan pada mesin yang berbeda: Mekanisme filesystem hanya dapat digunakan untuk berbagi data atau file konfigurasi antar container yang berjalan pada mesin yang sama. Jika container berjalan pada mesin yang berbeda, maka mekanisme ini tidak dapat digunakan.
- Memerlukan koordinasi yang baik: Mekanisme filesystem memerlukan koordinasi yang baik antara container yang berbagi volume untuk memastikan bahwa data atau file konfigurasi yang digunakan sama di setiap container.

Pemilihan mekanisme inter-container communication yang tepat harus dipertimbangkan berdasarkan kebutuhan bisnis dan teknologi yang digunakan dalam aplikasi microservice. Mekanisme filesystem dapat menjadi pilihan yang baik terutama jika container berjalan pada mesin yang sama dan membutuhkan akses bersama ke data atau file konfigurasi.

##### Database Records
Inter-container communication dengan database records mengacu pada kemampuan container dalam berbagi data atau informasi melalui database yang sama.

Dalam aplikasi yang terdiri dari beberapa container, terdapat kemungkinan beberapa container memerlukan akses ke data yang sama di database. Misalnya, aplikasi e-commerce dengan container untuk web front-end, container untuk pengelolaan persediaan, dan container untuk pemrosesan pembayaran. Semua container ini perlu akses ke database yang sama untuk mengambil data produk, informasi persediaan, dan transaksi pembayaran.

Pengimplementasian inter-container communication dengan database records dapat menggunakan berbagai teknologi dan bahasa pemrograman seperti Python, Java, atau Node.js. Salah satu contoh teknologi yang populer dalam inter-container communication dengan database records adalah ORM (Object-Relational Mapping) seperti Hibernate, Sequelize, atau SQLAlchemy. ORM memungkinkan untuk memetakan objek dalam aplikasi ke struktur tabel dalam database dan memungkinkan container untuk mengakses data yang sama dari database yang sama.

Beberapa keunggulan dari mekanisme database record pada inter-container communication pada aplikasi microservice adalah sebagai berikut:

- Konsistensi data - Inter-container communication dengan database records memastikan bahwa setiap container menggunakan data yang sama dari database yang sama, sehingga menjaga konsistensi data di seluruh aplikasi.
- Penghematan waktu - Penggunaan inter-container communication dengan database records memungkinkan container untuk berbagi data tanpa harus mengirimkan data melalui jaringan, sehingga dapat menghemat waktu dan meningkatkan kecepatan aplikasi.
- Skalabilitas - Dengan menggunakan database yang sama untuk semua container, aplikasi dapat dengan mudah ditingkatkan dengan menambahkan atau menghapus container tanpa mempengaruhi konsistensi data.

Namun, penggunaan mekanisme database record juga memiliki beberapa kelemahan, seperti:

- Kompleksitas konfigurasi - Inter-container communication dengan database records memerlukan konfigurasi yang kompleks untuk memastikan setiap container dapat mengakses database yang sama dengan benar.
- Risiko keamanan - Jika tidak diatur dengan benar, inter-container communication dengan database records dapat meningkatkan risiko keamanan aplikasi karena container dapat memiliki akses ke data sensitif dalam database yang sama.

Penting untuk diingat bahwa inter-container communication dengan database records memerlukan koordinasi yang baik antara container dan penggunaan teknologi yang tepat. Hal ini meliputi penggunaan teknologi yang dapat menangani concurrency, isolasi transaksi, dan penggunaan teknologi yang tepat untuk penggunaan yang sesuai dengan skala aplikasi. Selain itu, penggunaan teknologi dan bahasa pemrograman yang sama di semua container dapat mempermudah koordinasi dan interaksi antara container.

##### HTTP
HTTP (Hypertext Transfer Protocol) adalah protokol komunikasi yang umum digunakan dalam inter-container communication pada aplikasi microservice. HTTP adalah protokol request-response yang digunakan untuk mentransfer data melalui jaringan antar service atau container.

Pada aplikasi microservice, HTTP dapat digunakan sebagai mekanisme inter-container communication melalui RESTful API. Setiap service menyediakan RESTful API yang terbuka, dan service lain dapat mengakses API tersebut untuk berkomunikasi dengan service yang bersangkutan.

Dalam penggunaan HTTP sebagai mekanisme inter-container communication pada aplikasi microservice, HTTP dapat memiliki beberapa keunggulan, seperti:

- Mudah diimplementasikan: karena HTTP adalah protokol yang sangat umum, banyak bahasa pemrograman dan framework yang mendukung implementasi HTTP.
- Interoperabilitas yang baik: karena HTTP adalah protokol standar, service yang berbeda dapat saling berkomunikasi dengan mudah.
- Dukungan terhadap berbagai format data: HTTP mendukung berbagai format data seperti JSON, XML, dan lain-lain, sehingga memungkinkan service untuk berkomunikasi menggunakan format data yang berbeda-beda.
- Dukungan terhadap pengamanan: HTTP dapat digunakan dengan protokol HTTPS untuk memastikan keamanan dan kerahasiaan data yang ditransfer antar service atau container.

Namun, penggunaan HTTP sebagai mekanisme inter-container communication pada aplikasi microservice juga memiliki beberapa kelemahan, seperti:

- Overhead yang besar: HTTP memiliki overhead yang besar karena memerlukan header dan metadata yang cukup kompleks.
- Performa yang relatif lambat: karena overhead yang besar, performa HTTP relatif lambat dibandingkan dengan mekanisme inter-container communication yang lebih sederhana seperti Unix domain socket.
- Tidak mendukung streaming data: HTTP tidak mendukung streaming data secara efektif, sehingga tidak cocok untuk transfer data yang besar dan kompleks.

Pemilihan mekanisme inter-container communication yang tepat harus dipertimbangkan berdasarkan kebutuhan bisnis dan teknologi yang digunakan dalam aplikasi microservice. HTTP dapat menjadi pilihan yang baik terutama jika aplikasi microservice membutuhkan interoperabilitas yang baik dan dukungan terhadap pengamanan.

##### MQTT
MQTT (Message Queuing Telemetry Transport) adalah protokol komunikasi ringan yang digunakan untuk pertukaran data pada sistem IoT (Internet of Things) dan aplikasi yang membutuhkan pertukaran data real-time. MQTT dirancang untuk mengoptimalkan penggunaan bandwidth jaringan dan penggunaan daya pada perangkat yang terhubung dengan jaringan.

Pada aplikasi microservice, MQTT dapat digunakan sebagai mekanisme inter-container communication untuk komunikasi antara service atau container yang berjalan pada mesin yang sama atau mesin yang berbeda. MQTT menggunakan konsep publish-subscribe untuk mentransfer pesan antara service atau container.

Dalam konsep publish-subscribe, service atau container yang mengirim pesan disebut sebagai publisher, sedangkan service atau container yang menerima pesan disebut sebagai subscriber. Publisher hanya perlu mengirimkan pesan ke broker (sebuah server MQTT) tanpa harus mengetahui subscriber mana yang akan menerimanya. Kemudian broker akan menyebarluaskan pesan ke seluruh subscriber yang telah terdaftar.

![Inter-container communication dengan MQTT](../img/microservice-mqtt.png)

Beberapa keunggulan dari MQTT sebagai mekanisme inter-container communication pada aplikasi microservice adalah sebagai berikut:

- Ringan dan efisien: MQTT dirancang untuk mengoptimalkan penggunaan bandwidth jaringan dan penggunaan daya pada perangkat yang terhubung dengan jaringan, sehingga sangat cocok digunakan pada aplikasi microservice yang membutuhkan pertukaran data real-time.
- Skalabilitas yang baik: MQTT dapat digunakan pada sistem yang sangat besar dengan banyak service atau container, sehingga sangat cocok digunakan pada aplikasi microservice yang membutuhkan skalabilitas.
- Dukungan terhadap kualitas layanan (QoS): MQTT menyediakan tiga tingkat QoS (0, 1, 2) untuk memastikan pesan terkirim dengan benar dan tiba pada waktu yang tepat.
- Dukungan terhadap pengamanan: MQTT dapat digunakan dengan protokol SSL/TLS untuk memastikan keamanan dan kerahasiaan data yang ditransfer antar service atau container.

Namun, penggunaan MQTT juga memiliki beberapa kelemahan, seperti:

- Kurang umum: MQTT masih kurang umum dibandingkan dengan protokol komunikasi lainnya seperti HTTP.
- Kompleksitas implementasi: MQTT memerlukan broker sebagai perantara antara publisher dan subscriber, sehingga memerlukan kompleksitas implementasi yang lebih tinggi.

Pemilihan mekanisme inter-container communication yang tepat harus dipertimbangkan berdasarkan kebutuhan bisnis dan teknologi yang digunakan dalam aplikasi microservice. MQTT dapat menjadi pilihan yang baik terutama jika aplikasi microservice membutuhkan penggunaan bandwidth jaringan dan penggunaan daya yang efisien, serta dukungan terhadap kualitas layanan dan pengamanan.

### Membuat Aplikasi Microservice dengan Docker Compose

Pada implementasi kali ini, kita akan menggunakan Docker Compose untuk menjalankan aplikasi microservice pada satu mesin. Docker Compose memungkinkan kita untuk mendefinisikan dan menjalankan aplikasi multi-container dengan mudah menggunakan file konfigurasi YAML.

Untuk implementasi microservice dengan Docker Compose kali ini akan menggunakan aplikasi yang terdapat dalam folder [compose](compose/). Aplikasi ini terdiri dari 5 services yang akan berjalan pada satu mesin menggunakan Docker Compose.

Secara umum, gambaran tentang arsitektur aplikasi yang akan digunakan seperti berikut.

![Arsitektur App](../img/app-arch.png)

Pada gambar tersebut terdapat 5 services (**`nginx-frontend`**, **`frontend`**, **`nginx-backend`**, **`backend`**, dan **`database`**). **`nginx-frontend`** dan **`nginx-backend`** sama-sama bertugas sebagai web-server (penjelasan tentang masing-masing istilah dapat dilihat pada subbab [Pengantar Aplikasi Web](#pengantar-aplikasi-web)). **`nginx-frontend`** berjalan pada port **6000** dan **`nginx-backend`** berjalan pada port **5000**. Alur aplikasinya sebagai berikut:

1. Pada client side, client akan mengakses aplikasi menggunakan web browser masing-masing.
2. Web browser akan meneruskan permintaan client ke server, selanjutnya akan diurus oleh server (client sudah tidak perlu melakukan apa-apa lagi).
3. Pada server side, permintaan client akan diterima pertama kali oleh web server untuk **`frontend`** (**`nginx-frontend`**).
4. Permintaan client pada **`nginx-frontend`**, akan dilanjutkan untuk mendapat resource dari service **`frontend`**.
5. Jika halaman yang digunakan tidak memerlukan service dari **`backend`**, maka perjalanan permintaan client akan dikembalikan ke client. 
6. Jika memerlukan service **`backend`** (misal perlu data dari **`database`**, seperti login dan register), maka permintaan akan diteruskan ke web server untuk **`backend`** (**`nginx-backend`**).
7. Yang terakhir, **`backend`** akan berkomunikasi dengan **`database`** untuk memperoleh data dan akan dikembalikan hingga ke client.

#### 1. Download Resource Aplikasi

Implementasi microservice kali ini menggunakan aplikasi yang tersedia pada folder [compose](compose) di repository ini dengan melakukan cloning.

```
git clone https://github.com/arsitektur-jaringan-komputer/Pelatihan-Docker.git
```

Lalu masuk ke directory **`Pelatihan-Docker/4. Membangun Aplikasi di Docker/4.1 Membangun Aplikasi di Docker (without swarm)/compose/`** dan aplikasi siap untuk diimplementasikan.

#### 2. Konfigurasi File Nginx

> Jika tidak memiliki domain name atau tidak ingin mengkonfigurasikan domain name ke aplikasi, maka langkah ini bisa dilewati dan bisa langsung ke langkah [3. Konfigurasi File Environment](#3-konfigurasi-file-environment)

Pada folder **`nginx/`** terdapat 2 file, **`nginx-frontend.conf`** dan **`nginx-backend.conf`** yang perlu dikonfigurasi. Caranya dengan mengganti nilai **`server_name`** dari **`localhost`** ke domain name masing-masing.

![Nginx conf](../img/nginx-conf.png)

#### 3. Konfigurasi File Environment

> Jika dideploy pada localhost, maka lewati langkah berikut dan bisa langsung ke langkah [4. Build dan Menjalankan Aplikasi](#4-build-dan-menjalankan-aplikasi)

Pada folder **`frontend/`** terdapat file **`.env`** yang perlu dikonfigurasi. Yang perlu dilakukan adalah mengubah nilai nya sesuai yang diinginkan.

```
NEXT_PUBLIC_APP_NAME="<nama-aplikasi>"
NEXT_PUBLIC_API_ENDPOINT="http://<alamat-nginx-backend>:5000/" 
```

* **`NEXT_PUBLIC_APP_NAME`**: Nama dari aplikasi yang akan ditampilkan di Frontend
* **`NEXT_PUBLIC_API_ENDPOINT`**: Alamat dari web server untuk **`backend`** (menggunakan port 5000), dapat diisi dengan domain name atau IP public server. Contoh: `http://localhost:5000/` atau `http://192.168.1.100:5000/`

#### 4. Build dan Menjalankan Aplikasi

Untuk membangun dan menjalankan aplikasi dengan Docker Compose, kita dapat menggunakan perintah berikut:

```
docker compose build
```

Perintah ini akan membangun semua image yang diperlukan berdasarkan Dockerfile yang ada di masing-masing service. Setelah proses build selesai, semua images dapat dipastikan telah ter-build dengan menjalankan perintah berikut (total akan ada 5 images untuk service pada aplikasi ini).

```
docker images
```

![Docker images](../img/docker-images.png)

Setelah semua images dipastikan telah ter-build, selanjutnya adalah menjalankan aplikasi dengan perintah berikut:

```
docker compose up -d
```

Perintah **`-d`** akan menjalankan container dalam mode detached (background), sehingga terminal tetap dapat digunakan untuk perintah lainnya.

#### 5. Cek Aplikasi

Untuk mengecek aplikasi telah berjalan dengan sesuai atau tidak, dapat menjalankan perintah berikut:

```
docker compose ps
```

Perintah ini akan menampilkan status dari semua container yang sedang berjalan. Pastikan bahwa semua container memiliki status **`Up`**.

Selain itu, karena aplikasi ini berbasis web, aplikasi dapat dicek melalui web browser dengan memasukkan alamat **`http://localhost:6000`** atau **`http://<ip-server>:6000`**.

![Tampilan Aplikasi](../img/app-login.png)

#### 6. Menghentikan Aplikasi

Untuk menghentikan aplikasi yang sedang berjalan, dapat menggunakan perintah berikut:

```
docker compose down
```

Perintah ini akan menghentikan dan menghapus semua container yang dibuat oleh Docker Compose. Jika ingin menghapus volume juga (termasuk data database), dapat menggunakan perintah:

```
docker compose down -v
```

> **Peringatan**: Perintah **`docker compose down -v`** akan menghapus semua volume termasuk data database. Pastikan untuk melakukan backup data terlebih dahulu jika diperlukan.

#### 7. Melihat Logs

Untuk melihat logs dari semua service atau service tertentu, dapat menggunakan perintah berikut:

```
# Melihat logs semua service
docker compose logs

# Melihat logs service tertentu
docker compose logs <nama-service>

# Melihat logs dengan follow (real-time)
docker compose logs -f <nama-service>
```

#### 8. Restart Service

Jika ingin me-restart service tertentu tanpa menghentikan semua container, dapat menggunakan perintah berikut:

```
docker compose restart <nama-service>
```

Atau jika ingin me-restart semua service:

```
docker compose restart
```

### Tambahan

#### Pengantar Aplikasi Web

1. Frontend

    Frontend adalah bagian dari sebuah aplikasi web yang berhubungan langsung dengan pengguna. Ini mencakup elemen-elemen visual dan interaktif yang dilihat dan digunakan oleh pengguna akhir. Bahasa pemrograman yang umum digunakan untuk mengembangkan frontend adalah HTML, CSS, dan JavaScript.

2. Backend

    Backend adalah bagian dari aplikasi web yang berada di sisi server. Ini melibatkan pemrosesan data, logika bisnis, dan interaksi dengan database. Beberapa bahasa pemrograman yang digunakan untuk mengembangkan backend antara lain Python, JavaScript (Node.js), dan Ruby.

3. Web Server

    Web server adalah perangkat lunak yang menjalankan aplikasi web dan mengirimkan konten ke pengguna melalui protokol HTTP. Web server bertindak sebagai penghubung antara klien (misalnya, browser web) dan backend aplikasi. Beberapa web server yang populer adalah Apache dan Nginx.

4. Database

    Database adalah tempat penyimpanan yang digunakan untuk menyimpan data yang diperlukan oleh aplikasi web. Database memungkinkan penyimpanan, pengambilan, dan pembaruan data dengan cara yang terstruktur. Beberapa jenis database yang umum digunakan adalah MySQL, PostgreSQL, MongoDB, dan SQLite.

## Sumber Referensi
- https://datacommcloud.co.id/microservices-adalah-perbedaan-monolithic-architecture/
- https://medium.com/pujanggateknologi/berkenalan-dengan-teknologi-mqtt-7e63cab9d00d
- https://aws.amazon.com/id/microservices/
- https://docs.docker.com/compose/
- https://docs.docker.com/compose/compose-file/

