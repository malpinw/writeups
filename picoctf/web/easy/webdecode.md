# WebDecode

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2024 |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Easy |
| Status | Solved |

## Deskripsi

Challenge ini menguji kemampuan untuk menggunakan web inspector dan memeriksa bagian-bagian
halaman web yang tidak terlihat langsung pada tampilan browser.

## Informasi Awal

Instance challenge dapat diakses melalui:

```text
http://titan.picoctf.net:64501
```

Halaman utama menampilkan navigasi menuju:

- `index.html`
- `about.html`
- `contact.html`

Hint yang tersedia:

- `Use the web inspector on other files included by the web page.`
- `The flag may or may not be encoded`

## Tools

- Firefox
- Firefox Developer Tools, panel `Inspector`
- Base64 Decode and Encode

## Analisis

Halaman `about.html` memberikan petunjuk untuk memeriksa halaman menggunakan inspector.
Pada source HTML halaman tersebut terdapat atribut custom bernama `notify_true` pada elemen
`section`:

```html
<section class="about" notify_true="cGljb0NURnt3ZWJfc3VjY2Nlc3NmdWxseV9kM2N0b2RlZF8xZjgzMjYxNX0=">
```

Nilai atribut tersebut tidak ditampilkan sebagai teks biasa pada halaman. Bentuknya juga
terlihat seperti Base64, sehingga nilai tersebut didekode untuk memperoleh flag.

## Langkah Penyelesaian

### 1. Membuka halaman utama

Instance dibuka melalui Firefox:

```text
http://titan.picoctf.net:64501
```

Halaman utama menampilkan pesan:

```text
Ha!!!!!! You looking for a flag?
Keep Navigating
```

Pesan tersebut mengarahkan untuk melanjutkan navigasi ke halaman lain.

### 2. Membuka halaman About

Halaman `about.html` dibuka melalui tautan `ABOUT` atau URL berikut:

```text
http://titan.picoctf.net:64501/about.html
```

Halaman menampilkan pesan:

```text
Try inspecting the page!! You might find it there
```

Pesan tersebut mengarahkan pemeriksaan source HTML menggunakan web inspector.

### 3. Memeriksa source HTML

Firefox Developer Tools dibuka pada panel `Inspector`. Di dalam elemen `section` halaman
About terlihat atribut berikut:

```html
notify_true="cGljb0NURnt3ZWJfc3VjY2Nlc3NmdWxseV9kM2N0b2RlZF8xZjgzMjYxNX0="
```

Nilai atribut tersebut disalin untuk didekode.

### 4. Mendekode nilai Base64

Nilai Base64 dimasukkan ke tool `Base64 Decode and Encode`:

```text
cGljb0NURnt3ZWJfc3VjY2Nlc3NmdWxseV9kM2N0b2RlZF8xZjgzMjYxNX0=
```

Hasil decoding adalah:

```text
picoCTF{web_succ3ssfully_d3c0ded_1f832615}
```

## Flag

```text
picoCTF{web_succ3ssfully_d3c0ded_1f832615}
```

## Pembelajaran

- Informasi penting dapat disembunyikan di atribut HTML custom tanpa terlihat pada halaman.
- Firefox Developer Tools membantu memeriksa struktur dan source HTML halaman web.
- Base64 merupakan encoding yang dapat dikembalikan ke teks asli tanpa key khusus.
- Navigasi ke halaman lain dan membaca hint dapat mengarahkan proses enumerasi manual pada
  aplikasi web.
