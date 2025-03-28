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

#### Reflection Publisher-3
