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
