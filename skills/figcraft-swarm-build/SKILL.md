---
name: figcraft-swarm-build
description: Orchestrate multi-agent BA, PM, UX, UI, copywriting, parallel UI construction, and QA review on a live FigCraft desktop canvas using Hybrid Pattern B (Partitioned Multi-Writer Swarm) over the figcraft_figcraft-live MCP server. Use when building multi-section pages, dashboards, or multi-screen flows in FigCraft Live with a multi-agent team.
---

# FigCraft Swarm Build (Hybrid Pola B — Partitioned Multi-Writer)

SOP orkestrasi multi-agent end-to-end untuk membangun desain UI secara paralel di atas **Live Server FigCraft** (`figcraft --control 7979`) melalui MCP `figcraft_figcraft-live` tanpa konflik *selection* atau *race condition*.

> **Arsitektur Hybrid Swarm Antigravity:** Di Antigravity Swarm Board, worker tidak selalu mewarisi `call_mcp_tool` untuk lazy MCP server — akses eksekusi MCP `figcraft_figcraft-live` berada pada **`@coordinator`**. Karena itu, worker `figcraft-builder` bekerja **paralel** menyusun payload JSON `batch` yang valid terhadap schema MCP untuk partisi masing-masing, lalu `@coordinator` mengeksekusi setiap `batch` secara atomik ke kanvas live.

## Prasyarat

1. Aplikasi desktop FigCraft berjalan dengan control port aktif:
   ```powershell
   figcraft --control 7979
   ```
2. MCP server `figcraft_figcraft-live` aktif dan terhubung.
3. Sebelum memanggil tool MCP `figcraft_figcraft-live`, baca schema tool terkait jika belum ada di konteks (`inspect_document`, `create_node`, `batch`, `set_properties`, `get_node`, `screenshot`).

---

## Fase 0: Verifikasi Koneksi Live Canvas (Coordinator)

1. Panggil `inspect_document` (`{ "outline": true, "depth": 2 }`) pada `figcraft_figcraft-live`.
2. Catat `document` ID, `currentPage`, dan struktur layer yang sudah ada agar desain baru konsisten dan tidak menimpa karya sebelumnya.

---

## Fase 1: Discovery & Spesifikasi (Read-Only Specialists)

Jalankan agent spesialis (seluruhnya **read-only** terhadap kanvas) sesuai kompleksitas tugas:

1. **`figcraft-ba`** — Menyusun *Business Requirements Summary*: tujuan bisnis, aktor/peran, aturan bisnis, alur proses/data, dan daftar gap tanpa membuat asumsi bisnis.
2. **`figcraft-pm`** — Menyusun *Product Brief* berdasarkan hasil BA: daftar *section* terurut, ruang lingkup fungsional, dan *acceptance criteria*.
3. **`figcraft-ux-designer`** & **`figcraft-copywriter`** (dapat paralel setelah brief PM siap):
   - `figcraft-ux-designer`: merancang arsitektur informasi, *user flow*, dan hierarki *auto-layout* per *section* yang siap dipartisi.
   - `figcraft-copywriter`: menyusun teks final (*headline*, *subheadline*, CTA, label navigasi, *microcopy*) per slot teks.
4. **`figcraft-ui-designer`** — Menggabungkan *wireframe* UX dan *copy* final menjadi **Shared Design Token Sheet** (`#RRGGBB`, skala tipografi, *spacing*, *radius*, *stroke*) dan spesifikasi visual per *section*.

---

## Fase 2: Partisi Kanvas oleh Coordinator (Wajib Sebelum Paralel)

Coordinator adalah **satu-satunya** pihak yang menyiapkan kerangka partisi sebelum *builder* dijalankan paralel:

1. Buat (atau identifikasi) root container frame (mis. `Desktop 1440px` dengan auto-layout vertikal) atau halaman terpisah.
2. Buat frame kosong untuk setiap partisi/*section* yang akan dikerjakan paralel (mis. `Section/Navbar`, `Section/Hero`, `Section/Features`, `Section/Footer`) dalam satu `batch` dengan `"select": false`.
3. Catat **numeric `id` eksak** setiap frame partisi dari hasil `batch` atau `inspect_document`.

---

## Fase 3: Eksekusi Paralel Hybrid dengan `figcraft-builder`

1. Spawn worker `figcraft-builder` secara paralel melalui **`swarm`** (`workspace: "inherit"`), satu worker per partisi `parentId`.
2. Setiap brief worker **wajib** memuat:
   - `document` ID dan `parentId` eksklusif miliknya.
   - Spesifikasi BA/PM/UX, Copy final, dan Design Tokens dari Fase 1 khusus untuk *section* tersebut.
   - Instruksi **Hybrid Swarm**: langsung susun payload JSON `batch` lengkap (`document`, `label`, `onError: "abort"`, `commands` dengan `"select": false` dan `$ref` internal) berdasarkan schema `batch.json` & `create_node.json`, lalu posting ke `@coordinator` (atau eksekusi langsung bila `call_mcp_tool` tersedia di worker).
3. Begitu worker memposting payload JSON ke Swarm Board, **`@coordinator`** segera mengeksekusi setiap `batch` secara atomik ke `figcraft_figcraft-live` (tanpa konflik *selection* karena semua memakai `parent` ID terpisah dan `select: false`).

---

## Fase 4: Audit QA & Finalisasi

Setelah seluruh partisi tereksekusi di kanvas live:

1. Coordinator mengambil data `inspect_document` / `get_node` dan `screenshot` (per section dan root container) lalu mengaudit — atau menyerahkan ke **`figcraft-qa`** (*read-only*) — terhadap aturan bisnis (`figcraft-ba`), *acceptance criteria* (`figcraft-pm`), struktur UX, kesesuaian copy, dan konsistensi token UI.
2. Jika ditemukan isu `Critical` / `Major`, perbaiki berdasarkan `nodeId` & `parentId` spesifik menggunakan `ids` eksplisit, lalu verifikasi ulang dengan `screenshot`.
3. Setelah status QA `PASS`, hentikan (*kill*) worker swarm yang sudah selesai dan simpan dokumen (`save_file`) bila diminta pengguna.
