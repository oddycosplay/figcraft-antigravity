# FigCraft Live Canvas — Partitioned Multi-Writer Rules

Aturan ini berlaku setiap kali plugin `figcraft` aktif dan berinteraksi dengan MCP server `figcraft_figcraft-live` (`127.0.0.1:7979`).

## Protokol Anti-Konflik (Pola B: Partitioned Multi-Writer)

1. **Batas Partisi Wajib (`parent` ID Eksklusif)**
   - Setiap worker/builder hanya boleh membuat, mengubah, atau menghapus node di dalam `parent` ID yang secara eksplisit ditugaskan kepadanya.
   - Dilarang memodifikasi node di luar subtree milik sendiri atau mengubah properti root/page milik worker lain.

2. **Dilarang Mengandalkan `selection`**
   - Live Server FigCraft berbagi satu `selection` global. Setiap pemanggilan `set_properties`, `run_command`, `export`, atau mutasi lainnya **wajib** menyertakan parameter `ids` secara eksplisit.
   - Jangan memanggil `select` sebagai prasyarat mutasi saat mode multi-agent aktif.

3. **Gunakan `batch` untuk Mutasi Per-Section**
   - Bungkus rangkaian pembuatan/perubahan node untuk satu unit kerja ke dalam `batch` agar atomik dan tercatat sebagai satu langkah riwayat *undo* yang rapi.

4. **Dilarang Memanggil `undo` / `redo` Global Saat Paralel**
   - Riwayat *undo/redo* bersifat global untuk seluruh dokumen. Jika terjadi kesalahan pada node milik sendiri, perbaiki atau hapus node berdasarkan `id` eksplisit, bukan dengan `undo`.

5. **Koordinat & Auto Layout**
   - `create_node` dan `set_properties` memakai koordinat `x`/`y` **relatif terhadap parent frame**.
   - Untuk child di dalam frame auto-layout, atur `layout`, `gap`, `padding`, `mainAlign`, `crossAlign`, `sizingH`, dan `sizingV` daripada memaksa `x`/`y` manual.

6. **Verifikasi Mandiri Wajib**
   - Setelah menulis ke kanvas, verifikasi subtree milik sendiri menggunakan `get_node` (pada `parent` ID milik sendiri) dan `screenshot` (pada node spesifik) untuk memastikan tidak ada *overflow*, tumpang tindih, atau properti yang meleset.
