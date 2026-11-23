# FigCraft Plugin

Plugin lokal untuk menghubungkan Antigravity ke aplikasi desktop **FigCraft** melalui MCP Server dalam mode **Live Canvas (`--attach`)**, dilengkapi tim multi-agent **Pola B (Partitioned Multi-Writer)**.

## Komponen

- **MCP Server (`figcraft-live`)**: Menjalankan `figcraft-cli mcp --attach 7979` untuk terhubung ke instance aplikasi desktop FigCraft yang sedang berjalan.
- **Skill (`figcraft-swarm-build`)**: SOP orkestrasi multi-agent end-to-end Pola B (`BA → PM → UX + Copywriter → UI → Coordinator Partitioning → parallel figcraft-builder → QA`).
- **Agents**:
  - `figcraft-ba`: Business Analyst (*read-only* kanvas; merumuskan tujuan bisnis, aktor, aturan bisnis, dan alur proses/data).
  - `figcraft-pm`: Product Manager (*read-only* kanvas; menyusun *Product Brief*, *scope*, dan *acceptance criteria*).
  - `figcraft-ux-designer`: UX Designer (*read-only* kanvas; menyusun arsitektur informasi, *user flow*, dan hierarki *auto-layout* per *section*).
  - `figcraft-ui-designer`: UI Designer (*read-only* kanvas; menyusun *design tokens*, tipografi, warna `#RRGGBB`, dan spesifikasi visual per *section*).
  - `figcraft-copywriter`: Copywriter & UX Writer (*read-only* kanvas; menyusun teks final per slot UI).
  - `figcraft-builder`: *Partitioned Writer* paralel yang membangun UI hanya di dalam `parent` ID miliknya menggunakan `batch` dan `ids` eksplisit.
  - `figcraft-qa`: Visual & Design QA Reviewer (*read-only* kanvas; mengaudit layer tree & *screenshot* terhadap seluruh spesifikasi).
- **Rules (`rules/AGENTS.md`)**: Aturan wajib anti-konflik untuk Live Canvas FigCraft.

## Prasyarat

1. Pastikan `figcraft` dan `figcraft-cli` tersedia di `PATH` sistem.
2. Jalankan aplikasi desktop FigCraft dengan control port aktif:

```powershell
figcraft --control 7979
```

## Struktur Direktori

```text
~/.gemini/config/plugins/figcraft/
├── plugin.json
├── README.md
├── mcp_config.json
├── rules/
│   └── AGENTS.md
├── skills/
│   └── figcraft-swarm-build/
│       └── SKILL.md
└── agents/
    ├── figcraft-ba.md
    ├── figcraft-pm.md
    ├── figcraft-ux-designer.md
    ├── figcraft-ui-designer.md
    ├── figcraft-copywriter.md
    ├── figcraft-builder.md
    └── figcraft-qa.md
```
