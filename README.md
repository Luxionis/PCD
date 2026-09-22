# Buku Ajar & Dokumentasi Komprehensif: Pengolahan Citra Digital (PCD) Native Python
## Pembahasan Tuntas: Teori, Formulasi Matematis, Logika Algoritma, dan Bedah Baris per Baris Kode

---

## 📖 Kata Pengantar & Petunjuk Pembacaan

Dokumentasi ini disusun khusus sebagai modul pembelajaran komprehensif mata kuliah **Pengolahan Citra Digital (PCD)**. Seluruh program di dalam repositori ini dibangun secara murni (**100% Native Python**) menggunakan pustaka standar internal bawaan sistem operasi dan Python (`struct`, `math`, `os`, `sys`), **sama sekali tanpa mengandalkan paket pihak ketiga seperti NumPy, OpenCV, PIL/Pillow, maupun Matplotlib**.

Dokumentasi ini membedah setiap file secara terpisah, menguraikan:
1. **Tujuan dan Teori Dasar:** Menjelaskan fenomena fisis dan matematis yang mendasari algoritma.
2. **Kamus Istilah & Variabel:** Menjelaskan nama-nama variabel teknis dalam bahasa yang mudah dicerna.
3. **Bedah Kode Baris Demi Baris:** Menjelaskan secara rinci apa yang dikerjakan oleh komputer pada setiap baris kode, mengapa baris tersebut ditulis, dan apa dampaknya terhadap citra.
4. **Alur Kerja & Uji Konsistensi:** Menjelaskan apa yang terjadi saat program dieksekusi di terminal.

---

## 📑 Daftar Isi

- [1. Konsep Dasar & Arsitektur Citra Native](#1-konsep-dasar--arsitektur-citra-native)
- [2. Modul 1: Pembacaan & Representasi Citra BMP (`modul1_representasi_citra.py`)](#2-modul-1-pembacaan--representasi-citra-bmp-modul1_representasi_citrapy)
- [3. Modul 2: Operasi Titik / Point Processing (`modul2_operasi_titik.py`)](#3-modul-2-operasi-titik--point-processing-modul2_operasi_titikpy)
- [4. Modul 3: Operasi Geometri Spasial (`modul3_operasi_geometri.py`)](#4-modul-3-operasi-geometri-spasial-modul3_operasi_geometripy)
- [5. Modul 4: Operasi Berbasis Bingkai / Frame Processing (`modul4_operasi_bingkai.py`)](#5-modul-4-operasi-berbasis-bingkai--frame-processing-modul4_operasi_bingkaipy)
- [6. Modul 5: Operasi Global & Analisis Histogram (`modul5_operasi_global.py`)](#6-modul-5-operasi-global--analisis-histogram-modul5_operasi_globalpy)
- [7. Modul Pendukung: Antarmuka Terminal & Inisialisasi Paket (`main.py` & `__init__.py`)](#7-modul-pendukung-antarmuka-terminal--inisialisasi-paket-mainpy--__init__py)
- [8. Panduan Menjalankan & Menguji Seluruh Program](#8-panduan-menjalankan--menguji-seluruh-program)

---

## 1. Konsep Dasar & Arsitektur Citra Native

### 1.1 Apa Itu Citra Digital?
Secara fisis, cahaya yang dipantulkan oleh benda ditangkap oleh sensor kamera dalam bentuk sinyal analog kontinu $f(x, y)$, di mana $(x, y)$ menyatakan posisi spasial pada bidang dua dimensi, dan $f$ adalah intensitas energi cahaya di titik tersebut.

Komputer tidak dapat menyimpan sinyal kontinu yang tak terhingga. Oleh karena itu, dilakukan dua proses digitalisasi:
1. **Sampling (Pencuplikan Spasial):** Membagi bidang gambar menjadi kisi-kisi kotak kecil berhingga berukuran $N$ baris vertikal dan $M$ kolom horizontal. Setiap kotak kecil disebut **piksel** (*picture element*).
2. **Kuantisasi (Digitalisasi Nilai):** Memetakan intensitas cahaya kontinu ke dalam himpunan bilangan bulat berhingga. Pada kedalaman 8-bit, intensitas dibagi menjadi $2^8 = 256$ tingkat derajat keabuan (dari 0 = hitam pekat hingga 255 = putih cemerlang).

### 1.2 Sistem Koordinat Matriks Citra Komputer
Dalam sistem koordinat Kartesius matematika murni, sumbu $y$ mengarah ke atas ($+y$ ke atas). Namun, **dalam ilmu komputer dan pemrosesan citra digital standar**, titik origin $(0, 0)$ selalu berada di **sudut kiri atas**, dan sumbu $y$ bertambah ke arah **bawah**:
```text
(0,0) -------------> Sumbu X (Kolom: 0 sampai Lebar - 1)
  |      Piksel (x, y)
  |      matriks[y][x]
  v
Sumbu Y (Baris: 0 sampai Tinggi - 1)
```
* **PENTING:** Saat mengakses data matriks array di Python `matriks[y][x]`:
  * Indeks pertama adalah **nomor baris** vertikal ($y$).
  * Indeks kedua adalah **nomor kolom** horizontal ($x$).

---

## 2. Modul 1: Pembacaan & Representasi Citra BMP (`modul1_representasi_citra.py`)

### 2.1 Teori & Anatomi Berkas Windows Bitmap (BMP)
Berkas BMP adalah format penyimpanan citra mentah biner tanpa kompresi *lossy*. Susunan byte dalam berkas BMP terdiri dari empat segmen utama:

```text
+-------------------------------------------------------------+
| 1. Bitmap File Header (14 byte)                             |
|    - Magic Number 'BM' (2 byte)                             |
|    - Ukuran File Total (4 byte, integer unsigned)           |
|    - Reserved 1 & 2 (4 byte, bernilai 0)                    |
|    - Offset Piksel (4 byte, lokasi awal memori piksel)      |
+-------------------------------------------------------------+
| 2. DIB Header / BITMAPINFOHEADER (40 byte)                  |
|    - Ukuran Header (4 byte, harus >= 40)                    |
|    - Lebar Gambar (4 byte, integer bertanda)                |
|    - Tinggi Gambar (4 byte, positif = Bottom-Up)            |
|    - Color Planes (2 byte, selalu bernilai 1)               |
|    - Bits Per Pixel / BPP (2 byte: 8 = Gray, 24 = RGB)      |
|    - Kompresi (4 byte: 0 = BI_RGB tanpa kompresi)          |
|    - Ukuran Citra Mentah & Resolusi PPM                     |
+-------------------------------------------------------------+
| 3. Color Palette / Palet Warna (Khusus citra 8-bit)        |
|    - 256 entri warna x 4 byte (Blue, Green, Red, Reserved)  |
|    - Total: 1024 byte tabel warna                           |
+-------------------------------------------------------------+
| 4. Array Piksel (Pixel Data)                                |
|    - 8-bit: 1 byte per piksel (indeks derajat keabuan)     |
|    - 24-bit: 3 byte per piksel dengan urutan B-G-R          |
|    - Aturan Padding 4-byte di ujung setiap baris            |
|    - Disimpan dari baris terbawah menuju ke atas            |
+-------------------------------------------------------------+
```

#### Aturan Padding 4-Byte BMP (Penting!)
Arsitektur bus memori mikroprosesor membaca data dalam blok kelipatan 4 byte (32-bit dword). Jika lebar baris data citra dalam satuan byte tidak habis dibagi 4, Windows mewajibkan penambahan byte kosong bernilai 0 di akhir setiap baris data sebagai bantalan (*padding*):
$$\text{Padding} = (4 - (\text{LebarBarisByte} \pmod 4)) \pmod 4$$
* Contoh pada 8-bit dengan lebar 120 piksel: $120 \pmod 4 = 0 \rightarrow \text{Padding} = 0$.
* Contoh pada 24-bit dengan lebar 653 piksel: Lebar byte $= 653 \times 3 = 1959$ byte. $1959 \pmod 4 = 3$. Maka $\text{Padding} = (4 - 3) \pmod 4 = 1$ byte padding tiap baris.

---

### 2.2 Bedah Kode `modul1_representasi_citra.py` Baris per Baris

#### Bagian 1: Inisialisasi Pustaka & Path
```python
13: import os
14: import struct
15: import sys
16: from typing import Any, List, Optional, Tuple, Union
17: 
18: # Memastikan direktori modul ini terdaftar di sys.path
19: DIR_INI = os.path.dirname(os.path.abspath(__file__))
20: if DIR_INI not in sys.path:
21:     sys.path.insert(0, DIR_INI)
22: 
23: TipePiksel = Union[int, Tuple[int, int, int]]
```
* **Baris 13 (`import os`):** Mengimpor modul interaksi sistem operasi untuk manipulasi direktori dan berkas fisik.
* **Baris 14 (`import struct`):** Modul krusial Python untuk membaca dan menulis variabel biner C (*raw bytes*). Format `<` berarti *Little-Endian*, `H` berarti integer 16-bit tak bertanda (*unsigned short*), `I` berarti integer 32-bit tak bertanda (*unsigned int*), dan `i` berarti integer 32-bit bertanda (*signed int*).
* **Baris 15 (`import sys`):** Mengimpor modul interpreter sistem Python.
* **Baris 16 (`from typing import ...`):** Menyediakan dokumentasi tipe variabel sehingga editor kode dapat memberikan petunjuk pengetikan dan mencegah salah tipe.
* **Baris 19-21:** Mengambil alamat absolut folder aktif (`DIR_INI`) dan memasukkannya ke urutan pertama (`index 0`) pada daftar pencarian library Python (`sys.path`). Ini menjamin skrip dapat menemukan modul kawannya di mana pun terminal dijalankan.
* **Baris 23:** Alias tipe: sebuah piksel didefinisikan berupa bilangan bulat tunggal `int` (untuk derajat keabuan 0..255) atau himpunan 3 bilangan bulat `(R, G, B)` untuk citra berwarna.

#### Bagian 2: Kelas `CitraMatriks`
```python
29: class CitraMatriks:
37:     def __init__(self, lebar: int, tinggi: int, mode: str = "GRAYSCALE", nilai_dasar: TipePiksel = 0):
46:         if lebar <= 0 or tinggi <= 0:
47:             raise ValueError("Lebar dan tinggi citra harus bernilai positif.")
48:         if mode not in ("GRAYSCALE", "RGB"):
49:             raise ValueError("Mode citra yang didukung hanya 'GRAYSCALE' dan 'RGB'.")
50: 
51:         self.lebar = lebar
52:         self.tinggi = tinggi
53:         self.mode = mode
54: 
55:         if mode == "RGB" and isinstance(nilai_dasar, int):
56:             nilai_dasar = (nilai_dasar, nilai_dasar, nilai_dasar)
57: 
58:         self.matriks: List[List[TipePiksel]] = [
59:             [nilai_dasar for _ in range(lebar)] for _ in range(tinggi)
60:         ]
```
* **Baris 29:** Mendeklarasikan kelas `CitraMatriks` sebagai wadah data citra dua dimensi.
* **Baris 37:** Konstruktor pembuat citra. Menerima parameter `lebar` (kolom $M$), `tinggi` (baris $N$), `mode`, dan `nilai_dasar` piksel awal.
* **Baris 46-49:** Pemeriksaan validitas ukuran. Dimensi $\le 0$ atau mode yang tidak dikenal akan memicu pesan kesalahan `ValueError`.
* **Baris 51-53:** Menyimpan dimensi dan mode ke dalam atribut internal objek.
* **Baris 55-56:** Jika pengguna membuat citra RGB tetapi hanya memberikan angka integer (misal 0), baris ini otomatis mengubahnya menjadi triplet `(0, 0, 0)` agar tipe data konsisten.
* **Baris 58-60:** Alokasi memori matriks 2D menggunakan teknik *list comprehension*. Menghasilkan daftar berukuran $N$ baris yang masing-masing memiliki $M$ kolom elemen bernilai `nilai_dasar`.

```python
63:     def get_pixel(self, x: int, y: int) -> TipePiksel:
65:         if not (0 <= x < self.lebar and 0 <= y < self.tinggi):
66:             raise IndexError(f"Koordinat ({x}, {y}) di luar batas citra {self.lebar}x{self.tinggi}.")
67:         return self.matriks[y][x]
```
* **Baris 63:** Fungsi pengambil nilai piksel pada koordinat kolom $x$ dan baris $y$.
* **Baris 65-66:** Pelindung indeks (*boundary check*). Jika koordinat bernilai negatif atau melampaui batas resolusi, fungsi melempar `IndexError`.
* **Baris 67:** Mengembalikan nilai elemen pada `matriks[y][x]`. Ingat: baris $y$ diakses terlebih dahulu.

```python
69:     def set_pixel(self, x: int, y: int, nilai: TipePiksel) -> None:
71:         if not (0 <= x < self.lebar and 0 <= y < self.tinggi):
72:             raise IndexError(f"Koordinat ({x}, {y}) di luar batas citra {self.lebar}x{self.tinggi}.")
73:         self.matriks[y][x] = nilai
```
* **Baris 69-73:** Fungsi pengubah nilai piksel pada titik koordinat $(x, y)$. Setelah memverifikasi koordinat, isi piksel langsung diperbarui dengan nilai baru.

```python
75:     def salin(self) -> "CitraMatriks":
77:         duplikat = CitraMatriks(self.lebar, self.tinggi, self.mode)
78:         for y in range(self.tinggi):
79:             for x in range(self.lebar):
80:                 duplikat.matriks[y][x] = self.matriks[y][x]
81:         return duplikat
```
* **Baris 75-81:** Melakukan penggandaan mendalam (*deep copy*). Membuat objek kanvas baru dan menyalin seluruh isi piksel satu per satu. Dengan cara ini, pengeditan pada citra hasil salinan tidak akan merusak citra aslinya di memori.

```python
83:     def info(self) -> str:
85:         return f"Citra [{self.mode}] - Resolusi: {self.lebar}x{self.tinggi} piksel (N={self.tinggi}, M={self.lebar})"
```
* **Baris 83-85:** Mengembalikan teks rangkuman spesifikasi teknis citra.

#### Bagian 3: Fungsi Pembaca Berkas Biner BMP (`muat_bmp`)
```python
92: def muat_bmp(path_berkas: str) -> Tuple[CitraMatriks, dict]:
102:     if not os.path.exists(path_berkas):
103:         raise FileNotFoundError(f"Berkas tidak ditemukan: {path_berkas}")
105:     with open(path_berkas, "rb") as f:
```
* **Baris 92:** Mendeklarasikan fungsi parser biner yang mengembalikan objek `CitraMatriks` dan dictionary `metadata`.
* **Baris 102-103:** Memastikan file gambar benar-benar ada di media penyimpanan fisik.
* **Baris 105:** Membuka file dalam mode `"rb"` (*read binary*). Statement `with` menjamin file otomatis tertutup secara aman setelah selesai dibaca.

```python
108:         magic_bytes = f.read(2)
109:         if magic_bytes != b"BM":
110:             raise ValueError(f"Bukan berkas BMP valid. Tanda magic: {magic_bytes!r}")
112:         ukuran_berkas, _, _, offset_data = struct.unpack("<IHHI", f.read(12))
```
* **Baris 108-110:** Membaca 2 byte pertama. Standar BMP mewajibkan byte awal bernilai `b'BM'`. Jika tidak sesuai, pembacaan dihentikan.
* **Baris 112:** Membaca 12 byte sisa dari File Header (total 14 byte). Format `<IHHI` mengurai:
  * `ukuran_berkas`: Ukuran file dalam satuan byte.
  * `offset_data`: Alamat memori byte tempat data piksel dimulai.

```python
115:         ukuran_dib = struct.unpack("<I", f.read(4))[0]
116:         if ukuran_dib < 40:
117:             raise ValueError(f"Ukuran DIB Header ({ukuran_dib} byte) tidak didukung.")
119:         lebar_mentah, tinggi_mentah, planes, bpp, kompresi, ukuran_gambar, x_ppm, y_ppm, warna_digunakan, warna_penting = struct.unpack(
120:             "<iiHHIIiiII", f.read(36)
121:         )
124:         if ukuran_dib > 40:
125:             f.read(ukuran_dib - 40)
```
* **Baris 115-121:** Membaca DIB Header. Membaca ukuran header (standar 40 byte) lalu mengurai 36 byte berikutnya menggunakan pola `<iiHHIIiiII`. Di sini didapatkan `lebar_mentah`, `tinggi_mentah`, kedalaman bit `bpp`, dan tipe `kompresi`.
* **Baris 124-125:** Kompatibilitas untuk versi DIB modern (V4 atau V5 yang berukuran $> 40$ byte): byte ekstra dilewati dengan aman.

```python
128:         if kompresi != 0:
129:             raise ValueError(f"Kompresi BMP tipe {kompresi} belum didukung. Hanya uncompressed BMP (BI_RGB).")
130:         if bpp not in (8, 24):
131:             raise ValueError(f"Kedalaman bit {bpp}-bpp belum didukung. Hanya 8-bit dan 24-bit.")
135:         arah_terbalik = tinggi_mentah > 0
136:         tinggi = abs(tinggi_mentah)
137:         lebar = lebar_mentah
```
* **Baris 128-131:** Validasi integritas: memastikan berkas adalah BMP murni tanpa kompresi (`kompresi == 0`) dan hanya menerima 8-bit atau 24-bit.
* **Baris 135-137:** Jika `tinggi_mentah > 0`, citra disimpan secara *bottom-up* (`arah_terbalik = True`). Dimensi tinggi diambil nilai absolut positifnya `abs(tinggi_mentah)`.

```python
150:         palet: List[Tuple[int, int, int]] = []
151:         if bpp == 8:
152:             jumlah_warna = warna_digunakan if warna_digunakan > 0 else 256
153:             for _ in range(jumlah_warna):
154:                 b, g, r, _reserved = struct.unpack("4B", f.read(4))
155:                 palet.append((r, g, b))
```
* **Baris 150-155:** Membaca tabel palet warna jika citra bertipe 8-bit. Setiap entri warna terdiri dari 4 byte: Blue, Green, Red, dan Reserved (`4B`). Komponen ini disimpan ke list `palet` dalam format Python `(r, g, b)`.

```python
158:         f.seek(offset_data)
161:         mode = "RGB" if bpp == 24 else "GRAYSCALE"
162:         citra = CitraMatriks(lebar, tinggi, mode)
```
* **Baris 158:** Memindahkan kursor baca file langsung ke posisi byte data piksel pertama (`offset_data`).
* **Baris 161-162:** Mengalokasikan objek `CitraMatriks` kosong sesuai resolusi dan mode yang terdeteksi.

```python
164:         if bpp == 8:
166:             padding = (4 - (lebar % 4)) % 4
167:             baris_terbaca = []
168:             for _ in range(tinggi):
169:                 indeks_baris = list(f.read(lebar))
170:                 if padding > 0:
171:                     f.read(padding)
174:                 if palet:
175:                     baris_piksel = [palet[idx][0] for idx in indeks_baris]
176:                 else:
177:                     baris_piksel = indeks_baris
178:                 baris_terbaca.append(baris_piksel)
180:             if arah_terbalik:
181:                 baris_terbaca.reverse()
183:             citra.matriks = baris_terbaca
```
* **Baris 164-183 (Pembacaan 8-bit):**
  * **Baris 166:** Menghitung jumlah byte padding penyeimbang kelipatan 4.
  * **Baris 168-171:** Membaca data piksel sebanyak `lebar` byte, lalu membaca dan membuang byte padding kosong jika ada.
  * **Baris 174-177:** Mengambil nilai derajat keabuan dari tabel palet berdasarkan indeks yang dibaca.
  * **Baris 180-181:** Jika citra berorientasi *bottom-up*, urutan baris dibalik dengan `.reverse()` agar baris teratas citra berada di indeks $y = 0$.

```python
185:         elif bpp == 24:
186:             padding = (4 - ((lebar * 3) % 4)) % 4
187:             baris_terbaca = []
188:             for _ in range(tinggi):
189:                 baris_piksel: List[TipePiksel] = []
190:                 data_baris = f.read(lebar * 3)
191:                 for i in range(0, len(data_baris), 3):
192:                     b = data_baris[i]
193:                     g = data_baris[i + 1]
194:                     r = data_baris[i + 2]
195:                     baris_piksel.append((r, g, b))
196:                 if padding > 0:
197:                     f.read(padding)
198:                 baris_terbaca.append(baris_piksel)
200:             if arah_terbalik:
201:                 baris_terbaca.reverse()
203:             citra.matriks = baris_terbaca
```
* **Baris 185-203 (Pembacaan 24-bit RGB):**
  * Setiap baris terdiri dari $M \times 3$ byte data ditambah padding.
  * Format penyimpanan BMP adalah **BGR**. Kode membaca 3 byte berturutan (`b, g, r`) lalu menyusunnya menjadi triplet standar RGB `(r, g, b)`.
  * Membalik susunan baris jika file berformat *bottom-up*.
* **Baris 205:** Mengembalikan objek citra dan dictionary metadata.

#### Bagian 4: Fungsi Penulis Berkas Biner BMP (`simpan_bmp`)
```python
208: def simpan_bmp(citra: CitraMatriks, path_berkas: str) -> None:
218:     folder_induk = os.path.dirname(path_berkas)
219:     if folder_induk and not os.path.exists(folder_induk):
220:         os.makedirs(folder_induk, exist_ok=True)
222:     with open(path_berkas, "wb") as f:
```
* **Baris 208-222:** Membuka file target dalam mode tulis biner `"wb"` (*write binary*). Otomatis menciptakan folder penampung jika belum tersedia.

```python
223:         if citra.mode == "GRAYSCALE":
224:             bpp = 8
225:             padding = (4 - (lebar % 4)) % 4
226:             ukuran_data_piksel = (lebar + padding) * tinggi
227:             offset_data = 14 + 40 + (256 * 4)  # FileHeader + DIBHeader + Palet 256 warna
228:             ukuran_berkas = offset_data + ukuran_data_piksel
231:             f.write(b"BM")
232:             f.write(struct.pack("<IHHI", ukuran_berkas, 0, 0, offset_data))
235:             f.write(struct.pack(
236:                 "<IiiHHIIiiII",
237:                 40, lebar, tinggi, 1, bpp, 0, ukuran_data_piksel, 2835, 2835, 256, 0
238:             ))
241:             for val in range(256):
242:                 f.write(struct.pack("4B", val, val, val, 0))
245:             pad_bytes = b"\x00" * padding
246:             for y in range(tinggi - 1, -1, -1):
247:                 baris_bytes = bytearray(citra.matriks[y])
248:                 f.write(baris_bytes)
249:                 if padding > 0:
250:                     f.write(pad_bytes)
```
* **Baris 223-250 (Penyimpanan Citra Grayscale 8-bit):**
  * Menghitung total ukuran file termasuk palet warna 256 tingkatan (1024 byte).
  * Menulis 14 byte File Header dan 40 byte DIB Header.
  * Menulis tabel palet abu-abu linier ($B=G=R=val$) untuk nilai 0 s.d. 255.
  * Menulis data baris secara *bottom-up* (dari $y = N-1$ turun ke $0$) disertai byte padding kosong `b"\x00"`.

```python
252:         elif citra.mode == "RGB":
253:             bpp = 24
254:             padding = (4 - ((lebar * 3) % 4)) % 4
255:             ukuran_data_piksel = (lebar * 3 + padding) * tinggi
256:             offset_data = 14 + 40  # FileHeader + DIBHeader
257:             ukuran_berkas = offset_data + ukuran_data_piksel
260:             f.write(b"BM")
261:             f.write(struct.pack("<IHHI", ukuran_berkas, 0, 0, offset_data))
264:             f.write(struct.pack(
265:                 "<IiiHHIIiiII",
266:                 40, lebar, tinggi, 1, bpp, 0, ukuran_data_piksel, 2835, 2835, 0, 0
267:             ))
270:             pad_bytes = b"\x00" * padding
271:             for y in range(tinggi - 1, -1, -1):
272:                 baris_bytes = bytearray()
273:                 for x in range(lebar):
274:                     p = citra.matriks[y][x]
275:                     r, g, b = p[0], p[1], p[2]
276:                     baris_bytes.extend([b, g, r])  # Format BGR
277:                 f.write(baris_bytes)
278:                 if padding > 0:
279:                     f.write(pad_bytes)
```
* **Baris 252-280 (Penyimpanan Citra True Color 24-bit):**
  * Menulis File Header dan DIB Header tanpa palet warna.
  * Menyusun kembali piksel `(r, g, b)` ke dalam urutan biner **BGR** (`[b, g, r]`).
  * Menulis baris dari bawah ke atas disertai byte padding.

#### Bagian 5: Generator Sampel Sintetis & Uji Mandiri
* **Baris 286-298 (`buat_citra_sampel_gradien`):** Menghasilkan gradien intensitas horizontal linier `int((x / lebar) * 255)` dengan kotak pembalik kontras di tengahnya.
* **Baris 300-320 (`buat_citra_sampel_warna`):** Membagi kanvas menjadi 4 pita warna vertikal: merah, hijau, biru, dan kuning.
* **Baris 326-363 (`if __name__ == "__main__":`):** Menguji fungsionalitas modul secara otomatis saat dijalankan. Menghasilkan file `sampel_grayscale.bmp` dan `sampel_warna.bmp` di folder `hasil/`, lalu menguji kesesuaian piksel menggunakan pernyataan `assert`.

---

## 3. Bedah Kode Modul 2: `modul2_operasi_titik.py`

### 3.1 Teori Operasi Titik (Point Processing)
Operasi titik memetakan nilai intensitas setiap piksel input $K_i$ pada koordinat $(x, y)$ ke nilai intensitas baru $K_o$ tanpa terpengaruh oleh piksel tetangganya:
$$K_o = T(K_i)$$

```text
Piksel Masukan Ki(x, y) --------> [ Fungsi Transformasi T ] --------> Piksel Keluaran Ko(x, y)
```

---

### 3.2 Bedah Kode `modul2_operasi_titik.py` Baris per Baris

#### Bagian 1: Fungsi Penjepit Intensitas (`jepit`)
```python
42: def jepit(nilai: Union[int, float], batas_bawah: int = 0, batas_atas: int = 255) -> int:
44:     return max(batas_bawah, min(batas_atas, round(nilai)))
```
* **Baris 42-44:** Membatasi nilai matematis agar tidak melebihi rentang valid 8-bit $[0, 255]$. Jika hasil perhitungan menghasilkan angka negatif (misal $-20$), fungsi mengubahnya menjadi $0$. Jika melebihi $255$ (misal $290$), fungsi memotongnya menjadi $255$. Nilai pecahan dibulatkan dengan `round()`.

#### Bagian 2: Modifikasi Kecerahan (`ubah_kecerahan`)
```python
51: def ubah_kecerahan(citra: CitraMatriks, nilai_offset: int) -> CitraMatriks:
64:     hasil = CitraMatriks(citra.lebar, citra.tinggi, citra.mode)
65:     for y in range(citra.tinggi):
66:         for x in range(citra.lebar):
67:             p = citra.get_pixel(x, y)
68:             if citra.mode == "GRAYSCALE":
69:                 hasil.set_pixel(x, y, jepit(p + nilai_offset))
70:             else:
71:                 r, g, b = p
72:                 hasil.set_pixel(x, y, (
73:                     jepit(r + nilai_offset),
74:                     jepit(g + nilai_offset),
75:                     jepit(b + nilai_offset)
76:                 ))
77:     return hasil
```
* **Rumus Matematis:** $K_o = \text{clamp}(K_i + C)$
* **Baris 51:** Fungsi menerima citra sumber dan nilai pergeseran `nilai_offset` ($C$). Jika $C > 0$, citra menjadi terang. Jika $C < 0$, citra menjadi gelap.
* **Baris 64:** Membuat kanvas hasil dengan dimensi dan mode yang identik.
* **Baris 65-67:** Melakukan perulangan menyusuri setiap baris $y$ dan setiap kolom $x$, lalu mengambil nilai piksel aslinya `p`.
* **Baris 68-76:** Menambahkan `nilai_offset` ke intensitas piksel. Pada citra warna, penambahan dilakukan secara serentak ke saluran R, G, dan B. Seluruh hasil dijepit dengan `jepit()`.

#### Bagian 3: Peningkatan Kontras (`tingkatkan_kontras`)
```python
84: def tingkatkan_kontras(citra: CitraMatriks, faktor_gain: float, titik_poros: int = 128) -> CitraMatriks:
98:     hasil = CitraMatriks(citra.lebar, citra.tinggi, citra.mode)
99:     for y in range(citra.tinggi):
100:         for x in range(citra.lebar):
101:             p = citra.get_pixel(x, y)
102:             if citra.mode == "GRAYSCALE":
103:                 baru = jepit(faktor_gain * (p - titik_poros) + titik_poros)
104:                 hasil.set_pixel(x, y, baru)
105:             else:
106:                 r, g, b = p
107:                 hasil.set_pixel(x, y, (
108:                     jepit(faktor_gain * (r - titik_poros) + titik_poros),
109:                     jepit(faktor_gain * (g - titik_poros) + titik_poros),
110:                     jepit(faktor_gain * (b - titik_poros) + titik_poros)
111:                 ))
112:     return hasil
```
* **Rumus Matematis:** $K_o = \text{clamp}(G \times (K_i - P) + P)$
* **Baris 84:** Menerima `faktor_gain` ($G$) sebagai pengali kontras dan `titik_poros` ($P$, default 128) sebagai titik tumpu peregangan kontras.
* **Baris 103 & 108-110:** Nilai piksel dikurangi titik poros 128, dikalikan dengan faktor pengali $G$, lalu ditambahkan kembali dengan 128. Piksel yang lebih terang dari 128 ditarik semakin terang mendekati 255, sedangkan piksel yang lebih gelap dari 128 ditarik semakin gelap mendekati 0.

#### Bagian 4: Negasi / Inversi Citra (`negasi_citra`)
```python
119: def negasi_citra(citra: CitraMatriks) -> CitraMatriks:
131:     hasil = CitraMatriks(citra.lebar, citra.tinggi, citra.mode)
132:     for y in range(citra.tinggi):
133:         for x in range(citra.lebar):
134:             p = citra.get_pixel(x, y)
135:             if citra.mode == "GRAYSCALE":
136:                 hasil.set_pixel(x, y, 255 - p)
137:             else:
138:                 r, g, b = p
139:                 hasil.set_pixel(x, y, (255 - r, 255 - g, 255 - b))
140:     return hasil
```
* **Rumus Matematis:** $K_o = 255 - K_i$
* **Baris 119-140:** Membalik skala warna. Nilai hitam ($0$) berubah menjadi putih ($255$), nilai abu-abu terang ($200$) berubah menjadi abu-abu gelap ($55$). Pada citra berwarna, inversi dilakukan independen pada komponen R, G, dan B.

#### Bagian 5: Konversi Ruang Warna ke Grayscale (`konversi_keabuan`)
```python
147: def konversi_keabuan(citra: CitraMatriks, metode: str = "luminance") -> CitraMatriks:
161:     if citra.mode == "GRAYSCALE":
162:         return citra.salin()
164:     hasil = CitraMatriks(citra.lebar, citra.tinggi, "GRAYSCALE")
165:     for y in range(citra.tinggi):
166:         for x in range(citra.lebar):
167:             r, g, b = citra.get_pixel(x, y)
168:             if metode == "luminance":
169:                 abu = jepit(0.299 * r + 0.587 * g + 0.114 * b)
170:             elif metode == "average":
171:                 abu = jepit((r + g + b) / 3.0)
172:             else:
173:                 raise ValueError(f"Metode konversi '{metode}' tidak dikenal. Pilih 'luminance' atau 'average'.")
174:             hasil.set_pixel(x, y, abu)
175:     return hasil
```
* **Baris 161-162:** Optimasi: jika citra masukan sudah dalam mode `GRAYSCALE`, fungsi langsung mengembalikan duplikatnya tanpa komputasi ulang.
* **Baris 168-169 (Metode Luminance ITU-R BT.601):**
  $$Y = \text{round}(0.299 \cdot R + 0.587 \cdot G + 0.114 \cdot B)$$
  Mata manusia paling sensitif terhadap cahaya hijau ($58.7\%$), kemudian merah ($29.9\%$), dan paling kurang sensitif terhadap biru ($11.4\%$). Pembobotan ini menghasilkan derajat keabuan yang paling natural dan nyaman bagi mata manusia.
* **Baris 170-171 (Metode Average):** Rata-rata aritmatika sederhana dari penjumlahan ketiga komponen warna dibagi 3.

#### Bagian 6: Pengambangan Biner / Thresholding
```python
182: def pengambangan_tunggal(citra: CitraMatriks, ambang: int = 128) -> CitraMatriks:
195:     citra_gray = citra if citra.mode == "GRAYSCALE" else konversi_keabuan(citra)
196:     hasil = CitraMatriks(citra_gray.lebar, citra_gray.tinggi, "GRAYSCALE")
198:     for y in range(citra_gray.tinggi):
199:         for x in range(citra_gray.lebar):
200:             intensitas = citra_gray.get_pixel(x, y)
201:             biner = 255 if intensitas >= ambang else 0
202:             hasil.set_pixel(x, y, biner)
203:     return hasil
```
* **Rumus Thresholding Tunggal:**
  $$K_o = \begin{cases} 255, & \text{jika } K_i \ge \text{Ambang} \\ 0, & \text{jika } K_i < \text{Ambang} \end{cases}$$
* **Baris 195:** Otomatis mengubah citra berwarna menjadi grayscale terlebih dahulu sebelum dilakukan segmentasi.
* **Baris 201:** Membandingkan nilai intensitas dengan ambang batas (default 128). Piksel diubah menjadi putih murni ($255$) jika memenuhi ambang, atau hitam murni ($0$) jika di bawah ambang.

```python
206: def pengambangan_ganda(citra: CitraMatriks, ambang_bawah: int = 85, ambang_atas: int = 170) -> CitraMatriks:
220:     citra_gray = citra if citra.mode == "GRAYSCALE" else konversi_keabuan(citra)
221:     hasil = CitraMatriks(citra_gray.lebar, citra_gray.tinggi, "GRAYSCALE")
223:     for y in range(citra_gray.tinggi):
224:         for x in range(citra_gray.lebar):
225:             intensitas = citra_gray.get_pixel(x, y)
226:             biner = 255 if (ambang_bawah <= intensitas <= ambang_atas) else 0
227:             hasil.set_pixel(x, y, biner)
228:     return hasil
```
* **Rumus Thresholding Ganda (Band Thresholding):**
  $$K_o = \begin{cases} 255, & \text{jika } \text{Ambang}_{\text{bawah}} \le K_i \le \text{Ambang}_{\text{atas}} \\ 0, & \text{lainnya} \end{cases}$$
* **Baris 226:** Memilih piksel yang berada dalam interval intensitas tertentu. Sangat berguna untuk mengisolasi objek spesifik pada citra rontgen atau citra satelit.

---

## 4. Modul 3: Operasi Geometri Spasial (`modul3_operasi_geometri.py`)

### 4.1 Teori Transformasi Geometri & Masalah Forward Mapping
Transformasi geometri memindahkan posisi spasial koordinat asal $(x, y)$ ke koordinat tujuan $(x', y')$.
* **Masalah Pemetaan Maju (Forward Mapping):** Jika kita menghitung posisi baru dari posisi lama ($x \rightarrow x'$), pembulatan angka desimal hasil trigonometri akan meninggalkan celah kosong pada citra tujuan yang memunculkan bintik-bintik hitam berlubang (*aliasing/holes*).
* **Solusi Pemetaan Balik (Inverse Mapping):** Modul ini menggunakan metode *Inverse Mapping*. Algoritma menyusuri setiap koordinat pada kanvas hasil $(x', y')$, lalu menghitung mundur koordinat asalnya $(x, y)$ pada citra sumber. Hal ini menjamin setiap piksel kanvas tujuan terisi penuh tanpa ada lubang.

---

### 4.2 Bedah Kode `modul3_operasi_geometri.py` Baris per Baris

#### Bagian 1: Pencerminan (Flipping)
```python
48: def cermin_horizontal(citra: CitraMatriks) -> CitraMatriks:
55:     hasil = CitraMatriks(citra.lebar, citra.tinggi, citra.mode)
56:     w = citra.lebar
57:     for y in range(citra.tinggi):
58:         for x in range(w):
59:             hasil.set_pixel(w - 1 - x, y, citra.get_pixel(x, y))
60:     return hasil
```
* **Rumus:** $x' = M - 1 - x, \quad y' = y$
* **Baris 48-60:** Kolom paling kiri ($x=0$) dipindahkan ke kolom paling kanan ($x'=M-1$). Citra terbalik secara horizontal (kiri ke kanan).

```python
63: def cermin_vertikal(citra: CitraMatriks) -> CitraMatriks:
70:     hasil = CitraMatriks(citra.lebar, citra.tinggi, citra.mode)
71:     h = citra.tinggi
72:     for y in range(h):
73:         for x in range(citra.lebar):
74:             hasil.set_pixel(x, h - 1 - y, citra.get_pixel(x, y))
75:     return hasil
```
* **Rumus:** $x' = x, \quad y' = N - 1 - y$
* **Baris 63-75:** Baris paling atas ($y=0$) dipindahkan ke baris paling bawah ($y'=N-1$). Citra terbalik secara vertikal (atas ke bawah).

```python
78: def cermin_kombinasi(citra: CitraMatriks) -> CitraMatriks:
85:     hasil = CitraMatriks(citra.lebar, citra.tinggi, citra.mode)
86:     w, h = citra.lebar, citra.tinggi
87:     for y in range(h):
88:         for x in range(w):
89:             hasil.set_pixel(w - 1 - x, h - 1 - y, citra.get_pixel(x, y))
90:     return hasil
```
* **Rumus:** $x' = M - 1 - x, \quad y' = N - 1 - y$
* **Baris 78-90:** Membalik kedua sumbu secara bersamaan. Secara geometri, hasilnya identik dengan rotasi 180 derajat.

#### Bagian 2: Rotasi Citra (Rotation)
```python
97: def rotasi_90(citra: CitraMatriks, searah_jarum_jam: bool = True) -> CitraMatriks:
106:     w_lama, h_lama = citra.lebar, citra.tinggi
107:     w_baru, h_baru = h_lama, w_lama
108:     hasil = CitraMatriks(w_baru, h_baru, citra.mode)
110:     for y in range(h_lama):
111:         for x in range(w_lama):
112:             p = citra.get_pixel(x, y)
113:             if searah_jarum_jam:
114:                 nx = h_lama - 1 - y
115:                 ny = x
116:             else:
117:                 nx = y
118:                 ny = w_lama - 1 - x
119:             hasil.set_pixel(nx, ny, p)
120:     return hasil
```
* **Baris 97-120:** Rotasi 90 derajat menukar dimensi kanvas: lebar baru menjadi tinggi lama, dan tinggi baru menjadi lebar lama (`w_baru, h_baru = h_lama, w_lama`).
* **Baris 113-115 (Clockwise):** $nx = H_{\text{lama}} - 1 - y$ dan $ny = x$.
* **Baris 116-118 (Counter-Clockwise):** $nx = y$ dan $ny = W_{\text{lama}} - 1 - x$.

```python
131: def rotasi_bebas(
132:     citra: CitraMatriks,
133:     sudut_derajat: float,
134:     latar_belakang: Optional[TipePiksel] = None
135: ) -> CitraMatriks:
150:     rad = math.radians(sudut_derajat)
151:     cos_t = math.cos(rad)
152:     sin_t = math.sin(rad)
158:     w_baru = max(1, round(abs(w_lama * cos_t) + abs(h_lama * sin_t)))
159:     h_baru = max(1, round(abs(w_lama * sin_t) + abs(h_lama * cos_t)))
```
* **Baris 150-152:** Mengonversi derajat sudut ke radian: $\theta = \text{radians}(\text{sudut})$. Menghitung nilai kosinus dan sinus.
* **Baris 158-159 (Bounding Box Adaptif):** Menghitung dimensi kanvas baru yang membesar secara otomatis agar sudut-sudut gambar yang berputar tidak terpotong tepi kanvas (*anti-clipping*).

```python
161:     cx_lama = w_lama / 2.0
162:     cy_lama = h_lama / 2.0
163:     cx_baru = w_baru / 2.0
164:     cy_baru = h_baru / 2.0
166:     hasil = CitraMatriks(w_baru, h_baru, citra.mode, nilai_dasar=latar_belakang)
168:     for ny in range(h_baru):
169:         for nx in range(w_baru):
172:             dx = nx - cx_baru
173:             dy = ny - cy_baru
176:             sx = round(dx * cos_t + dy * sin_t + cx_lama)
177:             sy = round(-dx * sin_t + dy * cos_t + cy_lama)
180:             if 0 <= sx < w_lama and 0 <= sy < h_lama:
181:                 hasil.set_pixel(nx, ny, citra.get_pixel(sx, sy))
183:     return hasil
```
* **Baris 161-164:** Menentukan titik pusat rotasi (poros tengah citra lama dan kanvas baru).
* **Baris 168-177 (Inverse Mapping):** Menyisir setiap piksel pada kanvas baru $(nx, ny)$, menghitung jaraknya terhadap titik tengah $(dx, dy)$, lalu memutar balik ke koordinat sumber $(sx, sy)$ menggunakan rumus rotasi invers.
* **Baris 180-181:** Jika koordinat asal $(sx, sy)$ berada di dalam citra sumber, intensitas piksel disalin ke kanvas baru. Jika di luar batas, piksel dibiarkan terisi warna latar belakang (`latar_belakang`).

#### Bagian 3: Pemotongan Citra (`potong_citra`)
```python
190: def potong_citra(citra: CitraMatriks, x_awal: int, y_awal: int, lebar_potong: int, tinggi_potong: int) -> CitraMatriks:
209:     if x_awal < 0 or y_awal < 0 or lebar_potong <= 0 or tinggi_potong <= 0:
210:         raise ValueError("Parameter pemotongan tidak valid (harus positif).")
211:     if (x_awal + lebar_potong > citra.lebar) or (y_awal + tinggi_potong > citra.tinggi):
212:         raise ValueError("Area potong melebihi batas citra.")
217:     hasil = CitraMatriks(lebar_potong, tinggi_potong, citra.mode)
218:     for ny in range(tinggi_potong):
219:         for nx in range(lebar_potong):
220:             hasil.set_pixel(nx, ny, citra.get_pixel(x_awal + nx, y_awal + ny))
221:     return hasil
```
* **Baris 190-221:** Mengambil sub-area persegi panjang (Region of Interest / ROI). Memvalidasi bahwa kotak area pemotongan tidak keluar dari batas citra sumber, kemudian menyalin piksel dari koordinat $(x_{\text{awal}} + nx, y_{\text{awal}} + ny)$ ke kanvas potongan yang baru.

#### Bagian 4: Penskalaan Citra (`skala_citra`)
```python
228: def skala_citra(citra: CitraMatriks, faktor_x: float, faktor_y: float) -> CitraMatriks:
248:     w_baru = max(1, round(citra.lebar * faktor_x))
249:     h_baru = max(1, round(citra.tinggi * faktor_y))
251:     hasil = CitraMatriks(w_baru, h_baru, citra.mode)
253:     for ny in range(h_baru):
255:         sy = min(citra.tinggi - 1, int(ny / faktor_y))
256:         for nx in range(w_baru):
258:             sx = min(citra.lebar - 1, int(nx / faktor_x))
259:             hasil.set_pixel(nx, ny, citra.get_pixel(sx, sy))
261:     return hasil
```
* **Baris 248-249:** Menghitung resolusi baru berdasarkan faktor pembesaran horizontal ($f_x$) dan vertikal ($f_y$).
* **Baris 253-261 (Nearest-Neighbor Interpolation):** Mencari indeks piksel asal terdekat dengan membagi koordinat baru terhadap faktor skala: $sx = \text{int}(nx / f_x)$ dan $sy = \text{int}(ny / f_y)$. Fungsi `min()` memastikan indeks tidak pernah melebihi ukuran citra sumber.

---

## 5. Modul 4: Operasi Berbasis Bingkai / Frame Processing (`modul4_operasi_bingkai.py`)

### 5.1 Teori Pemrosesan Multi-Citra
Operasi bingkai (*frame operations*) menggabungkan piksel dari dua citra atau lebih pada koordinat yang sama:
$$C(x, y) = A(x, y) \circ B(x, y)$$

---

### 5.2 Bedah Kode `modul4_operasi_bingkai.py` Baris per Baris

#### Bagian 1: Penyelarasan Kanvas Berbeda Resolusi (`_selaraskan_kanvas`)
```python
47: def _selaraskan_kanvas(citra_a: CitraMatriks, citra_b: CitraMatriks):
51:     w_kanvas = max(citra_a.lebar, citra_b.lebar)
52:     h_kanvas = max(citra_a.tinggi, citra_b.tinggi)
54:     offset_ax = (w_kanvas - citra_a.lebar) // 2
55:     offset_ay = (h_kanvas - citra_a.tinggi) // 2
57:     offset_bx = (w_kanvas - citra_b.lebar) // 2
58:     offset_by = (h_kanvas - citra_b.tinggi) // 2
60:     return w_kanvas, h_kanvas, offset_ax, offset_ay, offset_bx, offset_by
```
* **Baris 47-60:** Jika dua citra yang dioperasikan memiliki ukuran yang berbeda, fungsi ini menciptakan kanvas berukuran maksimum dari keduanya dan menghitung nilai pergeseran (*offset*) agar citra yang lebih kecil diposisikan tepat di tengah (*centered*).

#### Bagian 2: Penggabungan Citra Berbobot / Blending (`gabung_citra`)
```python
67: def gabung_citra(citra_a: CitraMatriks, citra_b: CitraMatriks, bobot_a: float = 0.5) -> CitraMatriks:
88:     bobot_b = 1.0 - bobot_a
99:     kw, kh, oax, oay, obx, oby = _selaraskan_kanvas(a, b)
100:     hasil = CitraMatriks(kw, kh, mode_target)
```
* **Rumus Linear Blending:**
  $$C(x, y) = \text{clamp}(w_a \cdot A(x, y) + (1 - w_a) \cdot B(x, y))$$
* **Baris 88:** Menghitung bobot citra kedua: $w_b = 1.0 - w_a$.
* **Baris 112-127:** Pada area di mana kedua citra saling bertumpuk, nilai piksel dilebur sesuai proporsi bobot. Pada area yang hanya terisi oleh salah satu gambar, piksel gambar tersebut ditampilkan utuh. Menghasilkan efek transparansi / *watermarking*.

#### Bagian 3: Deteksi Gerakan / Frame Differencing (`deteksi_gerakan`)
```python
145: def deteksi_gerakan(frame_awal: CitraMatriks, frame_lanjut: CitraMatriks, ambang_gerak: int = 30):
171:             p1 = fa.get_pixel(x, y)
172:             p2 = fl.get_pixel(x, y)
173:             diff = abs(int(p1) - int(p2))
175:             citra_selisih.set_pixel(x, y, diff)
176:             citra_mask.set_pixel(x, y, 255 if diff >= ambang_gerak else 0)
178:     return citra_selisih, citra_mask
```
* **Rumus Selisih Absolut & Mask Biner:**
  $$\Delta(x, y) = |I_2(x, y) - I_1(x, y)|$$
  $$\text{Mask}(x, y) = \begin{cases} 255, & \text{jika } \Delta(x, y) \ge \text{ambang} \\ 0, & \text{jika } \Delta(x, y) < \text{ambang} \end{cases}$$
* **Baris 171-176:** Menghitung selisih intensitas antara dua frame video. Daerah yang statis (latar belakang) menghasilkan selisih mendekati 0 (hitam). Daerah tempat objek berpindah posisi menghasilkan selisih besar yang ditandai dengan warna putih ($255$) pada citra masker biner.

#### Bagian 4: Operasi Aljabar Logika Citra Biner
* **Baris 192-208 (`logika_and`):**
  Piksel output bernilai $255$ jika piksel $A > 0$ **DAN** piksel $B > 0$. Menghasilkan irisan dua bentuk (*intersection*).
* **Baris 211-227 (`logika_or`):**
  Piksel output bernilai $255$ jika piksel $A > 0$ **ATAU** piksel $B > 0$. Menggabungkan dua siluet (*union*).
* **Baris 229-246 (`logika_xor`):**
  Piksel bernilai $255$ jika hanya salah satu citra yang aktif `(va > 0) ^ (vb > 0)`. Menemukan area yang tidak beririsan.
* **Baris 248-268 (`logika_sub`):**
  Pengurangan logika ($A \land \neg B$). Memotong/menghapus siluet objek $B$ dari bentuk citra $A$.
* **Baris 270-278 (`logika_not`):**
  Inversi biner ($255 - \text{piksel}$).

---

## 6. Modul 5: Operasi Global & Analisis Histogram (`modul5_operasi_global.py`)

### 6.1 Teori Histogram & Ekualisasi Kumulatif (CDF)
Histogram adalah diagram frekuensi kemunculan intensitas keabuan $k \in [0, 255]$:
$$h(k) = n_k$$
Di mana $n_k$ adalah banyaknya piksel yang memiliki nilai intensitas $k$.

Citra yang memiliki kontras rendah memiliki kurva histogram yang sempit dan menumpuk di area tengah saja. **Ekualisasi Histogram** meratakan kurva tersebut ke seluruh rentang dinamis $[0, 255]$ menggunakan Fungsi Distribusi Kumulatif / Cumulative Distribution Function (CDF):
$$\text{CDF}(k) = \sum_{j=0}^{k} h(j)$$
Pemetaan intensitas baru dihitung melalui rumus standar:
$$s_k = \text{round}\left( \frac{\text{CDF}(k) - \text{CDF}_{\min}}{\text{TotalPiksel} - \text{CDF}_{\min}} \times 255 \right)$$

---

### 6.2 Bedah Kode `modul5_operasi_global.py` Baris per Baris

#### Bagian 1: Perhitungan Histogram (`hitung_histogram`)
```python
44: def hitung_histogram(citra: CitraMatriks) -> Dict[str, List[int]]:
52:     if citra.mode == "GRAYSCALE":
53:         h_gray = [0] * 256
54:         for y in range(citra.tinggi):
55:             for x in range(citra.lebar):
56:                 intensitas = citra.get_pixel(x, y)
57:                 h_gray[intensitas] += 1
58:         return {"gray": h_gray}
```
* **Baris 44-58:** Menyiapkan list 256 elemen bernilai 0. Menyisir setiap piksel pada baris $y$ kolom $x$, mengambil nilai intensitasnya, lalu menaikkan nilai pencacah pada indeks intensitas tersebut (`h_gray[intensitas] += 1`). Pada mode RGB, proses dilakukan pada 3 array terpisah (`h_r`, `h_g`, `h_b`).

#### Bagian 2: Ekualisasi Histogram Kumulatif
```python
76: def _bangun_tabel_ekualisasi(hist: List[int], total_piksel: int) -> List[int]:
79:     cdf = [0] * 256
80:     kumulatif = 0
81:     for i in range(256):
82:         kumulatif += hist[i]
83:         cdf[i] = kumulatif
86:     cdf_min = 0
87:     for val in cdf:
88:         if val > 0:
89:             cdf_min = val
90:             break
94:     lut = [0] * 256
95:     pembagi = total_piksel - cdf_min
99:     for k in range(256):
100:         if cdf[k] == 0:
101:             lut[k] = 0
102:         else:
103:             nilai_baru = round(((cdf[k] - cdf_min) / pembagi) * 255)
104:             lut[k] = max(0, min(255, nilai_baru))
106:     return lut
```
* **Baris 79-84:** Menghitung nilai kumulatif CDF dari indeks 0 sampai 255.
* **Baris 86-91:** Menemukan nilai CDF bukan nol terkecil (`cdf_min`).
* **Baris 94-106:** Membangun Lookup Table (LUT). Memetakan setiap derajat intensitas lama $k$ ke intensitas baru yang telah diratakan menggunakan rumus CDF standar.

```python
109: def ekualisasi_histogram(citra: CitraMatriks) -> CitraMatriks:
127:         lut = _bangun_tabel_ekualisasi(hist_data["gray"], total_piksel)
128:         for y in range(citra.tinggi):
129:             for x in range(citra.lebar):
130:                 lama = citra.get_pixel(x, y)
131:                 hasil.set_pixel(x, y, lut[lama])
```
* **Baris 109-142:** Menerapkan tabel pemetaan LUT ke seluruh piksel citra. Nilai lama digantikan dengan nilai ekualisasi baru: `lut[lama]`.

#### Bagian 3: Evaluasi Statistik Citra (`evaluasi_statistik`)
```python
148: def evaluasi_statistik(citra: CitraMatriks) -> Dict[str, float]:
166:     rata_rata = sum(nilai_list) / total_piksel
167:     variansi = sum((v - rata_rata) ** 2 for v in nilai_list) / total_piksel
168:     standar_deviasi = math.sqrt(variansi)
```
* **Baris 166 (Rata-rata / Mean $\mu$):** Total intensitas dibagi jumlah piksel. Mengukur kecerahan global citra.
* **Baris 167-168 (Standar Deviasi Kontras $\sigma$):** Akar kuadrat dari variansi. Mengukur seberapa lebar penyebaran warna dari nilai rata-ratanya. Nilai $\sigma$ yang tinggi membuktikan kontras citra sangat baik dan dinamis.
* **Baris 173-174:** Mencatat nilai piksel paling gelap (`min`) dan paling terang (`max`).

#### Bagian 4: Visualisasi Grafik Histogram ASCII di Terminal
```python
182: def tampilkan_histogram_ascii(hist: List[int], judul: str = "HISTOGRAM", tinggi_grafik: int = 10, jumlah_bin: int = 32):
191:     bin_size = 256 // jumlah_bin
192:     kelompok = [0] * jumlah_bin
193:     for i in range(256):
194:         b = min(jumlah_bin - 1, i // bin_size)
195:         kelompok[b] += hist[i]
```
* **Baris 182-210:** Karena program tidak menggunakan jendela GUI Matplotlib, fungsi ini menggambar grafik batang histogram langsung di layar terminal menggunakan karakter teks:
  * **Baris 191-195:** Mengelompokkan (*binning*) 256 tingkat keabuan menjadi 32 kelompok baris.
  * **Baris 201-209:** Mencetak grafik batang vertikal dari atas ke bawah menggunakan karakter `#` lengkap dengan sumbu horizontal.

---

## 7. Modul Pendukung: Antarmuka Terminal & Inisialisasi Paket (`main.py` & `__init__.py`)

### 7.1 Berkas `main.py`
Menyediakan antarmuka terminal interaktif berbasis teks (*Command Line Interface*):
* **Baris 28-111 (`jalankan_modul`):** Fungsi pengontrol yang mengeksekusi skenario uji modul 1 sampai 5 secara otomatis dan menyimpan hasilnya ke folder `hasil/`.
* **Baris 113-144 (`main`):** Menampilkan menu pilihan `[1]` s.d. `[6]` dan `[0]` untuk keluar.

### 7.2 Berkas `__init__.py`
Menjadikan folder `PCD` ini sebagai paket Python resmi yang dapat diimpor langsung dari proyek lain:
```python
from PCD import CitraMatriks, muat_bmp, simpan_bmp
```

---

## 8. Panduan Menjalankan & Menguji Seluruh Program

### 8.1 Menjalankan Melalui Menu Utama
Buka PowerShell atau Command Prompt pada direktori ini, lalu ketik:
```powershell
python main.py
```
Pilih angka **`6`** untuk mengeksekusi demonstrasi seluruh modul dari awal sampai akhir secara otomatis.

### 8.2 Menjalankan Modul Secara Terpisah
Anda dapat menguji masing-masing modul secara individual:
```powershell
# Modul 1: Parser & Writer Biner BMP
python modul1_representasi_citra.py

# Modul 2: Operasi Titik (Brightness, Kontras, Negasi, Grayscale, Threshold)
python modul2_operasi_titik.py

# Modul 3: Operasi Geometri (Flip, Rotasi Bebas 35°, Crop, Zoom)
python modul3_operasi_geometri.py

# Modul 4: Operasi Bingkai (Blending, Motion Diff, Logika Biner)
python modul4_operasi_bingkai.py

# Modul 5: Operasi Global (Histogram, Ekualisasi CDF, Grafik ASCII)
python modul5_operasi_global.py
```

### 8.3 Verifikasi Kompilasi Kode (Syntax Check)
Untuk menguji bahwa seluruh file bebas dari kesalahan sintaks:
```powershell
python -m py_compile __init__.py main.py modul1_representasi_citra.py modul2_operasi_titik.py modul3_operasi_geometri.py modul4_operasi_bingkai.py modul5_operasi_global.py
```
Jika tidak muncul pesan kesalahan di layar, berarti seluruh modul **100% valid dan siap digunakan**.
