# Includes

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2022 |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Easy |
| Status | Solved |

## Deskripsi

Challenge `Includes` meminta pemain menemukan flag pada sebuah halaman web. Hint yang tersedia
menanyakan apakah terdapat code lain selain yang ditampilkan oleh inspector pada awalnya.

## Informasi Awal

Instance challenge dapat diakses melalui:

```text
http://saturn.picoctf.net:53733
```

Halaman menampilkan judul `On Includes`, penjelasan tentang directive `include`, dan tombol
`Say hello`. Pada challenge juga tersedia hint berikut:

```text
Is there more code than what the inspector initially shows?
```

## Tools

- Firefox
- Firefox Developer Tools, panel `Style Editor`
- Firefox Developer Tools, panel `Debugger`

## Analisis

Flag tidak seluruhnya berada pada HTML yang terlihat. Saat tombol `Say hello` ditekan, halaman
menampilkan alert:

```text
This code is in a separate file!
```

Pesan tersebut mengarahkan pemeriksaan ke file eksternal. Pada `style.css` ditemukan bagian
pertama flag di dalam komentar CSS, sedangkan pada `script.js` ditemukan bagian keduanya di
dalam komentar JavaScript.

## Langkah Penyelesaian

### 1. Membuka instance challenge

Instance dibuka melalui Firefox pada alamat berikut:

```text
http://saturn.picoctf.net:53733
```

Halaman menampilkan judul `On Includes`, artikel singkat tentang directive `include`, dan
tombol `Say hello`.

### 2. Memeriksa file JavaScript terpisah

Setelah tombol `Say hello` ditekan, muncul alert berikut:

```text
This code is in a separate file!
```

Pesan ini mengonfirmasi bahwa sebagian code berada pada file eksternal.

### 3. Menemukan bagian pertama flag pada `style.css`

Firefox Developer Tools dibuka pada panel `Style Editor`. File `style.css` menampilkan komentar
yang berisi bagian pertama flag:

```css
/* picoCTF{1nclu51v17y_1of2_ */
```

### 4. Menemukan bagian kedua flag pada `script.js`

Panel `Debugger` kemudian digunakan untuk membuka `script.js`. Selain fungsi `greetings()` yang
menampilkan alert, terdapat komentar berisi bagian kedua flag:

```javascript
// f7w_2of2_df589022}
```

Kedua bagian tersebut digabungkan sesuai urutannya.

## Flag

```text
picoCTF{1nclu51v17y_1of2_f7w_2of2_df589022}
```

## Pembelajaran

- Source code eksternal seperti file CSS dan JavaScript perlu diperiksa ketika hint menyebutkan
  adanya code tambahan.
- Komentar pada file yang dimuat browser tetap dapat dibaca melalui Developer Tools.
- Flag dapat disimpan secara terpisah pada beberapa file sehingga perlu menggabungkan seluruh
  fragmen yang ditemukan.
- Pesan dari fungsi JavaScript dapat menjadi petunjuk bahwa pemeriksaan tidak cukup dilakukan
  pada HTML yang terlihat.

## Referensi

- Wikipedia, `Include directive`, sebagaimana tercantum pada halaman challenge.
