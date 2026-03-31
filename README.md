# BambangShop Receiver App
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
4.  `repository`: this module contains structs that serve as databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a Rocket web framework skeleton that you can work with.

As this is an Observer Design Pattern tutorial repository, you need to implement a feature: `Notification`.
This feature will receive notifications of creation, promotion, and deletion of a product, when this receiver instance is subscribed to a certain product type.
The notification will be sent using HTTP POST request, so you need to make the receiver endpoint in this project.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Receiver" folder.

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    ROCKET_PORT=8001
    APP_INSTANCE_ROOT_URL=http://localhost:${ROCKET_PORT}
    APP_PUBLISHER_ROOT_URL=http://localhost:8000
    APP_INSTANCE_NAME=Safira Sudrajat
    ```
    Here are the details of each environment variable:
    | variable                | type   | description                                                     |
    |-------------------------|--------|-----------------------------------------------------------------|
    | ROCKET_PORT             | string | Port number that will be listened by this receiver instance.    |
    | APP_INSTANCE_ROOT_URL   | string | URL address where this receiver instance can be accessed.       |
    | APP_PUUBLISHER_ROOT_URL | string | URL address where the publisher instance can be accessed.       |
    | APP_INSTANCE_NAME       | string | Name of this receiver instance, will be shown on notifications. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)
3.  To simulate multiple instances of BambangShop Receiver (as the tutorial mandates you to do so),
    you can open new terminal, then edit `ROCKET_PORT` in `.env` file, then execute another `cargo run`.

    For example, if you want to run 3 (three) instances of BambangShop Receiver at port `8001`, `8002`, and `8003`, you can do these steps:
    -   Edit `ROCKET_PORT` in `.env` to `8001`, then execute `cargo run`.
    -   Open new terminal, edit `ROCKET_PORT` in `.env` to `8002`, then execute `cargo run`.
    -   Open another new terminal, edit `ROCKET_PORT` in `.env` to `8003`, then execute `cargo run`.

## Mandatory Checklists (Subscriber)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop-receiver to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create SubscriberRequest model struct.`
    -   [ ] Commit: `Create Notification database and Notification repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Notification repository.`
    -   [ ] Commit: `Implement list_all_as_string function in Notification repository.`
    -   [ ] Write answers of your learning module's "Reflection Subscriber-1" questions in this README.
-   **STAGE 3: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Commit: `Implement receive_notification function in Notification service.`
    -   [ ] Commit: `Implement receive function in Notification controller.`
    -   [ ] Commit: `Implement list_messages function in Notification service.`
    -   [ ] Commit: `Implement list function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Subscriber-2" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Subscriber) Reflections

#### Reflection Subscriber-1

1. Karena framework web seperti Rocket memproses request secara multi-threading (banyak thread berjalan bersamaan), kita butuh mekanisme pengunci (lock) agar data Vec notifikasi kita tidak rusak saat diakses berbarengan (data race).

Mutex (Mutual Exclusion): Sangat ketat. Hanya membolehkan 1 thread saja yang mengakses data pada satu waktu, entah itu untuk membaca atau menulis. Thread yg lain jadi haru antri.

RwLock (Read-Write Lock): Lebih fleksibel. Memperbolehkan banyak thread membaca data secara bersamaan (asalkan tidak ada yang sedang menulis). Tapi kalau ada yang mau menulis, baru ia akan mengunci semuanya.
Di aplikasi Receiver ini, operasi membaca daftar notifikasi kemungkinan akan dilakukan lebih sering oleh user secara bersamaan. Kalau pakai Mutex, proses read saja harus saling tunggu sehingga bikin program jadi lambat. Dengan RwLock, banyak user bisa melihat notifikasi secara barengan tanpa harus mengantri, sehingga kinerjanya jadi jauh lebih optimal.


2. Di Java, variabel static memang bisa diubah kapan saja. Tapi sayangnya, fitur itu adalah salah satu penyebab utama terjadinya bug dan data race yang sulit dilacak saat program berjalan di banyak thread. Rust punya prinsip: keselamatan memori tanpa garbage collector. Kompilator Rust sangat ketat (dikenal dengan borrow checker) dan secara standar melarang mutasi pada variabel global (static mut) karena itu dianggap tidak aman dan berisiko tinggi menyebabkan bentrok antar thread. Oleh karena itu, kita butuh lazy_static. Pustaka ini membantu kita membuat variabel global yang inisialisasinya ditunda sampai variabel itu pertama kali dipanggil saat runtime. Di dalamnya, kita membungkus tipe data kita (seperti Vec atau DashMap) dengan pelindung thread-safe seperti RwLock atau Mutex, sehingga kompilator Rust bisa yakin 100% bahwa datanya aman untuk diakses dan diubah secara paralel.


#### Reflection Subscriber-2

1. Ya, saya mengeksplorasi file src/lib.rs pada aplikasi Publisher (Main App) karena saya sempat mengalami sebuah kejanggalan. Saat mencoba membuat produk baru di Publisher (yang berjalan di port 8000), pesan notifikasi yang diterima oleh Receiver justru mencantumkan URL produk yang mengarah ke port yang salah, yaitu http://localhost:8001/product/0. Karena penasaran, saya membedah file src/lib.rs milik Publisher dan menemukan penyebab utamanya. Pada blok implementasi Default untuk AppConfig, ternyata ada konfigurasi bawaan yang di-hardcode seperti ini: instance_root_url: String::from("http://localhost:8001"). Dari penemuan ini, saya belajar cara kerja sistem konfigurasi di Rust (menggunakan figment dan dotenvy). Karena awalnya saya tidak memiliki file .env di folder Publisher, fungsi generate() gagal melakukan merge dengan variabel environment. Alhasil, aplikasi menggunakan nilai fallback atau "setelan pabrik" tersebut. Ini mengajarkan saya bahwa lib.rs bertindak sebagai pondasi konfigurasi dasar, dan kita bisa dengan mudah menimpanya (override) tanpa harus mengubah kode inti aplikasi, yaitu cukup dengan mendefinisikan variabel APP_INSTANCE_ROOT_URL=http://localhost:8000 di dalam file .env.


2. Observer pattern sangat memudahkan penambahan subscriber baru (scalable) karena Main App (Publisher) dan Receiver saling lepas (decoupled). Main App tidak perlu dimodifikasi atau di restart setiap kali ada subscriber baru. Selama subscriber menembak endpoint /subscribe dengan format yang benar, Main App cukup menyimpannya ke dalam database memori (seperti Vec dengan pelindung RwLock) dan otomatis menyiarkan notifikasinya. Namun, untuk menjalankan lebih dari 1 instance Main App secara bersamaan, sistemnya tidak akan semudah itu untuk ditambahkan. Saat ini, Subscriber didesain hanya untuk memegang 1 URL Publisher. Jika ada banyak Publisher, Subscriber harus melacak semua URL tersebut. Untuk kasus multi-publisher, arsitektur ini kurang cocok dan biasanya membutuhkan perantara tambahan yang lebih kompleks seperti Message Broker (contoh: RabbitMQ atau Kafka) agar manajemen pesannya tidak berantakan.


3. Selama mengerjakan tutorial ini, saya belum membuat automated Tests secara ekstensif atau merombak total dokumentasinya. Namun, saya sangat bergantung pada dokumentasi Postman Collection yang sudah disediakan dan mencoba menyesuaikan beberapa environment variables (seperti mengganti port dan URL) untuk skenario pengujian multi-instance. Dari pengalaman menggunakan collection bawaan tersebut, saya menyadari bahwa dokumentasi API yang tertata rapi (dengan pengelompokan folder yang jelas untuk Publisher dan Receiver) sangatlah krusial untuk melacak alur kerja aplikasi. Praktik mendokumentasikan API dan menyediakan contoh request/response di Postman ini jelas akan sangat bermanfaat di dunia nyata.