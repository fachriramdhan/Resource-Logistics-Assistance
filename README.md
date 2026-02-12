
## **BLUEPRINT SISTEM: "RELA-ESTAFET"**

*(Resource & Logistics Assistance)*

### **1. Arsitektur Data (Logic-Centered)**

Sistem ini menggunakan relasi antar objek yang sangat ketat untuk memastikan tidak ada barang yang "hilang" di database.

* **Node (Posko/Gudang):** Menyimpan koordinat (hanya teks), nama kepala posko, jumlah kurir standby, dan kontak darurat.
* **Inventory (Stok):** Memisahkan barang berdasarkan kategori prioritas (**P1: Medis/Nyawa**, **P2: Balita/Rentan**, **P3: Umum**).
* **Transaction (Logistik):** Mencatat perjalanan barang dengan status: `Sedia` -> `Requested` -> `In-Transit` -> `Arrived`.

---

### **2. Alur Kerja Sistem (Step-by-Step)**

#### **A. Fase Identifikasi (AI Role)**

1. Admin Posko A input kebutuhan: *"Susu Balita - 10 Box - Status: Urgent"*.
2. **AI Matcher** memindai database dan menemukan stok di Posko B.
3. AI menghitung rute estafet: **Posko B -> Posko C -> Posko D -> Posko A**.
4. AI mengirim notifikasi ke Admin Posko B: *"Rekomendasi pengalihan stok ke Posko A via Estafet."*

#### **B. Fase Persetujuan (Human-in-the-Loop)**

1. Admin Posko B (setelah bicara dengan Kepala Posko) menekan tombol **"Setujui Request"**.
2. Sistem men-generate **Kode Transfer** (Contoh: `BANTU-77`).
3. Status barang di Posko B berubah dari `Available` menjadi `Locked (In-Transit)`.

#### **C. Fase Distribusi (Estafet Kode)**

* **Leg 1:** Kurir B membawa barang ke Posko C. Admin Posko C memasukkan kode `BANTU-77` di web. Status: *Diterima di Transit C*.
* **Leg 2:** Kurir C membawa barang ke Posko D. Admin Posko D memasukkan kode `BANTU-77`. Status: *Diterima di Transit D*.
* **Leg 3 (Final):** Kurir D sampai di Posko A. Admin Posko A memasukkan kode `BANTU-77`. Status: **Selesai (Stok Masuk ke Posko A)**.

---

### **3. Rancangan Antarmuka (UI/UX)**

#### **A. Dashboard Admin (Fokus Utama)**

* **Widget "Darurat":** Daftar barang P1 (Medis) yang sedang menuju posko mereka.
* **Input Box Besar:** *"Masukkan Kode Terima"* (Fokus utama saat kurir datang).
* **Inventory Tracker:** Grafik batang sederhana menunjukkan sisa stok vs jumlah pengungsi di posko tersebut.

#### **B. Dashboard Kurir (Mobile Friendly)**

* **Daftar Tugas:** Tampilan kartu berisi: [Kode Barang] - [Tujuan Berikutnya] - [Nama Barang].
* **Status Standby:** Switch ON/OFF untuk memberi tahu AI apakah kurir tersedia untuk estafet.

---

### **4. Logika AI Matcher (The Logic Engine)**

AI tidak hanya mencocokkan barang, tapi menggunakan bobot (**Weighting**):


* **Kategori:** Medis = 100, Balita = 80, Umum = 40.
* **Langkah Estafet:** Semakin banyak perpindahan posko, skor sedikit turun (AI akan mencari jalur estafet terpendek).

---

### **5. Strategi "Low-Bandwidth" (Realistis)**

* **Offline First:** Menggunakan *Service Workers*. Admin bisa input data saat sinyal hilang, dan web akan otomatis sinkronisasi saat sinyal muncul kembali (saat kurir bergerak ke area bersinyal).
* **No Media:** Tidak ada upload foto/video di jalur distribusi utama. Semua berbasis teks untuk kecepatan maksimal.

---

### **6. Struktur Tabel Database (Gambaran Kasar)**

| Tabel | Kolom Utama |
| --- | --- |
| **Posko** | `id`, `nama`, `lokasi_teks`, `jumlah_kurir`, `kontak` |
| **Barang** | `id`, `nama`, `kategori_id`, `total_stok` |
| **Kebutuhan** | `id`, `posko_id`, `barang_id`, `qty_butuh`, `tingkat_urgensi` |
| **Estafet** | `id`, `kode_unik`, `posko_asal`, `posko_tujuan`, `posko_posisi_sekarang`, `status` |

---
