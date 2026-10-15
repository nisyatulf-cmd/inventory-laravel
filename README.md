# Inventory Management System (Laravel 12 - Mini Project 4)

Aplikasi **Inventory Management System** yang telah dimigrasikan dan di-refactor secara penuh dari **PHP Native (Pertemuan 4)** ke framework **Laravel (Pertemuan 5: Laravel Fundamental)** berdasarkan dokumen silabus [Web 5.pptx](file:///D:/punya-nisyatul/mini-project-4/Web%205.pptx).

---

## 🌟 Transformasi & Kesesuaian Kriteria Modul (Slide 35, 40, & 41)

| Aspek | P4 PHP Native | P5 Laravel Refactoring |
| :--- | :--- | :--- |
| **Database DDL** | `database/schema.sql` manual | `database/migrations/` (Schema Blueprint & FK constraint) |
| **Data Dummy** | Manual INSERT SQL | `database/seeders/` (`UserSeeder`, `CategorySeeder`, `SupplierSeeder`, `ProductSeeder`) |
| **Query & Relasi** | PDO Prepared Statements manual | **Eloquent ORM** (`hasMany`, `belongsTo`, `belongsToMany` via junction) |
| **Arsitektur** | File CRUD terpisah | **MVC Pattern** (`routes/web.php` $\rightarrow$ Resource Controller $\rightarrow$ Model $\rightarrow$ Blade) |
| **Autentikasi & RBAC**| Manual session & `password_verify` | **Laravel Auth** (`Auth::attempt`, Session regenerate, middleware `role:admin`) |
| **User Interface** | Include PHP manual | **Blade Template Engine** (`@extends`, `@section`, `@forelse`, form error validation) |
| **Business Logic** | ACID Transaction manual | `DB::transaction()` & `Product::lockForUpdate()` (Anti Stok Minus) |

---

## 🗂️ Struktur Proyek (Sesuai Slide 36)

```text
inventory-laravel/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── AuthController.php            # Login & Logout dengan session regenerate
│   │   │   ├── DashboardController.php       # Agregat metrik stok & eager loading
│   │   │   ├── ProductController.php         # Resource Controller CRUD produk (7 methods)
│   │   │   ├── CategoryController.php        # Resource Controller master kategori (1:N)
│   │   │   ├── SupplierController.php        # Resource Controller master supplier (N:M)
│   │   │   └── StockMovementController.php   # Transaksi barang masuk/keluar & laporan agregasi
│   │   └── Middleware/
│   │       └── EnsureUserRole.php            # Otorisasi role:admin vs staff (403 Forbidden)
│   └── Models/
│       ├── User.php                          # Model pengguna & autentikasi
│       ├── Category.php                      # Model Kategori (hasMany ke Product)
│       ├── Supplier.php                      # Model Supplier (belongsToMany ke Product)
│       ├── Product.php                       # Model Produk (belongsTo Category, belongsToMany Supplier)
│       └── StockMovement.php                 # Model Histori Mutasi Stok (belongsTo Product & User)
├── database/
│   ├── migrations/                           # Migrasi database relasional
│   └── seeders/                              # Seeder data uji otomatis
├── resources/
│   └── views/
│       ├── layouts/app.blade.php             # Master layout Dark Teal responsif
│       ├── auth/login.blade.php              # Halaman login
│       ├── dashboard.blade.php               # Dashboard metrik & ringkasan stok
│       ├── products/                         # Blade views: index, create, edit, show
│       ├── categories/index.blade.php        # Master data kategori
│       ├── suppliers/index.blade.php         # Master data supplier
│       └── stock/                            # Blade views: in, out, history
└── routes/web.php                            # Routing resource, named routes & middleware
```

---

## 🚀 Panduan Instalasi & Menjalankan Proyek (Slide 48)

### 1. Masuk ke Direktori Proyek
```bash
cd D:\punya-nisyatul\mini-project-4\inventory-laravel
```

### 2. Konfigurasi Database (.env)
Pastikan konfigurasi `.env` telah mengarah ke MySQL:
```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=inventory_db
DB_USERNAME=root
DB_PASSWORD=
```

### 3. Eksekusi Migration & Seeding (Slide 41)
Jalankan perintah berikut untuk menginisialisasi seluruh tabel dan data pengujian awal:
```bash
php artisan migrate:fresh --seed
```

### 4. Menjalankan Server Aplikasi
Jalankan server pengembangan Laravel Artisan:
```bash
php artisan serve --port=8080
```
Buka peramban di: **[http://localhost:8080](http://localhost:8080)**

---

## 🔑 Kredensial Akun Pengujian (Seeder)
* **Akun Administrator (Akses Penuh):**
  * **Username:** `admin`
  * **Password:** `admin123`
  * *Hak Akses:* Dashboard, CRUD Produk, Kategori, Supplier, Transaksi Stok Masuk & Keluar.
* **Akun Staff (Operasional):**
  * **Username:** `staff`
  * **Password:** `staff123`
  * *Hak Akses:* Dashboard, Katalog Produk, Transaksi Stok Masuk & Keluar (Dilarang mengelola kategori/supplier - dilindungi middleware `role`).

---

## 🛡️ Verifikasi Business Rule P4 yang Dipertahankan
1. **Unik Kode SKU**: Divalidasi via Form Request Validation Rule `unique:products,sku`.
2. **Anti Stok Negatif**: Transaksi Barang Keluar ([StockMovementController::storeOut](file:///D:/punya-nisyatul/mini-project-4/inventory-laravel/app/Http/Controllers/StockMovementController.php#L68-L108)) dibungkus dalam `DB::transaction()` dengan `Product::lockForUpdate()`. Jika `quantity > stock`, transaksi di-abort dan di-rollback otomatis.
3. **Integritas Histori**: Produk yang sudah memiliki rekaman mutasi stok pada tabel `stock_movements` tidak dapat dihapus sembarangan.
4. **Listing Data Lanjutan**: Fitur pencarian nama/SKU, filter kategori, pengurutan kolom via allowlist, dan paginasi `withQueryString()` tetap berfungsi mulus.
