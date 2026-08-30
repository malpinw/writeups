# Pachinko

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2026 (via CyLab Security Academy) |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Medium |
| Status | Solved (Flag 1) |

## Deskripsi

Challenge `Pachinko` menyajikan "NAND Simulator": sebuah kanvas interaktif untuk
membangun sirkuit logika dari gerbang NAND. Empat node input dan empat node output
ditampilkan; pemain menyambungkan gerbang NAND (maksimal 2 input per gerbang), lalu
men-submit sirkuit untuk diperiksa server. Goal yang ditampilkan: "Flip the outputs!"
— output harus kebalikan (inverse) dari input.

## Informasi Awal

- Instance diakses melalui browser dengan URL yang berubah setiap instance:

  ```text
  http://activist-birds.picoctf.net:54309/
  ```

- Frontend mengirim sirkuit sebagai JSON ke `POST /check`:

  ```json
  {"circuit": [{"input1": 5, "input2": 5, "output": 1}]}
  ```

- File yang diberikan bersama challenge (full source):
  - `index.js`: server Express — endpoint `/check` (validasi sirkuit, jalankan
    program checker di CPU WASM) dan `/flag` (admin, butuh flag1+flag2).
  - `cpu.js` + `wasm/`: emulator CPU (hasil kompilasi Verilog) yang menjalankan
    program dari memori, dengan sinyal `clock`, `reset`, `write_enable`, `halted`,
    `flag`.
  - `utils.js`: `serializeCircuit` menaruh program di alamat 0, output state di
    0x1000, input state di 0x2000, dan sirkuit di 0x3000 (masing-masing gerbang
    3 word: input1, input2, output).
  - `programs/nand_checker.bin`: program checker; `programs/flag.bin`: program
    untuk endpoint admin.
- Logika keputusan di `index.js`:

  ```js
  if (flag) { resp += FLAG2; }
  else if (result === 0x1337) { resp += FLAG1; }
  else if (result === 0x3333) { resp += "wrong answer :("; }
  else { resp += "unknown error code: " + result; }
  ```

## Tools

- Python 3 (urllib, stdlib saja)
- curl (recon)
- Browser

## Analisis

### Apa yang diminta checker?

Server meng-generate 4 nilai input acak, masing-masing 0x0000 atau 0xffff, lalu
menyusun output harapan sebagai **inverse**-nya (0xffff ↔ 0x0000). Sirkuit kita
dievaluasi oleh program `nand_checker.bin` di atas input acak tersebut; hasil akhir
program di memori 0x1000 harus 0x1337 agar FLAG1 dikeluarkan.

### Sirkuit yang benar

Output = inverse input, dan NAND dengan kedua input disambungkan ke sumber yang sama
bertindak sebagai NOT:

```text
NAND(x, x) = NOT(NAND hasil) = NOT x
```

Jadi cukup 4 gerbang NAND — satu per bit — dengan kedua inputnya berasal dari node
input yang sama dan outputnya menunjuk node output yang sesuai.

### Jebakan penomoran node

Kode frontend `resetGame()` mengungkap urutan pembuatan node:

```js
nextNodeId = 5;          // reset setelah output nodes
// input nodes dibuat dulu  -> dapat id 5, 6, 7, 8
// createOutputNodes()      -> output nodes pakai id 1, 2, 3, 4
```

Penomorannya **terbalik dari intuisi**: input = node 5–8, output = node 1–4.
Hipotesis pertama yang menganggap input = 1–4 gagal dengan respons
"wrong answer :(" (program checker selesai tapi hasil evaluasi salah — format
sirkuit sudah benar, pemetaan node-nya yang keliru).

## Langkah Penyelesaian

### 1. Recon source code

Membaca `index.js`, `cpu.js`, `utils.js`, dan frontend untuk memahami format
`POST /check`, mekanisme evaluasi, dan channel feedback ("unknown error code: N"
bila hasil bukan 0x1337/0x3333).

### 2. Script pengujian hipotesis penomoran

Karena penomoran node di sisi checker tidak diketahui, script menguji beberapa
hipotesis sekaligus — tiap hipotesis adalah 4 gerbang NOT (NAND(x,x)) dengan
pemetaan node berbeda:

```python
def not_gates(input_offset, output_ids):
    return [
        {"input1": input_offset + i, "input2": input_offset + i, "output": out}
        for i, out in enumerate(output_ids)
    ]

hypotheses = {
    "inputs 5-8 -> outputs 1-4 (self-NAND)": not_gates(5, [1, 2, 3, 4]),
    "inputs 1-4 -> outputs 5-8 (self-NAND)": not_gates(1, [5, 6, 7, 8]),
    ...
}
```

Respons server untuk tiap hipotesis menjadi umpan balik: flag, "wrong answer :(",
atau "unknown error code".

### 3. Eksekusi

```text
[*] inputs 5-8 -> outputs 1-4 (self-NAND) -> HTTP 200:
    {"status":"success","flag":"picoCTF{p4ch1nk0_f146_0n3_e947b9d7}\n"}
```

Hipotesis pertama terbukti benar: inputs 5–8, outputs 1–4.

### 4. Konfirmasi di browser

Sirkuit yang sama dapat dibangun manual di NAND Simulator: tambahkan 4 gerbang NAND,
sambungkan tiap input node (5–8) ke kedua input gerbangnya, lalu sambungkan output
gerbang ke node output (1–4) yang seposisi, dan submit.

## Flag

```text
picoCTF{p4ch1nk0_f146_0n3_e947b9d7}
```

## Pembelajaran

- Identitas node dalam protokol tidak selalu sesuai urutan visual/UI — selalu cek
  kode yang membuat elemen tersebut (di sini `resetGame()` dan `nextNodeId`)
  sebelum menebak pemetaan.
- Gerbang NAND secara teknis lengkap (functionally complete): NOT cukup dengan
  NAND(x,x); dari NOT dan NAND bisa dibangun AND, OR, XOR — konsep dasar yang
  membuat satu jenis gerbang cukup untuk challenge sirkuit apa pun.
- Respons error yang berbeda-beda ("wrong answer" vs "unknown error code: N")
  adalah channel umpan balik yang berguna untuk membedakan "format salah" dan
  "logika salah" tanpa perlu melihat source server.
- Untuk soal ber-hipotesis, script pengujian massal jauh lebih efisien daripada
  mencoba satu per satu manual — tiap respons server menyaring hipotesis berikutnya.

## Referensi

- [picoCTF](https://picoctf.org/)
- [CyLab Security Academy](https://learn.cylabacademy.org/)
- [NAND gate - Wikipedia](https://en.wikipedia.org/wiki/NAND_gate)
- [Functional completeness - Wikipedia](https://en.wikipedia.org/wiki/Functional_completeness)
