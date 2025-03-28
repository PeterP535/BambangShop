# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Subscriber model struct.`
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Subscriber repository.`
    -   [ ] Commit: `Implement list_all function in Subscriber repository.`
    -   [ ] Commit: `Implement delete function in Subscriber repository.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1




# 📘 Analisis Desain dan Struktur Data BambangShop

Dokumen ini menjelaskan keputusan desain dalam implementasi sistem notifikasi *Observer Pattern* di proyek **BambangShop**, khususnya terkait kebutuhan akan trait/interface, pemilihan struktur data (`Vec` vs `DashMap`), dan implementasi thread safety (`DashMap` vs Singleton pattern).

---

## 1. Apakah Trait (Interface) Dibutuhkan untuk Subscriber?

###  Ringkasan
Dalam implementasi *Observer Pattern* klasik, interface atau trait digunakan agar publisher dapat menyimpan referensi ke berbagai observer yang memiliki perilaku seragam namun implementasi berbeda. Namun, dalam konteks **BambangShop**, penggunaan trait tidak diperlukan.

###  Alasan Mengapa Struct Saja Sudah Cukup

- **Tidak Ada Variasi Perilaku**  
  Semua subscriber hanya menyimpan data statis (`url`, `name`) dan tidak melakukan operasi atau logika spesifik yang berbeda-beda.

- **Tidak Ada Method Dinamis**  
  Kita tidak memerlukan method seperti `notify()` atau `update()` yang harus diimplementasikan secara berbeda oleh tiap subscriber.

- **Notifikasi Terpusat**  
  Pengiriman notifikasi (misalnya HTTP request) di-handle oleh sistem pusat, bukan oleh masing-masing subscriber secara mandiri.

- **Overhead Tidak Perlu**  
  Menambahkan trait hanya akan menambah kompleksitas tanpa ada keuntungan nyata dalam kasus ini.

###  Kesimpulan
Gunakan `struct Subscriber` saja sudah cukup. Trait hanya dibutuhkan jika ingin mendukung perilaku polymorphic antar berbagai jenis subscriber yang memiliki logika berbeda.

---

## 2. Apakah Vec Cukup atau Perlu DashMap?

###  Kebutuhan Sistem
- `url` pada `Subscriber` dan `id` pada `Program` harus bersifat **unik**
- Operasi utama: **insert**, **delete**, **lookup by key**
- Sistem bekerja secara **multi-threaded**

### 🔍 Perbandingan: `Vec<Subscriber>` vs `DashMap<String, Subscriber>`

| Aspek             | Vec                      | DashMap                        |
|-------------------|--------------------------|--------------------------------|
| Akses by Key      | O(n) (linear search)     | O(1) (hash lookup)             |
| Delete by Key     | O(n)                     | O(1)                           |
| Menjamin Unik     | Tidak otomatis            | Otomatis via key uniqueness    |
| Thread-Safe       |  Tidak                 |  Ya                          |
| Skalabilitas      | Terbatas                 | Sangat efisien untuk data besar|

###  Risiko Jika Menggunakan Vec

```rust
fn add(subscribers: &mut Vec<Subscriber>, new: Subscriber) {
    if !subscribers.iter().any(|s| s.url == new.url) {
        subscribers.push(new);
    }
}
```

Kode di atas memerlukan iterasi linear dan tidak efisien ketika jumlah subscriber meningkat.

###  Keunggulan DashMap
- Menjamin **uniqueness** secara otomatis melalui key
- Akses sangat cepat meski data besar
- Aman untuk digunakan di lingkungan konkuren (multi-threaded)

###  Kesimpulan
Gunakan `DashMap` karena:
- Menjamin integritas data unik
- Mendukung operasi cepat dan thread-safe
- Cocok dengan kebutuhan sistem konkuren

---

## 3. Apakah Perlu DashMap atau Cukup Singleton?

### 🔄 Apa Itu Singleton?
Pola desain Singleton menjamin hanya ada **satu instance** global dari suatu objek (contohnya `SUBSCRIBERS`).

Namun, **Singleton tidak otomatis thread-safe**, terutama jika koleksi datanya menggunakan `HashMap` biasa.

###  DashMap vs Singleton dengan RwLock

| Aspek                | Singleton + RwLock         | DashMap                        |
|----------------------|----------------------------|--------------------------------|
| Thread Safety         |  Tapi bisa jadi bottleneck |  Dengan sharding (lebih efisien) |
| Manual Lock Handling  |  Harus hati-hati          |  Sudah diurus secara internal |
| Performa High Load    |  Potensi kontensi         |  Terbagi ke banyak shard      |
| Kompleksitas Kode     | Tinggi                     | Sederhana dan idiomatik Rust   |

###  Contoh Masalah pada Singleton Manual

```rust
let repo = SubscriberRepo::instance();
repo.write().unwrap().insert(url.clone(), subscriber);
```

Risiko:
- Lock semua thread saat satu thread sedang menulis
- Rentan deadlock jika tidak hati-hati

###  Keunggulan DashMap
- **Sharded Mutex**: Kunci hanya bagian yang relevan, bukan seluruh map
- **Atomic Operation**: Insert, remove, get semua aman dan efisien
- **Lebih scalable** untuk sistem dengan traffic tinggi

###  Kesimpulan
- Gunakan **DashMap di dalam Singleton**, bukan mengganti DashMap dengan Singleton
- `lazy_static` atau `once_cell` bisa dipakai untuk membuat Singleton instance dari DashMap

---

##  Penutup

**Keputusan akhir:**
- ✅ `struct Subscriber` cukup tanpa trait
- ✅ Gunakan `DashMap` untuk koleksi subscriber dan program
- ✅ Kombinasi Singleton + DashMap adalah pendekatan optimal untuk thread-safe global state di Rust

Dengan pendekatan ini, sistem BambangShop menjadi:
- Lebih efisien
- Aman untuk concurrency
- Lebih mudah dirawat dan dikembangkan




#### Reflection Publisher-2

# Pemisahan Service dan Repository dalam Arsitektur MVC Modern

## Mengapa Perlu Memisahkan “Service” dan “Repository” dari Model?

Dalam pola dasar MVC, **Model** memang berfungsi sebagai pusat logika dan data. Namun, dalam praktik rekayasa perangkat lunak modern, pendekatan ini cenderung menimbulkan masalah jangka panjang, terutama saat sistem berkembang. Maka muncullah praktik pemisahan tanggung jawab melalui **Service Layer** dan **Repository Layer**.

### 1. Separation of Concerns (SoC)
- **Model** bertugas mendefinisikan struktur data dan merepresentasikan entitas (misalnya: `Program`, `Subscriber`, `Notification`).
- **Repository** bertanggung jawab atas **akses data** seperti mengambil, menyimpan, memperbarui, dan menghapus dari database.
- **Service** menangani **logika bisnis** yang berkaitan dengan proses domain (misalnya: siapa yang harus menerima notifikasi, validasi bisnis, atau transformasi data).

Dengan memisahkan tiga komponen ini, kode menjadi lebih modular, lebih mudah dipahami, dan lebih mudah diuji.

### 2. Pengujian Lebih Terarah
- Repository dapat diuji dengan **mock database**.
- Service diuji dengan logika bisnis tanpa perlu bergantung pada detail penyimpanan data.
- Model diuji sebagai struktur data murni.

### 3. Adaptasi Teknologi Lebih Mudah
- Kita bisa mengganti MySQL ke PostgreSQL atau ke penyimpanan file tanpa perlu mengubah logika bisnis.
- Service tidak peduli bagaimana data diambil, asalkan repository menyediakannya.

---

## Apa yang Terjadi Jika Hanya Menggunakan Model?

Jika kita memusatkan semua logika ke dalam Model, maka setiap model akan menanggung beban tanggung jawab yang terlalu besar. Hal ini bisa menyebabkan masalah berikut:

### Kompleksitas dan Ketergantungan
- Model **Program** harus menangani pengelolaan data, logika bisnis seperti pemrosesan pendaftaran, dan validasi.
- Model **Subscriber** bisa memiliki tanggung jawab ganda: menyimpan data dan memproses langganan atau berhenti langganan.
- Model **Notification** harus tahu cara menyimpan notifikasi, menentukan subscriber yang relevan, dan bahkan cara mengirimkannya.

### Dampaknya
- **Kode menjadi saling tergantung** satu sama lain. Model saling memanggil dan memodifikasi, sehingga debugging menjadi sulit.
- **Testing menjadi sulit**, karena Model tidak dapat diuji secara terpisah dari dependensinya.
- **Keterbacaan menurun**, dan akan sulit bagi developer baru untuk memahami peran dari setiap bagian kode.

### Imajinasikan Skenario Ini
Bayangkan `Program` perlu mengirim notifikasi ke semua `Subscriber`:
- Tanpa pemisahan, `Program` harus tahu bagaimana mengambil `Subscriber`, bagaimana memproses `Notification`, dan bagaimana mengirimkannya.
- Setiap Model tahu tentang Model lain, menciptakan jaringan dependensi yang kompleks dan rentan error.

---

## Eksplorasi dan Manfaat Postman

**Postman** adalah alat yang sangat bermanfaat untuk menguji dan mendokumentasikan REST API. Penggunaan Postman dalam proyek seperti BambangShop sangat membantu dalam mengembangkan dan memverifikasi fitur backend.

### Fitur Postman yang Membantu Pengembangan

1. **Pengujian HTTP Endpoint**
   - Cek semua metode: `GET`, `POST`, `PUT`, `DELETE`.
   - Kirim permintaan dengan body JSON, headers, dan parameter.

2. **Environment dan Variabel**
   - Definisikan variabel untuk base URL dan token auth sesuai environment (dev, staging, production).
   - Ganti-ganti konfigurasi tanpa ubah manual.

3. **Collection & Folder**
   - Simpan endpoint dalam koleksi berdasarkan modul atau fitur.
   - Memudahkan navigasi dan kolaborasi.

4. **Automated Testing dan Test Scripts**
   - Tulis skrip di tab “Tests” untuk memverifikasi status response, struktur JSON, dan nilai-nilai kunci.
   - Contoh: `pm.response.to.have.status(200);`

5. **Mock Server dan Monitoring**
   - Simulasikan endpoint yang belum jadi.
   - Jalankan pengujian terjadwal untuk cek kesehatan API.

6. **Kolaborasi Tim**
   - Share koleksi API dengan rekan satu tim.
   - Lacak perubahan dan histori request.

### Penggunaan Nyata dalam Proyek
- Menguji pendaftaran `Subscriber` dan pengiriman `Notification`.
- Verifikasi response API dari fitur `subscribe` dan `unsubscribe`.
- Cek validasi input untuk endpoint `Program`.
- Membantu dokumentasi API yang bisa dibagikan ke tim frontend.

---

## Kesimpulan

- Memisahkan Service dan Repository dari Model membuat arsitektur lebih bersih, scalable, dan maintainable.
- Jika hanya menggunakan Model, akan muncul kompleksitas tinggi akibat beban tanggung jawab ganda pada Model.
- Postman sangat membantu dalam pengujian API secara cepat, efisien, dan kolaboratif, serta cocok digunakan dalam proyek saat ini maupun proyek skala besar di masa depan.


#### Reflection Publisher-3

# Analisis Pola Observer dan Implementasi Multi-Threading dalam Tutorial BambangShop

Dokumen ini membahas implementasi pola Observer dalam konteks proyek tutorial BambangShop, serta mempertimbangkan variasi pendekatan dan dampaknya terhadap sistem, khususnya dalam hal performa dan arsitektur.



## Variasi Observer Pattern yang Digunakan

Dalam proyek ini, kita menggunakan **Push Model** dari pola Observer.

### Ciri-Ciri Push Model yang Terlihat di Kode
- Publisher (misalnya `NotificationService` atau `ProductService`) secara **aktif mengirim data** ke semua subscriber ketika suatu event terjadi.
- Subscriber **tidak perlu meminta data**; data dikirim langsung dalam bentuk notifikasi.
- Informasi yang dikirim sudah dikemas lengkap, biasanya dalam bentuk JSON atau payload tertentu.

Dengan kata lain, **publisher mendorong notifikasi kepada subscriber** secara langsung tanpa menunggu permintaan dari subscriber.



## Pertimbangan Jika Menggunakan Pull Model

### Apa Itu Pull Model?
Dalam Pull Model, subscriber diberi tahu bahwa ada update (bisa dengan sinyal sederhana), lalu mereka **menarik atau mengambil data** sendiri dari publisher.

### Kelebihan Pull Model
1. **Kontrol di Tangan Subscriber**  
   Subscriber dapat menentukan kapan dan bagaimana mereka mengambil data.

2. **Minim Beban Publisher**  
   Publisher tidak perlu menyiapkan data dan mengirimkannya ke semua subscriber secara langsung.

3. **Toleransi Kegagalan Lebih Baik**  
   Jika subscriber sedang tidak aktif, data tetap tersedia untuk diambil kemudian.

### Kekurangan Pull Model
1. **Delay Informasi**  
   Subscriber mungkin tidak mendapatkan update secara real-time, tergantung frekuensi polling.

2. **Overhead pada Subscriber dan Jaringan**  
   Subscriber harus secara berkala melakukan permintaan untuk memeriksa apakah ada update.

3. **Arsitektur Lebih Rumit**  
   Diperlukan endpoint dan logika tambahan untuk subscriber melakukan request terhadap data terbaru.

4. **Notifikasi Bisa Tidak Terlihat**  
   Jika subscriber tidak melakukan polling secara berkala, mereka bisa saja melewatkan update penting.

### Kesimpulan
Meskipun Pull Model memberikan fleksibilitas pada subscriber, dalam konteks BambangShop yang membutuhkan pengiriman notifikasi secara langsung dan tepat waktu, **Push Model lebih sesuai** karena memastikan semua subscriber menerima informasi tanpa keterlambatan.



## Dampak Jika Tidak Menggunakan Multi-Threading dalam Proses Notifikasi

### Apa yang Terjadi Saat Tidak Menggunakan Multi-Threading?

1. **Blocking pada Proses Notifikasi**
   - Pengiriman notifikasi dilakukan satu per satu secara sinkron.
   - Proses utama (misalnya penambahan produk) tertunda karena menunggu semua notifikasi selesai dikirim.

2. **Penurunan Respons Aplikasi**
   - Operasi API akan terasa lambat atau bahkan gagal ketika jumlah subscriber meningkat.
   - Permintaan HTTP dari subscriber yang lambat dapat menahan seluruh alur proses.

3. **Risiko Timeout**
   - Jika pengiriman notifikasi ke salah satu subscriber memakan waktu lama, seluruh proses dapat mengalami timeout, terutama pada sistem real-time.

4. **Kehilangan Skalabilitas**
   - Sangat sulit untuk menangani skenario dengan ratusan atau ribuan subscriber tanpa eksekusi paralel.

### Keuntungan Multi-Threading
- Notifikasi dikirim secara paralel ke semua subscriber.
- Publisher tidak perlu menunggu satu per satu.
- Performa sistem lebih stabil, cepat, dan responsif, terutama saat ada lonjakan jumlah subscriber.

### Kesimpulan
Tanpa multi-threading, sistem akan mengalami bottleneck serius dalam proses notifikasi. Oleh karena itu, **multi-threading adalah kebutuhan krusial** dalam skenario ini untuk menjaga performa dan pengalaman pengguna yang baik.



