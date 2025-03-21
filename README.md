### Commit 1 Reflection notes

Fungsi handle_connection bekerja dengan cara membaca permintaan HTTP dari klien menggunakan BufReader, kemudian memprosesnya hingga menemukan baris kosong yang menandakan akhir dari header HTTP. Setelah itu, permintaan dicetak ke konsol.
   ```rust
   let http_request: Vec<_> = buf_reader.lines()
       .map(|result| result.unwrap())
       .take_while(|line| !line.is_empty())
       .collect();
   ```
   - `buf_reader.lines()` mengembalikan iterator yang membaca baris per baris dari stream.
   - `.map(|result| result.unwrap())` digunakan untuk mengambil nilai dari `Result<String, std::io::Error>`.
   - `.take_while(|line| !line.is_empty())` menghentikan pembacaan ketika mendeteksi baris kosong, yang menandakan akhir dari header HTTP.
   - `.collect()` menyimpan hasil pembacaan dalam `Vec<String>` untuk dianalisis lebih lanjut.

### Commit 2 Reflection notes
Fungsi handle_connection sekarang juga mengirimkan respons berupa file HTML ke klien.
   ```rust
   let status_line = "HTTP/1.1 200 OK";
   let contents = fs::read_to_string("hello.html").unwrap();
   let length = contents.len();
   let response = format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");
   stream.write_all(response.as_bytes()).unwrap();
   ```
   - `status_line` adalah status HTTP yang menunjukkan bahwa permintaan berhasil.
   - `fs::read_to_string("hello.html").unwrap();` membaca isi file HTML yang akan dikirim sebagai respons.
   - `length` menghitung panjang konten yang akan dikirim agar dapat disertakan dalam header HTTP.
   - `format!` digunakan untuk menyusun string respons HTTP.
   - `stream.write_all(response.as_bytes()).unwrap();` mengirim respons ke klien dengan memastikan semua byte dikirim tanpa error.

![image](https://github.com/user-attachments/assets/28b2d0ac-8cc1-4673-822d-d3edac426d94)

### Commit 3 Reflection notes
Respon untuk request HTTP yang berbeda dibedakan melalui block if else. Refactoring dibutuhkan untuk mempermudah pemeliharaan kode. Jika kita ingin menambahkan alamat request lain, kita hanya tinggal update kode di satu tempat saja.

![image](https://github.com/user-attachments/assets/bcf7453f-2fc7-48ee-8905-3b84dd5797c4)

### Commit 4 Reflection notes
Karena server berjalan dalam mode single-thread, hanya ada satu thread yang menangani semua request.
Ketika kita membuka browser dan akses 127.0.0.1:7878/sleep. Lalu buka tab baru dan akses 127.0.0.1:7878/. Tab kedua akan tertunda selama 10 detik karena server hanya bisa menangani satu request dalam satu waktu.
Request lainnya harus menunggu hingga yang pertama selesai.

### Commit 5 Reflection notes
ThreadPool dalam konteks server multithreaded di Rust bekerja dengan menciptakan sejumlah thread tetap yang siap menangani tugas secara bersamaan. Saat permintaan masuk, server tidak langsung membuat thread baru untuk setiap koneksi (karena ini bisa menyebabkan overhead besar), melainkan memasukkan tugas ke dalam antrian tugas yang akan dikerjakan oleh thread yang tersedia di dalam pool. Setiap thread dalam ThreadPool akan mengambil tugas dari antrian dan menjalankannya, memungkinkan server untuk menangani banyak koneksi secara bersamaan tanpa menunggu satu tugas selesai sebelum memulai yang lain. 
