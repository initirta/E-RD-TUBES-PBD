# Sistem Informasi Posyandu Sehat - Negeri Konoa

## Pendahuluan
Sistem ini dirancang untuk mendata seluruh kegiatan posyandu, memantau vaksinasi, KB, serta parameter kecukupan gizi bayi dan balita (0–5 tahun) di Negeri Konoa.  
Dokumen ini berisi pemodelan konseptual basis data yang meliputi **entitas, atribut, relasi, kardinalitas, serta diagram ER** sebagai dasar implementasi sistem.

---

## Aturan Bisnis
1. Setiap posyandu memiliki nama, kelompok, warna, lokasi, dan daftar kegiatan.
2. Kader (dengan data lengkap) bertugas di satu posyandu.
3. Setiap kader mendata Kartu Keluarga (KK) dalam cakupannya. Sasaran: ibu hamil/nifas, ibu bayi/balita, atau keduanya.
4. Data ibu hamil/nifas meliputi HPHT, HPL, pemantauan bulanan, suplemen darah, imunisasi TT, persalinan, durasi nifas. Data KB dicatat untuk ibu bayi/balita.
5. Data bayi/balita: NIK, nama ayah, anak ke-, berat/tinggi/lingkar kepala lahir, cacat bawaan, jenis persalinan, penolong, kondisi hidup/mati.
6. Pemantauan tumbuh kembang bulanan: berat, tinggi, rasio gizi, sakit/penyakit, penyimpangan pertumbuhan & emosional.
7. Riwayat imunisasi: vaksin, tanggal pemberian, KIPI.
8. Data vaksin: kode, nama, klasifikasi, jadwal rekomendasi, ketersediaan per posyandu.

---

## Pemodelan Konseptual

### Entitas & Atribut

| Entitas | Atribut kunci | Atribut penting lainnya |
|---------|--------------|--------------------------|
| **Posyandu** | id_posyandu | nama, kelompok, warna, lokasi |
| **Kader** | id_kader | nama, tgl_lahir, jabatan, status, id_posyandu (FK) |
| **KartuKeluarga** | no_kk | alamat_kk, id_posyandu (FK) |
| **Ibu** | nik_ibu | nama, tgl_lahir, status_hamil, imunisasi_tt_terakhir, no_kk (FK) |
| **Kehamilan** | id_kehamilan | HPHT, HPL, tgl_persalinan, durasi_nifas, nik_ibu (FK) |
| **PemantauanKehamilan** | id_pemantauan_kehamilan | bulan_ke, kondisi, tgl_pemantauan, id_kehamilan (FK) |
| **BayiBalita** | nik_anak | nama_ayah, berat_lahir, cacat_bawaan, jenis_persalinan, kondisi_anak, id_kehamilan (FK) |
| **PemantauanTumbuhKembang** | id_pemantauan | berat_badan, rasio_gizi, penyimpangan, nik_anak (FK) |
| **Vaksin** | kode_vaksin | nama, klasifikasi, jadwal_rekomendasi |
| **RiwayatImunisasi** | id_imunisasi | tgl_pemberian, kipi, nik_anak (FK), kode_vaksin (FK) |
| **KetersediaanVaksin** | id_ketersediaan | stok, tgl_update, id_posyandu (FK), kode_vaksin (FK) |
| **KegiatanPosyandu** | id_kegiatan | nama_kegiatan, deskripsi, id_posyandu (FK) |
| **RiwayatSuplemenDarah** | id_suplemen | tgl_pemberian, periode_ke, nik_ibu (FK) |
| **RiwayatKB** | id_kb | tgl_mulai, tgl_selesai, jenis_kb, nik_ibu (FK) |

> **Catatan:** Untuk detail tipe data dan contoh nilai, lihat [laporan lengkap](link_ke_laporan_lengkap.md).

---

## Relasi Antar Entitas & Kardinalitas

Berikut adalah relasi binary utama beserta kardinalitas dan penjelasan:

| Relasi | Entitas 1 → Entitas 2 | Kardinalitas | Penjelasan |
|--------|----------------------|--------------|-------------|
| Mempekerjakan | Posyandu → Kader | 1 : N | Satu posyandu memiliki banyak kader. |
| Mencakup | Posyandu → KK | 1 : N | Satu posyandu mencakup banyak KK. |
| TerdaftarDi | Ibu → KK | N : 1 | Satu ibu terdaftar di satu KK. |
| Menjalani | Ibu → Kehamilan | 1 : N | Satu ibu bisa mengalami banyak kehamilan. |
| Melahirkan | Kehamilan → BayiBalita | 1 : 1 | Satu kehamilan melahirkan satu bayi (asumsi tunggal). |
| Dipantau | BayiBalita → PemantauanTumbuhKembang | 1 : N | Satu bayi punya banyak catatan bulanan. |
| MendapatImunisasi | BayiBalita → RiwayatImunisasi | 1 : N | Satu bayi bisa mendapat banyak vaksin. |
| MenggunakanVaksin | RiwayatImunisasi → Vaksin | N : 1 | Satu riwayat imunisasi menggunakan satu jenis vaksin. |
| TersediaDi | Posyandu → KetersediaanVaksin | 1 : N | Satu posyandu punya banyak stok vaksin. |
| MerupakanMasterVaksin | Vaksin → KetersediaanVaksin | 1 : N | Satu vaksin bisa tersedia di banyak posyandu. |
| Menyelenggarakan | Posyandu → KegiatanPosyandu | 1 : N | Satu posyandu punya banyak kegiatan. |
| MenerimaSuplemen | Ibu → RiwayatSuplemenDarah | 1 : N | Satu ibu mendapat banyak suplemen. |
| MenggunakanKB | Ibu → RiwayatKB | 1 : N | Satu ibu bisa punya riwayat KB beberapa periode. |

---

## ER Diagram (Mermaid)

> **Cara melihat:** Salin kode di bawah ke [Mermaid Live Editor](https://mermaid.live) atau gunakan plugin GitHub yang mendukung `erDiagram`.

```mermaid
erDiagram
    POSYANDU {
        int id_posyandu PK
        string nama_posyandu
        string kelompok
        string warna
        text lokasi
    }
    KADER {
        string id_kader PK
        string nama_kader
        enum jenis_kelamin
        date tgl_lahir
        text alamat
        string no_telp
        string jabatan
        date tgl_masuk
        enum status
        int id_posyandu FK
    }
    KK {
        string no_kk PK
        text alamat_kk
        int id_posyandu FK
    }
    IBU {
        string nik_ibu PK
        string nama_ibu
        date tgl_lahir_ibu
        text alamat_ibu
        string no_telp_ibu
        boolean status_hamil
        boolean status_memiliki_anak
        date imunisasi_tt_terakhir
        string no_kk FK
    }
    KEHAMILAN {
        int id_kehamilan PK
        string nik_ibu FK
        date tgl_terakhir_haid
        date hpl
        date tgl_persalinan
        int durasi_nifas
        int usia_kehamilan_mgg_lahir
    }
    PEMANTAUAN_KEHAMILAN {
        int id_pemantauan_kehamilan PK
        int id_kehamilan FK
        int bulan_ke
        text kondisi
        date tgl_pemantauan
    }
    BAYI_BALITA {
        string nik_anak PK
        string nama_anak
        string nama_ayah
        int anak_ke
        int dari_berapa_bersaudara
        date tgl_lahir_anak
        string tempat_lahir
        decimal berat_lahir
        decimal tinggi_lahir
        decimal lingkaran_kepala_lahir
        boolean cacat_bawaan
        text deskripsi_cacat
        enum jenis_persalinan
        enum penolong_persalinan
        enum kondisi_anak
        int id_kehamilan FK
    }
    PEMANTAUAN_TUMBUH_KEMBANG {
        int id_pemantauan PK
        string nik_anak FK
        int bulan_ke
        date tgl_pemantauan
        decimal berat_badan
        decimal tinggi_badan
        string rasio_gizi
        enum status_kesehatan
        string penyakit
        text penyimpangan_pertumbuhan
        text penyimpangan_emosional
    }
    VAKSIN {
        string kode_vaksin PK
        string nama_vaksin
        enum klasifikasi
        string jadwal_rekomendasi
    }
    RIWAYAT_IMUNISASI {
        int id_imunisasi PK
        string nik_anak FK
        string kode_vaksin FK
        date tgl_pemberian
        boolean kipi
        text bentuk_kipi
    }
    KETERSEDIAAN_VAKSIN {
        int id_ketersediaan PK
        int id_posyandu FK
        string kode_vaksin FK
        int stok
        date tgl_update
    }
    KEGIATAN_POSYANDU {
        int id_kegiatan PK
        int id_posyandu FK
        string nama_kegiatan
        text deskripsi
    }
    RIWAYAT_SUPLEMENT_DARAH {
        int id_suplemen PK
        string nik_ibu FK
        date tgl_pemberian
        int periode_ke
    }
    RIWAYAT_KB {
        int id_kb PK
        string nik_ibu FK
        date tgl_mulai
        date tgl_selesai
        string jenis_kb
        text keterangan
    }

    %% Relasi dengan kardinalitas
    POSYANDU ||--o{ KADER : "mempekerjakan"
    POSYANDU ||--o{ KK : "mencakup"
    KK ||--|| IBU : "terdaftar"
    IBU ||--o{ KEHAMILAN : "menjalani"
    KEHAMILAN ||--o{ PEMANTAUAN_KEHAMILAN : "dicatat_dalam"
    KEHAMILAN ||--|| BAYI_BALITA : "melahirkan"
    BAYI_BALITA ||--o{ PEMANTAUAN_TUMBUH_KEMBANG : "dipantau"
    BAYI_BALITA ||--o{ RIWAYAT_IMUNISASI : "mendapat_imunisasi"
    VAKSIN ||--o{ RIWAYAT_IMUNISASI : "digunakan"
    POSYANDU ||--o{ KETERSEDIAAN_VAKSIN : "tersedia_di"
    VAKSIN ||--o{ KETERSEDIAAN_VAKSIN : "stok_vaksin"
    POSYANDU ||--o{ KEGIATAN_POSYANDU : "menyelenggarakan"
    IBU ||--o{ RIWAYAT_SUPLEMENT_DARAH : "menerima_suplemen"
    IBU ||--o{ RIWAYAT_KB : "menggunakan_KB"
