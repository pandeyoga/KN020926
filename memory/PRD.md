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

## Backlog / Prioritas
- P2: refactor readability `merged_max` di PATCH approval_rules (catatan review, non-blocking).
- Roadmap repo (MASTER_ROADMAP.md): FASE I (Inspeksi & QC sebagai dokumen) → N (Notifikasi) → M (Makloon).

## Catatan Verifikasi Sesi Ini
- curl: POST rule entity ent_ksc → evaluate-approval mengembalikan rule_id tsb (mesin membaca);
  POST/PATCH entity bogus → 403; PATCH entity sah → OK.
- UI (screenshot): 9 aturan seed tampil, modal 3 kolom, dropdown entitas berisi 3 opsi,
  buat aturan ber-entitas KSC via UI berhasil (Cakupan = "PT Kain Suka Cita (KSC)"),
  Esc menutup modal, select Satuan/Grade 130px terbaca.
- Data uji dibersihkan; DB kembali 9 aturan seed, semua active.
