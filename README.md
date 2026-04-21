## Commit 1 Reflection Notes

Pada milestone ini, saya mempelajari cara membuat web server sederhana menggunakan Rust
dengan TcpListener. Program mendengarkan koneksi TCP di alamat 127.0.0.1:7878.

Fungsi `handle_connection` menerima TcpStream dan membaca HTTP request dari browser
menggunakan BufReader. Request dibaca baris per baris menggunakan `.lines()`, kemudian
diambil sampai menemukan baris kosong (yang menandakan akhir dari HTTP header) 
menggunakan `.take_while(|line| !line.is_empty())`.

Hasilnya berupa Vec<String> yang berisi header-header HTTP seperti method (GET), 
path, versi HTTP, Host, User-Agent, dan informasi lainnya yang dikirim browser.