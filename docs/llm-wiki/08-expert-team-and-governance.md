# ทีมผู้เชี่ยวชาญและธรรมาภิบาล — Expert Team & Governance

> เอกสารนี้ตอบคำถามว่า **"ใครคือคนที่สร้างและดูแล llm-wiki-library"** และ
> **"ใช้กติกาอะไรกำกับคุณภาพ/การเปลี่ยนแปลง"** — กล่าวคือ องค์ประกอบ **ทีมผู้เชี่ยวชาญ**
> (บทบาท/ความรับผิดชอบ/ทักษะ), ตาราง **RACI** บทบาท × กิจกรรม, **โมเดลธรรมาภิบาล**
> (editorial governance, human-in-the-loop, no-citation-no-answer, version control แบบ FRBR),
> **quality gates** ระดับเล่ม/ห้องสมุด, และตาราง **ความเสี่ยง+เจ้าของ**

> หมายเหตุการเชื่อมโยง: โมเดลกลางอยู่ที่ [00-overview.md](00-overview.md) (โมเดล 3 ชั้น) —
> โปรดอ่านก่อน เอกสารนี้ **ไม่นิยามใหม่** ว่าไปป์ไลน์/หนังสือ/ห้องสมุดคืออะไร แต่ map "คน" และ
> "กติกา" ลงบนชั้นเหล่านั้น โดยอ้างจาก [01-book-pipeline.md](01-book-pipeline.md) §5 (ความเสี่ยงรายสเตจ),
> [03-physical-library-principles.md](03-physical-library-principles.md) (FRBR/preservation),
> และ [04-llm-wiki-library.md](04-llm-wiki-library.md) §6 (governance & trust)
> — **ลำดับเฟส/timeline/gate** เป็นของ [07-implementation-roadmap.md](07-implementation-roadmap.md)
> และ **tech/infra/ops** เป็นของ [09-reference-stack-and-ops.md](09-reference-stack-and-ops.md)
> ที่นี่จะ cross-link แทนการอธิบายซ้ำ

---

## 1. ภาพรวม: ทีมเล็กที่ครอบทั้ง 3 ชั้น

ทีมที่ดีไม่จำเป็นต้องมีคนเยอะ แต่ต้องครอบ **ทั้ง 3 ชั้น** ของโมเดล (pipeline / book / library)
และมีเส้นแบ่ง **เจ้าของ (owner)** ชัดเจนในทุกกิจกรรม หนึ่งคนอาจสวมหลายหมวกในทีมเล็กได้
แต่ "เจ้าของ schema", "ผู้อนุมัติ scope", และ "ผู้รีวิวเชิงข้อเท็จจริง (SME)" ควรแยกบทบาท
เพื่อไม่ให้ผู้สร้างเนื้อหาตรวจงานตัวเอง

```
                ┌──────────────────────────────────────────────┐
                │  Product / Program Lead  (เจ้าของผลลัพธ์รวม)   │
                └───────────────┬──────────────────────────────┘
        ┌───────────────────────┼───────────────────────────┐
        ▼                       ▼                           ▼
  ชั้น 1 · PIPELINE        ชั้น 2 · BOOK              ชั้น 3 · LIBRARY
  Knowledge Engineer     LLM/Prompt & Schema Eng.    Information Architect /
  (ingest→synthesis      (schema = "สเปกคอมไพเลอร์")  Librarian (catalog,
   →lint→query owner)                                 classification, FRBR)
        │                       │                           │
        ├── Citation / Provenance Steward ── (คร่อมทุกชั้น: page→raw, cross-book)
        ├── Subject-Matter Experts / Reviewers ── (human-in-the-loop)
        ├── Platform / SRE ── (โครงสร้างพื้นฐาน → ดู 09)
        └── Legal / Rights & Privacy ── (license, สิทธิ์เข้าถึง, PII)
```

> เทียบวิทยาการคอมพิวเตอร์: ถ้า pipeline = คอมไพเลอร์ (Karpathy: raw=ซอร์ส, LLM=คอมไพเลอร์,
> wiki=artifact) ทีมนี้ก็เปรียบได้กับทีม **platform + compiler + release engineering + librarian**
> ที่ดูแลทั้ง toolchain, artifact registry, และ catalog

---

## 2. องค์ประกอบทีมและบทบาท (Roles)

### 2.1 Knowledge Engineer / Pipeline Owner

- **เป็นเจ้าของชั้น:** ชั้น 1 — ไปป์ไลน์ **ingest → synthesis → lint → query**
- **ความรับผิดชอบ:** รัน/ดูแลไปป์ไลน์ผลิตเล่ม, ตั้งรอบ lint, เปิด file-back, จับ failure modes
  รายสเตจ (provenance หาย, หน้าซ้ำ, orphan, ตอบนอกวิกิ — ดู [01-book-pipeline.md](01-book-pipeline.md) §2,§5)
- **ทักษะ:** orchestration/automation, ความเข้าใจ LLM workflow, data wrangling, เข้าใจ compiler-analogy

### 2.2 Information Architect / Librarian

- **เป็นเจ้าของชั้น:** ชั้น 3 — catalog, classification, metadata, authority control, FRBR
- **ความรับผิดชอบ:** ออกแบบ catalog/metadata (Dublin Core/MARC), classification (DDC/LCC),
  **authority control** (ชื่อ entity/หัวเรื่องให้เป็นมาตรฐานเดียว), แยกระดับ FRBR
  (Work/Expression/Manifestation/Item — ดู [03-physical-library-principles.md](03-physical-library-principles.md) §6),
  นโยบายพัฒนาคอลเลกชัน (รับเข้า/คัดออก)
- **ทักษะ:** library/information science, metadata schema, taxonomy/ontology, union catalog

### 2.3 Citation / Provenance Steward

- **เป็นเจ้าของ:** เส้นเลือด citation **คร่อมทุกชั้น** (page→raw ภายในเล่ม + cross-book)
- **ความรับผิดชอบ:** บังคับกฎ **no-citation, no-answer** ([04-llm-wiki-library.md](04-llm-wiki-library.md) §6),
  ดูแล **PID** ระดับเล่ม/หน้า + resolver (กัน link rot), ตรวจ provenance chain `raw→ingest→synthesis`,
  ตรึงระดับ FRBR ของการอ้างอิงไม่ให้กำกวม
- **ทักษะ:** bibliographic/PID standards (DOI/Handle/ARK), provenance modeling, link integrity

### 2.4 LLM / Prompt & Schema Engineer

- **เป็นเจ้าของชั้น:** ชั้น 2 — **schema** (กติกาบรรณาธิการของเล่ม = "สเปกของคอมไพเลอร์")
- **ความรับผิดชอบ:** เขียน/ดูแล schema ต่อเล่ม (โครงหน้า, naming/slug, รูปแบบ citation, lint rules,
  ส่วน Do-NOT — ดู [06-schema-example.md](06-schema-example.md)), ออกแบบ prompt ของแต่ละสเตจ,
  เวอร์ชัน schema (โยง expression ของ FRBR), บังคับกฎ "ตอบจากวิกิเท่านั้น" ใน query
- **ทักษะ:** prompt/schema design, evaluation/eval harness, ความเข้าใจ failure modes ของ LLM

### 2.5 Platform / SRE

- **เป็นเจ้าของ:** โครงสร้างพื้นฐานที่รันทั้งหมด — **รายละเอียดอยู่ที่ [09-reference-stack-and-ops.md](09-reference-stack-and-ops.md)**
- **ความรับผิดชอบ (สรุป):** storage ของ `raw/` immutable, backup/replication + fixity (preservation),
  resolver service, ความพร้อมใช้ของ query, observability — เนื้อหาเชิงเทคนิค **defer ไป 09**
- **ทักษะ:** SRE/DevOps, storage & backup, IaC, security ops

### 2.6 Subject-Matter Experts / Reviewers (Human-in-the-loop)

- **เป็นเจ้าของ:** ความถูกต้องเชิงข้อเท็จจริง — เป็น "คน" ในกฎ human-in-the-loop
- **ความรับผิดชอบ:** รีวิวการแก้เชิงข้อเท็จจริงที่ lint เสนอ (lint *เสนอ* คนอนุมัติ —
  [01-book-pipeline.md](01-book-pipeline.md) §2.3), ตัดสินข้อขัดแย้งในเล่มและ **ข้ามเล่ม**,
  ยืนยันคุณภาพก่อน release
- **ทักษะ:** เชี่ยวชาญหัวข้อเล่มนั้น, การประเมินหลักฐาน/แหล่งที่มา

### 2.7 Product / Program Lead

- **เป็นเจ้าของ:** ผลลัพธ์รวม, ขอบเขต (scope) ของแต่ละเล่ม, การอนุมัติเปลี่ยน scope, จังหวะ release
- **ความรับผิดชอบ:** กำหนด "หนึ่งหัวข้อ = หนึ่งเล่ม", คุม scope creep, จัด cadence รีวิว,
  เชื่อมกับ roadmap/เฟส ([07-implementation-roadmap.md](07-implementation-roadmap.md))
- **ทักษะ:** product/program management, การจัดลำดับความสำคัญ, การเจรจาขอบเขต

### 2.8 Legal / Rights & Privacy

- **เป็นเจ้าของ:** license ของ raw, สิทธิ์เข้าถึง (access model), ความเป็นส่วนตัว (PII)
- **ความรับผิดชอบ:** ตรวจสิทธิ์ก่อน ingest, บันทึก license ใน metadata เล่ม, กำหนด open/จำกัดต่อเล่ม
  (ดู [03-physical-library-principles.md](03-physical-library-principles.md) §8), จัดการ PII/redaction
- **ทักษะ:** IP/licensing, data privacy/compliance, การจัดการความเสี่ยง

---

## 3. ตาราง RACI (บทบาท × กิจกรรม)

R = Responsible (ลงมือทำ) · A = Accountable (เจ้าของ/ผู้รับผิด ชอบ — มีได้คนเดียว) ·
C = Consulted (ปรึกษา) · I = Informed (แจ้งให้ทราบ)

| กิจกรรม \ บทบาท | Knowledge Eng | InfoArch/Librarian | Citation Steward | LLM/Schema Eng | Platform/SRE | SME/Reviewer | Product Lead | Legal/Rights |
|---|---|---|---|---|---|---|---|---|
| **Ingest** (นำเข้า raw) | A/R | I | C | C | R | I | I | C |
| **Synthesis** (เขียนวิกิ) | A/R | C | C | R | I | C | I | I |
| **Lint** (ตรวจสุขภาพ) | A/R | C | C | R | I | C | I | I |
| **Schema authoring** | C | C | C | A/R | I | C | C | I |
| **Cataloging / metadata** | I | A/R | C | C | I | C | I | I |
| **Citation / PID** | C | C | A/R | C | R | I | I | I |
| **Cross-book review** | R | C | C | C | I | A/R | C | I |
| **Preservation / backup** | I | C | C | I | A/R | I | I | C |
| **Release / publish** | R | R | C | C | C | C | A | C |
| **Scope change** | C | C | I | C | I | C | A/R | C |
| **Rights / privacy clearance** | C | I | C | I | I | I | C | A/R |

> ออกแบบให้ **A ของแต่ละกิจกรรมไม่ทับกับ R ของผู้ที่ตรวจงานนั้น** เช่น Synthesis เป็นของ
> Knowledge Eng แต่ความถูกต้องเชิงข้อเท็จจริงตัดสินโดย SME (cross-book review) — กันคนตรวจงานตัวเอง

---

## 4. โมเดลธรรมาภิบาล (Governance)

### 4.1 Editorial governance — ใครเป็นเจ้าของอะไร

- **schema เป็นของ LLM/Schema Engineer** การแก้ schema ต้องผ่าน review (เพราะ schema = สเปกคอมไพเลอร์
  เปลี่ยนแล้วกระทบทั้งเล่ม) และถือเป็น **expression ใหม่** (โยง FRBR §4.4)
- **scope ของเล่มเป็นของ Product Lead** การเปลี่ยน scope (รับหัวข้อใหม่/แตกเล่ม/รวมเล่ม) ต้องอนุมัติ
  อย่างเป็นทางการ เพื่อกัน scope creep ([01-book-pipeline.md](01-book-pipeline.md) §5)
- **catalog/authority control เป็นของ Librarian** — การเพิ่ม/จัดหมวดเล่มใหม่ผ่านนโยบายพัฒนาคอลเลกชัน

### 4.2 Human-in-the-loop สำหรับการแก้เชิงข้อเท็จจริง

จาก [01-book-pipeline.md](01-book-pipeline.md) §2.3: **lint *เสนอ* การแก้ — ไม่ apply เองกับเรื่องสำคัญ**
การแก้ที่เป็น *ข้อเท็จจริง* (contradiction, stale claim, การลบหน้าที่มีค่า) ต้องมี **SME อนุมัติ**
ก่อนเขียนกลับ ส่วนการแก้เชิง *โครงสร้าง* (ร้อยลิงก์ orphan, สร้าง stub) ให้ LLM ทำอัตโนมัติได้
แต่ยังต้องผ่านรอบ lint

### 4.3 No-citation, no-answer

กติกาเหล็กจาก [04-llm-wiki-library.md](04-llm-wiki-library.md) §6: **ไม่มี citation = ไม่ปล่อยคำตอบออก**
ทุกหน้า wiki ต้องผูกกลับ `raw/` ทุกคำตอบ query ต้องแนบที่มา (ในเล่ม/ข้ามเล่ม) — Citation Steward
เป็นผู้รักษากฎนี้ และ lint เป็นด่านจับหน้า/คำตอบที่ขาดที่มา

### 4.4 Change / version control ของเล่มและ schema (โยง FRBR)

ผูกการควบคุมเวอร์ชันเข้ากับ FRBR ([03-physical-library-principles.md](03-physical-library-principles.md) §6):

| สิ่งที่เปลี่ยน | ระดับ FRBR | กติกาควบคุม | เจ้าของอนุมัติ |
|---|---|---|---|
| หัวข้อ/ขอบเขตเล่ม | **Work** | scope change ต้องอนุมัติเป็นทางการ | Product Lead |
| เนื้อหา wiki หลัง synthesis รอบหนึ่ง | **Expression** | บันทึกเวอร์ชันเนื้อหา + log | Knowledge Eng |
| แก้ schema (โครง/กติกา) | **Expression** ใหม่ | review + bump เวอร์ชัน schema | LLM/Schema Eng |
| snapshot/release ที่เผยแพร่ | **Manifestation** | แท็ก release + fixity/checksum | Product Lead + SRE |
| สำเนาที่ติดตั้งจริง | **Item** | inventory/preservation | Platform/SRE |

> `raw/` immutable เสมอ (append-only) — การเปลี่ยนทั้งหมดเกิดที่ชั้น wiki/schema ไม่ใช่ที่ความจริงต้นทาง

### 4.5 Review cadence (จังหวะรีวิว)

| รอบ | ทำอะไร | ผู้นำ |
|---|---|---|
| ต่อ ingest | บันทึก provenance + dedupe | Knowledge Eng |
| รายสัปดาห์ | lint ในเล่ม + อนุมัติการแก้ข้อเท็จจริง | Knowledge Eng + SME |
| รายเดือน | cross-book lint (ขัดแย้ง/link health) + authority control | Librarian + Citation Steward |
| ต่อ release | quality gates §5 + rights/privacy clearance | Product Lead + Legal |
| รายไตรมาส | นโยบายคอลเลกชัน (รับเข้า/คัดออก) + ทบทวน schema | Product Lead + Librarian |

---

## 5. Quality Gates / การประเมินคุณภาพ

วัดทั้ง **ระดับเล่ม** (book) และ **ระดับห้องสมุด** (library) — เกณฑ์เชิงคุณภาพก็พอ แต่ต้องมีเจ้าของวัด

| ตัวชี้วัด | ระดับ | เป้าหมาย (threshold) | เจ้าของวัด |
|---|---|---|---|
| Orphan pages (หน้ากำพร้า) | เล่ม | → **0** | Knowledge Eng |
| Broken cross-link / link rot | เล่ม + ห้องสมุด | → **0** | Citation Steward |
| Pages without citation | เล่ม | → **0** (no-citation-no-answer) | Citation Steward |
| Missing provenance (หน้าไร้ที่มา) | เล่ม | → **0** | Citation Steward |
| Contradiction rate (ในเล่ม) | เล่ม | ใกล้ 0; ทุกข้อมี note การตัดสิน | SME/Reviewer |
| Cross-book contradiction | ห้องสมุด | รายงานครบ + คลี่คลายมีบันทึกเหตุผล | SME + Librarian |
| Cross-book link health | ห้องสมุด | PID resolve ได้ 100% | Citation Steward |
| Answer faithfulness (ตอบจากวิกิ ไม่ใช่ความจำ) | เล่ม | สูง; ตัวอย่างสุ่มผ่าน eval | LLM/Schema Eng |
| Citation accuracy (ชี้ถูกหน้า/raw) | เล่ม + ห้องสมุด | สูง; sampling ตรวจ | Citation Steward |
| Stale-claim ratio | เล่ม | ต่ำ; ของเก่าถูก mark + ชี้หน้าใหม่ | Knowledge Eng |
| FRBR clarity (citation ตรึงระดับชัด) | ห้องสมุด | ไม่มี citation กำกวม | Librarian |
| Catalog/metadata completeness | ห้องสมุด | ทุกเล่มมี metadata + classification ครบ | Librarian |
| Preservation (fixity ผ่าน + backup ครบ) | ห้องสมุด | checksum ตรง, สำเนา ≥ N | Platform/SRE |

> Gate การ release: เล่มจะ "ขึ้นชั้น" ได้ก็ต่อเมื่อผ่าน gate ระดับเล่ม (orphan/broken/citation = 0)
> และเข้าห้องสมุดได้เมื่อ metadata + PID + cross-book link health ผ่าน — **ลำดับเฟสของ gate เหล่านี้
> อยู่ที่ [07-implementation-roadmap.md](07-implementation-roadmap.md)**

---

## 6. ความเสี่ยงและความรับผิดชอบ (Risk × Owner)

ต่อยอดจากตารางความเสี่ยงใน [03-physical-library-principles.md](03-physical-library-principles.md) และ
[04-llm-wiki-library.md](04-llm-wiki-library.md) §6 — ที่นี่ **กำหนดเจ้าของ (owner)** ให้ชัดทุกความเสี่ยง

| ความเสี่ยง | ผลกระทบ | การควบคุม (control) | เจ้าของ |
|---|---|---|---|
| Provenance gaps (หน้าไร้ที่มา) | คำตอบเชื่อไม่ได้ | บังคับ citation; lint จับหน้าไร้ raw | Citation Steward |
| Cross-book contradictions | เล่ม A/B ขัดกัน | cross-book lint + ตัดสินมีบันทึกเหตุผล | SME + Librarian |
| Rights / licensing | ใช้ raw ที่ไม่มีสิทธิ์ | ตรวจสิทธิ์ก่อน ingest; license ใน metadata; access control | Legal/Rights |
| Privacy / PII | รั่วข้อมูลส่วนบุคคล | redaction/แยกชั้นเข้าถึง; PII review ก่อน ingest | Legal/Rights |
| Model drift | คุณภาพ synthesis/answer เพี้ยนเมื่อเปลี่ยนรุ่น | eval harness ประจำ + pin/regression test ก่อนสลับรุ่น | LLM/Schema Eng |
| Link rot (PID พัง) | ลิงก์ข้ามเล่มเสีย | PID + resolver; ตรวจ link health รายเดือน | Citation Steward |
| Scope creep | เล่มเบลอ/บวม | คุม scope ด้วย schema; อนุมัติ scope change | Product Lead |
| Lint over-correction (แก้วน/ลบของมีค่า) | สูญความรู้ | human-in-the-loop กับการแก้ข้อเท็จจริง | Knowledge Eng + SME |
| Preservation loss | raw/wiki สูญหาย | backup หลายชุด + fixity/checksum (ดู 09) | Platform/SRE |

> หลักการแบ่งเจ้าของ: ความเสี่ยง *เนื้อหา* → Knowledge Eng/SME, *ความน่าเชื่อ/ที่มา* → Citation Steward,
> *กฎหมาย/สิทธิ์* → Legal, *โครงสร้างพื้นฐาน* → SRE, *ขอบเขต/ผลลัพธ์* → Product Lead

---

## 7. เช็กลิสต์: ตั้งทีมและธรรมาภิบาลให้พร้อม

- [ ] กำหนด **เจ้าของ (A)** สำหรับทุกกิจกรรมในตาราง RACI — A ห้ามทับกับผู้ตรวจงานนั้น
- [ ] แต่งตั้ง **เจ้าของ schema** (LLM/Schema Eng) แยกจาก **ผู้อนุมัติ scope** (Product Lead)
- [ ] แต่งตั้ง **Citation/Provenance Steward** ผู้รักษากฎ no-citation-no-answer + PID/resolver
- [ ] จัด **SME/Reviewer** สำหรับ human-in-the-loop การแก้เชิงข้อเท็จจริง (lint เสนอ → คนอนุมัติ)
- [ ] ตั้ง **review cadence** (ingest / สัปดาห์ / เดือน / release / ไตรมาส) ตาม §4.5
- [ ] ผูก **version control เข้ากับ FRBR** (Work=scope, Expression=เนื้อหา/schema, Manifestation=release)
- [ ] นิยาม **quality gates** ระดับเล่มและห้องสมุด + เจ้าของวัด (orphan/broken/citation = 0)
- [ ] ทำ **risk register** พร้อมเจ้าของและ control ทุกความเสี่ยง (§6)
- [ ] ตั้ง **rights/privacy clearance** เป็น gate ก่อน ingest และก่อน release (Legal)
- [ ] ตั้ง **eval harness** กัน model drift ก่อนสลับรุ่น LLM
- [ ] เชื่อมกับ **เฟส/timeline** ที่ [07-implementation-roadmap.md](07-implementation-roadmap.md)
      และ **tech/infra/ops** ที่ [09-reference-stack-and-ops.md](09-reference-stack-and-ops.md)

---

> อ้างอิงแนวคิดต้นทาง: Andrej Karpathy, *"LLM Wiki"*
> ([gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f), 2025–2026)
> — ทีมนี้คือ "คนที่ดูแลคอมไพเลอร์ความรู้ + ห้องสมุด artifact"
> · ดูโมเดลกลางและสารบัญชุดนี้ที่ [00-overview.md](00-overview.md)
