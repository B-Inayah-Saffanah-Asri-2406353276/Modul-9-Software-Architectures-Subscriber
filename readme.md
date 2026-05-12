# Modul 9 Software Architecture - Subscriber

### Questions
1. **What is amqp?**

Advanced Message Queuing Protocol (AMQP) adalah protokol komunikasi standar yang digunakan untuk pertukaran pesan antar aplikasi atau layanan. AMQP membuat sistem saling berkomunikasi melalui message broker seperti RabbitMQ.

2. **What does it mean? guest:guest@localhost:5672 , what is the first guest, and what is the second guest, and what is localhost:5672 is for?**

`guest` pertama adalah username untuk login ke server RabbitMQ, `guest` kedua untuk password akun tersebut, dan `localhost:5672` adalah alamat server RabbitMQ (`localhost` berarti server berjalan di komputer tersebut dan `5672` adalah port default yang digunakan RabbitMQ untuk komunikasi)

### Slow subscriber
![alt text](image-2.png)
Berdasarkan gambar, total pesan dalam queue adalah 16. Hal ini terjadi karena ini adalah simulasi slow subscriber, publisher mengirimkan banyak pesan dengan cepat secara berulang kali, namun subscriber memproses pesan lebih lambat, sehingga pesan-pesan yang belum sempat diproses menumpuk di queue. Angka 16 mencerminkan jumlah pesan yang sudah diterima subscriber tapi belum selesai diproses, dan jumlah tersebut akan terus bertambah setiap kali publisher dijalankan sebelum subscriber selesai memproses antrian sebelumnya.

### Running three subscriber
![alt text](image-3.png)
![alt text](image-5.png)
Berdasarkan gambar, terlihat bahwa dengan menjalankan 3 subscriber sekaligus membuat queue turun jauh lebih cepat dibanding hanya 1 subscriber. Hal ini terjadi karena beban pemrosesan dibagi rata ke semua subscriber yang terhubung ke queue yang sama. Ketika publisher mengirim banyak event sekaligus, RabbitMQ mendistribusikan pesan-pesan tersebut ke masing-masing subscriber, sehingga yang tadinya harus diproses satu per satu kini diproses secara paralel.

### Perbaikan code publisher dan susbcriber
**Publisher**: Kode publisher saat ini hardcode 5 pesan dengan nama yang statis. Perbaikan yang bisa dilakukan adalah membuat data lebih dinamis, misalnya membaca input dari user atau menggunakan loop untuk mengirim banyak pesan sekaligus. Selain itu, publisher tidak memiliki error handling yang proper sebaiknya di-handle dengan `match` atau `unwrap_or_else` agar error tidak terlewat.

**Subscriber**: Variabel `now` didefinisikan tapi now tidak pernah digunakan, ini code smell yang sebaiknya dibuang. Lalu, loop {} di main menggunakan CPU secara tidak efisien karena busy-waiting, lebih baik diganti dengan `std::thread::park()` atau mekanisme blocking lainnya agar tidak membuang resource.