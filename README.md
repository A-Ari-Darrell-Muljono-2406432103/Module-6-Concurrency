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