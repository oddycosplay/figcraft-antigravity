# FigCraft Plugin

Plugin lokal untuk menghubungkan Antigravity ke aplikasi desktop **FigCraft** melalui MCP Server dalam mode **Live Canvas (`--attach`)**, menggunakan arsitektur **Product Manager Coordinator + 5 Specialist Sub-Agents** (*Hybrid Pola B*).

## Komponen

- **MCP Server (`figcraft-live`)**: Menjalankan `figcraft-cli mcp --attach 7979` untuk terhubung ke instance aplikasi desktop FigCraft yang sedang berjalan.
- **Skill (`figcraft-swarm-build`)**: SOP orkestrasi multi-agent end-to-end dengan **Product Manager** sebagai Coordinator dan 5 sub-agent spesialis di Swarm Board:
  - `Coordinator (Product Manager)` → `figcraft-pm`: menyusun *Product Brief*, mempartisi frame (`parentId`), mengorkestrasi Swarm Board, mengeksekusi payload `batch` ke `figcraft_figcraft-live`, dan melakukan *sign-off*.
  - `@ba` (`figcraft-ba`): Business Analyst (*read-only*; merumuskan tujuan bisnis, aktor, aturan bisnis, dan alur proses/data).
  - `@ux-designer` (`figcraft-ux-designer`): UX Designer (*read-only*; menyusun arsitektur informasi, *user flow*, dan hierarki *auto-layout* per *section*).
  - `@copywriter` (`figcraft-copywriter`): Copywriter & UX Writer (*read-only*; menyusun teks final per slot UI).
  - `@ui-designer` (`figcraft-ui-designer`): UI Designer (*design tokens* `#RRGGBB`, tipografi, spesifikasi visual, dan menyusun payload JSON `batch` per `parentId` untuk dieksekusi Coordinator).
  - `@qa` (`figcraft-qa`): Visual & Design QA Reviewer (*read-only*; mengaudit layer tree & *screenshot* terhadap seluruh spesifikasi).
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
    ├── figcraft-pm.md
    ├── figcraft-ba.md
    ├── figcraft-ux-designer.md
    ├── figcraft-copywriter.md
    ├── figcraft-ui-designer.md
    └── figcraft-qa.md
```
