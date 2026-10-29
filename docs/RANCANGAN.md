# Rancangan Sistem Warehouse Internal PT SML

## 1. Prinsip desain

1. **Stok dicatat sebagai buku besar (ledger).** Setiap perubahan stok adalah baris `stock_movements` yang tidak bisa diubah. Saldo dihitung dari ledger, sehingga setiap selisih bisa ditelusuri (penting untuk audit emiten Tbk).
2. **Semua posting lewat fungsi database (RPC)**, misalnya `post_goods_receipt()` dan `issue_to_workorder()`, dijalankan atomik dengan pemeriksaan role.
3. **Scan dulu, ketik belakangan.** Rak, part, unit, dan baterai punya label QR.
4. **Bisa offline** untuk aplikasi lapangan (lokasi pelanggan dan tambang), sinkron saat ada sinyal.
5. **Multi-cabang sejak awal**, walaupun rollout pertama hanya di Cikarang.

## 2. Modul

| # | Modul | Isi |
|---|---|---|
| 1 | Master Data | Item/part, kategori, satuan, supplier, cabang, gudang, rak (bin), unit forklift, baterai |
| 2 | Inbound | PO → penerimaan barang (GR) → QC → putaway ke rak |
| 3 | Outbound | Permintaan part dari work order → picking → serah ke mekanik → retur sisa |
| 4 | Transfer | Pusat ke cabang, antar-cabang, gudang ke van mekanik, status in-transit |
| 5 | Unit Yard | Ready / Reserved / On-Rent / Inspeksi / Repair / Standby pengganti / Disposal |
| 6 | Baterai | Nomor seri, tipe (lead-acid/lithium), siklus, SoH, terpasang di unit mana |
| 7 | Stock Opname | Cycle count berbasis ABC, penyesuaian dengan persetujuan |
| 8 | Limbah B3 | Oli bekas dan aki bekas: TPS, manifest, serah ke pengolah berizin |
| 9 | Laporan & Dashboard | Nilai stok, slow-moving, fill rate, utilisasi unit, biaya part per unit |

## 3. Hierarki lokasi

```
Cabang (Cikarang, Surabaya, ...)
 └─ Gudang (Gudang Part, Yard Unit, Charging Bay, TPS-B3, Karantina QC)
     └─ Zona (A: fast-moving, B, C, oli, aki)
         └─ Rak / Bin (CKR-A-01-03)
Lokasi khusus: VAN-<plat no>   (stok bergerak mekanik)
               SITE-<pelanggan> (unit on-rent / stok di tambang)
               IN-TRANSIT
```

## 4. Model data inti (sketsa)

```sql
branches(id, code, name, address)
locations(id, branch_id, parent_id, code, type, is_active)
  -- type: warehouse|zone|bin|van|site|transit|qc|b3
items(id, sku, name, category, uom, abc_class, is_serialized, has_expiry,
      min_qty, max_qty, std_cost, compatible_models text[])
suppliers(id, name, contact)

units(id, unit_no, serial_no, brand, model, power, capacity_ton, hour_meter,
      status, location_id, customer_id, contract_no)
batteries(id, serial_no, chem, voltage, ah, cycles, soh_pct, status, unit_id, location_id)

-- Ledger stok (tidak boleh update/delete)
stock_movements(id, ts, item_id, lot_no, serial_no, from_location_id, to_location_id,
                qty, unit_cost, ref_type, ref_id, created_by)
  -- ref_type: GR|ISSUE|RETURN|TRANSFER|ADJ|SCRAP
stock_balances(item_id, location_id, lot_no, qty)  -- diisi fungsi posting

purchase_orders / po_lines
goods_receipts / gr_lines            -- draft → qc → posted
work_orders(id, wo_no, unit_id, type, hour_meter, mechanic_id, status, customer_id)
part_requests / pr_lines
transfers / transfer_lines           -- draft → shipped → received
stock_counts / count_lines           -- selisih → adjustment perlu approval
unit_events(id, unit_id, ts, from_status, to_status, location_id, note, photo_url)
b3_waste(id, type, qty_kg, location_id, in_date, manifest_no, out_date, vendor)
```

## 5. Alur proses utama

- **Penerimaan:** PO disetujui → barang tiba → scan PO, isi qty/lot/kedaluwarsa → karantina QC → QC lulus → putaway (scan label bin).
- **Pengeluaran ke WO:** mekanik buka WO (scan QR unit) → minta part → disetujui kepala workshop → picking (saran bin, FEFO) → serah terima → retur sisa. Biaya part menempel ke unit dan kontrak.
- **Siklus unit:** Ready → Reserved → On-Rent → Inspeksi (checklist + foto) → Ready atau Repair. Hour meter memicu WO PM otomatis (250/500/1.000 jam).
- **Transfer:** dokumen → pick → kirim (IN-TRANSIT) → cabang tujuan scan terima → selisih tercatat.
- **Isi ulang otomatis:** setiap malam, stok dibandingkan dengan `min_qty` + kebutuhan PM 30 hari ke depan → draft PO / transfer.

## 6. Role

| Role | Hak utama |
|---|---|
| Admin Gudang Pusat | Semua dokumen di semua cabang, master data |
| Kepala Gudang Cabang | Dokumen cabangnya, approve transfer & opname |
| Staf Gudang | GR, putaway, picking, opname (aplikasi scan) |
| Kepala Workshop | Approve permintaan part & WO |
| Mekanik | Buat WO, minta part, retur, inspeksi unit |
| Purchasing | PO & supplier |
| Finance / Auditor | Baca saja, laporan nilai stok |
| Manajemen | Dashboard |

## 7. KPI

Akurasi stok ≥ 98% · fill rate part ≥ 95% · dock-to-stock < 24 jam · utilisasi armada · nilai stok slow-moving (> 180 hari) · biaya part per jam operasi unit.

## 8. Tahapan pembangunan

| Fase | Lingkup | Perkiraan |
|---|---|---|
| 1 – MVP | Master data, lokasi, label QR, GR, issue ke WO, retur, kartu stok, opname | 6–8 minggu |
| 2 | Transfer multi-cabang, stok van mekanik, approval, min/max & draft PO | 4–6 minggu |
| 3 | Unit Yard & siklus status, baterai, PM berbasis hour meter | 4–6 minggu |
| 4 | Limbah B3, integrasi akuntansi, mode offline, telematics, analitik | Bertahap |

Saran: mulai Fase 1 hanya untuk gudang part Cikarang.
