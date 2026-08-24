# Hashgate

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2026 (via CyLab Security Academy) |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Medium |
| Status | Solved |

## Deskripsi

Challenge `Hashgate` menyediakan portal organisasi yang meminta email dan password untuk
login, lalu mengarahkan pengguna ke halaman profil masing-masing. Deskripsi challenge
menyebutkan bahwa akses ke admin memang tidak diekspos secara langsung, tetapi obscurity
bukanlah keamanan. Tujuannya adalah masuk ke profil admin organisasi dan mendapatkan flag.

## Informasi Awal

- Instance dapat diakses melalui
  `http://crystal-peak.picoctf.net:64764`.
- Halaman utama adalah form `Login` dengan field `Email` dan `Password`.
- Deskripsi challenge:

  ```text
  You have gotten access to an organisation's portal. Submit your email and password, and
  it redirects you to your profile. But be careful; just because access to the admin isn't
  directly exposed doesn't mean it's secure. Maybe someone forgot that obscurity isn't
  security... Can you find your way into the admin's profile for this organisation and
  capture the flag?
  ```

- Hint yang tersedia:

  ```text
  Notice anything about how the ID is being checked? It's not plain text... maybe a one-way
  function is involved.
  There are about 20 employees in this organisation.
  ```

## Tools

- Firefox
- Firefox Developer Tools, panel `Inspector`
- CrackStation
- MD5 Hash Generator (Dan's Tools)

## Analisis

Percobaan login dengan `admin@gmail.com` gagal dengan pesan `Invalid credentials.`, sehingga
kredensial admin tidak dapat ditebak langsung. Pemeriksaan source HTML pada panel `Inspector`
menemukan komentar yang membocorkan kredensial akun guest.

Setelah login sebagai guest, URL profil berubah menjadi `/profile/user/<nilai-heksadesimal>`
dan halaman menampilkan `Access level: Guest (ID: 3000)`. Nilai pada URL terbukti merupakan
MD5 dari ID numerik pengguna, sehingga profil pengguna lain dapat diakses melalui IDOR dengan
mengganti hash tersebut menjadi MD5 dari ID admin.

## Langkah Penyelesaian

### 1. Membuka instance dan mencoba login admin

Instance dibuka melalui Firefox. Percobaan login menggunakan `admin@gmail.com` menghasilkan
alert:

```text
Invalid credentials.
```

### 2. Menemukan kredensial guest pada komentar HTML

Panel **Inspector** dibuka untuk memeriksa source halaman. Pada bagian `<head>` terdapat
komentar HTML berikut:

```html
<!-- Email: guest@picoctf.org Password: guest -->
```

Komentar ini memberikan kredensial akun guest untuk login.

### 3. Login sebagai guest dan mengamati URL profil

Form login diisi dengan `guest@picoctf.org` dan password `guest`. Login berhasil dan browser
diarahkan ke:

```text
http://crystal-peak.picoctf.net:64764/profile/user/e93028bdc1aacdfb3687181f2031765d
```

Halaman profil menampilkan:

```text
Access level: Guest (ID: 3000). Insufficient privileges to view classified data. Only
top-tier users can access the flag.
```

### 4. Mengonfirmasi bahwa ID pada URL adalah MD5

Hint pertama menyebutkan one-way function, sedangkan nilai `e93028bdc1aacdfb3687181f2031765d`
memiliki panjang 32 karakter heksadesimal yang khas untuk MD5. Hash tersebut diperiksa pada
CrackStation dan hasilnya cocok:

```text
Hash: e93028bdc1aacdfb3687181f2031765d
Type: md5
Result: 3000
```

Ini mengonfirmasi bahwa segmen URL profil adalah MD5 dari ID numerik pengguna, sehingga
profil pengguna lain dapat diakses tanpa otorisasi jika ID-nya diketahui.

### 5. Menghasilkan MD5 untuk ID admin

Hint kedua menyebutkan organisasi memiliki sekitar 20 karyawan, dan ID guest adalah `3000`,
sehingga ID admin dicari pada rentang ID di sekitar nilai tersebut. String `3019` di-generate
menggunakan MD5 Hash Generator dengan hasil:

```text
Your String: 3019
MD5 Hash: a74c3bae3e13616104c1b25f9da1f11f
```

### 6. Mengakses profil admin dan mendapatkan flag

URL profil guest diubah dengan mengganti segmen hash menjadi MD5 milik ID `3019`:

```text
http://crystal-peak.picoctf.net:64764/profile/user/a74c3bae3e13616104c1b25f9da1f11f
```

Halaman menampilkan:

```text
Welcome, admin! Here is the flag: picoCTF{id0r_unl0ck_c814750d}
```

## Flag

```text
picoCTF{id0r_unl0ck_c814750d}
```

## Pembelajaran

- Komentar HTML yang tertinggal pada source dapat membocorkan kredensial akun lain.
- Identifier pengguna yang hanya di-hash menggunakan MD5 tidak memberikan perlindungan
  otorisasi; hash bersifat deterministik dan mudah diproduksi untuk ID yang ditebak.
- Menggabungkan hint (jumlah karyawan) dengan pola ID memungkinkan enumerasi ID admin
  melalui serangan IDOR pada URL profil.
- Otorisasi harus diverifikasi di server-side untuk setiap permintaan profil, bukan
  mengandalkan kerahasiaan URL.

## Referensi

- [picoCTF](https://picoctf.org/)
- [CrackStation](https://crackstation.net/)
- [MD5 Hash Generator - Dan's Tools](https://www.md5hashgenerator.com/)
