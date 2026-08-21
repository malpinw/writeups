# Unminify

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2024 |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Easy |
| Status | Solved |

## Deskripsi

Challenge ini menyediakan website yang source code-nya telah diminifikasi. Tujuannya adalah
memeriksa source halaman dan menemukan flag yang diterima oleh browser.

## Informasi Awal

Instance challenge dapat diakses melalui:

```text
http://titan.picoctf.net:52346
```

Deskripsi challenge menyebutkan bahwa source website telah dipadatkan agar tidak perlu
scrolling terlalu jauh dan halaman dapat dimuat lebih cepat.

Hint yang tersedia:

1. `Try CTRL+U / &#8984;+U in your browser to view the page source. You can also add "view-source:" before the URL, or try curl <URL> in your shell.`
2. `Minification reduces the size of code, but does not change its functionality.`
3. `What tools do developers use when working on a website? Many text editors and browsers include formatting.`

## Tools

- Firefox
- Firefox Developer Tools, panel `Inspector`

## Analisis

Halaman utama menampilkan pesan bahwa browser telah berhasil menerima flag, tetapi flag tidak
ditampilkan sebagai teks biasa pada tampilan halaman. Karena minification tidak mengubah
fungsi atau isi data halaman, source HTML tetap perlu diperiksa.

Pencarian string `picoCTF` pada source HTML menggunakan Firefox Developer Tools menemukan
flag pada elemen halaman. Flag tidak perlu dieksekusi atau didekode; nilainya sudah terlihat
sebagai teks biasa di source.

## Langkah Penyelesaian

### 1. Membuka instance challenge

Instance dibuka melalui Firefox pada alamat berikut:

```text
http://titan.picoctf.net:52346
```

Halaman menampilkan pesan:

```text
Welcome to my flag distribution website!
If you're reading this, your browser has successfully received the flag.
I just deliver flags, I don't know how to read them...
```

Pesan tersebut menunjukkan bahwa browser sudah menerima flag, sehingga langkah berikutnya
adalah memeriksa source halaman.

### 2. Membuka source HTML

Firefox Developer Tools dibuka pada panel `Inspector`. Sesuai hint, source juga dapat dibuka
dengan `Ctrl+U` atau menambahkan prefix `view-source:` pada URL.

### 3. Mencari flag pada source

Pada kolom pencarian Inspector, string berikut digunakan:

```text
picoCTF
```

Hasil pencarian memilih elemen yang memuat flag:

```text
picoCTF{r3tty_c0d3_ed938a7e}
```

## Flag

```text
picoCTF{r3tty_c0d3_ed938a7e}
```

## Pembelajaran

- Minification hanya mengurangi ukuran source dan tidak menghilangkan data yang ada di dalam
  halaman.
- Source HTML dapat memuat informasi yang tidak terlihat pada tampilan normal browser.
- Firefox Developer Tools membantu mencari string dan memeriksa elemen halaman.
- Hint challenge mengarahkan metode yang sesuai, yaitu membuka source dan menggunakan fitur
  formatting atau pencarian pada browser.
