# logon

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2019 (via CyLab Security Academy) |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Easy |
| Status | Solved |

## Deskripsi

Challenge ini menyediakan aplikasi web bernama **Factory Login**. Deskripsi challenge
menyatakan bahwa factory menyembunyikan sesuatu dari seluruh penggunanya. Tujuannya
adalah mendapatkan flag melalui aplikasi login tersebut.

## Informasi Awal

- Instance dapat diakses melalui
  `http://fickle-tempest.picoctf.net:60980`.
- Halaman login memiliki field `Username`, field `Password`, dan tombol `Sign In`.
- Hint yang diberikan:

  ```text
  Hmm it doesn't seem to check anyone's password, except for Joe's?
  ```

- Setelah login, endpoint `/flag` awalnya menampilkan pesan `No flag for you`.

## Tools

- Firefox
- Firefox Developer Tools, khususnya panel `Storage` dan `Cookies`

## Analisis

Login berhasil, tetapi halaman `/flag` belum memberikan flag. Pemeriksaan cookie pada
Firefox Developer Tools memperlihatkan bahwa status administrator disimpan dalam cookie
yang dapat dibaca dan diubah dari sisi klien. Nilai awal cookie `admin` adalah `False`.

Cookie lain yang terlihat adalah `username=administrator` dan `password=admin123`.
Karena nilai `admin` berubah menjadi `True` dan halaman kemudian menampilkan flag,
aplikasi mempercayai nilai cookie tersebut saat menentukan hak akses ke flag.

## Langkah Penyelesaian

### 1. Membuka instance challenge

Instance dibuka melalui Firefox. Halaman menampilkan judul `Factory Login` serta form
login. Pada modal challenge, hint mengarahkan perhatian pada pemeriksaan password dan
akun Joe.

### 2. Login ke aplikasi

Form diisi dengan username `administrator`. Password pada field ditampilkan sebagai
karakter tersamarkan. Setelah tombol **Sign In** ditekan, aplikasi
menampilkan pesan:

```text
Success: You logged in! Not sure you'll be able to see the flag though.
```

Namun, halaman `/flag` masih menampilkan:

```text
No flag for you
```

### 3. Memeriksa cookie aplikasi

Panel **Storage** pada Firefox Developer Tools dibuka dan cookie untuk domain instance
diperiksa. Nilai yang terlihat adalah:

```text
admin=False
password=admin123
username=administrator
```

Cookie `admin` menjadi indikator penting karena nilainya masih `False` meskipun login
sebagai `administrator` telah berhasil.

### 4. Mengubah status administrator

Nilai cookie `admin` diubah dari `False` menjadi `True`. Setelah halaman `/flag` dimuat
ulang, nilai cookie terlihat sebagai:

```text
admin=True
```

Halaman kemudian menampilkan bagian `Flag`, sehingga perubahan cookie berhasil mengubah
akses aplikasi.

## Flag

```text
picoCTF{th3_consp1r4cy_1l1v3s_4d184b0d}
```

## Pembelajaran

- Data yang menentukan hak akses tidak boleh dipercaya jika disimpan dan dapat diubah
  langsung oleh klien.
- Berhasil login tidak selalu berarti pengguna memiliki akses ke seluruh halaman;
  pemeriksaan otorisasi perlu dilakukan secara aman di server.
- Firefox Developer Tools dapat digunakan untuk memeriksa cookie yang dikirim aplikasi
  dan membantu menemukan perbedaan antara status login serta status administrator.

## Referensi

- [picoCTF](https://picoctf.org/)
