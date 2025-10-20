# Teks Bug — Pengujian Aman & Mitigasi

**PENTING:** Dokumen ini bertujuan untuk edukasi dan pengujian di lingkungan terisolasi milik pengembang (kev.store). Dilarang menggunakan teknik ini untuk menyerang, merusak, atau mengganggu perangkat/layanan orang lain.

---

## Deskripsi
Proyek ini berisi pedoman untuk:
- Menguji bagaimana aplikasi menangani input tak terduga (malformed input, very long strings, kontrol char, dsb.) secara **aman**.
- Mengimplementasikan mitigasi untuk mencegah crash, DoS lokal, dan kebocoran memori.
- Menyiapkan prosedur pengujian etis dan responsible disclosure.

---

## Prinsip Pengujian Aman
1. Lakukan hanya di lingkungan kontrol (local dev, container, VM) yang bisa di-reset.  
2. Jangan mengirim payload berbahaya ke layanan pihak ketiga tanpa izin tertulis.  
3. Catat hasil, tangani temuan secara bertanggung jawab, dan jangan publikasikan eksploit tanpa mitigasi.

---

## Contoh Kasus Uji (AMAN)
Gunakan contoh yang membatasi potensi destruktif:
- **String panjang terkendali** — uji dengan ukuran bertahap (1KB → 10KB → 100KB) bukan langsung 10MB+.  
- **Karakter kontrol non-printable** — gunakan contoh kecil untuk melihat bagaimana sistem merespons.  
- **Unicode normalization** — cek normalisasi (NFC/NFD) sebelum perbandingan.  
- **Fuzzing terkontrol** — atur timeouts dan batas output.

---

## Contoh Kode Mitigasi (Node.js / Express)

### 1. Batasi panjang payload (body parser)
```js
// server.js
const express = require('express');
const app = express();

// batasi body ke 100kb
app.use(express.json({ limit: '100kb' }));
app.use(express.urlencoded({ extended: true, limit: '100kb' }));

app.post('/submit', (req, res) => {
  // sanitize & validate sebelum proses
  const input = String(req.body.text || '').slice(0, 100000); // double-check
  // proses aman...
  res.send({ ok: true });
});

app.listen(3000);
