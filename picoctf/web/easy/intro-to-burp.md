# IntroToBurp

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2024 |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Easy |
| Status | Solved |

## Deskripsi

Challenge ini menyediakan aplikasi web dengan proses registrasi yang dilanjutkan oleh
verifikasi 2FA. Tujuannya adalah melewati permintaan OTP dan mendapatkan flag.

## Informasi Awal

Instance challenge dapat diakses melalui:

```text
http://titan.picoctf.net:64022
```

Form registrasi meminta beberapa data:

- Full Name
- Username
- Phone Number
- City
- Password

Hint challenge:

1. `Try using burpsuite to intercept request to capture the flag.`
2. `Try mangling the request, maybe their server-side code doesn't handle malformed requests very well.`

## Tools

- Firefox
- Firefox Developer Tools, panel `Storage`
- `flask-unsign`
- Terminal Linux

## Analisis

Setelah registrasi, aplikasi mengarahkan pengguna ke halaman `2fa authentication` yang
meminta OTP. Cookie browser bernama `session` menyimpan data session Flask dalam bentuk
signed cookie.

Nilai cookie tersebut diperiksa menggunakan `flask-unsign`. Hasil decoding memperlihatkan
bahwa session memuat field `otp`, sehingga OTP dapat diambil dari session tanpa melakukan
guessing.

## Langkah Penyelesaian

### 1. Membuka form registrasi

Instance dibuka melalui Firefox pada alamat berikut:

```text
http://titan.picoctf.net:64022
```

Halaman registrasi menampilkan field `Full Name`, `Username`, `Phone Number`, `City`, dan
`Password`.

### 2. Membuat akun

Form diisi dengan data berikut:

```text
Full Name: administrator
Username: administrator
Phone Number: 0123456789
City: city a
Password: admin123
```

Setelah tombol `Register` ditekan, aplikasi mengarahkan ke halaman dashboard yang meminta
OTP melalui form `2fa authentication`.

### 3. Memeriksa cookie session

Firefox Developer Tools dibuka pada panel `Storage`, kemudian cookie `session` untuk domain
challenge diperiksa. Cookie tersebut memiliki format signed session cookie Flask.

Nilai session tidak disalin penuh ke writeup karena merupakan token session aktif. Bentuk
command yang digunakan untuk mendekode nilainya adalah:

```bash
flask-unsign --decode --cookie '<nilai-cookie-session>'
```

Setelah command dijalankan dengan nilai cookie
sebagai satu argument, `flask-unsign` menampilkan isi session yang telah didekode.

### 4. Mengambil OTP dari hasil decoding

Hasil decoding session memuat data berikut:

```python
{
    'city': 'city a',
    'full_name': 'administrator',
    'otp': '384711',
    'password': 'admin123',
    'phone_number': '0123456789',
    'username': 'administrator'
}
```

Field `otp` memberikan nilai OTP yang dibutuhkan oleh halaman 2FA:

```text
384711
```

### 5. Mengirim OTP

Nilai `384711` dimasukkan ke field `Enter OTP`, kemudian tombol `Submit` ditekan.

Server menerima OTP dan menampilkan pesan bahwa permintaan OTP berhasil dilewati beserta
flag:

```text
Welcome, administrator you successfully bypassed the OTP request. Your Flag:
picoCTF{#OTP_Byp@ss_SuCc3$$_c94b61ac}
```

## Flag

```text
picoCTF{#OTP_Byp@ss_SuCc3$$_c94b61ac}
```

## Pembelajaran

- Data sensitif seperti OTP tidak boleh disimpan di client-side session yang dapat dibaca
  pengguna.
- Signed cookie dapat menjaga integritas data, tetapi tidak menyembunyikan isi data jika
  tidak dienkripsi.
- Firefox Developer Tools dapat digunakan untuk memeriksa cookie session pada browser.
- `flask-unsign` membantu membaca struktur data dari signed Flask session cookie.
