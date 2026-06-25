# 📦 E-Inventory — Sistem Manajemen Inventaris Barang

**Nama:** Aziz Tri Ramadhan  
**NIM:** 312410380  
**Mata Kuliah:** Pemrograman Web 2  
**Dosen:** Agung Nugroho, S.Kom., M.Kom.  
**Universitas Pelita Bangsa**

---

## 🏗️ Arsitektur Sistem

```
Browser (Vue SPA)  ←──Axios/JSON──→  CI4 API Server  ←──→  MySQL DB
   Port 5500                              Port 8080
```

## 📁 Struktur Repositori

```
e-inventory/
├── backend-api/                  ← Semua file CI4 (copy dari folder ci4/)
│   └── app/
│       ├── Config/
│       │   ├── Filters.php       ← ⭐ CORS global + AuthFilter per-route
│       │   └── Routes.php        ← Semua API endpoint
│       ├── Controllers/
│       │   ├── AuthController.php    ← POST /api/auth/login
│       │   ├── BarangController.php  ← CRUD RESTful Resource
│       │   ├── KategoriController.php
│       │   ├── SupplierController.php
│       │   └── StatistikController.php
│       ├── Models/
│       │   ├── BarangModel.php       ← JOIN query + validasi
│       │   ├── KategoriModel.php
│       │   ├── SupplierModel.php
│       │   └── UserModel.php
│       ├── Filters/
│       │   ├── AuthFilter.php        ← ⭐ Validasi Bearer Token
│       │   └── CorsFilter.php        ← ⭐ CORS headers
│       └── Database/
│           ├── Migrations/
│           │   └── ..._CreateInventarisTables.php
│           └── Seeds/
│               └── InventarisSeeder.php
│
└── frontend-spa/                 ← Frontend Vue SPA
    ├── index.html                ← ⭐ Entry point, CDN imports
    ├── app.js                    ← ⭐ Router + Axios Interceptors
    └── components/
        ├── Home.js               ← Landing page publik
        ├── Login.js              ← Form autentikasi
        ├── Dashboard.js          ← Ringkasan statistik
        └── Barang.js             ← CRUD tabel barang + modal
```

---

## 🚀 Panduan Setup & Instalasi

### LANGKAH 1 — Siapkan Backend CI4

**1a. Copy file proyek**
```
Salin isi folder `backend-api/app/` ke dalam proyek CI4 Anda,
menimpa file yang sudah ada (terutama Config/Filters.php & Routes.php)
```

**1b. Konfigurasi `.env`**
```env
CI_ENVIRONMENT = development
database.default.hostname = localhost
database.default.database = e_inventory    ← Buat database baru ini
database.default.username = root
database.default.password =
database.default.DBDriver = MySQLi
```

**1c. Buat Database**
```sql
CREATE DATABASE e_inventory CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

**1d. Jalankan Migrasi & Seeder**
```bash
cd backend-api
php spark migrate
php spark db:seed InventarisSeeder
```

**1e. Jalankan CI4 Server**
```bash
php spark serve --port=8080
```

✅ Test: buka http://localhost:8080 → harusnya muncul JSON info API

---

### LANGKAH 2 — Jalankan Frontend SPA

**2a. Sesuaikan Base URL di `app.js`**
```javascript
// Baris ~30 di frontend-spa/app.js
axios.defaults.baseURL = 'http://localhost:8080'
```

**2b. Serve menggunakan Live Server (VS Code) atau:**
```bash
# Dengan Python
cd frontend-spa
python -m http.server 5500

# Atau dengan Node.js
npx serve . -p 5500
```

**2c. Buka Browser**
```
http://localhost:5500
```

---

## 🔐 Kredensial Default

| Field    | Value      |
|----------|------------|
| Username | `admin`    |
| Password | `admin123` |

---

## 📡 Daftar API Endpoint

| Method | Endpoint             | Auth   | Keterangan               |
|--------|----------------------|--------|--------------------------|
| GET    | `/api/statistik`     | Publik | Statistik inventaris     |
| POST   | `/api/auth/login`    | Publik | Login → dapat token      |
| GET    | `/api/barang`        | Publik | Daftar semua barang      |
| POST   | `/api/barang`        | 🔒 Token | Tambah barang baru     |
| GET    | `/api/barang/{id}`   | Publik | Detail 1 barang          |
| PUT    | `/api/barang/{id}`   | 🔒 Token | Update barang            |
| DELETE | `/api/barang/{id}`   | 🔒 Token | Hapus barang             |
| GET    | `/api/kategori`      | Publik | Daftar kategori          |
| GET    | `/api/supplier`      | Publik | Daftar supplier          |

---

## 📸 Screenshot & Dokumentasi Laporan

### 1. Form Login / Autentikasi
![Login](docs/images/image.png)
**Keterangan:** Form autentikasi pengguna dengan field email/password

---

### 2. Dashboard Utama
![Dashboard](docs/images/3.png)
**Keterangan:** Ringkasan statistik inventaris dan menu navigasi utama

---

### 3. Halaman Daftar Produk
![Produk](docs/images/4.png)
**Keterangan:** Daftar produk dengan fitur search, filter, dan pagination

---

### 4. Detail Produk & Stok
![Detail Produk](docs/images/5.png)
**Keterangan:** Tampilan detail barang dengan informasi stok dan riwayat

---

### 5. Manajemen Supplier
![Supplier](docs/images/6.png)
**Keterangan:** Daftar supplier dan vendor dengan opsi tambah/edit/hapus

---

### 6. Form Transaksi
![Transaksi](docs/images/image1.png)
**Keterangan:** Form input penerimaan dan pengeluaran barang

---

## 🎯 Penjelasan Konsep Kunci (untuk Presentasi)

### 1. AuthFilter (Bearer Token)
File: `app/Filters/AuthFilter.php`

```
Request masuk → Baca header "Authorization: Bearer <token>"
             → Decode Base64 → Cek expiry (24 jam)
             → Cari user di DB → Jika valid: lanjut ke Controller
                               → Jika tidak: balas JSON 401
```

### 2. CorsFilter (CORS)
File: `app/Filters/CorsFilter.php`

```
Browser dari port 5500 → Kirim request ke port 8080
Browser block? → Tidak! Karena CorsFilter menambahkan header:
  Access-Control-Allow-Origin: *
  Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
```

### 3. Axios Request Interceptor
File: `frontend-spa/app.js`

```javascript
// Setiap request → otomatis tambah header:
config.headers['Authorization'] = `Bearer ${localStorage.getItem('token')}`
```

### 4. Axios Response Interceptor
File: `frontend-spa/app.js`

```javascript
// Setiap response → jika status 401:
// → Hapus token → Alert "Sesi Habis" → Redirect ke /login
```

### 5. Navigation Guard (Vue Router)
File: `frontend-spa/app.js`

```javascript
router.beforeEach((to, from, next) => {
  if (to.meta.requiresAuth && !localStorage.getItem('token')) {
    next('/login')  // Blokir akses, arahkan ke login
  } else {
    next()          // Izinkan navigasi
  }
})
```

---

## �️ Tech Stack

| Komponen  | Teknologi              | Versi  |
|-----------|------------------------|--------|
| Backend   | CodeIgniter 4          | 4.x    |
| Database  | MySQL                  | 5.7+   |
| Frontend  | Vue.js 3               | 3.x    |
| Router    | Vue Router             | 4.x    |
| HTTP      | Axios                  | 1.x    |
| CSS       | TailwindCSS            | 3.x    |
