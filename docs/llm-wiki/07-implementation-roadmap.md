# Implementation Roadmap — เวิร์กโฟลว์สร้าง llm-wiki-library แบบเป็นเฟส

> เอกสารนี้คือ **แผนลงมือทำ (delivery roadmap)** ที่พา "จากศูนย์ → หนึ่งเล่ม →
> ห้องสมุดเล็ก → ห้องสมุดที่สเกลได้" โดยร้อยทุกชั้นของโมเดลกลางเข้าด้วยกันเป็น
> **เฟสที่มี exit gate ชัดเจน (go/no-go)** — ตอบคำถามว่า "เริ่มตรงไหน ทำอะไรก่อน-หลัง
> อะไรบล็อกอะไร และจะรู้ได้อย่างไรว่าผ่านเฟสแล้ว"

> ยึดโมเดล 3 ชั้นใน [00-overview.md](00-overview.md) อย่างเคร่งครัด เอกสารนี้ **ไม่อธิบายซ้ำ**:
> ไปป์ไลน์ → [01-book-pipeline.md](01-book-pipeline.md) · หน่วยเล่ม → [02-llm-wiki-book.md](02-llm-wiki-book.md) ·
> หลักห้องสมุด → [03-physical-library-principles.md](03-physical-library-principles.md) ·
> การรวมร่าง → [04-llm-wiki-library.md](04-llm-wiki-library.md) ·
> ตัวอย่างลงมือทำ → [05-worked-example.md](05-worked-example.md) · schema → [06-schema-example.md](06-schema-example.md)
>
> เอกสารนี้ **เป็นเจ้าของเฉพาะ roadmap/เวิร์กโฟลว์** เท่านั้น —
> เรื่อง *ทีม/บทบาท/RACI/eval-gate* ดู [08-expert-team-and-governance.md](08-expert-team-and-governance.md) ·
> เรื่อง *tech stack/infra/ops* ดู [09-reference-stack-and-ops.md](09-reference-stack-and-ops.md)
> เราจะ **cross-link ไม่ลงลึก** ในสองหัวข้อนั้น
>
> แรงบันดาลใจเชิงแนวคิด: Andrej Karpathy, "LLM Wiki" — `raw/`=ซอร์ส, LLM=คอมไพเลอร์,
> `wiki/`=build artifact, `lint`=เทสต์, `query`=รันไทม์ ⇒ roadmap นี้คือ "การตั้งไลน์ผลิต
> แล้วค่อยขยายเป็นโรงงานหลายไลน์ + คลังสินค้า"

---

## 1. หลักคิดของ roadmap นี้

3 หลักที่กำกับลำดับของทุกเฟส:

1. **เล่มก่อนห้องสมุด (book-before-library):** อย่าสร้าง catalog/PID/cross-book
   ก่อนจะมีเล่มที่ "ฟอร์มดี" อย่างน้อยหนึ่งเล่ม — ชั้น 3 ตั้งอยู่บนชั้น 2 (ดู [04-llm-wiki-library.md](04-llm-wiki-library.md))
2. **ทำให้ซ้ำได้ก่อนสเกล (repeatable-before-scale):** เล่มที่สองต้องผลิตจาก
   *กระบวนการเดียวกัน* ไม่ใช่ฝีมือเฉพาะคน — แปลงไปป์ไลน์ให้เป็น "ผลิตภัณฑ์" ก่อนเพิ่มจำนวนเล่ม
3. **citation เป็นเงื่อนไขผ่าน ไม่ใช่ของแถม:** ทุก gate มีเกณฑ์ provenance —
   "no-citation, no-answer" (ดู [01-book-pipeline.md](01-book-pipeline.md) §2.4)

> หลัก compounding ของ Karpathy แปลงเป็น roadmap: ทุกเฟส **ต่อยอด artifact ของเฟสก่อน**
> ไม่รื้อทำใหม่ — pilot กลายเป็น template, template กลายเป็น product, product รองรับ scale

---

## 2. ภาพรวมเฟสและไทม์ไลน์ (ASCII)

```
 PHASE 0        PHASE 1         PHASE 2          PHASE 3         PHASE 4        PHASE 5
 Foundation     First Book      Pipeline-as-     Library Layer   Scale &        Continuous
 / Scope        (pilot)         Product          (catalog+PID)   Operate        Improvement
 │              │               │                │               │              │
 ▼              ▼               ▼                ▼               ▼              ▼
[scope/schema] [05 flow:       [repeatable      [catalog·PID·   [N เล่ม·       [metrics·
 raw policy]    ingest→synth→   ingest/synth/    citation·       cross-book·    re-compile·
                lint→query]     lint/query]      cross-book]     preservation]  schema vN]
 │              │               │                │               │              │
 ◇G0            ◇G1             ◇G2              ◇G3             ◇G4            ◇(loop)
 scope          BOOK DoD        pipeline         LIBRARY DoD     ops           health
 frozen         ผ่าน(02§8)      ซ้ำได้ ≥2 เล่ม    ผ่าน(04§9)       SLO ผ่าน        SLO คงที่
 │              │               │                │               │              │
 └─ สัปดาห์ 1-2 ─┴─ เดือน 1 ─────┴─ เดือน 2 ───────┴─ ไตรมาส 1 ─────┴─ ไตรมาส 2 ───┴─ ต่อเนื่อง ─►

 ◇ = exit gate (go/no-go)     เส้นทางหลักไหลซ้าย→ขวา; gate ที่ไม่ผ่าน = วนกลับเฟสเดิม
```

> หมายเหตุ: ใช้กรอบเวลา **สัมพัทธ์** (สัปดาห์/เดือน/ไตรมาส) ไม่ใช่วันที่จริง —
> ความเร็วจริงขึ้นกับขนาดทีมและ stack (ดู [08](08-expert-team-and-governance.md), [09](09-reference-stack-and-ops.md))

---

## 3. รายละเอียดรายเฟส

แต่ละเฟสระบุ: **วัตถุประสงค์ · กิจกรรมหลัก · ส่งมอบ · exit gate · ดึงจากเอกสารใด**

### 3.1 Phase 0 — Foundation / Scope (วางรากฐาน)

| ด้าน | รายละเอียด |
|------|-----------|
| **วัตถุประสงค์** | ตัดสินใจว่า "เล่มแรกคือเรื่องอะไร" และตั้งกติกาพื้นฐานก่อนแตะ raw |
| **กิจกรรมหลัก** | นิยาม scope เล่มแรก (one book = one topic), ร่าง schema เริ่มต้น (`CLAUDE.md`), ตั้งนโยบาย raw immutable + append-only log, กำหนดรูปแบบ citation, ตั้งชื่อ PID เล่มล่วงหน้า |
| **ส่งมอบ** | เอกสาร scope 1 หน้า · `schema` ฉบับ v0 · นโยบาย raw/citation · PID เล่ม (เช่น `lib:book/<topic>`) |
| **ดึงจาก** | [02-llm-wiki-book.md](02-llm-wiki-book.md) §3 (boundary), [06-schema-example.md](06-schema-example.md) (โครง schema), [00-overview.md](00-overview.md) §0 (Wiki-compile-centric) |

**Exit gate G0 — Scope frozen:**
- [ ] scope เล่มแรกเขียนเป็นลายลักษณ์ + ระบุ "นอกขอบเขต" ชัด (one topic)
- [ ] มี `schema` v0 ครบหัวข้อตามเช็กลิสต์ [06-schema-example.md](06-schema-example.md) §5
- [ ] นโยบาย raw immutable + รูปแบบ citation ตายตัว ตกลงแล้ว
- [ ] PID เล่มถูกจอง + ทีม/owner เริ่มต้นถูกระบุ (ราย role ดู [08](08-expert-team-and-governance.md))

---

### 3.2 Phase 1 — First Book / Pilot (เล่มแรก)

| ด้าน | รายละเอียด |
|------|-----------|
| **วัตถุประสงค์** | ผลิต llm-wiki-book "ที่ฟอร์มดี" หนึ่งเล่ม end-to-end เพื่อพิสูจน์ว่าไปป์ไลน์ใช้ได้จริง |
| **กิจกรรมหลัก** | เดินตาม flow ของ [05-worked-example.md](05-worked-example.md): ingest 2–3 source → synthesis หน้า wiki + citation → lint รอบแรก (orphan/contradiction/gap) → query + file-back |
| **ส่งมอบ** | หนึ่งเล่ม (`raw/` + `wiki/` + `schema`) ที่ผ่านเช็กลิสต์ [02-llm-wiki-book.md](02-llm-wiki-book.md) §8 · log การ lint รอบแรก · บันทึก "บทเรียน" เพื่อปรับ schema |
| **ดึงจาก** | [05-worked-example.md](05-worked-example.md) (ทั้งฉบับ), [01-book-pipeline.md](01-book-pipeline.md) (ทฤษฎีสเตจ), [02-llm-wiki-book.md](02-llm-wiki-book.md) §8 (DoD เล่ม) |

**Exit gate G1 — Book DoD ผ่าน (ดู §6.1):**
- [ ] ครบ `raw/`/`wiki/`/`schema`; `raw/` immutable; `wiki/` regenerate ได้จาก `raw/`
- [ ] ทุกข้ออ้างมี citation กลับ raw; orphan/broken-link/หน้าไร้ provenance ≈ 0 (lint ผ่าน)
- [ ] ขอบเขตหนึ่งหัวข้อชัด; query อ่านจาก wiki + file-back ได้จริง
- [ ] schema ถูกปรับเป็น v1 ตามบทเรียน pilot

---

### 3.3 Phase 2 — Pipeline-as-Product (ไปป์ไลน์ที่ซ้ำได้)

| ด้าน | รายละเอียด |
|------|-----------|
| **วัตถุประสงค์** | เปลี่ยน "ฝีมือทำเล่มแรก" ให้เป็น **กระบวนการที่ใครก็ทำซ้ำได้** กับเล่มที่สอง |
| **กิจกรรมหลัก** | ทำ runbook ของ ingest/synthesis/lint/query เป็นขั้นตอนตายตัว, ทำ schema เป็น **template** ที่ fork ไปเล่มใหม่ได้, ตั้งรอบ lint ประจำ, วาง eval ของแต่ละสเตจ (รายละเอียด eval-gate ดู [08](08-expert-team-and-governance.md); เครื่องมือ/ระบบ ดู [09](09-reference-stack-and-ops.md)) |
| **ส่งมอบ** | runbook ไปป์ไลน์ · schema template · เล่มที่สอง (พิสูจน์ว่าซ้ำได้) · เกณฑ์ลินต์ที่วัดได้ (orphan/contradiction/no-citation count) |
| **ดึงจาก** | [01-book-pipeline.md](01-book-pipeline.md) §6 (เช็กลิสต์รันไปป์ไลน์), [06-schema-example.md](06-schema-example.md) (schema → template) |

**Exit gate G2 — Pipeline ซ้ำได้:**
- [ ] เล่มที่สองผลิตด้วย **กระบวนการเดียวกัน** (ไม่ใช่ ad-hoc) และผ่าน Book DoD
- [ ] มี runbook + schema template ที่คนใหม่ทำตามแล้วได้เล่มฟอร์มดี
- [ ] รอบ lint ประจำทำงาน + ตัวเลขสุขภาพเล่ม (orphan/no-citation) เข้าเกณฑ์
- [ ] eval รายสเตจผ่านเกณฑ์ขั้นต่ำ (นิยามเกณฑ์ → [08](08-expert-team-and-governance.md))

---

### 3.4 Phase 3 — Library Layer (catalog + PID + citation ข้ามเล่ม)

| ด้าน | รายละเอียด |
|------|-----------|
| **วัตถุประสงค์** | ยกหลายเล่มขึ้นเป็น **ห้องสมุด**: ค้นเจอ จัดหมวด อ้างถึงกันได้ โดยคงขอบเขตเล่ม |
| **กิจกรรมหลัก** | สร้าง catalog/metadata (Dublin Core/MARC) + classification (DDC/LCC), ตั้ง **PID 2 ระดับ (เล่ม/หน้า) + resolver** กันลิงก์เน่า, เปิด cross-book citation (อ้างถึง ไม่ก๊อป), สร้าง union index + cross-book query (route → query/เล่ม → synthesize), เพิ่ม cross-book lint จับขัดแย้งข้ามเล่ม |
| **ส่งมอบ** | catalog record ทุกเล่ม · resolver ที่ใช้งานได้ · ตัวอย่าง cross-book citation ที่ resolve ผ่าน · cross-book query ที่ตอบ + แนบ citation หลายเล่ม |
| **ดึงจาก** | [04-llm-wiki-library.md](04-llm-wiki-library.md) §3–§5 (citation/catalog/cross-book), [03-physical-library-principles.md](03-physical-library-principles.md) (มาตรฐาน metadata/PID), [05-worked-example.md](05-worked-example.md) §8 (catalog record ตัวอย่าง) |

**Exit gate G3 — Library DoD ผ่าน (ดู §6.2):**
- [ ] ≥2 เล่มมี catalog record + PID เล่ม/หน้า + resolver ที่ลิงก์ไม่พังเมื่อย้ายไฟล์
- [ ] cross-book citation อย่างน้อยหนึ่งเส้น resolve ถูกต้อง (อ้างถึง ไม่ก๊อป)
- [ ] cross-book query ตอบจากหลายเล่ม + แนบ citation ของแต่ละเล่ม
- [ ] cross-book lint รายงานข้อขัดแย้งข้ามเล่มได้ + บันทึกการตัดสินใจ

---

### 3.5 Phase 4 — Scale & Operate (ขยายและดำเนินการ)

| ด้าน | รายละเอียด |
|------|-----------|
| **วัตถุประสงค์** | เพิ่มจำนวนเล่มให้ห้องสมุดโตได้แบบ modular โดยคุณภาพ/ความเชื่อถือไม่ตก |
| **กิจกรรมหลัก** | ออนบอร์ดเล่มใหม่ตามนโยบายคอลเลกชัน, จัดหมวดต่อเนื่อง, ตั้ง access control/ownership/license ต่อเล่ม, วาง preservation (สำรองหลายชุด + fixity/checksum), ตั้ง SLO ของ query/lint/resolver (infra/ops จริง → [09](09-reference-stack-and-ops.md); สิทธิ์/ธรรมาภิบาล → [08](08-expert-team-and-governance.md)) |
| **ส่งมอบ** | นโยบายออนบอร์ดเล่ม · access/license ต่อเล่ม · แผน+งานสำรองที่ทำงานจริง · dashboard สุขภาพห้องสมุด |
| **ดึงจาก** | [04-llm-wiki-library.md](04-llm-wiki-library.md) §4, §6 (governance ย่อ), [03-physical-library-principles.md](03-physical-library-principles.md) (preservation/circulation) |

**Exit gate G4 — พร้อมดำเนินการต่อเนื่อง:**
- [ ] เพิ่มเล่มใหม่ได้โดยไม่กระทบเล่มเดิม (modular, boundary คง)
- [ ] preservation ทำงาน: สำรอง ≥2 ชุด + ตรวจ fixity เป็นรอบ
- [ ] access/ownership/license กำหนดครบทุกเล่ม
- [ ] SLO ของ query/resolver/รอบ lint อยู่ในเกณฑ์ที่ตกลง

---

### 3.6 Phase 5 — Continuous Improvement (ปรับปรุงต่อเนื่อง · loop)

| ด้าน | รายละเอียด |
|------|-----------|
| **วัตถุประสงค์** | ทำให้ห้องสมุด **ฉลาดขึ้นเรื่อย ๆ** (compound) แทนที่จะนิ่งหรือเน่า |
| **กิจกรรมหลัก** | ใช้ query→file-back สะสมหน้าใหม่, re-compile `wiki/` เมื่อ schema เปลี่ยน (FRBR expression ใหม่ — ดู [06](06-schema-example.md) §4), วัด metric สุขภาพ + ปิด knowledge gap จาก lint, ปรับ schema เป็น vN, ทบทวนนโยบายคอลเลกชัน |
| **ส่งมอบ** | รายงาน metric เป็นรอบ · schema vN + เล่มที่ re-compile · บันทึกการปิด gap |
| **ดึงจาก** | [01-book-pipeline.md](01-book-pipeline.md) §2.4 (file-back/compound), [02-llm-wiki-book.md](02-llm-wiki-book.md) §4 (versioning), [04-llm-wiki-library.md](04-llm-wiki-library.md) §9 |

**Gate (loop) — Health คงที่:**
- [ ] metric สุขภาพ (orphan/no-citation/ขัดแย้งค้าง) ทรงตัว/ดีขึ้นทุกรอบ
- [ ] file-back เกิดจริงและผ่าน lint (ความรู้ compound ไม่ใช่ขยะบวม)
- [ ] re-compile จาก raw ได้เสมอเมื่อ schema เปลี่ยน (regenerable พิสูจน์ได้)

---

## 4. Milestones, ลำดับ และ effort สัมพัทธ์

| # | Milestone | เฟส | กรอบสัมพัทธ์ | effort สัมพัทธ์ | ส่งมอบหลัก |
|---|-----------|-----|--------------|-----------------|------------|
| M0 | scope + schema v0 + นโยบาย raw | 0 | สัปดาห์ 1-2 | S | เอกสาร scope, schema v0 |
| M1 | เล่มแรกผ่าน Book DoD | 1 | เดือน 1 | L | หนึ่งเล่มฟอร์มดี (05 flow) |
| M2 | runbook + schema template + เล่มที่สอง | 2 | เดือน 2 | L | ไปป์ไลน์ซ้ำได้ |
| M3 | catalog + PID + resolver | 3 | ไตรมาส 1 | M | library layer ใช้งานได้ |
| M4 | cross-book citation + query + lint | 3 | ไตรมาส 1 (ปลาย) | M | Library DoD ผ่าน |
| M5 | preservation + access + SLO | 4 | ไตรมาส 2 | M | พร้อม operate |
| M6 | metric loop + re-compile + schema vN | 5 | ต่อเนื่อง | S/รอบ | compound ต่อเนื่อง |

> effort: S=เล็ก, M=กลาง, L=ใหญ่ (สัมพัทธ์ ไม่ใช่หน่วยเวลา) — งานหนักจริงกระจุกที่
> **M1/M2 (เขียน wiki + ทำให้ซ้ำได้)** ตรงกับที่ [01-book-pipeline.md](01-book-pipeline.md) §4 ชี้ว่า
> "งานหนักอยู่ที่ ingest + synthesis" ไม่ใช่ตอน query

---

## 5. Dependencies & Critical Path

```
G0 ──► G1 ──► G2 ──► G3 ──► G4 ──► (G5 loop)
scope  book   repeat  library ops    improve
        DoD    pipeline DoD
                              ▲
   schema v0 ─────────────────┘ (schema เป็น input ของทุกเฟส; vN วนปรับใน P5)
```

**Critical path (สิ่งที่บล็อกสิ่งถัดไป):**

| ต้องเสร็จก่อน | จึงจะเริ่ม | เหตุผล |
|----------------|-----------|--------|
| scope + schema (G0) | ingest เล่มแรก (P1) | ไม่มี scope/citation rule → wiki เพี้ยน, raw ไม่มีระเบียบ |
| เล่มแรกฟอร์มดี (G1) | ทำ pipeline-as-product (P2) | ต้องมี "ของจริง" ก่อนจึงถอดเป็นกระบวนการ |
| ไปป์ไลน์ซ้ำได้ + ≥2 เล่ม (G2) | library layer (P3) | catalog/cross-book ต้องมีหลายเล่มที่ boundary คง (ดู [04](04-llm-wiki-library.md) §5) |
| PID + resolver (P3) | cross-book citation/query (P3) | ลิงก์ข้ามเล่มต้องมี PID indirection ก่อน ไม่งั้นลิงก์เน่า ([04](04-llm-wiki-library.md) §3.3) |
| Library DoD (G3) | scale & operate (P4) | สเกลก่อนระเบียบพร้อม = หนี้ทางเทคนิคทบต้น |

> **ทำขนานได้:** ภายใน P1 การ ingest source หลายชิ้นทำขนานกันได้; ภายใน P3
> งาน catalog/metadata กับงาน PID/resolver เดินขนานแล้วมาบรรจบที่ cross-book citation

---

## 6. Definition of Done (DoD) รายชั้น

DoD นี้ **อ้างอิงตรง** เช็กลิสต์ใน 02 และ 04 เพื่อให้ gate สอดคล้องกับโมเดลกลาง

### 6.1 Book DoD (ชั้น 2 — ใช้ที่ G1 และทุกเล่มใหม่ใน P4)

อิงเช็กลิสต์ [02-llm-wiki-book.md](02-llm-wiki-book.md) §8:

- [ ] ครบ `raw/` + `wiki/` + `schema`; `raw/` immutable (append-only)
- [ ] `wiki/` regenerate ได้ทั้งหมดจาก `raw/` (ไม่มีความรู้ลอยไร้ raw)
- [ ] มี `index.md` ครอบทุกหน้า + `log.md` append-only
- [ ] ทุกข้ออ้างมี citation กลับ `raw/`; ไม่มี orphan/broken-link ค้าง (lint ผ่าน)
- [ ] ขอบเขตหนึ่งหัวข้อชัด; schema ระบุ scope/naming/citation/lint
- [ ] query อ่านจาก `wiki/` (ไม่ค้นสด raw) + file-back ได้
- [ ] เล่มมี identity + boundary คม → หยิบเข้าชั้น 3 ได้

### 6.2 Library DoD (ชั้น 3 — ใช้ที่ G3)

อิงเช็กลิสต์ [04-llm-wiki-library.md](04-llm-wiki-library.md) §9:

- [ ] มี ≥1 เล่มที่ผ่าน Book DoD (ปกติ ≥2 เพื่อให้ cross-book มีความหมาย)
- [ ] บังคับ citation ภายในเล่มทุกเล่ม (no-citation, no-answer)
- [ ] มี PID ระดับเล่ม + ระดับหน้า + resolver (กันลิงก์เน่า)
- [ ] มี catalog/metadata (Dublin Core/MARC) + classification (DDC/LCC)
- [ ] มี union index → route คำถามข้ามเล่ม
- [ ] cross-book citation (อ้างถึง ไม่ก๊อป) + cross-book lint จับขัดแย้ง
- [ ] cross-book query: route → query/เล่ม → synthesize + แนบ citation ทุกเล่ม
- [ ] กำหนด ownership/สิทธิ์/license ต่อเล่ม + วางแผน preservation (fixity/สำรอง)
- [ ] file-back ข้ามเล่มเขียนกลับเป็นหน้าใหม่ในเล่มที่เหมาะสม

---

## 7. ความเสี่ยงต่อการส่งมอบ + การบรรเทา (ย่อ)

โฟกัสที่ความเสี่ยง **เชิง delivery/ลำดับงาน** — ธรรมาภิบาลเชิงลึก → [08](08-expert-team-and-governance.md), infra → [09](09-reference-stack-and-ops.md)

| ความเสี่ยง | เฟสที่เสี่ยง | ผลต่อการส่งมอบ | การบรรเทา (สั้น) |
|-----------|--------------|----------------|------------------|
| ข้าม G1 ไปทำ catalog เลย | 0→3 | ห้องสมุดว่างเปล่า ไม่มีของจริง | บังคับ book-before-library; G ต้องผ่านตามลำดับ |
| เล่มแรกเป็นฝีมือเฉพาะคน | 1→2 | เล่มที่สองทำไม่ได้ สเกลตัน | P2 ต้องพิสูจน์ "เล่มที่สองจากกระบวนการเดิม" |
| scope creep ของเล่ม | 1,4 | boundary เบลอ จัดหมวด/อ้างข้ามเล่มไม่ได้ | คุมด้วย schema scope (ดู [06](06-schema-example.md) §1) |
| ผูกลิงก์ข้ามเล่มกับ path ตรง | 3 | ลิงก์เน่าเมื่อย้ายไฟล์ | ตั้ง PID + resolver **ก่อน** เปิด cross-book ([04](04-llm-wiki-library.md) §3.3) |
| lint ถูกข้ามตอนเร่งสเกล | 4 | คุณภาพตก ความรู้ขัดแย้งสะสม | รอบ lint เป็นส่วนของ DoD เล่มใหม่ ไม่ใช่ทางเลือก |
| file-back กลายเป็นขยะบวม | 5 | wiki โตแต่คุณภาพตก | หน้า file-back ต้องผ่าน lint รอบถัดไปเสมอ ([01](01-book-pipeline.md) §2.4) |

---

## 8. Master Checklist — rollout ทั้งโครงการ

ลำดับเช็กจากบนลงล่าง; ขีดได้ครบในกลุ่ม = ผ่าน gate ของเฟสนั้น

```
[ ] P0  scope + นอกขอบเขต เขียนชัด                              ─┐
[ ] P0  schema v0 ครบ (06 §5) + นโยบาย raw/citation + PID เล่ม   ─┴► G0 scope frozen
[ ] P1  ingest 2–3 source (raw immutable + log)                ─┐
[ ] P1  synthesis หน้า wiki + citation กลับ raw                 │
[ ] P1  lint รอบแรกผ่าน (orphan/contradiction/gap)              │
[ ] P1  query + file-back ทำงาน → Book DoD (§6.1)              ─┴► G1
[ ] P2  runbook ไปป์ไลน์ + schema template                     ─┐
[ ] P2  เล่มที่สองจากกระบวนการเดิม + eval รายสเตจผ่าน            ─┴► G2 repeatable
[ ] P3  catalog/metadata + classification ทุกเล่ม              ─┐
[ ] P3  PID เล่ม/หน้า + resolver (กันลิงก์เน่า)                  │
[ ] P3  cross-book citation + union index + cross-book query   │
[ ] P3  cross-book lint → Library DoD (§6.2)                   ─┴► G3
[ ] P4  นโยบายออนบอร์ดเล่ม + access/license ต่อเล่ม             ─┐
[ ] P4  preservation (สำรอง ≥2 + fixity) + SLO query/resolver  ─┴► G4 operate
[ ] P5  metric loop + file-back compound                       ─┐
[ ] P5  re-compile เมื่อ schema เปลี่ยน + schema vN             ─┴► (loop) health
```

**แก่นของ roadmap:** เดินตามลำดับ **G0→G1→G2→G3→G4→loop** โดยทุก gate ยึดหลักเดียวกับ
ทั้งชุด — **raw คือความจริง, ทุกคำตอบสาวกลับได้, หนึ่งเล่มหนึ่งหัวข้อ** — เริ่มจากเล่มเดียว
ที่ฟอร์มดี แล้ว *ต่อยอด* (ไม่รื้อ) เป็นกระบวนการที่ซ้ำได้ แล้วจึงครอบด้วยระเบียบห้องสมุด
จนสเกลและ compound ได้อย่างเชื่อถือ

---

> เอกสารพี่น้องที่เติมเต็ม roadmap นี้: ทีม/บทบาท/RACI/eval-gate →
> [08-expert-team-and-governance.md](08-expert-team-and-governance.md) ·
> tech stack/infra/ops → [09-reference-stack-and-ops.md](09-reference-stack-and-ops.md) ·
> โมเดลกลาง → [00-overview.md](00-overview.md)
>
> แรงบันดาลใจเชิงแนวคิด: Andrej Karpathy, "LLM Wiki"
> ([gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f), 2025–2026)
