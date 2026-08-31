# PRD — Kain Nusantara (ERP Tekstil)

## Problem Statement Asli
Lanjutkan development dari repo https://github.com/wakasajanamasa/KN (ERP tekstil multi-entitas:
React + FastAPI + MongoDB). Development sebelumnya berhenti setelah iteration_273 (semua tes PASS,
tersisa action items minor).

## Arsitektur
- Backend: FastAPI modular (`backend/routers/*`, `backend/services/*`), MongoDB via Motor.
- Frontend: React CRA, **dilayani dari bundle statis `frontend/build` oleh `static_server.js`
  (TIDAK ada hot reload — setelah edit `frontend/src` WAJIB `bash scripts/rebuild_frontend.sh`)**.
- Multi-entitas: PT Kain Suka Cita (ent_ksc / KSC) + CV Kanda Suka (KANDA); isolasi via
  `entity_scope.py` (entity_ctx / resolve_scope_ids / assert_entity_access), header `X-Entity-Id`.
- Restore lingkungan setelah clone: `bash /app/.restore_env.sh` (pip + yarn + mongo + seed_realistic.py + build).
- Kredensial demo: lihat `/app/memory/test_credentials.md` (admin@kainnusantara.id / demo12345).

## Persona
- Admin (Budi) — akses penuh; Manager (Dewi) — persetujuan; Admin Sales, Finance, Sales, Gudang, Desainer.

## Yang Sudah Diimplementasikan
- s/d iteration_273: Aturan Persetujuan dikolapskan ke skema mesin tunggal
  {doc_type, entity_id, min_amount, max_amount, required_role, sort, active, is_percent};
  UI + mesin (config_service.evaluate_approval) membaca koleksi yang sama. 3 fix kosmetik
  (blanket modal unit 130px, label mini PO create, thead COA).
- 2026-06 (sesi ini — action items iteration_273):
  1. CSS `.form-row-3col` ditambahkan (`styles/components.css`) — modal aturan kini 3 kolom.
  2. Cakupan entitas pada Aturan Persetujuan: dropdown `rule-entity-id` di form
     (Semua entitas / KSC / KANDA), payload POST & PATCH kirim `entity_id`, kolom Cakupan
     menampilkan nama entitas. Backend memvalidasi `entity_id` ∈ allowed_entity_ids
     (403 bila tidak berwenang) pada POST & PATCH (`_assert_rule_entity_allowed`).
  3. GET /approval-rules/{id}: cek 404 dipindah SEBELUM assert_entity_access.
  4. Lebar select Satuan & Grade pada baris Tambah Item PO create: 104px → 130px (kedua grid).
  5. Escape menutup FormModal — sudah tertangani `useEscapeClose` (INV-UI-10), diverifikasi ulang.

- 2026-06 (verifikasi FASE I): FASE I (Inspeksi & QC sebagai dokumen) TERNYATA SUDAH SELESAI
  di repo (plan.md §STATUS I, 2026-08-24, POC 93/93). Diverifikasi ulang di lingkungan ini:
  layar SPK Inspeksi & QC (Gudang → Operasi Gudang → tab SPK Inspeksi & QC) render 2 SPK KSC
  + banner kebijakan; kebijakan default qc.color_mismatch_action=tahan /
  qc.handfeel_mismatch_action=peringatkan; release-hold oleh warehouse → 403
  (HOLD_RELEASE_ROLES = admin, manager); seed 3 SPK + 7 complaint_reasons ada.

- 2026-06 (Dasbor Tahanan QC): beranda manajer kini memuat kartu KPI "Barang ditahan QC"
  (klik → layar SPK Inspeksi, testid manager-home-qc-hold-kpi) + papan antrean
  `inspection_hold` (baris per SPK ber-tahanan). Backend: `MANAGER_BOARD_KEYS =
  HOME_BOARD_KEYS + ("inspection_hold",)` di home_service.manager_home — definisi antrean
  tetap satu di approval_backlog_service (INV-HOME-01), Control Tower admin tidak diubah.

- 2026-06 (FASE N + M ditutup): DRIFT terakhir FASE N dibereskan — notifikasi
  `contra_bon_cycle` kini beralamat `create_addressed(roles=("finance","manager"))`
  (contra_bon_reminder.py), 0 dokumen `recipient_role="all"`. FASE M dibukukan dengan POC
  baru `backend/test_core_makloon_lini_poc.py` (M1–M6, 35/35 PASS, nol residu).
  `audit_md_erp_readiness`: SELESAI=96 · BELUM=0 · DRIFT=0. Rencana MD-ERP habis.

- 2026-06 (sesi lanjutan ke-7 — penutupan & backlog): restore diverifikasi (audit 96/0/0,
  POC N 35/35, POC M 35/35, demo persis, residu 0); commit WIP di-amend jadi pesan penutup
  N+M. Backlog kecil selesai: P2 merged_max (POC approval 41/41), Bug #7 badge tab
  (center_delta 0.00px), 3 pintu RFID verify → DOOR_EXEMPT (penjaga 233 cek HIJAU).
  MASTER_ROADMAP EPIC 0 & 1 diverifikasi TERNYATA SUDAH DIBANGUN sesi lampau — flag
  ui.show_coming_soon terbukti mengendalikan sidebar (dibalik via API lalu dikembalikan),
  role-home per peran hidup, akses sales bersih dari HPP/vendor bill. Regresi
  iteration_274: backend 100% · frontend 100% · nol residu. Header MASTER_ROADMAP dibetulkan.

## Backlog / Prioritas
- ~~P2: refactor readability `merged_max` di PATCH approval_rules~~ — SELESAI 2026-06
  (ternary bersarang → dua baris jelas; POC approval coverage tetap 41/41, PATCH menolak
  min>max dengan benar). Bug #7 badge tab "Menunggu" juga SELESAI (screenshot before/after).
  Bonus: 3 pintu RFID verify (scan fisik label, bukan keputusan manusia) didaftarkan ke
  `DOOR_EXEMPT` penjaga INV-APPR-01 — penjaga kembali HIJAU 233 cek.
- Rencana fase MD-ERP selesai semua; backlog berikutnya dari BUG_BACKLOG.md / permintaan pemilik.

## Catatan Verifikasi Sesi Ini
- curl: POST rule entity ent_ksc → evaluate-approval mengembalikan rule_id tsb (mesin membaca);
  POST/PATCH entity bogus → 403; PATCH entity sah → OK.
- UI (screenshot): 9 aturan seed tampil, modal 3 kolom, dropdown entitas berisi 3 opsi,
  buat aturan ber-entitas KSC via UI berhasil (Cakupan = "PT Kain Suka Cita (KSC)"),
  Esc menutup modal, select Satuan/Grade 130px terbaca.
- Data uji dibersihkan; DB kembali 9 aturan seed, semua active.
