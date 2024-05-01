# Refleksi Tutorial Pemrograman Lanjut

<details>
<summary style="font-size:24px">Tutorial 9</summary>

1. Apa perbedaan antara metode RPC (_Remote Procedure Call_) _unary_, server _streaming_, dan _bi-directional_ _streaming_, dan dalam situasi apa masing-masing yang paling cocok?

- _Unary_ RPC adalah jenis RPC yang paling sederhana. Klien mengirimkan satu permintaan dan mendapatkan satu respons kembali. _Unary_ _RPC_ cocok untuk situasi yang mana klien perlu membuat permintaan dan menerima respons tanpa perlu _streaming_ atau komunikasi berkelanjutan. Contohnya termasuk pengambilan data pengguna, pengiriman formulir atau mengeksekusi satu operasi yang membutuhkan respons.

- Server _Streaming_ RPC: Server _streaming_ RPC mirip dengan _unary_ RPC. Bedanya, server ini mengembalikan aliran pesan sebagai respons terhadap permintaan klien. Server _streaming_ RPC cocok untuk situasi server perlu mengirimkan sejumlah data besar ke klien secara berurutan. Contohnya termasuk pembaruan saham secara _real-time_, pengiriman berita, atau transfer _file_ yang besar.

- _Bi-directional Streaming_ RPC: Dalam RPC _streaming_ dua arah, panggilan diinisiasi oleh klien yang memanggil fungsi. Dua aliran bersifat independen sehingga klien dan server dapat membaca dan menulis pesan dalam urutan apa pun. RPC _streaming_ dua arah cocok untuk situasi saat ada kebutuhan untuk komunikasi berlanjut antara klien dan server dalam kedua arah. Contohnya termasuk aplikasi obrolan, _game_ _online_ _multiplayer_, atau pengeditan dokumen kolaboratif.

2. Apa pertimbangan keamanan potensial yang terlibat dalam mengimplementasikan layanan gRPC di Rust, terutama mengenai otentikasi, otorisasi, dan enkripsi data?
    
    Implementing a gRPC service in Rust involves several security considerations, especially concerning authentication, authorization, and data encryption:

   - **Autentikasi**:
      - Pastikan bahwa klien diautentikasi dengan benar sebelum diizinkan mengakses sumber daya atau operasi yang sensitif.
      - Implementasikan mekanisme autentikasi seperti TLS (_Transport Layer Security_) untuk membentuk kepercayaan secara aman antara klien dan server.
      - Gunakan protokol autentikasi seperti OAuth, JWT (JSON Web Tokens), atau skema autentikasi kustom untuk memverifikasi identitas klien.

   - **Otorisasi**:
      - Setelah klien diautentikasi, berlakukan kebijakan otorisasi untuk mengontrol tindakan apa yang dapat dilakukan klien dan data apa yang dapat diakses.
      - Tentukan aturan kontrol akses berdasarkan peran pengguna, izin, atau atribut lainnya.
      - Implementasikan _middleware_ atau _interceptor_ untuk menerapkan kebijakan otorisasi sebelum memproses permintaan.

   - **Enkripsi Data**:
      - Enkripsi data sensitif yang ditransmisikan melalui jaringan untuk melindunginya dari penyadapan dan pemalsuan.
      - Manfaatkan TLS untuk mengenkripsi saluran komunikasi gRPC dan memastikan enkripsi _end-to-end_ antara klien dan server.
      - Konfigurasikan server untuk memerlukan koneksi yang aman dan menolak permintaan tanpa enkripsi.
      - Pertimbangkan penggunaan teknik enkripsi tambahan seperti enkripsi tingkat aplikasi untuk data yang disimpan secara persisten atau ditransmisikan dalam layanan.

3. Apa tantangan atau isu potensial yang mungkin muncul saat menangani _streaming_ dua arah dalam Rust gRPC, terutama dalam skenario seperti aplikasi _chat_?

   - **_Concurrency_ dan Sinkronisasi**:
      - Mengelola aliran yang bersamaan dan memastikan sinkronisasi yang tepat antara klien dan server dapat menjadi kompleks.
      - Sistem kepemilikan dan peminjaman Rust mungkin memerlukan desain yang hati-hati untuk mengelola akses bersama ke sumber daya dengan aman.

   - **_Error_ _Handling_**:
      - Menangani kesalahan yang terjadi secara asinkron di kedua sisi klien dan server.
      - Mekanisme penanganan kesalahan Rust, seperti jenis _Result_ dan propagasi kesalahan perlu dimanfaatkan secara efektif untuk menangani kesalahan dengan baik.

   - **_Message Ordering and Duplication_**:
      - Memastikan pesanan pesan yang benar dan menghindari penggandaan pesan dalam aliran dua arah bisa menantang, terutama dalam skenario dengan kondisi jaringan yang tidak dapat diandalkan.
      - Mengimplementasikan mekanisme untuk mendeteksi dan menangani pesan yang keluar dari urutan atau duplikat sangat penting untuk menjaga integritas data.

   - **_Testing and Debugging_**:
      - Menguji skenario _streaming_ dua arah secara komprehensif, termasuk kasus-kasus ujung dan skenario kegagalan, bisa sangat rumit.
      - Memecahkan masalah terkait komunikasi asinkron dan konkurensi mungkin memerlukan alat dan teknik khusus.

4. Apa kelebihan dan kekurangan dari penggunaan `tokio_stream::wrappers::ReceiverStream` untuk _streaming_ respons dalam layanan Rust gRPC:

    **Kelebihan:**

    - **Fleksibilitas**: `ReceiverStream` memungkinkan pembuatan stream dari jenis data apa pun yang mengimplementasikan _trait_ `tokio::sync::mpsc::Receiver`, memberikan fleksibilitas dalam jenis sumber data yang dapat di-_stream_.

    - **Operasi Asinkron**: Ini memungkinkan pemrosesan asinkron data yang di-_stream_, memungkinkan layanan untuk menangani respons _streaming_ secara efisien tanpa memblokir _thread_ eksekusi.

    - **Asynchronous Operations**: It enables asynchronous processing of streamed data, allowing the service to handle streaming responses efficiently without blocking the execution thread.

    - **Kompatibilitas dengan Tokio**: `ReceiverStream` berintegrasi dengan baik dengan _runtime_ asinkron Tokio, sehingga sangat cocok untuk aplikasi Rust yang dibangun di atas ekosistem Tokio.

    **Kekurangan:**

    - **Kompleksitas _Debugging_**: _Debugging_ kode asinkron, terutama saat menangani aliran data yang kompleks dan eksekusi konkuren, dapat menantang dan mungkin memerlukan alat dan teknik khusus.

    - **Kompleksitas**: Mengelola _stream_ dan saluran asinkron memperkenalkan kompleksitas tambahan ke dalam basis kode, terutama saat menangani penanganan kesalahan, konkurensi, dan manajemen sumber daya.

    - **Potensi _Deadlock_ dan _Race Condition_**: Penanganan operasi asinkron yang salah, seperti memblokir _loop_ _event_ atau gagal menangani kesalahan dengan benar dapat menyebabkan _deadlock_, ra_ce condition_, atau masalah lain yang terkait dengan konkurensi.

5. ICara agar kode Rust gRPC dapat terstruktur untuk memfasilitasi penggunaan kembali dan modularitas sehingga dapat memudahkan pemeliharaan dan perluasan:

   - **Pemisahan Definisi Layanan**:
      - Mendefinisikan antarmuka layanan gRPC secara terpisah dari implementasinya.
      - Simpan definisi layanan di modul atau file terpisah untuk memudahkan navigasi kode dan pemeliharaan.

   - **_Dependency Injection_**:
      - Gunakan prinsip injeksi dependensi atau inversi kontrol untuk memisahkan komponen dan mempromosikan pengujian.
      - Lewatkan dependensi sebagai objek trait atau parameter generik untuk memungkinkan penggantian dan modularitas.

   - **_Reusable Components_**:
      - Identifikasi fungsionalitas atau pola umum yang dapat dienkapsulasi ke dalam komponen atau pustaka yang dapat digunakan kembali.
      - Ekstraksi fungsionalitas bersama ke dalam crate terpisah yang dapat dengan mudah digunakan kembali di banyak proyek.

6. In the **MyPaymentService** implementation, what additional steps might be necessary to handle more complex payment processing logic?

   - **_Error Handling_**:
      - Implementasikan mekanisme penanganan kesalahan yang tangguh untuk mengatasi berbagai skenario kesalahan, seperti kegagalan pembayaran, kesalahan validasi, atau gangguan layanan.
      - Kembalikan kode status gRPC yang sesuai dan pesan kesalahan kepada klien untuk mengkomunikasikan hasil operasi pemrosesan pembayaran secara efektif.

   - **_Transaction Handling_**:
      - Integrasi dengan database transaksional atau gateway pembayaran eksternal untuk mengelola transaksi pembayaran dengan aman dan andal.
      - Implementasikan mekanisme untuk menangani konsistensi transaksional, pembatalan, dan pemulihan dalam kasus kegagalan atau kesalahan selama pemrosesan pembayaran.

   - **_Validation and Authorization_**:
      - Validasi parameter permintaan pembayaran, seperti ID pengguna dan jumlah pembayaran, untuk memastikan integritas data dan mencegah aktivitas penipuan atau jahat.
      - Implementasikan pemeriksaan otorisasi untuk memverifikasi bahwa pengguna yang memulai pembayaran memiliki izin yang diperlukan untuk melakukan operasi tersebut.

7.  Adopsi gRPC sebagai protokol komunikasi dapat memiliki dampak signifikan pada arsitektur dan desain keseluruhan dari sistem terdistribusi, terutama dalam hal interoperabilitas dengan teknologi dan platform lain. Berikut adalah beberapa dampaknya:

    - **_Service Interface Definition_**:
       - gRPC mengandalkan Protocol Buffers (protobuf) untuk definisi antarmuka layanan, yang mempromosikan definisi API yang jelas dan tidak tergantung pada bahasa pemrograman.
       - Penggunaan protobuf memungkinkan tipedata yang kuat dan versioning, sehingga lebih mudah untuk mengembangkan API dari waktu ke waktu tanpa merusak kompatibilitas.

    - **_Efficient Communication_**:
       - gRPC menggunakan HTTP/2 sebagai protokol transportasinya, yang menawarkan fitur seperti multiplexing, kompresi header, dan kontrol aliran.
       - Hal ini menghasilkan komunikasi yang lebih efisien dibandingkan dengan API REST tradisional, mengurangi laten, peningkatan throughput, dan penggunaan sumber daya yang lebih baik.

    - **_Code Generation and Tooling_**:
       - gRPC menyediakan alat pembangkitan kode untuk beberapa bahasa pemrograman, yang memungkinkan pengembang untuk menghasilkan kode klien dan server dari definisi layanan protobuf.
       - Hal ini mempercepat pengembangan dan mengurangi kemungkinan inkonsistensi antara implementasi klien dan server.

8. Menggunakan HTTP/2 sebagai protokol dasar untuk gRPC memiliki beberapa kelebihan dan kekurangan dibandingkan dengan menggunakan HTTP/1.1 atau HTTP/1.1 dengan WebSocket untuk REST API:

    **Kelebihan**

    - **_Multiplexing_**: HTTP/2 mendukung multiplexing, memungkinkan beberapa permintaan dan tanggapan dikirim dan diterima secara bersamaan melalui satu koneksi TCP tunggal. Ini meningkatkan efisiensi dan mengurangi laten, terutama untuk aplikasi dengan banyak permintaan kecil dan independen.

    - **_Header Compression_**: HTTP/2 mengompresi header, mengurangi overhead dan meningkatkan pemanfaatan bandwidth. Ini sangat bermanfaat untuk permintaan dengan header besar, seperti yang digunakan dalam REST API dengan metadata yang luas.

    - **_Binary Protocol_**: HTTP/2 menggunakan mekanisme framing biner, yang lebih ringkas dan efisien dibandingkan dengan representasi teks yang digunakan dalam HTTP/1.1. Ini mengurangi overhead parsing dan meningkatkan kinerja, terutama untuk payload besar.

    **Kekurangan**

    - **_Complexity_**: HTTP/2 lebih kompleks daripada HTTP/1.1, baik dalam hal implementasi protokol maupun pertimbangan operasional. Kompleksitas ini dapat meningkatkan kurva pembelajaran bagi pengembang dan memperkenalkan potensi masalah dalam konfigurasi dan implementasi.

    - **_Server_ _Compatibility_**: Meskipun HTTP/2 didukung secara luas oleh server dan klien web modern, masih mungkin ada masalah kompatibilitas dengan infrastruktur lama atau sistem warisan. Ini dapat membatasi adopsi di lingkungan di mana interoperabilitas menjadi perhatian.

    - **_Resource Consumption_**: Fitur-fitur HTTP/2, seperti multiplexing dan kompresi header, dapat meningkatkan konsumsi sumber daya pada server dan klien, terutama dalam skenario dengan konkurensi tinggi atau jumlah koneksi terbuka yang besar. Ini mungkin memerlukan penyetelan dan manajemen sumber daya yang hati-hati untuk menghindari penurunan kinerja.

9.  Model _request-response_ dari REST API dan kemampuan _streaming_ bidireksional dari gRPC menawarkan pendekatan yang berbeda dalam hal komunikasi waktu nyata dan responsivitas:

    **REST APIs (Request-Response Model):**

    - **_Client-Initiated Requests_**:
      - Dalam REST API, klien biasanya memulai permintaan ke server untuk mengambil atau memanipulasi sumber daya.
      - Setiap permintaan dari klien memicu tanggapan server yang terpisah, mengikuti model permintaan-tanggapan yang stateless.
      - Komunikasi waktu nyata dalam REST API biasanya dicapai melalui mekanisme polling atau long-polling, di mana klien secara berulang-ulang mengirim permintaan ke server untuk memeriksa pembaruan.

    - **_Latency and Scalability_**:
      - REST API cocok untuk skenario di mana rendahnya latensi bukan merupakan kebutuhan kritis, dan volume permintaan relatif rendah atau sporadis.
      - Namun, dalam aplikasi waktu nyata atau sangat interaktif, overhead dari siklus permintaan-tanggapan yang sering dan latensi jaringan dapat mempengaruhi responsivitas dan pengalaman pengguna.

    - **_Synchronous Communication_**:
      - Komunikasi dalam REST API biasanya sinkron, yang berarti bahwa klien harus menunggu tanggapan dari server sebelum melanjutkan dengan tindakan berikutnya.
      - Sifat sinkron ini dapat menyebabkan perilaku blocking dan penurunan konkurensi, terutama dalam skenario dengan banyak klien konkuren atau koneksi yang berlangsung lama.

    **gRPC (Bidirectional Streaming):**

    - **_Simultaneous Data Transfer_**:
      - gRPC supports bidirectional streaming, allowing both clients and servers to send multiple messages asynchronously over a single connection.
      - This bidirectional communication enables real-time data exchange and interaction between clients and servers without the need for repeated request-response cycles.

    - **_Low Latency and High Throughput_**:
      - Dengan streaming bidireksional, gRPC dapat mencapai latensi yang lebih rendah dan throughput yang lebih tinggi dibandingkan dengan REST API, terutama dalam skenario dengan pembaruan yang sering atau persyaratan waktu nyata.
      - Klien dan server dapat bertukar data secara waktu nyata, memungkinkan respons yang lebih cepat dan pengalaman pengguna yang lebih baik, terutama dalam aplikasi interaktif seperti sistem obrolan atau pembaruan langsung.

    - **Asynchronous Communication**:
      - gRPC mempromosikan komunikasi asinkron, memungkinkan klien dan server untuk melanjutkan pemrosesan tugas lain saat menunggu pesan masuk.
      - Sifat asinkron ini meningkatkan konkurensi dan pemanfaatan sumber daya, mengarah pada pola komunikasi yang lebih efisien dan dapat diskalakan.

10. Implikasi dari pendekatan berbasis skema gRPC menggunakan Protocol Buffers dibandingkan dengan sifat yang lebih fleksibel dan tanpa skema dari JSON dalam payload REST API adalah sebagai berikut:

    **Pendekatan berbasis skema gRPC dengan Protocol Buffers**

    - **_Strong Typing and Contractual Agreement_**:
       - Protocol Buffers menerapkan definisi skema yang ketat untuk pesan yang dipertukarkan antara klien dan server dalam gRPC.
       - Tipe kuat ini memastikan bahwa kedua pihak mematuhi kontrak yang telah ditentukan sebelumnya, mengurangi kemungkinan kesalahan dan inkonsistensi komunikasi.

    - **_Efficiency and Compactness_**:
       - Protocol Buffers menggunakan format enkode biner, yang lebih efisien dan kompak dibandingkan dengan format teks seperti JSON.
       - Efisiensi ini menghasilkan ukuran pesan yang lebih kecil dan konsumsi bandwidth jaringan yang lebih rendah, menyebabkan kinerja yang lebih baik dan latensi yang lebih rendah.

    - **_Code Generation_**:
       - Protocol Buffers dapat digunakan untuk menghasilkan kode klien dan server dalam beberapa bahasa pemrograman dari satu definisi skema.
       - Dapat menyederhanakan pengembangan dan memastikan konsistensi antara implementasi klien dan server, mengurangi kemungkinan kesalahan dan ketidaksesuaian.

    **Sifat fleksibel, tanpa skema JSON dalam payload REST API**

    - **_Adaptability and Interoperability_**:
       - Sifat tanpa skema JSON memungkinkan fleksibilitas dan keandalan yang lebih besar dalam merepresentasikan struktur data yang kompleks dan heterogen.
       - Fleksibilitas ini membuat JSON cocok untuk skenario di mana skema data dapat berevolusi dengan cepat atau bervariasi secara luas antara klien dan server yang berbeda.

    - **_Human-Readability and Debugging_**:
       - Format teks JSON mudah dibaca dan dipahami oleh manusia, sehingga cocok untuk debugging dan menemukan masalah interaksi API.
       - Pengembang dapat memeriksa payload JSON secara langsung di browser atau klien API mereka, memfasilitasi prototyping dan pengembangan yang cepat.


</details>