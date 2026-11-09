# task-tugas-akhir

Aplikasi mobile Flutter untuk memantau dan memprioritaskan tugas perkuliahan
berdasarkan bobot mata kuliah, tingkat kesulitan, dan sisa waktu menuju deadline.

> **Masalah yang diselesaikan:** mahasiswa tidak kekurangan daftar tugas. Yang tidak ada
> adalah cara menentukan mana yang harus dikerjakan lebih dulu — karena deadline terdekat
> belum tentu yang paling penting.

Dibuat untuk memenuhi **UAS Praktikum Mobile Programming**.

## Masalah

> **Mahasiswa tidak kekurangan daftar tugas — yang tidak ada adalah cara menentukan
> mana yang harus dikerjakan lebih dulu.**

Tugas dari beberapa mata kuliah datang hampir bersamaan dengan deadline yang berdekatan.
Aplikasi to-do list biasa hanya menampilkan daftar itu apa adanya, urut sesuai waktu
ditambahkan atau sesuai tanggal deadline. Akibatnya:

- **Deadline terdekat belum tentu yang paling penting.** Tugas ringan dari mata kuliah
  2 SKS bisa menutupi tugas berat dari mata kuliah 4 SKS yang sebenarnya lebih menentukan
  nilai.
- **Yang paling mendesak justru sering terlewat**, karena yang paling dekat tanggalnya
  belum tentu yang paling besar bobotnya.
- **Mahasiswa mengerjakan berdasarkan urutan yang terasa mendesak**, bukan yang terukur.
  Keputusan diambil pakai perasaan, bukan angka.

### Sasaran pengguna

| | |
| --- | --- |
| **Siapa** | Mahasiswa aktif yang mengambil 5–8 mata kuliah per semester |
| **Kondisi** | Menerima tugas dari banyak mata kuliah dengan deadline berdekatan |
| **Kebiasaan sekarang** | Mencatat di buku, Google Tasks, atau grup WhatsApp — semuanya berupa daftar tanpa bobot |
| **Yang dibutuhkan** | Urutan pengerjaan yang bisa dipertanggungjawabkan, bukan sekadar daftar |
| **Kemampuan teknis** | Terbiasa aplikasi mobile sehari-hari, tidak perlu dipandu |

## Solusi dan korelasinya dengan masalah

Aplikasi ini tidak menambah satu daftar tugas lagi. Yang ditambahkan adalah **satu angka
yang menjawab "kerjakan yang mana dulu"**, dihitung dari tiga hal yang menentukan
mendesak atau tidaknya sebuah tugas:

```
skor = (sks × kesulitan) / sisa_hari
```

| Masalah | Yang dipakai di rumus | Akibatnya pada daftar tugas |
| --- | --- | --- |
| Deadline terdekat belum tentu paling penting | `sks` — bobot mata kuliah | Tugas dari mata kuliah berbobot besar naik peringkat, tidak tenggelam oleh tugas ringan |
| Kesulitan tidak pernah diperhitungkan | `kesulitan` (1–5) — diisi saat membuat tugas | Tugas yang butuh usaha lebih didahulukan meski deadline-nya masih agak jauh |
| Yang mendesak sering terlewat | `sisa_hari` — pembagi | Semakin dekat deadline, skor naik. Sisa 1 hari mengalahkan sisa 14 hari |
| Urutan ditentukan perasaan | Diurutkan menurun berdasarkan skor | Urutan bisa dijelaskan: "ini di atas karena SKS-nya besar dan sisanya 2 hari" |

Konsekuensi yang perlu disadari: **rumus ini menempatkan urgensi di atas kemudahan.**
Tugas ringan yang harus selesai besok tetap masuk kategori dekat, tapi tugas berat yang
deadline-nya dua minggu bisa berada di atasnya. Itu memang disengaja — yang dicari bukan
"apa yang bisa cepat selesai", melainkan "apa yang paling mahal kalau terlewat".

Skor dihitung di sisi Flutter (bukan disimpan di database) karena `sisa_hari` berubah
setiap hari. Kalau disimpan, nilainya akan basi setiap tengah malam dan butuh proses
pembaruan tersendiri.

Setiap kartu tugas menampilkan skornya sebagai badge **KRITIS / DEKAT / SANTAI**, sehingga
keputusan bisa diambil tanpa membuka halaman detail lebih dulu.

Angka skor dari rumus ini **tidak dibatasi 0–1** — tugas dengan sisa 1 hari bisa bernilai 6,
tugas dengan sisa 14 hari bernilai 0,71. Karena itu badge tidak memakai ambang absolut,
melainkan perbandingan terhadap skor tertinggi di daftar saat itu:

| Badge | Syarat | Artinya |
| --- | --- | --- |
| **KRITIS** | skor ≥ 60% dari skor tertinggi | Mengerjakan hal lain lebih dulu berisiko kehilangan nilai besar |
| **DEKAT** | 25% – 59% | Perlu masuk rencana minggu ini |
| **SANTAI** | < 25% | Masih aman, tapi jangan sampai lupa |

Dua hal teknis yang harus ditangani di kode:

- **Sisa hari bisa nol** kalau deadline-nya hari ini. Pembagian dengan nol menghasilkan
  `Infinity`, jadi `sisa_hari` dibatasi minimal 1 sebelum dipakai.
- **Daftar kosong atau satu tugas** membuat "skor tertinggi" tidak bermakna sebagai
  pembanding. Untuk kasus itu badge dihitung langsung dari sisa hari, bukan dari rasio.

## Rancangan antarmuka

Wireframe sketsa tangan, 4 halaman, skala 1:1 (360 × 760 dp).
Berkas sumber: [`design/wireframe-kasar.html`](design/wireframe-kasar.html) —
render satu lembar dengan `design/wireframe-kasar@2x.png`.

<table>
<tr>
<td width="300" valign="top"><img src="design/p1.png" width="280" alt="Wireframe halaman daftar tugas"></td>
<td valign="top">

**1. Daftar Tugas** — `/`

Halaman utama yang langsung menjawab pertanyaan "kerjain yang mana dulu?".

- Kolom cari menyaring judul dan deskripsi tugas.
- Chip filter memilih status: semua, belum, proses, selesai.
- Kartu tugas diurutkan menurun berdasarkan skor prioritas, bukan tanggal.
- Tiap kartu memuat nama mata kuliah, bobot SKS, sisa hari, tingkat kesulitan,
  serta badge urgensi: **KRITIS**, **DEKAT**, atau **SANTAI**.
- Bar tipis di bawah judul menunjukkan besaran skor secara visual.
- Tombol `+` membuka form tambah tugas.
- Navigasi bawah berpindah antara Tugas, Mata Kuliah, dan Profil.

Data: `GET /tugas` (mendukung `?q=` dan `?status=`).

</td>
</tr>

<tr>
<td width="300" valign="top"><img src="design/p2.png" width="280" alt="Wireframe halaman form tugas"></td>
<td valign="top">

**2. Tambah / Ubah Tugas** — `/form`

Satu halaman dipakai untuk dua keperluan: menambah tugas baru dan mengubah tugas lama.

- Mata kuliah dipilih dari daftar yang sudah ada, tidak diketik bebas,
  supaya bobot SKS-nya pasti ikut terbaca.
- Judul wajib diisi, deskripsi opsional.
- Deadline diambil lewat pemilih tanggal sehingga formatnya seragam.
- Tingkat kesulitan memakai tombol 1–5, bukan kolom teks, agar nilai di luar
  rentang tidak mungkin masuk.
- Status dipilih lewat chip.
- Setiap kolom punya pesan kesalahan sendiri di bawahnya. Contoh pada gambar
  adalah keadaan ketika judul dikosongkan.

Data: `POST /tugas` untuk data baru, `PUT /tugas/:id` untuk perubahan.

</td>
</tr>

<tr>
<td width="300" valign="top"><img src="design/p3.png" width="280" alt="Wireframe halaman detail tugas"></td>
<td valign="top">

**3. Detail Tugas** — `/detail`

Dibuka dengan menekan salah satu kartu di halaman daftar.

- Angka besar di atas adalah sisa hari menuju deadline, dihitung ulang setiap
  halaman dibuka.
- Tabel informasi memuat mata kuliah, bobot SKS, kesulitan, skor prioritas
  beserta kategorinya, dan status terkini.
- Deskripsi ditampilkan penuh.
- Status bisa diganti langsung dari sini tanpa membuka form.
- Tombol hapus meminta konfirmasi lebih dulu, karena tindakan ini tidak bisa
  dibatalkan.

Data: `GET /tugas/:id` untuk isi halaman, `PUT /tugas/:id/status` untuk ganti
status cepat, `DELETE /tugas/:id` untuk hapus.

</td>
</tr>

<tr>
<td width="300" valign="top"><img src="design/p4.png" width="280" alt="Wireframe halaman mata kuliah"></td>
<td valign="top">

**4. Mata Kuliah** — `/mata-kuliah`

Halaman pendukung. Bobot SKS yang diisi di sini adalah salah satu pembilang pada
rumus prioritas, jadi data mata kuliah harus benar lebih dulu.

- Daftar mata kuliah beserta jumlah tugas yang menempel padanya.
- Tiap baris bisa diubah atau dihapus.
- Form tambah berisi nama dan bobot SKS 1–6.
- Menghapus mata kuliah ikut menghapus seluruh tugas di dalamnya
  (`ON DELETE CASCADE`), karena itu ada peringatan sebelum tombol simpan.

Data: `GET` / `POST` / `PUT` / `DELETE /mata-kuliah`.

</td>
</tr>
</table>

## Alur halaman

```
Daftar Tugas ──tap kartu──> Detail Tugas ──tombol ubah──> Form (mode edit)
     │                                                          ▲
     └──tombol +──> Form (mode tambah) ─────────────────────────┘
     │
     └──navigasi bawah──> Mata Kuliah
```

Navigasi memakai named route, dan halaman Detail menerima `id` tugas lewat
`arguments` — sesuai pola Modul 08.

## Pemetaan ke modul praktikum

Struktur kode mengikuti pola yang dipakai modul, bukan pola baru. Bagian yang sengaja
dibuat berbeda hanya sumber data (API dan database), karena modul masih memakai
JSONPlaceholder dan API publik yang bersifat baca saja.

| Ketentuan UAS | Modul acuan | Implementasi |
| --- | --- | --- |
| UI (Row, Column, Card, TextField) | 03, 04, 05 | `Row`, `Column`, `Flexible`/`Expanded`, `SizedBox`, `Spacer`, `Card` |
| Daftar data | 06 | `ListView.builder` + kartu tugas |
| Navigasi minimal 3 halaman | 07, 08 | Named route + `arguments` saat ke halaman detail |
| Model & JSON | 10, 13 | Model class + `fromJson()` / `toJson()` |
| REST API | 10, 13 | package `http`, `GET`/`POST`/`PUT`/`DELETE` |
| Asynchronous | 13 | `Future`, `async`, `await` + loading/error state |
| State management | 11 | `GetX` — `GetxController`, `.obs`, `Obx`, `GetMaterialApp` |
| Form & validasi | 13 | `TextField`/`Form`, pesan kesalahan, `SnackBar` notifikasi |
| Fitur tambahan | 06, 09, 10 | Pencarian, filter status, sorting prioritas |

## Struktur proyek

Mengikuti pembagian `model` / `view` / `viewmodel` seperti pada Modul 10 dan 11.

```
lib/
├── main.dart                  // GetMaterialApp, tema, daftar named route
├── model/
│   ├── mata_kuliah.dart       // model + fromJson/toJson
│   └── tugas.dart             // model + fromJson/toJson + getter skorPrioritas
├── view/
│   ├── home.dart              // daftar tugas + pencarian + filter
│   ├── form_tugas.dart        // form tambah/edit + validasi
│   ├── detail_tugas.dart      // detail + countdown
│   └── daftar_matakuliah.dart // CRUD mata kuliah
└── viewmodel/
    ├── tugas_controller.dart  // GetX controller + pemanggilan HTTP
    └── matakuliah_controller.dart
```

Pemanggilan API diletakkan di dalam `viewmodel` seperti pada Modul 10 dan 13, sehingga
`view` hanya mengurus tampilan.

## Teknologi

- Flutter (Dart)
- GetX — state management dan routing
- package `http` — HTTP client
- package `intl` — format tanggal deadline
- Backend: REST API sendiri (Node.js + PostgreSQL), bukan JSONPlaceholder

## Menjalankan

```bash
flutter pub get
flutter run
```

Alamat API diletakkan pada `lib/config.dart` (tidak di-commit).

## Status

Dalam pengerjaan — UAS Praktikum Mobile Programming.
