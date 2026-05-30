# CCTV Installation Design — CN37

## Physical Infrastructure

ถนนเมนซ้าย (เหนือ) และขวา (ใต้) มีเสาไฟโครงการตลอดแนว — ระบบเดินสายพาด ADSS fiber บนเสา และดึงไฟ 220VAC จากวงจรไฟส่องสว่างของโครงการ (เปิด 24 ชั่วโมง)

## Key Distances (คำนวณจาก GPS จริง)

| เส้นทาง | ระยะ |
| --- | --- |
| ป้อมหน้า → ป้อมหลัง (ถนนเมน) | **1,026 m** |
| ป้อมหน้า → สำนักงาน | 504 m |
| สำนักงาน → ป้อมหลัง | 527 m |

---

## Network Architecture — แยก Backbone / Camera อย่างสิ้นเชิง

### หลักการออกแบบ

| ชั้น | วัตถุประสงค์ | อุปกรณ์ที่ผ่าน | ถ้าขาด |
| --- | --- | --- | --- |
| **Backbone** | สื่อสารระหว่างป้อม/สำนักงาน | ไม่ผ่าน JB ใดเลย | กระทบเฉพาะการสื่อสาร node-to-node |
| **Camera Chain** | ส่งภาพกล้องกลับ NVR | ผ่าน JB ตามสาย | กระทบเฉพาะกล้องใน chain นั้น |

---

## Backbone — สาย 2 เส้น ตรงถึงกัน ไม่ผ่าน JB

```text
MDF-FRONT ──────── 2-core 510m ────────► CORE-OFFICE ──────── 2-core 530m ────────► IDF-BACK
  (ป้อมหน้า)        วิ่งตามเสา                (สำนักงาน)          วิ่งตามเสา              (ป้อมหลัง)
                    ไม่หยุดที่ JB ใด                              ไม่หยุดที่ JB ใด
```

- รับประกันการสื่อสาร Intercom / VoIP / Access Control ระหว่าง 3 node หลัก
- ไม่กระทบจาก JB ไฟดับ, สายกล้องขาด, หรือ switch ใน JB พัง
- ใช้ BiDi SFP (1 core per link) หรือ standard SFP (2 core) ก็ได้

---

## Camera Chains — แบ่งตาม Zone

กล้อง 32 ตัวแบ่งเป็น 3 zone ตาม main node ที่ใกล้ที่สุด:

### Zone A — MDF-FRONT (ป้อมหน้า)

```text
MDF-FRONT
  ├── [Local PoE] CAM-01, 02, 03, 05, 06  (Cat6 ตรง ไม่เกิน 30m)
  ├── [Fiber] MDF-FRONT ──── JB-N2   (ตามเสาไฟ)
  │              JB-N2 ···· JB-N1 ···· JB-N4   (Cat6 LAN)
  └── [Fiber] MDF-FRONT ──── JB-S2   (ตามเสาไฟ)
                 JB-S2 ···· JB-S1 ···· JB-S3   (Cat6 LAN)
```

| สาย | JB | กล้อง | ระยะ | ชนิด |
| --- | --- | --- | --- | --- |
| Fiber | JB-N2 | CAM-04, CAM-31 | ~80 m จาก MDF-FRONT | ADSS 2-core |
| LAN | JB-N1 | CAM-07 | ~25 m จาก JB-N2 | Cat6 |
| LAN | JB-N4 | CAM-15 | ~60 m จาก JB-N1 | Cat6 |
| Fiber | JB-S2 | CAM-09, CAM-10 | ~55 m จาก MDF-FRONT | ADSS 2-core |
| LAN | JB-S1 | CAM-08 | ~30 m จาก JB-S2 | Cat6 |
| LAN | JB-S3 | CAM-16 | ~45 m จาก JB-S1 | Cat6 |

> JB-N1, JB-N4, JB-S1, JB-S3 ไม่ต้องใช้ SFP — ใช้ PoE Switch ธรรมดา + Cat6 uplink ได้เลย

### Zone B — CORE-OFFICE (สำนักงาน)

```text
CORE-OFFICE
  ├── [Local PoE] CAM-17, 18  (Cat6 ตรง)
  ├── [North-West Chain B]  CORE-OFFICE → JB-N7 → JB-N6 → JB-N5 → JB-N3
  ├── [North-East Chain B]  CORE-OFFICE → JB-N8
  └── [South-East Chain B]  CORE-OFFICE → JB-S4 → JB-S5 → JB-S6
```

| Chain | JB | กล้อง | ระยะสาย (ประมาณ) |
| --- | --- | --- | --- |
| NW Chain B | JB-N7 | CAM-14 | 115 m จาก CORE-OFFICE |
| NW Chain B | JB-N6 | CAM-13 | 70 m จาก JB-N7 |
| NW Chain B | JB-N5 | CAM-12 | 70 m จาก JB-N6 |
| NW Chain B | JB-N3 | CAM-11 | 65 m จาก JB-N5 |
| NE Chain B | JB-N8 | CAM-19, CAM-20 | 65 m จาก CORE-OFFICE |
| SE Chain B | JB-S4 | CAM-21 | 100 m จาก CORE-OFFICE |
| SE Chain B | JB-S5 | CAM-22 | 70 m จาก JB-S4 |
| SE Chain B | JB-S6 | CAM-23 | 70 m จาก JB-S5 |

### Zone C — IDF-BACK (ป้อมหลัง)

```text
IDF-BACK
  ├── [Local PoE] CAM-28, 29, 30, 32  (Cat6 ตรง ไม่เกิน 30m)
  └── [South Chain C]  IDF-BACK → JB-S9 → JB-S8 → JB-S7
```

| Chain | JB | กล้อง | ระยะสาย (ประมาณ) |
| --- | --- | --- | --- |
| South C | JB-S9 | CAM-26, CAM-27 | 38 m จาก IDF-BACK |
| South C | JB-S8 | CAM-25 | 75 m จาก JB-S9 |
| South C | JB-S7 | CAM-24 | 75 m จาก JB-S8 |

---

## Main Node Switch — SFP Port Requirements

| Node | Backbone SFP | Camera Chain SFP | Local PoE | แนะนำ Switch |
| --- | --- | --- | --- | --- |
| MDF-FRONT | 1 (→ CORE-OFFICE) | 2 (North A + South A) | 5 กล้อง | 8-port PoE + 3 SFP |
| CORE-OFFICE | 2 (← MDF + → IDF) | 3 (NW N3-N7 + NE N8 + SE S4-S6) | 2 กล้อง | 16-port PoE + 5 SFP |
| IDF-BACK | 1 (← CORE-OFFICE) | 1 (South C S7-S9) | 4 กล้อง | 8-port PoE + 2 SFP |

---

## Failure Scenarios

| เหตุการณ์ | Backbone | Camera | ผลกระทบ |
| --- | --- | --- | --- |
| JB ไฟดับ (1 จุด) | ✅ ปกติ | ❌ กล้องใน chain นั้นหาย | ป้อมยังสื่อสารกันได้ |
| JB ไฟดับ ทั้ง chain | ✅ ปกติ | ❌ กล้องทั้ง chain หาย | ป้อมยังสื่อสารกันได้ |
| Backbone สายขาด 1 เส้น | ❌ node 2 ตัวติดต่อไม่ได้ | ✅ กล้องปกติ | ต้องซ่อม backbone |
| ไฟดับทั้งโครงการ | ✅ UPS ที่ป้อม/สนง. | ❌ กล้องบนเสาดับหมด | ป้อมยังโทรหากันได้ |

---

## Cable BOM

### Backbone (ใหม่ — dedicated)

| เส้น | ระยะ + slack 10% | ชนิดสาย |
| --- | --- | --- |
| MDF-FRONT → CORE-OFFICE | ~560 m | ADSS 2-core SM figure-8 |
| CORE-OFFICE → IDF-BACK | ~580 m | ADSS 2-core SM figure-8 |
| **รวม Backbone** | **~1,140 m** | |

### Camera Chains (Zone A — จาก MDF-FRONT)

| Segment | ระยะ + slack 10% |
| --- | --- |
| MDF-FRONT → JB-N2 | ~90 m |
| JB-N2 → JB-N1 | ~30 m |
| JB-N1 → JB-N4 | ~65 m |
| JB-N4 → JB-N3 | ~25 m |
| JB-N3 → JB-N5 | ~75 m |
| MDF-FRONT → JB-S2 | ~60 m |
| JB-S2 → JB-S1 | ~40 m |
| JB-S1 → JB-S3 | ~50 m |
| **รวม Zone A** | **~435 m** |

### Camera Chains (Zone B — จาก CORE-OFFICE)

| Segment | ระยะ + slack 10% |
| --- | --- |
| CORE-OFFICE → JB-N7 | ~125 m |
| JB-N7 → JB-N6 | ~75 m |
| CORE-OFFICE → JB-N8 | ~70 m |
| CORE-OFFICE → JB-S4 | ~110 m |
| **รวม Zone B** | **~380 m** |

### Camera Chains (Zone C — จาก IDF-BACK)

| Segment | ระยะ + slack 10% |
| --- | --- |
| IDF-BACK → JB-S9 | ~45 m |
| JB-S9 → JB-S8 | ~85 m |
| JB-S8 → JB-S7 | ~85 m |
| JB-S7 → JB-S6 | ~85 m |
| JB-S6 → JB-S5 | ~85 m |
| **รวม Zone C** | **~385 m** |

### สรุปสายทั้งหมด

| ประเภท | รวม |
| --- | --- |
| Backbone | ~1,140 m |
| Camera chains | ~1,200 m |
| **Grand Total ADSS 2-core** | **~2,340 m** |

---

## Equipment BOM

| รายการ | จำนวน | หมายเหตุ |
| --- | --- | --- |
| ADSS 2-core SM figure-8 | **~2,400 m** | เผื่อ ~60m สำหรับ offcuts |
| Junction Box IP66 + DIN-rail | **17 ชุด** | |
| Fiber Splice Closure 4-splice | **17 ชุด** | 2 สาย × 2 core ต่อ JB |
| PoE Switch 4-port + 2 SFP managed | **17 ชุด** | JBs (upstream + downstream SFP) |
| BiDi SFP module คู่ (1310/1550nm) | **36 คู่** | 17 JB × 2 port + 3 backbone links |
| DIN-rail PSU 220VAC→48VDC/60W | **17 ชุด** | |
| ELCB 10A + Surge Protector SPD II | **17 ชุด** | |
| Managed Switch 8-port PoE + 3 SFP | **2 ชุด** | MDF-FRONT, IDF-BACK |
| Managed Switch 16-port PoE + 5 SFP | **1 ชุด** | CORE-OFFICE |
| NVR 32ch | **1 ชุด** | CORE-OFFICE เท่านั้น — รับกล้องทั้ง 32 ตัว |
| UPS 1000VA | **2 ชุด** | MDF-FRONT, IDF-BACK (ไม่มี NVR แล้ว) |
| UPS 3000VA | **1 ชุด** | CORE-OFFICE (Switch + NVR 32ch + Server) |

---

## Per-Pole Power Design

```text
วงจรไฟเสา 220VAC
    ├── หลอด LED ส่องสว่าง (โหลดเดิม)
    └── ELCB 10A → Surge Protector → DIN-rail PSU (48VDC/60W)
                                              │
                                    PoE Switch 4-port + 2 SFP
                                    SFP-upstream ← สาย 2-core จาก node ก่อนหน้า
                                    SFP-downstream → สาย 2-core ไป node ถัดไป
                                              │
                                        กล้อง IP (PoE ~12W)
```

โหลดสูงสุดต่อ JB ≈ 84W — ไม่ต้องมี UPS ที่ JB เพราะไฟเปิด 24h กล้องจะกลับมาเองเมื่อไฟคืน

---

## VLAN Design

| VLAN | ใช้งาน | ผ่าน |
| --- | --- | --- |
| VLAN 10 | CCTV — กล้องทุกตัว + NVR | Camera chains + Backbone |
| VLAN 20 | Voice/Intercom ป้อม ↔ สำนักงาน | **Backbone เท่านั้น** |
| VLAN 30 | Management — Switch, NVR | Backbone + chains |
