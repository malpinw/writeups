# dont-use-client-side

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2019 |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Easy |
| Status | Solved |

## Deskripsi

Challenge ini menyediakan halaman portal login sederhana bertuliskan `Secure Login Portal`. Tujuannya adalah membongkar logika verifikasi kredensial yang berjalan di sisi client (*client-side*) untuk mendapatkan password valid yang merupakan flag.

## Informasi Awal

Instance challenge dapat diakses melalui:

```text
http://fickle-tempest.picoctf.net:50002
```

Halaman web menampilkan sebuah form input kredensial dengan pesan:

```text
This is the secure login portal
Enter valid credentials to proceed
```

Petunjuk (*hint*) yang diberikan pada challenge:

1. `Never trust the client`

## Tools

- Browser (Chromium / Firefox)
- Burp Suite Community Edition (Proxy, HTTP history)
- Text Editor

## Analisis

Saat pengguna memasukkan password sembarang dan menekan tombol `verify`, browser langsung menampilkan dialog alert:

```text
Incorrect password
```

Tidak ada HTTP request baru yang dikirim ke server saat tombol ditekan. Hal ini mengindikasikan bahwa proses validasi password dilakukan secara lokal di browser (*client-side*).

Pemeriksaan pada source HTML / response HTTP menunjukkan adanya fungsi JavaScript bernama `verify()`. Fungsi tersebut mengambil nilai input dari elemen `#pass`, menetapkan variabel `split = 4`, lalu membandingkan potongan-potongan substring password secara bersarang (*nested if*). Karena logika perbandingan dan potongan karakter berada langsung di dalam script, nilai flag dapat direkonstruksi dengan mengurutkan substring berdasarkan indeksnya.

## Langkah Penyelesaian

### 1. Mengakses instance challenge

Instance dibuka melalui browser pada alamat berikut:

```text
http://fickle-tempest.picoctf.net:50002
```

Halaman menyajikan form login sederhana dengan field input dan tombol `verify`.

### 2. Menguji form login

Sebuah nilai acak dimasukkan ke dalam field input, kemudian tombol `verify` ditekan. Browser memunculkan alert:

```text
Incorrect password
```

### 3. Memeriksa source code JavaScript

Melalui Burp Suite `HTTP history` (atau inspect element browser), source HTML pada response `GET /` diperiksa. Ditemukan tag `<script>` yang memuat fungsi validasi berikut:

```javascript
<script type="text/javascript">
  function verify() {
    checkpass = document.getElementById("pass").value;
    split = 4;
    if (checkpass.substring(0, split) == 'pico') {
      if (checkpass.substring(split*6, split*7) == 'eb02') {
        if (checkpass.substring(split, split*2) == 'CTF{') {
         if (checkpass.substring(split*4, split*5) == 'ts_p') {
          if (checkpass.substring(split*3, split*4) == 'lien') {
            if (checkpass.substring(split*5, split*6) == 'lz_2') {
              if (checkpass.substring(split*2, split*3) == 'no_c') {
                if (checkpass.substring(split*7, split*8) == 'b45}') {
                  alert("Password Verified")
                  }
                }
              }
      
            }
          }
        }
      }
    }
    else {
      alert("Incorrect password");
    }
  }
</script>
```

### 4. Menyusun potongan string flag

Dengan nilai `split = 4`, setiap potongan substring dihitung indeks posisinya:

- `substring(0, split)` $\rightarrow$ indeks `0` s.d. `4`: `'pico'`
- `substring(split, split*2)` $\rightarrow$ indeks `4` s.d. `8`: `'CTF{'`
- `substring(split*2, split*3)` $\rightarrow$ indeks `8` s.d. `12`: `'no_c'`
- `substring(split*3, split*4)` $\rightarrow$ indeks `12` s.d. `16`: `'lien'`
- `substring(split*4, split*5)` $\rightarrow$ indeks `16` s.d. `20`: `'ts_p'`
- `substring(split*5, split*6)` $\rightarrow$ indeks `20` s.d. `24`: `'lz_2'`
- `substring(split*6, split*7)` $\rightarrow$ indeks `24` s.d. `28`: `'eb02'`
- `substring(split*7, split*8)` $\rightarrow$ indeks `28` s.d. `32`: `'b45}'`

Potongan string digabungkan secara berurutan:

```text
pico + CTF{ + no_c + lien + ts_p + lz_2 + eb02 + b45} = picoCTF{no_clients_plz_2eb02b45}
```

## Flag

```text
picoCTF{no_clients_plz_2eb02b45}
```

## Pembelajaran

- Logika validasi kredensial tidak boleh dijalankan di client-side karena seluruh kode JavaScript dapat dilihat dan dianalisis oleh pengguna.
- Membagi rahasia menjadi potongan-potongan substring dalam logika perbandingan tidak memberikan keamanan jika logika tersebut dapat diakses secara publik.
- Autentikasi dan pengecekan rahasia harus selalu dilakukan di sisi server (*server-side*).

## Referensi

- [MDN Web Docs - String.prototype.substring()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/substring)
- [OWASP Cheat Sheet - Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
