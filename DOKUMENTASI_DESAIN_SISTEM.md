# DOKUMENTASI DESAIN SISTEM — KOPKAR RSI JEMURSARI

> **Proyek:** Point of Sales & Koperasi Karyawan

---

# 1. USE CASE DIAGRAM

### 1.1 Kasir

```mermaid
graph LR
    subgraph "👤 Aktor"
        A["👤 Kasir"]
    end

    subgraph "📦 Manajemen Penjualan"
        direction TB
        UC1["Kasir / Penjualan"]
        UC2["Penjualan Kredit"]
        UC3["Penjualan Voucher"]
        UC4["Retur Penjualan"]
        UC29["Login"]
    end

    A --> UC1
    A --> UC2
    A --> UC3
    A --> UC4
    A --> UC29
```

### 1.2 Gudang

```mermaid
graph LR
    subgraph "👤 Aktor"
        G["👤 Gudang"]
    end

    subgraph "📦 Manajemen Barang"
        direction TB
        UC5["Barang Masuk"]
        UC6["Konsinyasi Masuk"]
        UC7["Retur Konsinyasi"]
        UC8["Purchase Order"]
        UC9["Ganti Unit Barang"]
        UC10["Stock Opname"]
        UC29["Login"]
    end

    G --> UC5
    G --> UC6
    G --> UC7
    G --> UC8
    G --> UC9
    G --> UC10
    G --> UC29
```

### 1.3 Keuangan

```mermaid
graph LR
    subgraph "👤 Aktor"
        K["👤 Keuangan"]
    end

    subgraph "📦 Manajemen Keuangan"
        direction TB
        UC11["Simpanan / Modal Anggota"]
        UC12["Pinjaman Anggota"]
        UC13["Pelunasan Kredit"]
        UC14["Pembayaran Pinjaman"]
        UC15["Jurnal Akuntansi"]
        UC29["Login"]
    end

    K --> UC11
    K --> UC12
    K --> UC13
    K --> UC14
    K --> UC15
    K --> UC29
```

### 1.4 Admin

```mermaid
graph LR
    subgraph "👤 Aktor"
        AD["👤 Admin"]
    end

    subgraph "📦 Master Data"
        direction TB
        UC16["Kelola Anggota"]
        UC17["Kelola Barang"]
        UC18["Kelola Supplier"]
        UC19["Kelola Unit/Gudang"]
        UC20["Kelola Satuan"]
    end

    subgraph "📦 Sistem"
        direction TB
        UC30["Manajemen Pengguna"]
        UC31["Manajemen Role &amp; Hak Akses"]
        UC32["Konfigurasi Perusahaan"]
        UC33["Manajemen Menu"]
        UC29["Login"]
    end

    AD --> UC16
    AD --> UC17
    AD --> UC18
    AD --> UC19
    AD --> UC20
    AD --> UC30
    AD --> UC31
    AD --> UC32
    AD --> UC33
    AD --> UC29
```

### 1.5 Pimpinan

```mermaid
graph LR
    subgraph "👤 Aktor"
        P["👤 Pimpinan"]
    end

    subgraph "📦 Laporan"
        direction TB
        UC21["Laporan Penjualan"]
        UC22["Laporan Stok Barang"]
        UC23["Laporan HPP"]
        UC24["Laporan Kartu Stok"]
        UC25["Laporan Kredit"]
        UC26["Laporan Potong Gaji"]
        UC27["Laporan Kasir"]
        UC28["Laporan Konsinyasi"]
        UC29["Login"]
    end

    P --> UC21
    P --> UC22
    P --> UC23
    P --> UC24
    P --> UC25
    P --> UC26
    P --> UC27
    P --> UC28
    P --> UC29
```

## 1.2 Aktor Sistem

| Aktor              | Deskripsi                                                                |
| ------------------ | ------------------------------------------------------------------------ |
| **Kasir**    | Melayani transaksi penjualan tunai, kredit, dan voucher kepada anggota   |
| **Gudang**   | Mengelola barang masuk, konsinyasi, PO, retur, dan stock opname          |
| **Keuangan** | Mengelola simpanan, pinjaman, pelunasan, dan jurnal akuntansi            |
| **Admin**    | Mengelola master data (anggota, barang, supplier) dan konfigurasi sistem |
| **Pimpinan** | Melihat seluruh laporan dan monitoring                                   |

## 1.3 Hak Akses (Role-Based Access Control)

Sistem menggunakan kontrol akses berbasis role dengan 5 permission per menu:

| Permission     | Kode          | Keterangan              |
| -------------- | ------------- | ----------------------- |
| Read           | `r`         | Melihat data (list)     |
| Create         | `c`         | Menambah data baru      |
| Update         | `u`         | Mengedit data           |
| Print          | `p`         | Mencetak laporan        |
| Delete         | `d`         | Menghapus data          |
| Approve        | `approve`   | Menyetujui transaksi    |
| Cancel Approve | `c_approve` | Membatalkan persetujuan |

---

# 2. SEQUENCE DIAGRAM

## 2.1 Sequence: Login

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant index.php
    participant DB as MySQL
  
    User->>Browser: Akses halaman login
    Browser->>index.php: GET /
    index.php->>index.php: Cek session (belum login)
    index.php-->>Browser: Tampilkan form login
  
    User->>Browser: Input username & password
    Browser->>index.php: POST (login, pwd)
    index.php->>DB: SELECT * FROM pengguna WHERE login=? AND kk=md5(?) AND del=0
    DB-->>index.php: Data pengguna
  
    alt Login Berhasil
        index.php->>index.php: Set session (login, userid, userlengkap, userrole, usercab, userpgw)
        index.php-->>Browser: Redirect ke main.php
    else Login Gagal
        index.php-->>Browser: Tampilkan error "Login Atau Kata Kunci Tidak Sesuai"
    end
```

## 2.2 Sequence: Penjualan Kasir (CRUD via Framework)

```mermaid
sequenceDiagram
    actor Kasir
    participant Browser
    participant main.php
    participant common_list.php
    participant common_entry.php
    participant common_api.php
    participant Common_CUQuery
    participant DB as MySQL

    Note over Kasir,DB: === LIST DATA ===
    Kasir->>Browser: Klik menu Kasir
    Browser->>main.php: GET main.php?act=kasir
    main.php->>DB: SELECT role_menu (cek hak akses)
    main.php->>DB: SELECT master_menu (dapatkan path)
    main.php->>common_list.php: include()
    common_list.php->>DB: SELECT form_list (konfigurasi list)
    common_list.php->>DB: SELECT sql_sel (data)
    DB-->>common_list.php: Data penjualan
    common_list.php-->>Browser: Tampilkan tabel data

    Note over Kasir,DB: === TAMBAH DATA ===
    Kasir->>Browser: Klik "Tambah"
    Browser->>main.php: GET main.php?act=kasir&tipe=1
    main.php->>common_entry.php: include()
    common_entry.php->>DB: SELECT form_entry_std (konfigurasi form)
    common_entry.php->>DB: SELECT form_attr (field-field form)
    common_entry.php->>Common_Generator: Render field
    Common_Generator-->>common_entry.php: HTML input
    common_entry.php-->>Browser: Tampilkan form entry

    Note over Kasir,DB: === SIMPAN DATA ===
    Kasir->>Browser: Isi form & klik "Simpan"
    Browser->>common_api.php: POST (data form, tipe=1)
    common_api.php->>Common_CUQuery: new Common_CUQuery($Konek, $_POST, $act, $userid)
    Common_CUQuery->>DB: SELECT table_name FROM form_entry_std
    Common_CUQuery->>DB: SELECT field_name, input_type FROM form_attr
    Common_CUQuery->>Common_CUQuery: Masking_Value() - formatting data
    Common_CUQuery-->>common_api.php: INSERT INTO ... VALUES ...
    common_api.php->>DB: Exec insert query
    DB-->>common_api.php: Success
    common_api.php-->>Browser: Redirect ke main.php

    Note over Kasir,DB: === HAPUS DATA ===
    Kasir->>Browser: Klik "hapus"
    Browser->>common_api.php: POST (id, tipe=3) via AJAX
    common_api.php->>DB: SELECT sql_del FROM form_list
    common_api.php->>DB: Exec delete query
    DB-->>common_api.php: Success
    common_api.php-->>Browser: JSON {status:1}
    Browser->>Browser: Reload halaman
```

## 2.3 Sequence: Konsinyasi Masuk (KOSY)

```mermaid
sequenceDiagram
    actor Gudang
    participant Browser
    participant api.php
    participant modul/kosy/kosy.php
    participant DB as MySQL
  
    Gudang->>Browser: Klik menu Konsinyasi
    Browser->>main.php: GET main.php?act=kosy
    main.php->>modul/kosy/kosy.php: include()
    kosy.php->>DB: SELECT data konsinyasi
    DB-->>kosy.php: List data
    kosy.php-->>Browser: Tampilkan daftar konsinyasi
  
    Gudang->>Browser: Klik Tambah
    Browser->>modul/kosy/add.php: include()
    add.php-->>Browser: Form konsinyasi masuk
  
    Gudang->>Browser: Isi no faktur, supplier, gudang, detail barang
    Browser->>api.php: POST act=kosy&tpact=add
    api.php->>modul/kosy/api_kosy.php: process add
    api_kosy.php->>DB: INSERT INTO tr_arus_brg (header)
    api_kosy.php->>DB: INSERT INTO tr_arus_brg_dtl (detail)
    DB-->>api_kosy.php: Success
    api_kosy.php-->>Browser: Redirect ke list
```

## 2.4 Sequence: Pinjaman Anggota

```mermaid
sequenceDiagram
    actor Keuangan
    participant Browser
    participant modul/keu/sp_pinjam
    participant DB as MySQL
  
    Keuangan->>Browser: Menu Keuangan > Pinjaman
    Browser->>modul/keu/sp_pinjam/add.php: include()
    add.php-->>Browser: Form pinjaman (jenis, akun debet/kredit, list anggota)
  
    Keuangan->>Browser: Pilih anggota & nominal pinjaman
    Browser->>DB: INSERT INTO keu_pinjaman (idpinjaman, idtr, jenis, idanggota, tanggal, nominal)
    Browser->>DB: INSERT INTO keu_dt_tr_acc (jurnal akuntansi)
    DB-->>Browser: Success
```

## 2.5 Sequence: Stock Opname / Restock (skrip_restock.php)

```mermaid
sequenceDiagram
    actor Admin
    participant Terminal
    participant skrip_restock.php
    participant DB as MySQL
  
    Admin->>Terminal: Jalankan skrip_restock.php
    Terminal->>skrip_restock.php: php skrip_restock.php
    skrip_restock.php->>DB: SELECT COUNT(*) FROM mt_barang
    DB-->>skrip_restock.php: jumlah_barang
  
    loop Untuk setiap barang
        skrip_restock.php->>DB: SELECT id_barang FROM mt_barang
        loop Untuk setiap unit/gudang
            skrip_restock.php->>DB: SELECT SUM(qty) FROM tr_brg_msk (barang masuk < tgl)
            skrip_restock.php->>DB: SELECT SUM(qty) FROM tr_brg_ret (retur < tgl)
            skrip_restock.php->>DB: SELECT SUM(qty) FROM tr_arus_brg_dtl (penjualan < tgl)
          
            Note over skrip_restock.php: saldo = barang_masuk - penjualan - retur
          
            skrip_restock.php->>DB: INSERT INTO tr_brg_msk (saldo sebagai stok awal)
        end
    end
  
    skrip_restock.php->>DB: DELETE data lama (cleanup)
    Note over skrip_restock.php: Hapus tr_arus_brg_dtl, tr_arus_brg, tr_brg_msk, tr_kredit, tr_voucher, tr_brg_ret yang < tgl
```

---

# 3. DOMAIN MODEL & CLASS DIAGRAM

## 3.1 Domain Model (Entity Relationship)

```mermaid
erDiagram
    mt_anggota ||--o{ tr_arus_brg : "melakukan"
    mt_unit ||--o{ tr_arus_brg : "transaksi di"
    mt_supplier ||--o{ tr_arus_brg : "supply ke"
    mt_gudang ||--o{ tr_arus_brg : "disimpan di"
    mt_barang ||--o{ tr_arus_brg_dtl : "terdapat di"
    mt_satuan ||--o{ mt_barang : "satuan"
  
    tr_arus_brg ||--o{ tr_arus_brg_dtl : "memiliki detail"
    tr_arus_brg ||--o{ tr_kredit : "menghasilkan"
    tr_arus_brg ||--o{ tr_voucher : "menggunakan"
  
    tr_brg_msk }o--|| mt_barang : "barang"
    tr_brg_msk }o--|| mt_unit : "gudang"
    tr_brg_ret }o--|| mt_barang : "barang"
    tr_brg_ret }o--|| mt_unit : "gudang"
  
    pengguna ||--o{ tr_arus_brg : "input oleh"
    pengguna }o--|| master_role : "memiliki role"
    role_menu }o--|| master_role : "hak akses"
    role_menu }o--|| master_menu : "menu"
  
    mt_anggota ||--o{ keu_pinjaman : "meminjam"
    mt_anggota ||--o{ keu_modal : "menyimpan"
    mt_anggota ||--o{ keu_sp : "simpanan/pinjaman"
  
    keu_pinjaman ||--o{ keu_pel_pinjaman : "pelunasan"
    keu_pinjaman ||--o{ keu_dt_tr_acc : "jurnal"
    keu_modal ||--o{ keu_dt_tr_acc : "jurnal"
  
    keu_tr_acc ||--o{ keu_dt_tr_acc : "detail akuntansi"
    keu_dt_tr_acc }o--|| keu_acc : "akun"
    keu_acc }o--|| keu_kel_acc : "kelompok"
    keu_kel_acc }o--|| keu_neraca : "neraca"
  
    tr_kredit ||--o{ tr_pelunasan : "dilunasi"
    mt_anggota ||--o{ tr_pelunasan : "melunasi"

    mt_anggota {
        int id_anggota PK
        string nama
        tinyint aktif
        string tempat
        date ttl
        tinyint jk
        string alamat
        date tglmsk
        tinyint jabatan
        tinyint unit
        date tglberhenti
        tinyint del
    }
  
    mt_barang {
        int id_barang PK
        string kode_barang
        string nama
        int id_satuan FK
        int sup FK
        decimal hargabeli
        decimal hargajual
        tinyint del
    }
  
    mt_unit {
        int id_unit PK
        string nama
        tinyint tipe
        tinyint del
    }
  
    mt_supplier {
        int id_supplier PK
        string nama
        string alamat
        string telp
        tinyint del
    }
  
    mt_gudang {
        int id PK
        string nama
        tinyint del
    }
  
    mt_satuan {
        int id_satuan PK
        string nama
        tinyint del
    }
  
    tr_arus_brg {
        varchar id_tr PK
        tinyint mk
        int unit FK
        int anggota FK
        int supplier FK
        int gudang FK
        datetime waktu
        decimal total
        decimal bayar
        decimal voucher
        decimal kredit
        decimal kembali
        int create_user FK
        datetime create_date
        tinyint del
    }
  
    tr_arus_brg_dtl {
        varchar id_tr_dtl PK
        varchar id_tr FK
        int id_brg FK
        decimal qty
        decimal harga
        decimal disc
        datetime create_date
        tinyint del
    }
  
    tr_brg_msk {
        varchar id PK
        varchar brg FK
        varchar gdg FK
        date tgl
        decimal qty
        int entry_user FK
        datetime entry_time
        tinyint del
    }
  
    tr_brg_ret {
        varchar id PK
        varchar brg FK
        varchar gdg FK
        date tgl
        decimal qty
        int entry_user FK
        datetime entry_time
        tinyint del
    }
  
    tr_kredit {
        varchar id_kredit PK
        varchar id_tr FK
        int anggota FK
        date tgl
        decimal nominal
        int tenor
        decimal angsuran
        tinyint lunas
        datetime create_date
        tinyint del
    }
  
    tr_pelunasan {
        varchar id_pelunasan PK
        varchar id_kredit FK
        int anggota FK
        date tgl
        decimal nominal
        datetime create_date
        tinyint del
    }
  
    tr_voucher {
        varchar id_voucher PK
        varchar id_tr FK
        varchar kode
        decimal nominal
        datetime create_date
        tinyint del
    }
  
    pengguna {
        int idpeng PK
        string login
        string kk
        string lengkap
        int role FK
        int cab FK
        int idpgw FK
        tinyint del
    }
  
    master_role {
        int id_role PK
        string desk
    }
  
    master_menu {
        int id_menu PK
        string desk
        int level
        int parent
        string form_id
        int tipe
        string path
        string auth
        int ord
        tinyint del
    }
  
    role_menu {
        int id PK
        int role FK
        varchar link
        tinyint r
        tinyint c
        tinyint u
        tinyint p
        tinyint d
        tinyint approve
        tinyint c_approve
    }
  
    keu_acc {
        int id_acc PK
        int id_kel_acc FK
        int kode
        string Deskripsi
        tinyint karakter
        tinyint del
    }
  
    keu_kel_acc {
        int id_kel_acc PK
        int id_neraca FK
        int kode
        string deskripsi
        tinyint del
    }
  
    keu_neraca {
        int id_neraca PK
        string deskripsi
        tinyint del
    }
  
    keu_tr_acc {
        varchar id_tr_keu PK
        date tgl
        string reff
        string ket
        int create_user FK
        datetime create_date
        tinyint del
    }
  
    keu_dt_tr_acc {
        varchar id_dt_tr PK
        varchar id_tr FK
        int akun FK
        tinyint dk
        decimal nominal
        datetime create_date
        tinyint del
    }
  
    keu_pinjaman {
        varchar idpinjaman PK
        varchar idtr FK
        tinyint jenis
        int idanggota FK
        date tanggal
        decimal nominal
        datetime create_date
        tinyint del
    }
  
    keu_modal {
        varchar idmodal PK
        varchar idtr FK
        tinyint jenis
        int idanggota FK
        date tanggal
        decimal nominal
        datetime create_date
        tinyint del
    }
  
    keu_sp {
        varchar id_sp PK
        varchar id_tr FK
        tinyint jenis
        tinyint hb
        varchar anggota FK
        datetime create_date
        tinyint del
    }
  
    keu_pel_pinjaman {
        varchar idpel PK
        varchar idpinjaman FK
        decimal nominal
        decimal bghs
        date tanggal
        datetime create_date
        tinyint del
    }
```

## 3.2 Class Diagram — Framework CRUD

```mermaid
classDiagram
    class Konek_Wrapper {
        -connection: PDO|mysql
        +Exec_Query_Mysql(query) Result
        +Fetch_Array(result) array
        +Jumlah_Baris_Selected(result) int
        +Jumlah_Kolom_Selected(result) int
        +Field_Name(result, index) string
        +GetError() bool
        +Close_Mysql()
    }
  
    class Common_Generator {
        -Konek: Konek_Wrapper
        -field_name: string
        -field_label: string
        -input_type: string
        -init_val: string
        -editable: string
        -size: string
        -val_type: string
        -data_statics: string
        -sql: string
        -required: string
        +Render(hidden?) string
    }
  
    class Common_CUQuery {
        -Konek: Konek_Wrapper
        -param: array
        -formid: string
        -table: string
        -hasil_attr: Result
        -user: int
        -Masking_Value(value, tipe) string
        +Insert_Data() string
        +Update_Data() string
    }
  
    class Common_CUQuery_DTL {
        -Konek: Konek_Wrapper
        -param: array
        -formid: string
        +Insert_Data() string
        +Update_Data() string
    }
  
    class Common_CUQuery_Giant {
        -Konek: Konek_Wrapper
        -param: array
        -formid: string
        +Insert_Data() string
    }
  
    class Common_StrToDec {
        +StrToDec(string) decimal
    }
  
    class form_attr {
        +form_id: string
        +field_name: string
        +field_label: string
        +input_type: int
        +init_val: string
        +editable: int
        +required: int
        +data_static: text
        +data_sql: text
    }
  
    class form_entry_std {
        +form_id: string
        +page_title: string
        +table_name: string
    }
  
    class form_list {
        +form_id: string
        +page_title: string
        +sql_sel: text
        +sql_del: text
        +va, vp, ve, vd: bool
    }
  
    class form_generator_giant {
        +tbl: string
        +field_tbl: string
        +tipe_tbl: string
        +data_sql_tbl: text
    }
  
    Konek_Wrapper <.. Common_Generator : uses
    Konek_Wrapper <.. Common_CUQuery : uses
    Konek_Wrapper <.. Common_CUQuery_DTL : uses
    Common_CUQuery <|-- Common_CUQuery_DTL : extends
    Common_CUQuery <|-- Common_CUQuery_Giant : extends
  
    form_attr -- form_entry_std : form_id
    form_entry_std .. Common_CUQuery : reads table_name
    form_attr .. Common_Generator : configures rendering
    form_list .. common_list : configures listing
    form_generator_giant .. Common_CUQuery_Giant : configures giant form
```

## 3.3 Class Diagram — Modul Bisnis

```mermaid
classDiagram
    class Pengguna {
        +idpeng: int
        +login: string
        +kk: string
        +lengkap: string
        +role: int
        +cab: int
        +idpgw: int
        +del: bool
        +Login(login, password) bool
        +Logout()
        +HasAccess(menu, permission) bool
    }
  
    class Role {
        +id_role: int
        +desk: string
        +GetPermissions(menu) array
    }
  
    class Anggota {
        +id_anggota: int
        +nama: string
        +aktif: bool
        +ttl: date
        +jk: int
        +alamat: string
        +tglmsk: date
        +jabatan: int
        +unit: int
        +GetSaldoKredit() decimal
        +GetHistoryPinjaman() array
    }
  
    class Barang {
        +id_barang: int
        +kode_barang: string
        +nama: string
        +id_satuan: int
        +sup: int
        +hargabeli: decimal
        +hargajual: decimal
        +GetStok(unit, tanggal) decimal
        +GetKartuStok(unit, start, end) array
    }
  
    class TrArusBrg {
        +id_tr: string
        +mk: int
        +unit: int
        +anggota: int
        +waktu: datetime
        +total: decimal
        +bayar: decimal
        +voucher: decimal
        +kredit: decimal
        +kembali: decimal
        +DetailItems: TrArusBrgDtl[]
        +Save()
        +GetTotalByDate(start, end) decimal
    }
  
    class TrArusBrgDtl {
        +id_tr_dtl: string
        +id_tr: string
        +id_brg: int
        +qty: decimal
        +harga: decimal
        +disc: decimal
    }
  
    class TrB rgMsk {
        +id: string
        +brg: string
        +gdg: string
        +tgl: date
        +qty: decimal
        +entry_user: int
    }
  
    class TrB rgRet {
        +id: string
        +brg: string
        +gdg: string
        +tgl: date
        +qty: decimal
    }
  
    class Kredit {
        +id_kredit: string
        +id_tr: string
        +anggota: int
        +tgl: date
        +nominal: decimal
        +tenor: int
        +angsuran: decimal
        +lunas: bool
        +GetSisaAngsuran() decimal
    }
  
    class Pelunasan {
        +id_pelunasan: string
        +id_kredit: string
        +anggota: int
        +tgl: date
        +nominal: decimal
        +ProsesPelunasan()
    }
  
    class Voucher {
        +id_voucher: string
        +id_tr: string
        +kode: string
        +nominal: decimal
        +IsValid() bool
    }
  
    class Pinjaman {
        +idpinjaman: string
        +idtr: string
        +jenis: int
        +idanggota: int
        +tanggal: date
        +nominal: decimal
        +GetSisaPinjaman() decimal
    }
  
    class PelPinjaman {
        +idpel: string
        +idpinjaman: string
        +nominal: decimal
        +bghs: decimal
        +tanggal: date
    }
  
    class Modal {
        +idmodal: string
        +idtr: string
        +jenis: int
        +idanggota: int
        +tanggal: date
        +nominal: decimal
    }
  
    class JurnalAkuntansi {
        +id_tr_keu: string
        +tgl: date
        +reff: string
        +ket: string
        +DetailJurnal: KeuDtTrAcc[]
    }
  
    class KeuDtTrAcc {
        +id_dt_tr: string
        +id_tr: string
        +akun: int
        +dk: int
        +nominal: decimal
    }
  
    class Akun {
        +id_acc: int
        +id_kel_acc: int
        +kode: int
        +Deskripsi: string
        +karakter: int
    }
  
    class Supplier {
        +id_supplier: int
        +nama: string
        +alamat: string
        +telp: string
    }
  
    class Unit {
        +id_unit: int
        +nama: string
        +tipe: int
    }
  
    class Gudang {
        +id: int
        +nama: string
    }
  
    TrArusBrg "1" --> "*" TrArusBrgDtl : contains
    TrArusBrg "1" --> "0..1" Kredit : generates
    TrArusBrg "1" --> "0..1" Voucher : uses
    TrArusBrg "*" --> "1" Anggota : belongs to
    TrArusBrg "*" --> "1" Unit : transacted at
    Kredit "1" --> "*" Pelunasan : repaid by
    Anggota "1" --> "*" Kredit : has
    Anggota "1" --> "*" Pinjaman : borrows
    Pinjaman "1" --> "*" PelPinjaman : repaid by
    Anggota "1" --> "*" Modal : owns
    Barang "1" --> "*" TrArusBrgDtl : in transaction
    Barang "1" --> "*" TrB rgMsk : received
    Barang "1" --> "*" TrB rgRet : returned
    JurnalAkuntansi "1" --> "*" KeuDtTrAcc : contains
    KeuDtTrAcc "*" --> "1" Akun : posts to
```

## 3.4 Arsitektur Sistem

```mermaid
flowchart TB
    subgraph "Presentation Layer"
        index[index.php - Login]
        main[main.php - Router Utama]
        header[header.php - Navigasi]
        footer[footer.php]
        home[home.php - Dashboard]
    end
  
    subgraph "Business Logic Layer"
        direction TB
        common_list[common_list.php - List Generator]
        common_entry[common_entry.php - Form Generator]
        common_api[common_api.php - API CRUD]
        common_cuquery[Common_CUQuery - Query Builder]
        common_generator[Common_Generator - Input Renderer]
        common_generator_dtl[Common_Generator_DTL - Detail Renderer]
    end
  
    subgraph "Modules"
        kasir[kasir/ - Penjualan]
        kosy[kosy/ - Konsinyasi]
        retur[retur/ - Retur]
        brg_msk[brg_msk/ - Barang Masuk]
        po[po/ - Purchase Order]
        keu[keu/ - Keuangan]
        laporan[laporan/ - Reports]
        master[master/ - Master Data]
        pelunasan[pelunasan/ - Pelunasan]
        stock_opt[stock_opt/ - Stok Opname]
        voucher[voucher/ - Voucher]
        sistem[sistem/ - System Config]
    end
  
    subgraph "Data Access Layer"
        connect[include/connect.php]
        class_mysql[class_Mysql_connect.php]
        class_pdo[class_PDO_connect.php]
    end
  
    subgraph "Database"
        db[(MySQL kopkar_rsi<br/>39 Tabel)]
    end
  
    main --> common_list
    main --> common_entry
    main --> home
    main --> kasir & kosy & retur & brg_msk & po & keu & laporan & master & pelunasan & stock_opt & voucher & sistem
  
    common_entry --> common_generator
    common_entry --> common_generator_dtl
    common_api --> common_cuquery
  
    common_list --> connect
    common_entry --> connect
    common_api --> connect
    modules --> connect
  
    connect --> class_mysql
    connect --> class_pdo
    class_mysql --> db
    class_pdo --> db
```

## 3.5 Deskripsi Tabel Database (39 Tabel)

### Tabel Konfigurasi Form (Framework Dinamis)

| Tabel                       | Fungsi                                                             |
| --------------------------- | ------------------------------------------------------------------ |
| `form_attr`               | Atribut field form entry (nama field, tipe input, validasi, label) |
| `form_attr_dtl`           | Atribut field untuk form detail                                    |
| `form_entry_std`          | Konfigurasi form entry standar (judul, nama tabel)                 |
| `form_entry_std_dtl`      | Konfigurasi form entry detail                                      |
| `form_generator_giant`    | Konfigurasi giant form (multi-tabel)                               |
| `form_generator_id_giant` | Konfigurasi ID generator giant form                                |
| `form_list`               | Konfigurasi halaman list (SQL select, SQL delete, hak akses)       |
| `tipe_input`              | Master tipe input (text, date, combo, checkbox, dll)               |

### Master Data

| Tabel           | Fungsi                            |
| --------------- | --------------------------------- |
| `mt_anggota`  | Data anggota koperasi             |
| `mt_barang`   | Data barang/jasa yang dijual      |
| `mt_gudang`   | Data gudang penyimpanan           |
| `mt_satuan`   | Data satuan barang (pcs, kg, dll) |
| `mt_supplier` | Data supplier/pemasok             |
| `mt_unit`     | Data unit/cabang retail           |

### Pengguna & Akses

| Tabel                 | Fungsi                                              |
| --------------------- | --------------------------------------------------- |
| `pengguna`          | Data pengguna sistem                                |
| `master_role`       | Master role pengguna                                |
| `role_menu`         | Mapping hak akses role ke menu (r,c,u,p,d, approve) |
| `master_menu`       | Daftar menu navigasi                                |
| `sys_comp`          | Konfigurasi sistem                                  |
| `sys_role`          | Role sistem (backup)                                |
| `sys_user`          | User sistem (backup)                                |
| `master_perusahaan` | Data perusahaan/cabang                              |

### Transaksi

| Tabel               | Fungsi                                |
| ------------------- | ------------------------------------- |
| `tr_arus_brg`     | Header transaksi penjualan/konsinyasi |
| `tr_arus_brg_dtl` | Detail item transaksi                 |
| `tr_brg_msk`      | Barang masuk / stok awal              |
| `tr_brg_ret`      | Retur barang                          |
| `tr_kredit`       | Transaksi kredit anggota              |
| `tr_pelunasan`    | Pelunasan kredit                      |
| `tr_voucher`      | Voucher penjualan                     |

### Keuangan/Akuntansi

| Tabel                | Fungsi                                         |
| -------------------- | ---------------------------------------------- |
| `keu_neraca`       | Klasifikasi neraca (aset, liabilitas, ekuitas) |
| `keu_kel_acc`      | Kelompok akun                                  |
| `keu_acc`          | Chart of account (daftar akun)                 |
| `keu_tr_acc`       | Header transaksi akuntansi/jurnal              |
| `keu_dt_tr_acc`    | Detail jurnal (debet/kredit)                   |
| `keu_pinjaman`     | Pinjaman anggota                               |
| `keu_pel_pinjaman` | Pelunasan pinjaman                             |
| `keu_modal`        | Modal/simpanan anggota                         |
| `keu_sp`           | Simpanan/pinjaman (SP)                         |
| `keu_shu_awal`     | Saldo SHU awal                                 |

---

## 3.6 Alur Data (Data Flow)

```mermaid
flowchart LR
    subgraph "INPUT"
        A[Kasir] --> B[Penjualan<br/>tr_arus_brg]
        C[Gudang] --> D[Konsinyasi<br/>tr_arus_brg mk=1]
        C --> E[Barang Masuk<br/>tr_brg_msk]
        C --> F[Retur<br/>tr_brg_ret]
        K[Keuangan] --> G[Pinjaman<br/>keu_pinjaman]
        K --> H[Modal Anggota<br/>keu_modal]
        K --> I[Pelunasan<br/>tr_pelunasan]
    end
  
    subgraph "PROSES"
        B --> J[Hitung Stok<br/>skrip_restock.php]
        D --> J
        E --> J
        F --> J
        G --> L[Jurnal<br/>keu_dt_tr_acc]
        H --> L
    end
  
    subgraph "OUTPUT"
        J --> M[Laporan Stok]
        J --> N[Kartu Stok]
        J --> O[HPP]
        L --> P[Laporan Keuangan]
        B --> Q[Laporan Penjualan]
        G --> R[Laporan Kredit]
    end
```

---

> **Dokumentasi ini dibuat oleh Giant Rachman Junaedi untuk menggambarkan arsitektur lengkap** **Sistem: KOPKAR RSI JEMURSARI — Point of Sales & Manajemen Koperasi Karyawan** dari sisi bisnis (use case), alur interaksi (sequence), struktur data (domain model/ERD), organisasi kode (class diagram), hingga dependensi antar modul.
