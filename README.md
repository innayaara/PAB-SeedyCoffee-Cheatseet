# ⚡ CHEATSHEET UJIAN — SeedyCoffee PAB

---

## 1. PENJELASAN PROJECT (1 Menit)

> *"SeedyCoffee adalah aplikasi pemesanan kopi berbasis **Flutter + Supabase** dengan **3 role**: Customer, Admin, dan Kasir. Customer browse menu, checkout, dapat **QR Code** untuk bayar ke kasir. Admin kelola menu/banner dan punya **dashboard AI** (Google Gemini). Kasir scan QR atau input kode untuk konfirmasi pembayaran. State management pakai **Provider**. Fitur unggulan: QR payment, AI insight, multi-role."*

---

## 2. ALUR APLIKASI

```
main.dart → SplashScreen (loading ~2 detik)
    ↓ cek role dari AppProvider
    ├── customer  → /main  (HomeTab, CartTab, NotifTab, ProfileTab)
    ├── admin     → /admin (Dashboard, Menu, Banner, Orders, Promo)
    ├── cashier   → /kasir (Input Kode | Scan QR)
    └── tdk login → /login (Login / Register + OTP email)
```

**Navigasi Customer (Bottom Nav 4 Tab):**
- 🏠 Home → grid menu + banner slider
- 🛒 Cart → keranjang belanja
- 🔔 Notif → notifikasi pesanan & promo
- 👤 Profil → edit nama, HP, foto

---

## 3. ALUR CHECKOUT & PEMBAYARAN

```
[Home] Tap menu → [MenuDetail] Pilih size/gula/es → "Tambah ke Keranjang"
    ↓
[CartTab] Lihat item, ubah qty → "Checkout"
    ↓
[CheckoutScreen] Ringkasan + total → "Konfirmasi & Checkout"
    → AppProvider.checkout() → OrderService.createOrder()
    → INSERT ke tabel 'orders' (status: PENDING)
    → Cart dikosongkan + notifikasi dibuat
    ↓
Tampil QR Code + Kode Unik (mis: SEEDY-AB12) + Total
    ↓
[KasirScreen] Scan QR / input kode → "Konfirmasi Pembayaran"
    → OrderService.confirmPayment()
    → UPDATE orders: status → CONFIRMED
    → INSERT ke tabel 'payments' (method: cash)
    → Customer dapat notifikasi: "Pesanan dikonfirmasi"
```

> ⚠️ Pembayaran adalah **SIMULASI cash di kasir**, bukan payment gateway online.

---

## 4. CRUD DATABASE (Supabase)

| Operasi | Contoh | File |
|---|---|---|
| **CREATE** | Buat order baru saat checkout | `order_service.dart` → `createOrder()` |
| **CREATE** | Tambah menu baru (admin) | `menu_service.dart` → `addMenu()` |
| **READ** | Load semua menu + kategori | `menu_service.dart` → `loadMenus()` |
| **READ** | Load pesanan user (dengan join) | `order_service.dart` → `loadOrders()` |
| **UPDATE** | Konfirmasi bayar (status pending→confirmed) | `order_service.dart` → `confirmPayment()` |
| **UPDATE** | Edit profil user | `auth_service.dart` → `updateProfile()` |
| **DELETE** | Hapus menu | `menu_service.dart` → `deleteMenu()` |
| **DELETE** | Hapus banner | `banner_service.dart` → `deleteBanner()` |

**Fallback jika tanpa internet:** SharedPreferences → StaticDatabase (data dummy)

---

## 5. PACKAGE PENTING

| Package | Fungsi |
|---|---|
| `provider` | State management (ChangeNotifier) |
| `supabase_flutter` | Backend: Auth + DB + Storage |
| `shared_preferences` | Simpan session & cart lokal |
| `qr_flutter` | **Generate** QR Code |
| `mobile_scanner` | **Scan** QR Code (kamera kasir) |
| `fl_chart` | Grafik BarChart dashboard admin |
| `http` | Request ke Google Gemini AI |
| `image_picker` | Ambil foto dari galeri |
| `crop_your_image` | Crop foto sebelum upload |
| `google_fonts` | Font Playfair Display + DM Sans |

---

## 6. FILE PENTING YANG WAJIB DIBUKA

| File | Kenapa Penting |
|---|---|
| `lib/main.dart` | Entry point, semua route, setup Provider |
| `lib/providers/app_provider.dart` | Semua state + logika bisnis (375 baris) |
| `lib/screens/shared/splash_screen.dart` | Routing berdasarkan role |
| `lib/screens/user/checkout_screen.dart` | Checkout + generate QR Code |
| `lib/screens/kasir/kasir_screen.dart` | Scan QR + konfirmasi bayar |
| `lib/services/order_service.dart` | CRUD pesanan di Supabase |
| `lib/services/auth_service.dart` | Login, register, OTP, session |
| `lib/models/order_model.dart` | Struktur data pesanan + enum status |
| `lib/core/config/env_config.dart` | Baca API key dari .env |
| `lib/services/static_database.dart` | Data dummy fallback |

---

## 7. 10 PERTANYAAN UJIAN PALING MUNGKIN

**Q1: State management apa yang digunakan?**
→ **Provider** dengan `ChangeNotifier`. Semua state di `AppProvider`. Saat data berubah, panggil `notifyListeners()` → widget yang pakai `context.watch()` otomatis rebuild.

**Q2: Apa bedanya `context.watch` dan `context.read`?**
→ `watch` = subscribe, widget rebuild otomatis. `read` = baca sekali, tidak rebuild. `read` dipakai di fungsi tombol.

**Q3: Backend apa yang digunakan?**
→ **Supabase** — menyediakan Auth (OTP email), Database (PostgreSQL), dan Storage (gambar).

**Q4: Bagaimana sistem multi-role bekerja?**
→ `UserModel` punya field `role` (enum: customer/admin/cashier). Di `SplashScreen`, setelah data dimuat, cek role → navigate ke halaman yang sesuai.

**Q5: Apakah pembayaran nyata atau simulasi?**
→ **Simulasi.** Pembayaran cash di kasir. Customer tunjukkan QR → kasir scan/input kode → konfirmasi → status order berubah `pending` → `confirmed`.

**Q6: Bagaimana OTP registrasi bekerja?**
→ User daftar → Supabase Auth kirim OTP 6 digit ke email → user input OTP di app → `verifyOtp()` → akun aktif → masuk ke app.

**Q7: Bagaimana cart disimpan?**
→ Di **SharedPreferences** sebagai JSON. Disimpan tiap ada perubahan (`_saveCart()`), dimuat ulang saat app start (`_loadCart()`).

**Q8: Package apa untuk QR Code?**
→ **Generate QR:** `qr_flutter`. **Scan QR (kamera):** `mobile_scanner`.

**Q9: Apa fungsi Google Gemini di aplikasi ini?**
→ **AI Insight** di dashboard admin. Data penjualan dikirim ke Gemini API lewat HTTP. Gemini membalas analisis bisnis (ringkasan, rekomendasi, prediksi) dalam Bahasa Indonesia.

**Q10: Bagaimana jika Supabase tidak dikonfigurasi?**
→ App tetap berjalan. Fallback ke **SharedPreferences** (data lokal), lalu ke **StaticDatabase** (data dummy hardcoded di `static_database.dart`).

---

> ☕ *SeedyCoffee — Proyek Akhir PAB | Flutter + Supabase + Provider*
