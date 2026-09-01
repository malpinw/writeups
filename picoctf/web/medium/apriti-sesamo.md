# Apriti Sesamo

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2025 |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Medium |
| Status | Solved |

## Deskripsi

Aplikasi web dengan halaman "impossible login": form username/password yang diklaim
tidak mungkin ditembus. Flag diperoleh dengan mengelabui logika perbandingan di sisi
server lewat manipulasi tipe data.

## Informasi Awal

Instance dapat diakses melalui:

```text
http://verbal-sleep.picoctf.net:59371/
```

Halaman depan hanya berisi tombol Login yang mengarah ke `impossibleLogin.php`.
Form melakukan POST dengan dua field:

```html
<form action="impossibleLogin.php" method="post">
    <input type="text" id="username" name="username">
    <input type="password" id="pwd" name="pwd">
</form>
```

Tidak ada source code yang diberikan challenge — analisis dilakukan lewat probing
perilaku server.

## Tools

- curl (probing dan eksploitasi; satu-dua request, tidak perlu Burp)

## Analisis

### Pemetaan perilaku server (black-box probing)

| Uji | Input | Respon |
| --- | --- | --- |
| sama persis | `admin` / `admin` | Failed! No flag for you |
| beda biasa | `admin` / `adminx` | Failed! No flag for you |
| dua-duanya kosong | `` / `` | Failed! No flag for you |
| angka berbeda | `1` / `01` | Failed! No flag for you |
| magic hash 0e | `aaroZmOk` / `aaK1STfY` | Failed! No flag for you |
| **array** | `username[]=a` / `pwd[]=b` | **Warning sha1() + flag** |

Dari pola ini terbaca logika server (tanpa melihat source):

1. Kedua nilai **tidak boleh sama** (`admin`/`admin` langsung gagal)
2. Kedua nilai lalu di-`sha1()` dan hasilnya **harus identik** (`===` ketat)

Sha1 magic hash (`0e...`) tidak lolos — berarti perbandingannya benar-benar ketat,
bukan `==` longgar. Jalur yang tersisa: membuat kedua perhitungan hash **gagal**.

### Vektor: PHP sha1() dan array

Fungsi `sha1()` di PHP hanya menerima string. Ketika dikirim array (nama field
`username[]` membuat PHP mengumpulkannya sebagai array), `sha1()` melempar warning
dan mengembalikan **NULL** — untuk kedua field sekaligus.

Akibatnya pemeriksaan ketat menjadi: `NULL === NULL` → **true**. Kedua syarat lolos:
array `a` dan `b` memang tidak sama (syarat 1), dan hash keduanya "identik" karena
sama-sama NULL (syarat 2). Server mencetak flag.

Pesan warning bahkan membocorkan lokasi file server:

```text
Warning: sha1() expects parameter 1 to be string, array given
in /var/www/html/impossibleLogin.php on line 38
```

## Langkah Penyelesaian

Satu perintah curl dari PowerShell (array trick via raw POST body):

```text
curl.exe -s -X POST "http://verbal-sleep.picoctf.net:59371/impossibleLogin.php" -d "username[]=a&pwd[]=b"
```

Output yang relevan:

```text
Warning: sha1() expects parameter 1 to be string, array given
in /var/www/html/impossibleLogin.php on line 38
Warning: sha1() expects parameter 1 to be string, array given
in /var/www/html/impossibleLogin.php on line 38
picoCTF{w3Ll_d3sErV3d_Ch4mp_471fee75}
```

## Flag

```text
picoCTF{w3Ll_d3sErV3d_Ch4mp_471fee75}
```

## Pembelajaran

- **Type juggling bukan cuma `==` vs `===`.** Bahkan dengan perbandingan ketat,
  memaksa kedua sisi ke nilai yang sama (NULL dari fungsi yang error) menghasilkan
  kesetaraan palsu.
- **Nama field `[]` mengubah tipe data.** PHP otomatis mengubah `username[]=a` menjadi
  array — cara paling murah untuk mengirim tipe yang tidak diduga oleh logika server.
- **Warning PHP adalah kebocoran informasi.** Pesan error mengungkap nama fungsi,
  tipe yang diharapkan, path file, dan nomor baris — empat petunjuk gratis untuk attacker.
- **Black-box probing berlapis bermakna.** Setiap baris tabel probing menguji satu
  hipotesis (identik? beda? kosong? hash longgar? tipe data lain?) sehingga logika
  server bisa dibaca tanpa source code.
- Perlindungan defensif: validasi tipe input di server, nonaktifkan display_errors
  di production, dan jangan mengandalkan `sha1()` untuk verifikasi.

## Catatan Proses

- Sempat diuji SHA-1 collision (arah "dua input berbeda, hash sama") melalui penalaran
  mandiri sebelum probing array — jalur ini tidak diperlukan karena perbandingan server
  ternyata ketat; yang berhasil justru membuat kedua hash gagal menjadi NULL.
- Challenge ini diselesaikan dengan pola mentoring baru: pemandu memberi arahan
  langkah-demi-langkah, pelaku menjalankan eksploitasi manual di mesinnya sendiri.

## Referensi

- Dokumentasi PHP `sha1()`: https://www.php.net/manual/en/function.sha1.php
  (mencakup perilaku non-string input)
