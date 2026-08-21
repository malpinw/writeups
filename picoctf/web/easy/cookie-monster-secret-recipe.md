# Cookie Monster Secret Recipe

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2025 |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Easy |
| Status | Solved |

## Deskripsi

Cookie Monster menyembunyikan resep cookie rahasianya di dalam website. Tujuan challenge
adalah menemukan secret tersebut melalui aplikasi web yang disediakan.

## Informasi Awal

Instance challenge dapat diakses melalui:

```text
http://verbal-sleep.picoctf.net:55755
```

Halaman awal menampilkan form login dengan field `Username`, `Password`, dan tombol `Login`.
Challenge menyediakan hint berikut:

1. `Sometimes, the most important information is hidden in plain sight. Have you checked all parts of the webpage?`
2. `Cookies aren't just for eating - they're also used in web technologies!`
3. `Web browsers often have tools that can help you inspect various aspects of a webpage, including things you can't see directly.`

## Tools

- Firefox
- Burp Suite Community Edition
- Base64 Decode and Encode

## Analisis

Login menggunakan username `administrator` dan password `admin123` tidak memberikan akses
ke halaman rahasia. Respons server justru menampilkan pesan bahwa password tidak diperlukan
dan cookie yang benar dibutuhkan:

```text
Access Denied
Cookie Monster says: 'Me no need password. Me just need cookies!'
Hint: Have you checked your cookies lately?
```

Dengan memeriksa HTTP response pada Burp Suite, terlihat bahwa server mengirim cookie bernama
`secret_recipe`. Nilai cookie tersebut merupakan Base64 yang bagian padding-nya ditulis dalam
bentuk URL encoding. Setelah didekode, nilainya menghasilkan flag.

## Langkah Penyelesaian

### 1. Membuka halaman login

Instance dibuka melalui Firefox pada alamat berikut:

```text
http://verbal-sleep.picoctf.net:55755
```

Halaman menampilkan judul `Cookie Monster's Secret Recipe` dan form login.

### 2. Mengirim kredensial login

Form diisi dengan username dan password berikut:

```text
Username: administrator
Password: admin123
```

Request yang terlihat pada Burp Suite adalah:

```http
POST /login.php HTTP/1.1
Host: verbal-sleep.picoctf.net:55755
Content-Type: application/x-www-form-urlencoded

username=administrator&password=admin123
```

### 3. Memeriksa response dan cookie

Server merespons dengan halaman `Access Denied`, tetapi response juga memiliki header
`Set-Cookie` berikut:

```http
Set-Cookie: secret_recipe=cGljb0NURntjMDBrMWVfbTBuc3Rlcl9sM3Zlc19jMDBraWVzX0IzQUQ5NEMyfQ%3D%3D
```

Pesan `Me just need cookies!` mengonfirmasi bahwa cookie tersebut merupakan artefak penting
untuk penyelesaian challenge.

### 4. Mendekode nilai cookie

Nilai cookie dimasukkan ke tool `Base64 Decode and Encode`. Setelah bagian URL encoding pada
padding diproses, nilai Base64-nya adalah:

```text
cGljb0NURntjMDBrMWVfbTBuc3Rlcl9sM3Zlc19jMDBraWVzX0IzQUQ5NEMyfQ==
```

Hasil decoding adalah:

```text
picoCTF{c00k1e_m0nster_l0ves_c00kies_B3AD94C2}
```

## Flag

```text
picoCTF{c00k1e_m0nster_l0ves_c00kies_B3AD94C2}
```

## Pembelajaran

- Cookie dapat menyimpan informasi penting yang tidak boleh dipercaya atau diekspos tanpa
  perlindungan yang sesuai.
- HTTP response header seperti `Set-Cookie` dapat diperiksa menggunakan Burp Suite.
- Base64 adalah encoding, bukan enkripsi, sehingga nilainya dapat dikembalikan ke bentuk asli.
- Pesan error dan hint pada halaman dapat mengarahkan pemeriksaan ke cookie browser.
