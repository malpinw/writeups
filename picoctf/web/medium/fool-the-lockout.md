# Fool the Lockout

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2026 (via CyLab Security Academy) |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Medium |
| Status | Solved |

## Deskripsi

Challenge `Fool the Lockout` (by David Garcia, picoCTF 2026) menyajikan sebuah website
dengan halaman login yang dilindungi rate limit berbasis IP. Untuk mencegah brute force
dan credential stuffing, server membatasi jumlah percobaan login: melebihi ambang batas
membuat IP diblokir sementara. Deskripsi challenge menyatakan bahwa pembuatnya yakin
hal ini membuat menebak kredensial "mustahil", dan menantang untuk membuktikan
sebaliknya. Sebagai bahan pengujian tersedia:

- Sebuah dummy account dengan pasangan username-password acak dari daftar kredensial publik.
- File username dan password yang diberikan bersama challenge.
- Full source code aplikasi.

Tujuannya: bypass rate limit, login, dan ambil flag.

## Informasi Awal

- Instance diakses melalui browser dengan port yang berubah setiap instance
  (`62317`, `55399`, `54021` pada beberapa sesi):

  ```text
  http://candy-mountain.picoctf.net:54021/login
  ```

- Halaman login sederhana dengan form POST (`username`, `password`), judul
  "Silly Little Page". `/` redirect ke `/login` bila belum login.
- File yang diberikan bersama challenge:
  - `app.py`: full source code aplikasi Flask.
  - `creds-dump.txt`: 100 pasangan kredensial format `username;password`.
- Recon endpoint lain (`/register`, `/signup`, `/robots.txt`, `/console`) semuanya
  404/400 — tidak ada jalur alternatif selain form login.

## Tools

- Python 3 (urllib, stdlib saja)
- curl (recon)
- Browser

## Analisis

Parameter rate limit terlihat jelas di `app.py`:

```python
MAX_REQUESTS = 10      # max failed attempts before a user is locked out
EPOCH_DURATION = 30     # timeframe for failed attempts (in seconds)
LOCKOUT_DURATION = 120      # duration a user will be locked out for (in seconds)
```

Dua detail implementasi yang menjadi celah:

1. **Counter per epoch di-reset, tidak diakumulasi.** Fungsi `refresh_request_rates_db`
   mengembalikan `num_requests` ke 0 begitu `curr_time - epoch_start > 30`. Artinya
   defense ini hanya memperlambat serangan, tidak menghentikannya: cukup kirim
   maksimal 10 percobaan per jendela 30 detik lalu tunggu reset.
2. **Pengecekan memakai `>` bukan `>=`:**

   ```python
   if request_rates[client_ip]['num_requests'] > MAX_REQUESTS:
   ```

   Dengan `MAX_REQUESTS = 10`, yang memicu lockout adalah percobaan ke-11 — jadi
   tepat 10 POST gagal per epoch masih aman.

Matematis: 100 kredensial ÷ 10 per epoch = 10 epoch × ~31 detik ≈ 5 menit. Rate limiter
mengubah brute force instan menjadi serangan "pelan tapi pasti", bukan memblokirnya.

Jebakan teknis kedua ada di sisi klien: `urllib.request.urlopen` **mengikuti redirect
secara otomatis**. Login sukses di Flask menghasilkan `302 → /`, dan tanpa cookie
sesi halaman `/` me-redirect lagi ke `/login`, sehingga status akhir yang dilihat
script adalah 200 — identik dengan respons login gagal. Run pertama script karena itu
melaporkan "no success" padahal kredensial valid ada di daftar. Solusinya memakai
opener tanpa redirect handler agar HTTP 302 mentah terlihat sebagai tanda sukses.

## Langkah Penyelesaian

### 1. Recon dan verifikasi instance

```bash
curl -sS http://candy-mountain.picoctf.net:54021/login   # form login standar
curl -sS -o /dev/null -w "%{http_code}" http://candy-mountain.picoctf.net:54021/login  # 200 = instance hidup
```

### 2. Membuat script brute force dengan epoch pacing

Script `bruteforce.py` (stdlib saja, tanpa dependency):

- Membaca `creds-dump.txt`, memecah tiap baris pada `;`.
- Mengirim POST `/login` satu per satu; setiap 10 percobaan tidur 31 detik
  (`EPOCH_PAUSE > EPOCH_DURATION`) agar counter server ter-reset.
- Memakai `HTTPRedirectHandler` yang dinonaktifkan supaya status 302 dari login
  sukses tidak "ditelan" oleh redirect otomatis.
- Berhenti begitu menemukan `[+] SUCCESS`.

Bagian deteksi sukses yang menjadi kunci:

```python
class NoRedirect(urllib.request.HTTPRedirectHandler):
    def redirect_request(self, *args, **kwargs):
        return None

opener = urllib.request.build_opener(NoRedirect)
try:
    with opener.open(req, timeout=10) as resp:
        return resp.status          # 302 = login sukses
except urllib.error.HTTPError as e:
    return e.code
```

### 3. Menjalankan brute force

```bash
python bruteforce.py   # dari folder yang berisi creds-dump.txt
```

Output berjalan per epoch tanpa pernah menyentuh rate limit:

```text
[*] Loaded 100 credential pairs
[-] rora:winner1 -> failed (HTTP 200)
...
[*] Epoch 7 done, sleeping 31s for counter reset...
[+] SUCCESS: deane:shoe  (redirect to /, flag should be on homepage)
```

Kredensial valid adalah `deane:shoe` (baris 76 dari dump).

### 4. Login manual dan ambil flag

Login di browser sebagai `deane` / `shoe` pada instance yang sama. Homepage
menyambut dengan flag:

```text
Welcome deane
picoCTF{f00l_7h4t_l1m1t3r_b9fcf635}
```

## Flag

```text
picoCTF{f00l_7h4t_l1m1t3r_b9fcf635}
```

## Pembelajaran

- Rate limiting berbasis window yang **di-reset** (bukan penalty akumulatif) hanya
  memperlambat brute force: 10 request per 30 detik tetap membuka jalan penuh ke
  seluruh wordlist. Mitigasi yang lebih kuat: lockout progresif, captcha, atau
  pembatasan berbasis akun (bukan hanya IP).
- Detail operator penting: `>` vs `>=` menentukan berapa permintaan "aman" per epoch.
  Membaca source code selalu mengungkap margin seperti ini.
- `urllib` (dan banyak HTTP client lain) mengikuti redirect secara default — saat
  mendeteksi sukses lewat status code, nonaktifkan auto-redirect atau periksa
  `resp.url`/isi body. Bug deteksi semacam ini membuat run pertama salah menyimpulkan
  "tidak ada kredensial valid".
- Kesabaran adalah teknik: 100 kredensial ± 5 menit, tanpa satu pun lockout, asalkan
  pacing menghormati parameter window server.

## Referensi

- [picoCTF](https://picoctf.org/)
- [CyLab Security Academy](https://learn.cylabacademy.org/)
- [Python urllib.request documentation](https://docs.python.org/3/library/urllib.request.html)
