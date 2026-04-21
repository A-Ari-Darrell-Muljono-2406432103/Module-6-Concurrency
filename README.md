## Commit 1 Reflection Notes

Pada milestone ini, saya mempelajari cara membuat web server sederhana menggunakan Rust
dengan TcpListener. Program mendengarkan koneksi TCP di alamat 127.0.0.1:7878.

Fungsi `handle_connection` menerima TcpStream dan membaca HTTP request dari browser
menggunakan BufReader. Request dibaca baris per baris menggunakan `.lines()`, kemudian
diambil sampai menemukan baris kosong (yang menandakan akhir dari HTTP header) 
menggunakan `.take_while(|line| !line.is_empty())`.

Hasilnya berupa Vec<String> yang berisi header-header HTTP seperti method (GET), 
path, versi HTTP, Host, User-Agent, dan informasi lainnya yang dikirim browser.

## Commit 2 Reflection Notes

Pada milestone ini, server sudah bisa mengembalikan halaman HTML ke browser.
Fungsi `handle_connection` dimodifikasi untuk membaca file `hello.html` menggunakan
`fs::read_to_string()`, lalu mengirimkannya sebagai HTTP response.

Format HTTP response yang benar terdiri dari status line (`HTTP/1.1 200 OK`),
header `Content-Length` yang berisi panjang isi file, dua CRLF (`\r\n\r\n`) 
sebagai pemisah antara header dan body, lalu isi HTML-nya.

`Content-Length` penting agar browser tahu berapa byte yang harus dibaca 
sebagai body response, sehingga koneksi bisa dikelola dengan benar.

## Commit 3 Reflection Notes

Pada milestone ini, server sudah bisa membedakan request yang valid dan tidak valid.
Hanya request `GET / HTTP/1.1` yang akan mendapatkan response `200 OK` dengan 
halaman `hello.html`. Selain itu akan mendapat response `404 NOT FOUND` dengan 
halaman `404.html`.

Refactoring dilakukan dengan memisahkan penentuan `status_line` dan `filename` 
ke dalam satu blok kondisional, sehingga logika pembacaan file dan penulisan 
response hanya ditulis sekali. Ini menerapkan prinsip DRY (Don't Repeat Yourself)
agar kode lebih bersih dan mudah di-maintain.

## Commit 4 Reflection Notes

Pada milestone ini, ditambahkan endpoint `/sleep` yang mensimulasikan proses lambat
dengan `thread::sleep(Duration::from_secs(10))`.

Ketika dua tab browser dibuka bersamaan — satu ke `/sleep` dan satu ke `/` — 
tab kedua tidak langsung mendapat response, melainkan ikut menunggu 10 detik 
sampai request `/sleep` selesai diproses. Ini terjadi karena server masih 
single-threaded, sehingga hanya bisa memproses satu request pada satu waktu.

Ini membuktikan kelemahan single-threaded server: jika ada satu request yang 
lambat, semua request lain akan ter-blokir (blocking). Solusinya adalah 
menggunakan multithreading.

## Commit 5 Reflection Notes

Pada milestone ini, server diubah menjadi multithreaded menggunakan ThreadPool.
ThreadPool mengelola sejumlah Worker thread yang siap mengeksekusi job yang masuk.

Cara kerja ThreadPool:
- `ThreadPool::new(size)` membuat sejumlah Worker dan sebuah channel (mpsc).
- Sender disimpan di ThreadPool, Receiver dibungkus Arc<Mutex<>> dan dibagikan ke semua Worker.
- Setiap Worker berjalan dalam loop, menunggu job dari channel via `receiver.lock().unwrap().recv()`.
- `pool.execute(f)` mengirim closure sebagai job melalui channel.
- Arc memungkinkan receiver dimiliki banyak Worker, Mutex memastikan hanya satu Worker 
  yang mengambil job pada satu waktu.

Dengan ThreadPool berisi 4 thread, server kini bisa memproses hingga 4 request 
secara bersamaan. Tab yang mengakses `/sleep` tidak lagi memblokir tab lain.