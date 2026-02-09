# User Manual
# Accurate Enterprise 5 - Clone System

## Document Information
- **Project Name**: Accurate Enterprise 5 Clone
- **Version**: 1.0
- **Date**: February 2026
- **Audience**: End Users, Accountants, Managers

---

## TABLE OF CONTENTS

1. [Introduction](#1-introduction)
2. [Getting Started](#2-getting-started)
3. [Dashboard](#3-dashboard)
4. [Chart of Accounts](#4-chart-of-accounts)
5. [General Ledger](#5-general-ledger)
6. [Accounts Receivable](#6-accounts-receivable)
7. [Accounts Payable](#7-accounts-payable)
8. [Inventory Management](#8-inventory-management)
9. [Sales Management](#9-sales-management)
10. [Purchase Management](#10-purchase-management)
11. [Reports](#11-reports)
12. [Administration](#12-administration)

---

## 1. INTRODUCTION

### 1.1 Tentang Sistem

Accurate Enterprise 5 adalah sistem ERP (Enterprise Resource Planning) berbasis web yang dirancang untuk mengelola seluruh aspek bisnis Anda, meliputi:

- Akuntansi dan Keuangan
- Manajemen Persediaan (Inventory)
- Penjualan dan Piutang
- Pembelian dan Hutang
- Pelaporan Keuangan

### 1.2 Kebutuhan Sistem

**Browser yang Didukung**:
- Google Chrome (versi terbaru)
- Mozilla Firefox (versi terbaru)
- Microsoft Edge (versi terbaru)
- Safari (versi terbaru)

**Koneksi Internet**: Minimal 2 Mbps

**Resolusi Layar**: Minimal 1366 x 768 pixels

### 1.3 Konvensi dalam Manual

- **Menu > Submenu**: Menunjukkan navigasi menu
- `Tombol`: Tombol yang dapat diklik
- **Bold**: Istilah penting
- *Italic*: Catatan khusus

---

## 2. GETTING STARTED

### 2.1 Login ke Sistem

1. Buka browser dan akses URL: `https://erp.yourcompany.com`
2. Masukkan **Username** dan **Password** Anda
3. Klik tombol `Login`

![Login Screen](images/login-screen.png)

**Catatan**: Jika Anda lupa password, klik link "Forgot Password?" dan ikuti instruksi yang dikirim ke email Anda.

### 2.2 Memilih Perusahaan

Jika Anda memiliki akses ke lebih dari satu perusahaan:

1. Setelah login, Anda akan melihat daftar perusahaan
2. Klik perusahaan yang ingin Anda akses
3. Sistem akan membuka dashboard perusahaan tersebut

### 2.3 Antarmuka Utama

**Komponen Utama**:

1. **Header Bar**: Logo, nama perusahaan, notifikasi, profil user
2. **Sidebar Menu**: Menu navigasi utama
3. **Content Area**: Area kerja utama
4. **Footer**: Informasi versi dan copyright

### 2.4 Navigasi

**Menggunakan Menu**:
- Klik menu di sidebar untuk membuka submenu
- Submenu akan menampilkan fungsi-fungsi yang tersedia

**Breadcrumb**:
- Menunjukkan lokasi Anda dalam sistem
- Klik breadcrumb untuk kembali ke halaman sebelumnya

**Search**:
- Gunakan search bar di header untuk mencari transaksi, customer, atau item
- Ketik minimal 3 karakter dan tekan Enter

---

## 3. DASHBOARD

### 3.1 Overview Dashboard

Dashboard memberikan ringkasan kondisi keuangan dan operasional perusahaan Anda.

**Informasi yang Ditampilkan**:
- Total Penjualan (bulan ini)
- Total Pembelian (bulan ini)
- Piutang yang Jatuh Tempo
- Hutang yang Jatuh Tempo
- Saldo Bank
- Stok Minimum (alert)

### 3.2 Widgets Dashboard

#### Sales Chart
Menampilkan grafik penjualan 6 bulan terakhir

#### Top Customers
Menampilkan 10 pelanggan dengan pembelian terbesar

#### Recent Transactions
Menampilkan 10 transaksi terakhir

#### Cash Flow
Menampilkan arus kas masuk dan keluar

### 3.3 Kustomisasi Dashboard

1. Klik icon `⚙️ Settings` di pojok kanan atas dashboard
2. Pilih widget yang ingin ditampilkan
3. Atur posisi widget dengan drag & drop
4. Klik `Save` untuk menyimpan konfigurasi

---

## 4. CHART OF ACCOUNTS

### 4.1 Tentang Chart of Accounts

Chart of Accounts (COA) adalah daftar semua akun yang digunakan untuk mencatat transaksi keuangan perusahaan.

**Tipe Akun**:
- **ASSET**: Harta (Kas, Bank, Piutang, Persediaan, Aset Tetap)
- **LIABILITY**: Kewajiban (Hutang, Hutang Pajak)
- **EQUITY**: Modal
- **REVENUE**: Pendapatan
- **EXPENSE**: Beban/Biaya

### 4.2 Melihat Chart of Accounts

**Menu**: Accounting > Chart of Accounts

Anda akan melihat daftar akun dalam bentuk hierarki (tree structure).

**Kolom yang Ditampilkan**:
- Kode Akun
- Nama Akun
- Tipe Akun
- Saldo Akun
- Status (Aktif/Tidak Aktif)

### 4.3 Membuat Akun Baru

1. **Menu**: Accounting > Chart of Accounts
2. Klik tombol `+ New Account`
3. Isi formulir:
   - **Account Code**: Kode akun (contoh: 1-10100)
   - **Account Name**: Nama akun (contoh: Bank BCA)
   - **Account Type**: Pilih tipe akun
   - **Parent Account**: Pilih akun induk (opsional)
   - **Normal Balance**: DEBIT atau CREDIT
   - **Currency**: Pilih mata uang (default: IDR)
4. Klik `Save`

**Best Practices**:
- Gunakan kode akun yang konsisten
- Kode level 1: 1 digit (contoh: 1)
- Kode level 2: 2 digit (contoh: 1-10)
- Kode level 3: 5 digit (contoh: 1-10100)

### 4.4 Mengedit Akun

1. Cari akun yang ingin diedit
2. Klik icon `✏️ Edit`
3. Ubah informasi yang diperlukan
4. Klik `Save`

**Catatan**: Akun yang sudah memiliki transaksi tidak dapat dihapus, hanya dapat dinonaktifkan.

### 4.5 Menonaktifkan Akun

1. Cari akun yang ingin dinonaktifkan
2. Klik icon `⚙️ Actions`
3. Pilih `Deactivate`
4. Konfirmasi tindakan

---

## 5. GENERAL LEDGER

### 5.1 Tentang General Ledger

General Ledger (Buku Besar) adalah catatan utama semua transaksi keuangan yang terjadi di perusahaan.

### 5.2 Membuat Jurnal Umum

**Menu**: Accounting > Journals > General Journal

1. Klik tombol `+ New Journal`
2. Isi informasi header:
   - **Journal Date**: Tanggal jurnal
   - **Description**: Deskripsi jurnal
3. Isi detail jurnal (minimal 2 baris):

   **Baris 1 (Debit)**:
   - Account: Pilih akun
   - Description: Deskripsi detail
   - Debit Amount: Masukkan jumlah

   **Baris 2 (Credit)**:
   - Account: Pilih akun
   - Description: Deskripsi detail
   - Credit Amount: Masukkan jumlah

4. Pastikan Total Debit = Total Credit
5. Klik `Save as Draft`

**Contoh Jurnal Penyusutan**:
```
Tanggal: 28 Februari 2026
Deskripsi: Penyusutan Peralatan Bulan Februari

Debit:  Beban Penyusutan Peralatan    Rp 5.000.000
Credit: Akumulasi Penyusutan          Rp 5.000.000
```

### 5.3 Posting Jurnal

Jurnal yang sudah disimpan dengan status "Draft" harus di-posting agar masuk ke buku besar.

1. Buka jurnal yang akan di-posting
2. Review detail jurnal
3. Klik tombol `Post`
4. Konfirmasi posting

**Catatan**: Jurnal yang sudah di-posting tidak dapat diubah. Untuk koreksi, buat jurnal pembalik.

### 5.4 Membatalkan Jurnal

Untuk membatalkan jurnal yang sudah di-posting:

1. Buka jurnal yang akan dibatalkan
2. Klik tombol `Void`
3. Masukkan alasan pembatalan
4. Konfirmasi

Sistem akan membuat jurnal pembalik secara otomatis.

### 5.5 Melihat Buku Besar

**Menu**: Accounting > Reports > General Ledger

1. Pilih **Account** yang ingin dilihat
2. Pilih **Date Range** (dari - sampai)
3. Klik `Generate Report`

Report akan menampilkan:
- Saldo Awal
- Daftar transaksi dengan debit/credit
- Saldo Akhir

---

## 6. ACCOUNTS RECEIVABLE

### 6.1 Tentang Piutang

Accounts Receivable (Piutang) adalah jumlah uang yang harus dibayar oleh pelanggan kepada perusahaan.

### 6.2 Mengelola Customer

#### Membuat Customer Baru

**Menu**: Sales > Customers

1. Klik `+ New Customer`
2. Isi informasi customer:
   - **Customer Code**: Kode customer (auto-generate atau manual)
   - **Customer Name**: Nama customer
   - **Tax ID**: NPWP (untuk perusahaan)
   - **Email**: Email customer
   - **Phone**: Nomor telepon
   - **Address**: Alamat lengkap
   - **Customer Type**: Individual atau Company
   - **Payment Term**: Termin pembayaran (contoh: Net 30 days)
   - **Credit Limit**: Limit kredit
3. Klik `Save`

#### Melihat Informasi Customer

1. **Menu**: Sales > Customers
2. Cari customer yang ingin dilihat
3. Klik nama customer untuk melihat detail

**Informasi yang Ditampilkan**:
- Data customer
- Outstanding invoices
- Payment history
- Credit limit dan available credit

### 6.3 Membuat Sales Invoice

**Menu**: Sales > Invoices

1. Klik `+ New Invoice`
2. Isi Header Invoice:
   - **Customer**: Pilih customer
   - **Invoice Date**: Tanggal invoice
   - **Due Date**: Tanggal jatuh tempo (auto-fill berdasarkan payment term)
   - **Customer PO Number**: Nomor PO dari customer (opsional)
3. Tambah Item Invoice:
   - Klik `Add Item`
   - **Item**: Pilih item dari dropdown
   - **Description**: Deskripsi (auto-fill dari master item)
   - **Quantity**: Jumlah
   - **Unit Price**: Harga satuan (auto-fill dari master item)
   - **Discount %**: Diskon dalam persen (opsional)
   - **Tax**: PPN 11% (otomatis)
4. Sistem akan menghitung:
   - Subtotal
   - Discount Amount
   - Tax Amount
   - Total Amount
5. Isi Notes (opsional)
6. Klik `Save as Draft`

### 6.4 Approve dan Send Invoice

#### Approve Invoice

1. Buka invoice yang sudah dibuat
2. Review detail invoice
3. Klik tombol `Approve`

#### Send Invoice ke Customer

1. Buka invoice yang sudah di-approve
2. Klik tombol `Send`
3. Isi:
   - **Recipient Email**: Email tujuan (auto-fill dari customer email)
   - **CC**: Email tambahan (opsional)
   - **Subject**: Subject email
   - **Message**: Pesan tambahan
4. Klik `Send Email`

Invoice akan dikirim dalam format PDF ke email customer.

### 6.5 Mencatat Pembayaran dari Customer

**Menu**: Sales > Payments

1. Klik `+ New Payment`
2. Isi informasi payment:
   - **Customer**: Pilih customer
   - **Receipt Date**: Tanggal terima pembayaran
   - **Payment Method**: Pilih metode (Bank Transfer, Cash, Check, dll)
   - **Payment Amount**: Jumlah pembayaran
   - **Bank Account**: Pilih rekening bank (jika bank transfer)
   - **Reference Number**: Nomor referensi transfer
3. Alokasikan pembayaran ke invoice:
   - Sistem akan menampilkan outstanding invoices
   - Centang invoice yang dibayar
   - Masukkan jumlah pembayaran untuk setiap invoice
4. Klik `Save`

**Catatan**:
- Jika pembayaran lebih besar dari invoice, sisanya akan menjadi advance payment (uang muka)
- Advance payment dapat dialokasikan ke invoice berikutnya

### 6.6 Laporan Piutang

#### AR Aging Report

**Menu**: Sales > Reports > AR Aging

Menampilkan umur piutang berdasarkan jatuh tempo:
- Current (belum jatuh tempo)
- 1-30 days overdue
- 31-60 days overdue
- 61-90 days overdue
- Over 90 days overdue

#### Customer Statement

**Menu**: Sales > Reports > Customer Statement

1. Pilih **Customer**
2. Pilih **Date Range**
3. Klik `Generate`

Menampilkan:
- Saldo awal
- Semua transaksi (invoice dan payment)
- Saldo akhir

---

## 7. ACCOUNTS PAYABLE

### 7.1 Tentang Hutang

Accounts Payable (Hutang) adalah jumlah uang yang harus dibayar oleh perusahaan kepada supplier.

### 7.2 Mengelola Vendor

#### Membuat Vendor Baru

**Menu**: Purchase > Vendors

1. Klik `+ New Vendor`
2. Isi informasi vendor:
   - **Vendor Code**: Kode vendor
   - **Vendor Name**: Nama vendor
   - **Tax ID**: NPWP vendor
   - **Email**: Email vendor
   - **Phone**: Nomor telepon
   - **Address**: Alamat lengkap
   - **Payment Term**: Termin pembayaran
3. Klik `Save`

### 7.3 Membuat Purchase Invoice

**Menu**: Purchase > Invoices

1. Klik `+ New Invoice`
2. Isi Header:
   - **Vendor**: Pilih vendor
   - **Invoice Date**: Tanggal invoice vendor
   - **Due Date**: Tanggal jatuh tempo
   - **Vendor Invoice Number**: Nomor invoice dari vendor
3. Tambah Item:
   - **Item**: Pilih item
   - **Quantity**: Jumlah
   - **Unit Price**: Harga satuan
   - **Tax**: PPN (Input) 11%
4. Klik `Save`

### 7.4 Membuat Pembayaran ke Vendor

**Menu**: Purchase > Payments

1. Klik `+ New Payment`
2. Isi informasi payment:
   - **Vendor**: Pilih vendor
   - **Payment Date**: Tanggal pembayaran
   - **Payment Method**: Pilih metode
   - **Payment Amount**: Jumlah pembayaran
   - **Bank Account**: Pilih rekening bank
3. Alokasikan pembayaran ke invoice
4. Klik `Save and Post`

### 7.5 Laporan Hutang

#### AP Aging Report

**Menu**: Purchase > Reports > AP Aging

Menampilkan umur hutang berdasarkan jatuh tempo.

---

## 8. INVENTORY MANAGEMENT

### 8.1 Tentang Inventory

Inventory Management mengelola persediaan barang di gudang.

### 8.2 Membuat Item Baru

**Menu**: Inventory > Items

1. Klik `+ New Item`
2. Isi informasi item:

   **Basic Information**:
   - **SKU**: Kode item (Stock Keeping Unit)
   - **Item Name**: Nama item
   - **Description**: Deskripsi lengkap
   - **Item Type**: Stock, Non-Stock, atau Service
   - **Category**: Kategori item
   - **Base Unit**: Satuan dasar (PCS, KG, M, dll)

   **Barcode**:
   - **Barcode**: Kode barcode
   - **Barcode Type**: EAN13, CODE128, dll

   **Pricing**:
   - **Purchase Price**: Harga beli
   - **Selling Price**: Harga jual

   **Inventory**:
   - **Track Inventory**: Ya/Tidak
   - **Cost Method**: FIFO, Average, atau Standard
   - **Reorder Point**: Titik pemesanan ulang
   - **Minimum Stock**: Stok minimum

3. Upload gambar produk (opsional)
4. Klik `Save`

### 8.3 Stock Adjustment

Untuk menyesuaikan stok aktual dengan sistem:

**Menu**: Inventory > Stock Adjustment

1. Klik `+ New Adjustment`
2. Pilih **Warehouse**
3. Pilih **Adjustment Date**
4. Tambah Item:
   - **Item**: Pilih item
   - **System Quantity**: Stok di sistem (auto-fill)
   - **Physical Quantity**: Stok aktual (hasil stock opname)
   - **Difference**: Selisih (auto-calculate)
   - **Notes**: Alasan penyesuaian
5. Klik `Save and Post`

### 8.4 Stock Transfer

Untuk memindahkan stok antar gudang:

**Menu**: Inventory > Stock Transfer

1. Klik `+ New Transfer`
2. Pilih **From Warehouse** (gudang asal)
3. Pilih **To Warehouse** (gudang tujuan)
4. Pilih **Transfer Date**
5. Tambah Item yang akan dipindahkan
6. Klik `Save and Post`

### 8.5 Laporan Inventory

#### Stock on Hand Report

**Menu**: Inventory > Reports > Stock on Hand

Menampilkan stok yang tersedia di setiap gudang.

#### Stock Movement Report

**Menu**: Inventory > Reports > Stock Movement

Menampilkan pergerakan stok (masuk/keluar) dalam periode tertentu.

#### Slow Moving Items

**Menu**: Inventory > Reports > Slow Moving Items

Menampilkan item yang pergerakan nya lambat (jarang terjual).

---

## 9. SALES MANAGEMENT

### 9.1 Sales Quotation

Membuat penawaran harga kepada customer.

**Menu**: Sales > Quotations

1. Klik `+ New Quotation`
2. Isi informasi quotation (mirip dengan invoice)
3. Set **Valid Until** (tanggal berlaku penawaran)
4. Klik `Save`
5. Klik `Send` untuk mengirim ke customer

#### Convert Quotation to Sales Order

1. Buka quotation yang sudah di-approve customer
2. Klik `Convert to Sales Order`

### 9.2 Sales Order

Pesanan dari customer yang harus dipenuhi.

**Menu**: Sales > Sales Orders

#### Membuat Sales Order Baru

1. Klik `+ New Sales Order`
2. Isi informasi order (sama seperti invoice)
3. Klik `Save`
4. Klik `Approve`

### 9.3 Delivery Order

Surat jalan untuk pengiriman barang ke customer.

**Menu**: Sales > Delivery Orders

1. Klik `+ New Delivery Order`
2. Pilih **Sales Order**
3. Sistem akan auto-fill item dari sales order
4. Ubah quantity jika partial delivery
5. Pilih **Warehouse**
6. Klik `Save and Post`

Sistem akan mengurangi stok di warehouse secara otomatis.

### 9.4 Create Invoice from Sales Order

1. Buka **Sales Order** yang sudah di-deliver
2. Klik `Create Invoice`
3. Review dan edit jika perlu
4. Klik `Save`

---

## 10. PURCHASE MANAGEMENT

### 10.1 Purchase Requisition

Permintaan pembelian dari departemen.

**Menu**: Purchase > Requisitions

1. Klik `+ New Requisition`
2. Isi:
   - **Department**: Departemen peminta
   - **Required Date**: Tanggal dibutuhkan
3. Tambah item yang dibutuhkan
4. Klik `Save and Submit`
5. Requisition akan masuk approval workflow

### 10.2 Purchase Order

Order pembelian ke supplier.

**Menu**: Purchase > Purchase Orders

1. Klik `+ New Purchase Order`
2. Atau convert dari Purchase Requisition
3. Isi informasi order
4. Klik `Save and Approve`
5. Klik `Send` untuk mengirim PO ke supplier

### 10.3 Goods Receipt

Penerimaan barang dari supplier.

**Menu**: Purchase > Goods Receipt

1. Klik `+ New Goods Receipt`
2. Pilih **Purchase Order**
3. Verify quantity yang diterima
4. Pilih **Warehouse** tujuan
5. Klik `Save and Post`

Sistem akan menambah stok di warehouse.

### 10.4 Create Invoice from Purchase Order

1. Buka **Purchase Order** yang sudah di-receipt
2. Klik `Create Invoice`
3. Match dengan invoice dari supplier
4. Klik `Save`

---

## 11. REPORTS

### 11.1 Financial Reports

#### Balance Sheet (Neraca)

**Menu**: Reports > Financial > Balance Sheet

Menampilkan posisi keuangan (Harta, Hutang, Modal) pada tanggal tertentu.

#### Profit & Loss (Laba Rugi)

**Menu**: Reports > Financial > Profit & Loss

Menampilkan pendapatan dan beban dalam periode tertentu, dan hasil laba/rugi.

#### Cash Flow Statement

**Menu**: Reports > Financial > Cash Flow

Menampilkan arus kas masuk dan keluar.

#### Trial Balance

**Menu**: Reports > Financial > Trial Balance

Menampilkan saldo debit dan credit semua akun.

### 11.2 Tax Reports

#### PPN Report

**Menu**: Reports > Tax > PPN Report

Menampilkan:
- PPN Keluaran (dari penjualan)
- PPN Masukan (dari pembelian)
- PPN yang harus dibayar/lebih bayar

#### Export e-Faktur

**Menu**: Reports > Tax > e-Faktur Export

Export data dalam format yang bisa di-import ke aplikasi e-Faktur.

### 11.3 Custom Reports

**Menu**: Reports > Custom Reports

Buat laporan custom sesuai kebutuhan:

1. Klik `+ New Report`
2. Pilih **Data Source** (table/view)
3. Pilih **Columns** yang ingin ditampilkan
4. Set **Filters** dan **Sorting**
5. Klik `Save Report`
6. Klik `Generate` untuk melihat hasil

---

## 12. ADMINISTRATION

### 12.1 User Management

**Menu**: Settings > Users

#### Membuat User Baru

1. Klik `+ New User`
2. Isi:
   - **Username**: Unique username
   - **Email**: Email address
   - **Full Name**: Nama lengkap
   - **Role**: Pilih role (Admin, Manager, User, dll)
3. Set **Password**
4. Klik `Save`

Email aktivasi akan dikirim ke user.

#### Mengatur Permission

1. **Menu**: Settings > Roles & Permissions
2. Pilih **Role**
3. Centang permission yang diinginkan:
   - Read
   - Create
   - Update
   - Delete
4. Klik `Save`

### 12.2 Company Settings

**Menu**: Settings > Company

Mengatur:
- Company Profile
- Fiscal Year
- Default Currency
- Number Format
- Date Format
- Document Numbering

### 12.3 Integration Settings

#### Payment Gateway

**Menu**: Settings > Integrations > Payment Gateway

Setup payment gateway (Midtrans, Xendit, dll) untuk menerima pembayaran online.

#### E-Commerce Integration

**Menu**: Settings > Integrations > E-Commerce

Sinkronisasi dengan platform e-commerce (Shopify, WooCommerce, dll).

---

## 13. TIPS & BEST PRACTICES

### 13.1 Backup Data

Lakukan backup data secara rutin:
- **Menu**: Settings > Backup
- Set schedule automatic backup
- Download backup manual jika perlu

### 13.2 Period Closing

Tutup periode setiap akhir bulan:
- **Menu**: Accounting > Period Closing
- Review semua transaksi
- Rekonsiliasi bank
- Tutup periode

Period yang sudah ditutup tidak bisa diubah (untuk menjaga integritas data).

### 13.3 Reconciliation

Lakukan rekonsiliasi bank setiap bulan:
- **Menu**: Accounting > Bank Reconciliation
- Match transaksi di sistem dengan statement bank
- Catat perbedaan jika ada

### 13.4 Stock Opname

Lakukan stock opname minimal 1 bulan sekali:
- Hitung fisik stok di gudang
- Bandingkan dengan sistem
- Buat stock adjustment untuk perbedaan

### 13.5 Review Reports

Review laporan secara berkala:
- Balance Sheet (bulanan)
- Profit & Loss (bulanan)
- AR Aging (mingguan)
- AP Aging (mingguan)
- Slow Moving Items (bulanan)

---

## 14. FREQUENTLY ASKED QUESTIONS (FAQ)

**Q: Bagaimana cara mengubah password?**

A: Menu > User Profile > Change Password

**Q: Bagaimana jika lupa password?**

A: Klik "Forgot Password" di halaman login dan ikuti instruksi di email.

**Q: Bagaimana cara membatalkan invoice yang salah?**

A: Buka invoice > Klik tombol "Void" > Masukkan alasan pembatalan

**Q: Apakah bisa menggunakan multi currency?**

A: Ya, sistem mendukung multi currency. Set currency saat membuat transaksi.

**Q: Bagaimana cara export data ke Excel?**

A: Di setiap laporan ada tombol "Export" > Pilih format Excel

**Q: Bagaimana cara print invoice?**

A: Buka invoice > Klik tombol "Print" atau "Download PDF"

**Q: Apakah bisa akses dari mobile?**

A: Ya, sistem responsive dan dapat diakses dari mobile browser atau download mobile app.

---

## 15. GETTING HELP

### 15.1 Support Contact

- **Email**: support@yourcompany.com
- **Phone**: +62-21-1234567
- **WhatsApp**: +62-812-3456-7890
- **Support Hours**: Senin-Jumat, 09:00 - 17:00 WIB

### 15.2 In-App Help

- Klik icon `?` Help di pojok kanan atas
- Gunakan fitur Search untuk mencari topik
- Video tutorials tersedia di Help Center

### 15.3 Training

Hubungi tim support untuk:
- On-site training
- Online training
- User workshop

---

## APPENDICES

### Appendix A: Keyboard Shortcuts
### Appendix B: Error Messages
### Appendix C: Glossary
### Appendix D: Sample Reports

---

**Document Version**: 1.0
**Last Updated**: February 2026
**Feedback**: documentation@yourcompany.com
