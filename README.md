# HRIS Presensi Mobile PWA — Starter Production Architecture

Aplikasi presensi mobile dengan geofence, face verification/liveness, izin/sakit/cuti/dinas luar, WhatsApp notification, rekap admin multi-role, Excel/PDF, dan job otomatis.

## Stack
- Next.js 16 App Router + TypeScript + Bootstrap 5
- Supabase PostgreSQL + Auth + private Storage + RLS + Cron
- `@vladmandic/human` untuk face descriptor/liveness di browser; model disalin ke `public/models/human` saat `npm install`
- Fonnte untuk WhatsApp
- Luxon untuk `Asia/Makassar`
- ExcelJS + pdf-lib untuk export

## Instalasi
```bash
cp .env.example .env.local
npm install
npm run dev
```

## Setup Supabase
1. Buat project Supabase.
2. Jalankan migration `supabase/migrations/0001_schema.sql` di SQL Editor / Supabase CLI.
3. Isi `office_locations` dengan koordinat kantor yang benar.
4. Pastikan setiap `staff.schedule_id` menunjuk ke jadwal kerja.
5. Buat user admin pada Supabase Auth. Setelah itu insert profile:
```sql
insert into public.profiles(id,full_name,role,division_id)
values ('AUTH-USER-UUID','Nama Admin','super_admin',null);
```
6. Isi env Supabase publishable key + service role key. Service role **jangan** diberi prefix `NEXT_PUBLIC_`.
7. Konfigurasi Fonnte token dan `WA_HR_TARGETS`.
8. Deploy ke domain HTTPS (kamera dan geolocation butuh secure context).

## Enrollment wajah
Enrollment tersedia di `/admin/face-enroll` dan hanya untuk `super_admin`. Sistem mengambil 3 sampel descriptor, merata-ratakannya, menormalisasi hasil, menonaktifkan template lama, lalu menyimpan template baru melalui route server-only. Idealnya enrollment dilakukan HR dengan staf hadir langsung. Kalibrasi `face_similarity_threshold` terhadap sampel nyata sebelum go-live.

## Cron
Supabase Cron menggunakan UTC pada contoh migration. Karena WITA = UTC+8:
- 08:15 WITA → 00:15 UTC
- 23:55 WITA → 15:55 UTC

Endpoint cron wajib memakai header `Authorization: Bearer <CRON_SECRET>`.

## Aturan izin
- `Sakit`: surat dokter + foto resep/obat wajib.
- `Izin/Cuti/Dinas Luar`: lampiran opsional pada starter; ubah menjadi wajib jika kebijakan perusahaan mengharuskan.
- Link Maps adalah titik lokasi **saat submit**, bukan live tracking.
- Link dokumen di WhatsApp menggunakan signed URL 24 jam.

## Multi-role
- `super_admin`: seluruh divisi.
- `manager` / `supervisor`: dibatasi `division_id` miliknya.
- Public attendance tidak membaca tabel langsung; semua query publik melewati server routes.

## Anti-fraud
Implemented:
- server timestamp
- server geofence validation
- GPS accuracy threshold
- face similarity to selected staff
- random liveness challenge
- anti-spoof/liveness score from Human
- private evidence photo
- idempotency + DB transaction lock
- audit logs

Not guaranteed by a PWA:
- reliable Android mock-location detection
- hardware/device attestation
- proof that the front-end JavaScript was not tampered with on a rooted/compromised device

Untuk risiko tinggi, gunakan Android wrapper/native, Play Integrity, per-device registration, dan MDM/managed devices.

## Penalty keterlambatan
Kolom config disediakan namun `penalty_enabled=false`. Sistem sebaiknya hanya menghitung simulasi/indikator sampai kebijakan HR/payroll dan aspek hukum perusahaan sudah disahkan; jangan otomatis memotong payroll dari starter ini.

## Production checklist
- [ ] HTTPS + HSTS
- [ ] Rotate all secrets
- [ ] Disable public signup admin
- [ ] Rate limit public routes (WAF/Redis/Vercel Firewall)
- [ ] CAPTCHA/abuse control bila endpoint diserang
- [ ] Configure backups / PITR
- [ ] Privacy notice dan acknowledgement karyawan
- [ ] Retention policy untuk selfie, biometric template, medical attachments, GPS
- [ ] Incident response & audit review
- [ ] QA di Android/iOS; browser permissions; GPS indoor/outdoor
- [ ] Face threshold calibration lintas lighting/device
- [ ] Uji paralel 20+ user dan double tap
- [ ] Uji hari libur/off-day agar Alfa tidak terbentuk keliru
