# Local Authority

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2022 |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Easy |
| Status | Solved |

## Deskripsi

Challenge ini menyediakan sebuah `Secure Customer Portal` dengan form login. Tujuannya adalah
menemukan cara aplikasi memeriksa password, kemudian mendapatkan akses ke halaman admin dan
flag.

## Informasi Awal

Instance challenge dapat diakses melalui:

```text
http://saturn.picoctf.net:60017
```

Halaman login menampilkan pesan:

```text
Only letters and numbers allowed for username and password.
```

Hint challenge:

```text
How is the password checked on this website?
```

## Tools

- Firefox
- Firefox Developer Tools, panel `Inspector`
- Firefox Developer Tools, panel `Debugger`

## Analisis

Percobaan login pada instance awal menghasilkan `Log In Failed`. Source HTML halaman login
kemudian diperiksa. Source tersebut memuat file JavaScript eksternal `secure.js` dan form
tersembunyi yang mengarah ke `admin.php`.

File `secure.js` berisi fungsi `checkPassword` yang membandingkan username dan password secara
langsung di sisi client. Nilai kredensial tersebut dapat dibaca melalui Developer Tools,
sehingga tidak perlu menebak password.

## Langkah Penyelesaian

### 1. Membuka portal

Portal dibuka melalui Firefox. Pada instance awal, alamat yang terlihat adalah:

```text
http://saturn.picoctf.net:60017
```

Halaman menampilkan judul `Secure Customer Portal` dan form login.

### 2. Menguji login awal

Percobaan login pada instance awal menghasilkan halaman:

```text
Log In Failed
```

Karena login biasa tidak berhasil, source halaman diperiksa untuk mencari cara aplikasi
melakukan validasi password.

### 3. Memeriksa source HTML

Pada Firefox Developer Tools, panel `Inspector` memperlihatkan bagian penting berikut:

```html
<script src="secure.js"></script>
<form id="hiddenAdminForm" hidden="" action="admin.php" method="post"></form>
```

File `secure.js` dan form tersembunyi `admin.php` menjadi petunjuk bahwa validasi login
dilakukan oleh JavaScript dan halaman admin tersedia sebagai endpoint terpisah.

### 4. Membaca fungsi validasi password

File `secure.js` dibuka pada panel `Debugger`. Fungsi validasi yang terlihat adalah:

```javascript
function checkPassword(username, password)
{
    if( username === 'admin' && password === 'strongPassword098765' )
    {
        return true;
    }
    else
    {
        return false;
    }
}
```

Kredensial yang diberikan oleh source JavaScript adalah:

```text
Username: admin
Password: strongPassword098765
```

### 5. Login sebagai admin

Kredensial tersebut dimasukkan ke form login pada instance:

```text
http://saturn.picoctf.net:60017
```

Setelah login berhasil, browser diarahkan ke `admin.php` dan menampilkan flag.

## Flag

```text
picoCTF{j5_15_7r4n5p4r3n7_05df90c8}
```

## Pembelajaran

- Validasi kredensial yang dilakukan sepenuhnya di client-side dapat dibaca oleh pengguna.
- File JavaScript eksternal harus diperiksa ketika aplikasi web menggunakan validasi di
  browser.
- Hidden form atau endpoint tersembunyi bukan mekanisme authorization.
- Kredensial dan logika autentikasi tidak boleh ditanam langsung di source JavaScript.
