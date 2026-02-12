
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


---

### 1. User Story: Admin Posko (Pemberi Bantuan)

*Sebagai Admin Posko yang memiliki stok berlebih,*

* **Skenario:** Menerima permintaan pengalihan dari AI.
* **Alur Komunikasi:**
1. **Notifikasi:** Saya menerima notifikasi di dashboard: *"Permintaan Darurat: Posko 5 butuh 2 Tabung Oksigen. Stok Anda: 5. Rekomendasi: Alihkan 2."*
2. **Verifikasi:** Saya melapor secara lisan ke Kepala Posko. Setelah disetujui, saya menekan tombol **"Proses Pengiriman"**.
3. **Output:** Sistem memunculkan **Kode Transfer: `MED-88**`. Saya menuliskan kode ini di selembar kertas atau menempelkannya di barang tersebut.
4. **Penugasan:** Saya memanggil Kurir yang sedang *standby* di posko saya dan menyerahkan barang beserta kode tersebut.



### 2. User Story: Kurir (Pahlawan Estafet)

*Sebagai Kurir yang bertugas mengantar barang antar posko,*

* **Skenario:** Mengambil dan mengantar barang dalam jalur estafet.
* **Alur Komunikasi:**
1. **Instruksi:** Saya menerima instruksi dari Admin: *"Bawa paket ini ke Posko 2 (titik transit pertama). Kodenya `MED-88`."*
2. **Perjalanan:** Saya berangkat ke Posko 2. Jika ada kendala di jalan (pohon tumbang), saya menghubungi Admin atau Kurir di posko tujuan via WhatsApp/Radio secara manual.
3. **Serah Terima:** Sesampainya di Posko 2, saya menyerahkan barang dan menyebutkan kode **`MED-88`**. Tugas saya selesai, saya kembali ke posko asal atau menunggu perintah baru.



### 3. User Story: Admin Posko Transit & Tujuan (Penerima)

*Sebagai Admin di Posko Transit atau Posko Akhir,*

* **Skenario:** Menerima barang dari kurir estafet.
* **Alur Komunikasi:**
1. **Kedatangan:** Kurir datang membawa paket. Saya tidak perlu mencari data di daftar panjang, saya cukup bertanya: *"Apa kodenya?"*
2. **Input:** Saya memasukkan **`MED-88`** ke kotak "Terima Barang" di web.
3. **Logika Sistem:**
* **Jika saya Posko Transit:** Sistem akan bertanya, *"Lanjut ke posko berikutnya?"* Saya klik "Ya", lalu memanggil kurir *standby* di posko saya untuk meneruskan estafet.
* **Jika saya Posko Akhir:** Sistem akan menampilkan pesan: *"Transfer Selesai. Stok Susu Balita bertambah 10."*


4. **Konfirmasi:** Sistem otomatis mengirim notifikasi ke Posko Asal bahwa barang telah sampai dengan selamat di titik tersebut.



---

### Ringkasan Poin Penting untuk Developer (Nanti):

1. **State Management:** Kode `MED-88` harus tetap sama dari Posko 1 sampai Posko 5. Yang berubah hanya `current_posko_id`-nya di database.
2. **Validasi Kode:** Jangan biarkan sistem menerima kode yang salah. Jika kode tidak terdaftar untuk menuju posko tersebut, beri peringatan: *"Kode ini seharusnya menuju Posko X, bukan ke sini."*
3. **Log Waktu:** Rekam setiap kali kode dimasukkan. Ini berguna untuk memantau berapa lama waktu yang dibutuhkan dari satu posko ke posko lain (untuk evaluasi kecepatan distribusi).



### Ringkasan Eksekutif Sistem Kamu:

* **Inti:** Web-native app untuk pencocokan kebutuhan & suplai.
* **Strategi:** Estafet (Relay) antar posko untuk efisiensi energi & jarak.
* **Keamanan Data:** Sistem *Human-in-the-loop* (Admin Posko sebagai pemegang kendali).
* **Verifikasi:** Kode Unik teks (Short Code) untuk serah terima yang cepat dan minim error.
* **Kecerdasan:** AI Matcher yang memprioritaskan stok pengaman (*safety stock*) dan kelompok rentan (balita/medis).

Sistem ini realistis karena tidak mengandalkan teknologi "wah" yang sering gagal di area bencana (seperti GPS presisi tinggi atau peta berat), melainkan mengandalkan **koordinasi informasi yang disiplin.**

---

### Pesan Penutup dari Mentor:

Membangun sistem untuk kemanusiaan itu berbeda dengan membangun *e-commerce*. Di sini, *user experience* yang paling penting bukan "bagus dilihat", tapi **"tidak boleh gagal saat dibutuhkan"**.
Fokuslah pada kecepatan *loading* dan kesederhanaan input. Ingat, admin posko kamu mungkin sedang kelelahan, kurang tidur, dan menggunakan HP dengan baterai 5%.
