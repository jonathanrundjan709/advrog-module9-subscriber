# Tutorial 9

## Pertanyaan

### a. Apa itu AMQP?

AMQP adalah singkatan dari Advanced Message Queuing Protocol. AMQP merupakan protokol standar terbuka yang digunakan untuk komunikasi berbasis pesan. Protokol ini memungkinkan aplikasi untuk mengirim dan menerima pesan melalui message broker, seperti RabbitMQ.

Pada repository ini, publisher mengirim event `user_created` ke message broker, sedangkan subscriber mendengarkan queue atau event yang sama dan memproses pesan yang diterima. Dengan AMQP, publisher dan subscriber dapat berkomunikasi secara asynchronous, sehingga publisher tidak perlu memanggil subscriber secara langsung.

### b. Apa arti `guest:guest@localhost:5672`?

Connection string yang digunakan pada kode adalah:

```text
amqp://guest:guest@localhost:5672
```

Connection string tersebut adalah alamat yang digunakan aplikasi untuk terhubung ke AMQP broker.

- `guest` yang pertama adalah username yang digunakan untuk autentikasi.
- `guest` yang kedua adalah password yang digunakan untuk autentikasi.
- `localhost` berarti broker berjalan di komputer yang sama dengan aplikasi.
- `5672` adalah port default yang digunakan RabbitMQ untuk koneksi AMQP.

Jadi, `guest:guest@localhost:5672` berarti aplikasi akan terhubung ke server RabbitMQ yang berjalan secara lokal pada port `5672`, dengan username `guest` dan password `guest`.

## Dokumentasi

Berikut adalah hasil ketika subscriber menerima message dari publisher:

![Hasil subscriber menerima message dari publisher](assets/subscriber-output.png)

## Simulasi Slow Subscriber

Berikut adalah hasil monitoring RabbitMQ ketika subscriber dibuat lebih lambat dalam memproses message:

![Monitoring RabbitMQ saat slow subscriber](assets/slow-subscriber-queue.png)

Pada screenshot tersebut, jumlah queued messages sempat naik hingga sekitar 25 message. Angka ini muncul karena publisher dapat terus mengirim event ke message broker, sedangkan subscriber memproses message secara lebih lambat satu per satu.

Program publisher mengirim 5 message setiap kali dijalankan. Jika publisher dijalankan beberapa kali dalam waktu singkat ketika subscriber sedang lambat, message baru akan masuk lebih cepat daripada kemampuan subscriber untuk memprosesnya. Akibatnya, message menumpuk sementara di queue RabbitMQ. Setelah subscriber selesai memproses message-message tersebut, jumlah queue turun kembali ke 0.

## Reflection and Running at Least Three Subscribers

Berikut adalah hasil monitoring RabbitMQ ketika subscriber dijalankan lebih dari satu:

![Monitoring RabbitMQ dengan beberapa subscriber](assets/three-subscribers-monitoring.png)

Pada percobaan ini, spike pada queued messages berkurang lebih cepat dibandingkan ketika hanya ada satu subscriber. Hal ini terjadi karena beberapa subscriber dapat mengambil dan memproses message dari queue yang sama secara paralel. Publisher tetap dapat mengirim banyak event ke message broker, tetapi beban pemrosesan tidak hanya ditangani oleh satu consumer saja.

Jika hanya ada satu subscriber dan setiap message membutuhkan waktu sekitar 1 detik untuk diproses, maka queue akan turun perlahan. Ketika ada beberapa subscriber, RabbitMQ membagikan message ke consumer yang tersedia, sehingga beberapa message dapat diproses pada waktu yang berdekatan. Karena itu, total queue lebih cepat berkurang setelah spike muncul.

Dari kode publisher dan subscriber, ada beberapa hal yang bisa diperbaiki. Pada publisher, data event masih ditulis satu per satu dengan beberapa pemanggilan `publish_event`; ini bisa dibuat lebih rapi dengan menyimpan data dalam array atau vector lalu melakukan loop. Pada subscriber, nilai delay masih ditulis langsung di kode menggunakan `thread::sleep`; akan lebih baik jika durasi delay dibuat sebagai konfigurasi agar lebih mudah diubah untuk simulasi. Selain itu, bagian `loop {}` di akhir subscriber sebaiknya diganti dengan mekanisme menunggu yang lebih jelas agar program tidak melakukan busy waiting.
