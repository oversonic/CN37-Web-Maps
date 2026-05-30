# ผังหมู่บ้านจัดสรร The Connect 37

Interactive map viewer สำหรับโครงการ **The Connect 37** — หมู่บ้านจัดสรร 495 หลัง พร้อมระบบ Fiber Network และ CCTV 32 ตัว

---

## Live Demo

เปิด `docs/index.html` ตรงในเบราว์เซอร์ได้เลย — ไม่ต้องใช้ server (SVG อยู่ใน file)

```
open docs/index.html
```

หรือ serve ผ่าน GitHub Pages จาก branch `main` → folder `docs/`

---

## Features

| Feature | รายละเอียด |
| --- | --- |
| 🗺️ Vector floor plan | SVG ผังบ้านทับบนแผนที่ OSM / Esri satellite |
| 📍 GPS-calibrated | ผังถูก align ด้วย 2-point transform จาก GPS จริง |
| 🔄 Pan / Zoom / Rotate | ลากแผนที่, scroll zoom, หมุน ±15° หรือ reset |
| 📡 CCTV markers | กล้อง 32 ตัว (CAM-01–CAM-32) พร้อม tooltip |
| 🔌 Fiber layer | เส้น Backbone + Camera Chains + LAN cross-connect |
| 🏠 Collapsible sidebar | กด ☰ ใน header เพื่อยุบ/ขยาย sidebar |
| 📱 Responsive | รองรับมือถือและ tablet |

---

## ข้อมูลโครงการ

| | |
| --- | --- |
| ชื่อโครงการ | The Connect 37 (CN37) |
| จำนวนหลัง | 495 หลัง |
| ซอย | 28 ซอย |
| ป้อม รปภ. | 2 จุด (หน้า/หลัง) |
| กล้อง CCTV | 32 ตัว |
| พื้นที่ | Lat 13.913–13.920 · Lng 100.585–100.598 |

---

## การใช้งาน

### Sidebar Controls

| Panel | ทำอะไร |
| --- | --- |
| ประเภทที่อยู่อาศัย | Legend สีแต่ละประเภทบ้าน |
| การแสดงผล | toggle แผนที่พื้นหลัง / พื้นขาวผัง |
| กล้องวงจรปิด | แสดง/ซ่อนตำแหน่งกล้อง 32 ตัว |
| Fiber & Junction Box | แสดง/ซ่อน layer สาย Fiber + JB |

### Map Controls (มุมขวาบน)

| ปุ่ม | ทำอะไร |
| --- | --- |
| ผังโครงการ (slider) | ปรับความโปร่งใสผัง 0–100% |
| ↺ / ↻ | หมุนแผนที่ ±15° |
| ⊙ | Reset การหมุนกลับ 338° |
| + / − | Zoom in / out |
| ⤢ | Fit bounds ทั้งโครงการ |

### Fiber Layer สี

| สี | ความหมาย |
| --- | --- |
| 🔵 น้ำเงินหนา | Backbone — ป้อมหน้า ↔ สำนักงาน ↔ ป้อมหลัง |
| 🟠 ส้มทึบ | Camera chain (fiber ADSS 2-core) |
| 🟠 ส้มประ | LAN Cat6 cross-connect |

---

## โครงสร้างไฟล์

```
CN37-Map-Web/
├── docs/
│   └── index.html          # Primary app (self-contained)
├── CCTV_DESIGN.md          # Fiber & CCTV infrastructure design
├── COST_ESTIMATE.md        # Equipment price estimate (ราคาตลาดไทย 2025)
├── CN37_CCTV_Design.docx   # Word document สรุปการออกแบบ
├── CNfiber_coordinates_v2.csv  # GPS พิกัดกล้อง 32 ตัว (source of truth)
├── CN37_MasterPlan3_cctv.svg   # Source SVG จาก Illustrator
└── CN37_masterplan.html    # Legacy version (ต้องใช้ HTTP server)
```

---

## CCTV & Fiber Design

### Network Architecture

```
[ป้อมหน้า MDF-FRONT] ──backbone──► [สำนักงาน CORE-OFFICE] ──backbone──► [ป้อมหลัง IDF-BACK]
         │                                    │                                    │
    Zone A chains                       Zone B chains                       Zone C chains
  (CAM-01–16)                         (CAM-17–23)                         (CAM-24–32)
```

- **Backbone**: ADSS 2-core SM ตรง ไม่ผ่าน JB — รับประกันสื่อสารป้อม/สำนักงาน
- **Camera Chains**: Daisy-chain ผ่าน JB — ถ้า JB ดับ กล้องหายแต่ป้อมยังสื่อสารได้
- **NVR**: 1 จุดที่สำนักงาน (32ch) รับกล้องทั้งหมด

รายละเอียดเพิ่มเติม → [`CCTV_DESIGN.md`](CCTV_DESIGN.md)

ประมาณราคาอุปกรณ์ → [`COST_ESTIMATE.md`](COST_ESTIMATE.md)

---

## Tech Stack

- [Leaflet.js 1.9.4](https://leafletjs.com/) — map engine
- [leaflet-rotate 0.2.8](https://github.com/Raruto/leaflet-rotate) — map rotation
- OpenStreetMap / Esri World Imagery — tile layers
- Inline SVG — floor plan (ไม่ต้อง fetch ไฟล์แยก)
- Vanilla HTML/CSS/JS — no build step, no dependencies to install

---

## GitHub Pages

เปิด Settings → Pages → Source: **Deploy from branch** → Branch: `main` / Folder: `/docs`

URL จะเป็น: `https://<username>.github.io/<repo>/`
