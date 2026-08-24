# Insp3ct0r

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2019 (via CyLab Security Academy) |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Easy |
| Status | Solved |

## Deskripsi

Challenge `Insp3ct0r` memberikan sebuah website sederhana dan petunjuk bahwa code pada
website perlu diperiksa. Tujuannya adalah menemukan tiga bagian flag yang disembunyikan
di dalam HTML, CSS, dan JavaScript, kemudian menggabungkannya sesuai urutan.

## Informasi Awal

- Instance dapat diakses melalui
  `http://fickle-tempest.picoctf.net:55285`.
- Halaman memiliki judul browser `My First Website :)` dan judul utama `Inspect Me`.
- Modal challenge memberikan deskripsi:

  ```text
  Kishor Balan tipped us off that the following code may need inspection:
  ```

- Hint yang tersedia:

  ```text
  How do you inspect web code on a browser?
  There's 3 parts
  ```

- Halaman menyediakan dua tab, yaitu `What` dan `How`. Tab `How` menyebutkan bahwa
  website dibuat menggunakan HTML, CSS, dan JS (JavaScript).

## Tools

- Firefox
- Firefox Developer Tools, panel `Inspector`
- Firefox Developer Tools, panel `Style Editor`
- Firefox Developer Tools, panel `Debugger`

## Analisis

Flag tidak ditampilkan pada teks utama halaman. Hint mengarahkan penggunaan web inspector,
sedangkan hint kedua menyatakan bahwa flag terdiri dari tiga bagian. Pemeriksaan terhadap
source HTML, stylesheet `mycss.css`, dan script `myjs.js` menemukan satu bagian flag pada
masing-masing file.

## Langkah Penyelesaian

### 1. Membuka instance challenge

Instance dibuka melalui Firefox. Halaman menampilkan `Inspect Me` dengan dua tab:
`What`, yang berisi teks `I made a website`, dan `How`, yang mencantumkan HTML, CSS,
serta JS (JavaScript).

### 2. Menemukan bagian pertama pada HTML

Firefox Developer Tools dibuka pada panel **Inspector**. Pada source HTML terdapat komentar
yang menyimpan bagian pertama flag:

```html
<!-- Html is neat. Anyways have 1/3 of the flag: picoCTF{tru3_d3 -->
```

Bagian pertama yang diperoleh adalah:

```text
picoCTF{tru3_d3
```

Source HTML juga memperlihatkan bahwa halaman memuat stylesheet `mycss.css` dan script
`myjs.js`, sehingga kedua file tersebut perlu diperiksa.

### 3. Menemukan bagian kedua pada CSS

Panel **Style Editor** digunakan untuk membuka `mycss.css`. Di dalam komentar CSS terdapat
bagian kedua flag:

```css
/* You need CSS to make pretty pages. Here's part 2/3 of the flag: t3ct1ve_0r_ju5t */
```

Bagian kedua yang diperoleh adalah:

```text
t3ct1ve_0r_ju5t
```

### 4. Menemukan bagian ketiga pada JavaScript

Panel **Debugger** kemudian digunakan untuk membuka `myjs.js`. Komentar pada file tersebut
menyimpan bagian terakhir:

```javascript
/* Javascript sure is neat. Anyways part 3/3 of the flag: _lucky?302945a7} */
```

Bagian ketiga yang diperoleh adalah:

```text
_lucky?302945a7}
```

### 5. Menggabungkan ketiga bagian

Ketiga bagian digabungkan sesuai urutan part 1, part 2, lalu part 3:

```text
picoCTF{tru3_d3 + t3ct1ve_0r_ju5t + _lucky?302945a7}
```

Hasil penggabungan tersebut membentuk flag lengkap.

## Flag

```text
picoCTF{tru3_d3t3ct1ve_0r_ju5t_lucky?302945a7}
```

## Pembelajaran

- Web inspector dapat digunakan untuk memeriksa source HTML dan file yang dimuat halaman.
- Informasi sensitif seperti flag dapat disembunyikan di dalam komentar HTML, CSS, atau
  JavaScript tanpa tampil pada halaman utama.
- Ketika hint menyebutkan beberapa bagian, seluruh file terkait perlu diperiksa dan bagian
  tersebut digabungkan sesuai urutan.

## Referensi

- [picoCTF](https://picoctf.org/)
