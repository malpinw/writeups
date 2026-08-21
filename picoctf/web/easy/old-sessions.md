# Old Sessions

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2026 |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Easy |
| Status | Solved |

## Deskripsi

Challenge ini membahas pentingnya pengaturan waktu kedaluwarsa session. Jika pengguna
tidak melakukan logout dengan benar, session dapat tetap aktif pada browser yang sama.
Challenge menyediakan aplikasi web yang memiliki session aktif untuk beberapa akun.

Tujuannya adalah menemukan cara untuk mengakses akun lain melalui session yang masih
valid dan mendapatkan flag.

## Informasi Awal

Instance yang terlihat pada screenshot dapat diakses melalui:

```text
http://dolphin-cove.picoctf.net:51198
```

Endpoint yang terlihat selama penyelesaian:

- `/login`
- `/register`
- `/sessions`

Pada halaman login dan register tersedia field `Username` serta `Password`. Akun
`administrator` dibuat dan digunakan untuk login dengan password `admin123`.

## Tools

- Firefox
- Firefox Developer Tools, khususnya panel `Storage` dan `Cookies`

## Analisis

Setelah login sebagai `administrator`, halaman utama menampilkan komentar dari beberapa
pengguna. Salah satu komentar menyebutkan adanya halaman aneh pada `/sessions`.

Halaman tersebut menampilkan dua session beserta data internalnya:

```text
1) session: QHubbuvA11tDc3YKR1vhYIOLKaKG1X-3CzhfpVYAg0, {'_permanent': True, 'key': 'admin'}
2) session: CjMq1Oveg9lYsBgJFy0wOhOaGgQSXTfAQrVJFB8K0tM, {'_permanent': True, 'key': 'administrator'}
```

Informasi ini menunjukkan bahwa nilai cookie `session` untuk akun `admin` dapat diketahui.
Cookie session akun `administrator` juga terlihat pada Firefox Developer Tools. Dengan
mengganti nilainya menggunakan session milik `admin`, browser dapat digunakan untuk
mengakses akun tersebut tanpa memasukkan kredensial akun `admin`.

## Langkah Penyelesaian

### 1. Membuka aplikasi web

Instance menampilkan halaman login pada endpoint `/login`. Tersedia tombol `Login` dan
`Register`.

### 2. Membuat akun dan login

Pada halaman `/register`, akun dengan username `administrator` dibuat menggunakan
password `admin123`. Setelah itu, akun tersebut digunakan untuk login melalui `/login`
dengan password yang sama.

### 3. Membaca petunjuk pada halaman utama

Setelah berhasil login, halaman utama menampilkan pesan `Welcome administrator` serta
daftar komentar. Salah satu komentar berbunyi:

```text
Hey I found a strange page at /sessions
```

Komentar tersebut menjadi petunjuk untuk membuka endpoint `/sessions`.

### 4. Mengakses endpoint `/sessions`

Endpoint `/sessions` menampilkan session yang tersimpan beserta account key-nya. Session
yang terkait dengan akun `admin` terlihat sebagai berikut:

```text
QHubbuvA11tDc3YKR1vhYIOLKaKG1X-3CzhfpVYAg0
```

Session yang sedang digunakan akun `administrator` terlihat sebagai berikut:

```text
CjMq1Oveg9lYsBgJFy0wOhOaGgQSXTfAQrVJFB8K0tM
```

Dengan demikian, endpoint ini membocorkan identifier session untuk akun lain.

### 5. Mengganti cookie session

Pada Firefox Developer Tools, panel `Storage` dibuka dan cookie `session` untuk domain
instance dipilih. Nilai cookie yang awalnya mengarah ke akun `administrator` kemudian
diganti menjadi nilai session milik `admin`:

```text
QHubbuvA11tDc3YKR1vhYIOLKaKG1X-3CzhfpVYAg0
```

Setelah halaman dimuat ulang, aplikasi menampilkan `Welcome admin`. Halaman tersebut
juga menampilkan flag.

## Flag

```text
picoCTF{s3t_s3ss10n_3xp1rat10n5_11cae9aa}
```

## Pembelajaran

- Endpoint internal yang menampilkan session dapat menyebabkan session hijacking.
- Cookie session harus memiliki masa berlaku dan invalidasi yang sesuai.
- Logout dan penutupan tab browser tidak boleh dianggap sebagai pengganti invalidasi
  session di server.
- Informasi session tidak boleh ditampilkan kepada pengguna atau dibocorkan melalui
  endpoint aplikasi.

