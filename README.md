# 🏪 KOPKAR RSI JEMURSARI — POS & Cooperative Management System

### System Design Case Study | Built from Scratch · Rescued · Re-maintained

> **Role:** Senior Database Administrator · System Architect
> **Client:** Koperasi Pegawai Rumah Sakit Islam Jemursari, Surabaya
> **Build Phase:** 2013 - 2016 (original system, built from scratch)
> **Rescue Phase:** August 2019 (database recovery + system re-engagement)
> **Stack:** PHP · MySQL · Bootstrap · jQuery · Custom Dynamic Form Framework
> **Database:** `kopkar_rsi` · 39 Tables

---

## 🔥 The Mess — Why This System Had to Be Built

Before this system existed, Koperasi Pegawai RSI Jemursari — a staff cooperative serving over 1,000 hospital employees — was running its entire retail, credit, and financial operation on Excel spreadsheets.

With 1,000+ members transacting daily across multiple operational domains — cash sales, credit purchases, voucher redemptions, goods receiving, consignment, member loans, and savings — the spreadsheet model was not just inefficient. It was a liability. Stock discrepancies were invisible. Credit exposure was untracked. Loan repayment schedules existed only in manually maintained files. There was no mechanism to detect revenue leakage, and no way for management to see the financial position of the cooperative in real time.

In 2013, I was engaged to replace this entirely — building, delivering, and maintaining the system through 2016. Three years after the maintenance contract ended, the system I had built was still running in production. It was only in 2019, after a power failure corrupted the database, that they called me back.

---

## 🔍 The Diagnosis — What Needed to Be Solved

The core challenge was scope. This was not a single-domain system. It needed to cover:

- **Retail POS** — cash sales, credit sales, voucher transactions, returns
- **Inventory** — goods receiving, consignment in/out, purchase orders, stock opname
- **Cooperative Finance** — member savings (modal), member loans (pinjaman), loan repayments, salary deductions
- **Accounting** — double-entry journal ledger (debit/credit) auto-generated from every financial transaction
- **Reporting** — sales reports, stock cards, HPP (cost of goods sold), credit reports, salary deduction reports, consignment reports
- **Multi-role Access Control** — Cashier, Warehouse, Finance, Admin, and Management each with scoped permissions

Beyond scope, there was a deeper architectural problem: with 12+ operational modules, building each with its own bespoke form, list, and CRUD logic would create a maintenance nightmare. Every new module or field change would require touching multiple files across the codebase.

---

## 🏗️ The Architecture — How It Was Solved

### The Core Innovation: A Dynamic Form Framework

The most significant architectural decision was building a **custom data-driven form engine** rather than hard-coding each module individually.

Instead of writing separate list pages, entry forms, and CRUD handlers for every module, the system reads its own configuration from the database:

| Config Table       | Purpose                                                                            |
| ------------------ | ---------------------------------------------------------------------------------- |
| `form_list`      | Defines how a list page renders — SQL query, columns, delete logic, access rights |
| `form_entry_std` | Defines form metadata — title, target table, module identity                      |
| `form_attr`      | Defines every field in a form — name, input type, validation, label               |
| `form_attr_dtl`  | Same as above, for master-detail form sections                                     |

At runtime, `Common_CUQuery` reads these tables and **auto-generates INSERT/UPDATE queries** without hard-coded SQL. `Common_Generator` renders the HTML input fields. `common_list.php` renders the data grid. `common_entry.php` renders the form.

The result: adding a new module required only database configuration entries — not new PHP files. This eliminated redundancy across 12+ modules and made the system significantly easier to maintain and extend.

```
Database Config Tables
        ↓
  Common_CUQuery        ← reads form_entry_std + form_attr
  Common_Generator      ← renders HTML inputs
  common_list.php       ← renders data grids
  common_entry.php      ← renders entry forms
  common_api.php        ← handles all CRUD via POST
        ↓
    MySQL Database
```

### Deployment Topology

```
[ 1,000+ Members / Staff ]
         ↓ (LAN / Intranet)
    [ Apache + PHP ]
         ↓
  [ Custom Framework Engine ]
  (main.php → common_* → modules)
         ↓
  [ MySQL — kopkar_rsi ]
     (39 Tables)
```

### Multi-Role Access Control

The system implements a **granular 7-permission RBAC** model — beyond the typical read/write split:

| Permission     | Code          | Scope                      |
| -------------- | ------------- | -------------------------- |
| Read           | `r`         | View data lists            |
| Create         | `c`         | Add new records            |
| Update         | `u`         | Edit existing records      |
| Print          | `p`         | Generate and print reports |
| Delete         | `d`         | Remove records             |
| Approve        | `approve`   | Authorize transactions     |
| Cancel Approve | `c_approve` | Reverse an approval        |

Each role's permissions are scoped per menu item — giving fine-grained control over what each user type can do in each module.

### The Credit & Loan Engine

One of the most operationally significant modules was the integrated credit system. Members could purchase goods or vouchers on credit — with repayment tracked and deducted from salary. The system handled:

- Credit transaction creation linked to member identity
- Repayment scheduling and tracking (`tr_kredit` → `tr_pelunasan`)
- Loan management separate from purchase credit (`keu_pinjaman` → `keu_pel_pinjaman`)
- Auto-generated accounting journal entries (debit/credit) for every financial event

Every financial transaction — whether a cash sale, a credit purchase, a loan disbursement, or a repayment — automatically posted to the double-entry journal ledger in `keu_dt_tr_acc`, keeping the cooperative's books current without manual accounting entries.

### Stock Reconciliation Engine

The `skrip_restock.php` engine continuously reconciles stock levels by iterating across all items and warehouse units — calculating real-time stock positions from the movement history in `tr_arus_brg` and `tr_arus_brg_dtl`. This gave management accurate HPP (cost of goods sold) and stock card reports without manual counting.

---

## ⚠️ Edge Cases — Where It Gets Hard

### The Concurrent Credit Transaction Problem

With 1,000+ members and multiple cashiers operating simultaneously, the risk of two cashiers approving credit transactions for the same member at the same moment — pushing them over their credit limit — was real.

The system handles this with a **pre-transaction balance check**: before any credit sale is committed, the system calculates the member's current outstanding balance from `tr_kredit` and compares it against their approved limit. The transaction is rejected at the application layer if the limit would be breached — before any INSERT reaches the database.

### The Stock Count Drift Problem

In a high-transaction retail environment, stock figures can drift from reality due to unrecorded returns, consignment adjustments, or timing gaps between physical movement and data entry. The `skrip_restock.php` reconciliation script was built to be run on-demand — recalculating stock from the full transaction history rather than relying on a running counter. This means stock figures are always derivable from source data, not just a cached sum that can silently drift.

### The Accounting Integrity Problem

Every financial transaction must produce a balanced journal entry (total debits = total credits). Rather than relying on application logic to enforce this at the UI layer, the system uses predefined debit/credit account mappings per transaction type. The journal entries in `keu_dt_tr_acc` are generated programmatically from these mappings — removing human error from the bookkeeping process entirely.

---

## 📊 The Impact — What Changed

| Before                                | After                                                       |
| ------------------------------------- | ----------------------------------------------------------- |
| Excel spreadsheets across all domains | Unified system — 39 normalized tables                      |
| Stock discrepancies invisible         | Real-time stock reconciliation via restock engine           |
| Credit exposure untracked             | Full credit lifecycle — issuance, tracking, repayment      |
| Loan repayments in manual files       | Automated loan tracking with salary deduction reports       |
| No accounting visibility              | Auto-generated double-entry journal for every transaction   |
| No management dashboard               | Real-time sales, stock, and financial reports               |
| Member purchases cash-only            | Credit and voucher system increased member purchasing power |
| 12+ modules = 12x redundant code      | Single dynamic form engine serving all modules              |

The credit system directly enabled a measurable increase in sales — members who previously could not purchase due to cash constraints could now buy goods and vouchers on credit, with repayment automatically managed through salary deductions.

---

## 💡 Key Architectural Decisions

**Why a custom dynamic form framework instead of an existing one?**
At the time of build, the available PHP frameworks either introduced too much overhead for an intranet deployment or required significant learning curve for future maintainers. Building a lean, data-driven engine on top of vanilla PHP gave full control over query generation, form rendering, and permission enforcement — with zero external dependencies.

**Why auto-generate accounting journals from transactions?**
Manual journal entry is the single biggest source of bookkeeping error in small cooperative systems. By mapping transaction types to debit/credit account pairs at the system level, the accounting layer becomes a byproduct of normal operations — not a separate task requiring a trained accountant for every entry.

**Why MySQL over a more modern database engine?**
The cooperative's IT environment was standardized on MySQL, the operations team was familiar with it, and the query patterns — while complex — were well within MySQL's capabilities at this data scale. Introducing PostgreSQL or another engine would have created a maintenance dependency without delivering meaningful benefit.

---

## 📐 System Design Diagrams

For the complete system architecture including Use Case Diagrams, Sequence Diagrams (Login, POS, Consignment, Loans, Stock Opname), Entity-Relationship Diagram (39 tables), Class Diagram, System Architecture Flowchart, and Data Flow Diagram — see:

📄 **[DOKUMENTASI_DESAIN_SISTEM.md](./DOKUMENTASI_DESAIN_SISTEM.md)**

---

*This case study is part of my system design portfolio. The client is a hospital staff cooperative — a real operational system serving 1,000+ members. All diagrams reflect the actual system architecture as built and maintained.*
