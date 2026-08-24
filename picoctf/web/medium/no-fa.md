# No FA

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2026 (via CyLab Security Academy) |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Medium |
| Status | Solved |

## Deskripsi

Challenge `No FA` menyatakan bahwa terdapat data yang bocor dan menanyakan apakah flag
dapat diperoleh dari data tersebut. Aplikasi menggunakan login dan verifikasi 2FA untuk
akun tertentu.

## Informasi Awal

- Instance terlihat pada screenshot dan dapat diakses melalui
  `http://foggy-cliff.picoctf.net:52703`.
- Halaman login berada di endpoint `/login` dan memiliki field `Username`, `Password`,
  serta tombol `LOGIN`.
- Hint yang tersedia:

  ```text
  What happens when there's no salt?
  rockyou rockyou rockyou
  What makes 2FA safe?
  ```

- Source code dan database `users.db` terlihat sebagai artefak yang tersedia selama
  penyelesaian.

## Tools

- Firefox
- Terminal Linux
- DB Browser for SQLite
- CrackStation
- `flask-unsign`

## Analisis

Source code menunjukkan bahwa password dibandingkan dengan hash SHA-256 dari input,
sedangkan database menyimpan kolom `password` dan `two_fa`. Baris akun `admin` pada
database memiliki `two_fa=1`, sehingga login yang benar akan mengarahkan ke halaman OTP.

Source code juga memperlihatkan bahwa OTP disimpan ke dalam session Flask pada field
`otp_secret`. Cookie session yang terbaca oleh browser dapat didekode menggunakan
`flask-unsign`, sehingga OTP dapat diambil tanpa menebak atau menerima email.

## Langkah Penyelesaian

### 1. Memeriksa halaman login

Instance dibuka melalui Firefox. Saat mengakses halaman login, aplikasi menampilkan pesan:

```text
Please login to access this page
```

Form login berisi username dan password. Percobaan login yang terlihat kemudian menghasilkan
pesan:

```text
Invalid username or password
```

### 2. Menganalisis source code aplikasi

Source code Flask yang terlihat menunjukkan alur pemeriksaan password dan 2FA berikut:

```python
if user and hashlib.sha256(password.encode()).hexdigest() == user['password']:
    if user['two_fa']:
        otp = str(random.randint(1000, 9999))
        session['otp_secret'] = otp
        session['otp_timestamp'] = time.time()
        session['username'] = username
        session['logged'] = 'false'
        return redirect(url_for('two_fa'))
    else:
        session['username'] = username
        session['logged'] = 'true'
```

Untuk halaman utama, source code juga menunjukkan bahwa flag hanya diberikan jika username
di session adalah `admin`:

```python
flag = "No flag for you!!"
if session.get('username') == 'admin':
    flag = os.getenv('FLAG')
```

### 3. Menemukan hash password akun admin

Database `users.db` dibuka menggunakan DB Browser for SQLite. Tabel `users` memiliki kolom
`username`, `password`, dan `two_fa`. Baris username `admin` memiliki nilai `two_fa` sebesar
`1` dan hash password berikut:

```text
c20fa16907343eef642d10f0bdb81bf629e6aaf6c906f26eabda079ca9e5ab67
```

Hash tersebut dimasukkan ke CrackStation. Hasil yang ditampilkan adalah:

```text
Type: sha256
Result: apple@123
```

Dengan demikian, password akun `admin` yang digunakan untuk login adalah `apple@123`.

### 4. Login sebagai admin

Username `admin` dan password hasil pemulihan dimasukkan ke form login. Setelah login
berhasil, aplikasi mengarahkan ke endpoint `/two_fa` yang menampilkan halaman:

```text
OTP Verification
Enter the OTP sent to your registered email:
```

Halaman menyediakan empat field angka dan tombol `VERIFY & LOGIN`.

### 5. Memeriksa signed session cookie

Firefox Developer Tools dibuka pada panel **Storage** dan cookie `session` untuk domain
challenge diperiksa. Nilai cookie session yang digunakan pada penyelesaian adalah:

```text
.eJwty0sKgCAQANC7zFpC--jgZWLISQR_qK2iu9ei7YN3QyzeswMLJ8XOIKCMunc-Go8P1SzxtxES90GpglUGzYa4LnLSShuFAq7OLVPi75BLIcPzAiv1HC4.aoxvTg.2PIrSfUWm7WBjkG8TYw9FRrF1Cg
```

Cookie tersebut didekode menggunakan `flask-unsign`:

```bash
flask-unsign --decode --cookie '.eJwty0sKgCAQANC7zFpC--jgZWLISQR_qK2iu9ei7YN3QyzeswMLJ8XOIKCMunc-Go8P1SzxtxES90GpglUGzYa4LnLSShuFAq7OLVPi75BLIcPzAiv1HC4.aoxvTg.2PIrSfUWm7WBjkG8TYw9FRrF1Cg'
```

Hasil decoding menampilkan data session berikut:

```python
{
    'logged': 'false',
    'otp_secret': '1208',
    'otp_timestamp': 1787588430.616718,
    'username': 'admin'
}
```

Field `otp_secret` memberikan OTP yang diperlukan oleh halaman verifikasi:

```text
1208
```

### 6. Mengirim OTP dan mendapatkan flag

Nilai `1208` dimasukkan ke empat field OTP, lalu tombol **VERIFY & LOGIN** ditekan. Aplikasi
menampilkan pesan `Login successful!`, halaman `Welcome!!`, dan flag.

## Flag

```text
picoCTF{n0_r4t3_n0_4uth_6db141c5}
```

## Pembelajaran

- Hash password tanpa salt memudahkan password lemah ditemukan melalui database hash publik.
- Database yang terekspos dapat membocorkan hash password dan konfigurasi 2FA pengguna.
- Signed Flask session cookie tetap dapat dibaca jika hanya ditandatangani dan tidak dienkripsi.
- OTP tidak aman jika nilainya disimpan langsung di session yang dapat dibaca oleh klien.
- Verifikasi 2FA seharusnya menggunakan penyimpanan server-side yang tidak membocorkan secret
  kepada pengguna.

## Referensi

- [picoCTF](https://picoctf.org/)
- [CrackStation](https://crackstation.net/)
