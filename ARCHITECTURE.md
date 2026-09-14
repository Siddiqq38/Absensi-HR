# Arsitektur Sistem

```text
HP Staff (PWA/HTTPS)
  ├─ Geolocation API
  ├─ Camera / MediaDevices
  ├─ Human face descriptor + liveness challenge
  └─ Next.js UI
          │ HTTPS multipart/JSON
          ▼
Next.js 16 App Router (server routes)
  ├─ validasi geofence ulang (server)
  ├─ validasi face similarity + liveness token
  ├─ timestamp WITA dari server
  ├─ private upload
  ├─ admin role/scope
  ├─ Excel / PDF
  └─ Fonnte adapter
          │
          ▼
Supabase
  ├─ PostgreSQL: master, attendance, leave, audit
  ├─ Auth: Super Admin / Manager / Supervisor
  ├─ Storage private: selfie + dokumen medis
  ├─ RLS: scope divisi
  └─ Cron: reminder 08:15 + close-day/Alfa
```

## Prinsip keamanan
1. Browser hanya UX layer; keputusan final dibuat server.
2. Service-role key dan token WhatsApp hanya ada di server.
3. Timestamp menggunakan `now()` server, disajikan `Asia/Makassar`.
4. Geofence dihitung ulang di server dengan Haversine.
5. Storage privat; dashboard menerima signed URL sementara.
6. Face template tidak pernah dapat dibaca user publik.
7. Attendance RPC memakai advisory lock + unique key untuk mencegah double submit/paralel.
8. Semua perubahan setting penting masuk audit log.

## Batas anti Fake-GPS di PWA
Web Geolocation API tidak memberi sinyal terpercaya bahwa lokasi berasal dari Android mock provider. Versi ini memakai radius, accuracy threshold, risk score, dan bukti wajah/liveness. Untuk proteksi lebih tinggi, bungkus modul sebagai Android native/Capacitor dan tambahkan pemeriksaan mock-location + Play Integrity/device attestation.

## Privasi
Selfie, face descriptor, dokumen sakit, dan koordinat adalah data berisiko tinggi. Terapkan privacy notice internal, dasar pemrosesan yang sesuai, pembatasan akses, retensi, penghapusan, log akses, serta prosedur insiden. Jangan membuat bucket foto publik.
