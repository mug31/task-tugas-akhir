# task-tugas-akhir

Aplikasi mobile Flutter untuk memantau dan memprioritaskan tugas akhir perkuliahan
berdasarkan bobot mata kuliah, tingkat kesulitan, dan sisa waktu menuju deadline.

Dibuat untuk memenuhi **UAS Praktikum Mobile Programming**.

## Latar belakang

Tugas kuliah datang bersamaan dari banyak mata kuliah dengan deadline yang berdekatan.
Daftar tugas biasa (to-do list) hanya menampilkan apa yang harus dikerjakan tanpa memberi
tahu mana yang paling mendesak. Aplikasi ini menghitung skor prioritas setiap tugas
sehingga mahasiswa tahu urutan pengerjaan yang paling masuk akal.

## Rumus prioritas

```
skor = (sks × kesulitan) / sisa_hari
```

Skor dihitung di sisi Flutter (bukan disimpan di database) karena `sisa_hari` berubah
setiap hari. Semakin besar skor, semakin mendesak tugas tersebut.

## Fitur

- CRUD tugas dan CRUD mata kuliah melalui REST API
- Daftar tugas terurut otomatis berdasarkan skor prioritas
- Badge status urgensi: kritis / dekat / santai
- Countdown sisa hari menuju deadline
- Form tambah/edit dengan validasi input
- Penanganan state loading, error, dan retry

## Kebutuhan teknis praktikum yang dipenuhi

| Ketentuan | Implementasi |
| --- | --- |
| User Interface | Row, Column, Card, TextField, ListTile, Badge |
| Navigation | 4 halaman dengan named route |
| REST API | Supabase REST (PostgREST) via package `http` |
| CRUD | Create, Read, Update, Delete pada `tugas` dan `mata_kuliah` |
| Model & JSON | Model class + `fromJson()` / `toJson()` |
| State management | GetX (`GetxController`, `.obs`, `Obx`) |
| Asynchronous | `Future`, `async`, `await` + state loading/error |
| Form & validasi | Form tambah/edit tugas dengan validator |
| Fitur tambahan | Sorting prioritas otomatis + filter status |

## Teknologi

- Flutter (Dart)
- GetX — state management dan routing
- Supabase / PostgREST — backend REST API
- package `http` — HTTP client

## Struktur proyek

```
lib/
├── main.dart
├── models/
│   ├── mata_kuliah.dart
│   └── tugas.dart
├── controllers/
│   ├── mata_kuliah_controller.dart
│   └── tugas_controller.dart
├── services/
│   └── api_service.dart
├── pages/
│   ├── tugas_list_page.dart
│   ├── tugas_form_page.dart
│   ├── tugas_detail_page.dart
│   └── mata_kuliah_page.dart
└── widgets/
    └── tugas_card.dart
```

## Menjalankan

```bash
flutter pub get
flutter run
```

Konfigurasi URL dan API key Supabase diletakkan pada `lib/config.dart`.

## Status

Dalam pengerjaan — UAS Praktikum Mobile Programming.
