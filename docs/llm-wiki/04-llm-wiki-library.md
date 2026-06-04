# LLM-Wiki-Library — การรวมร่างเป็นห้องสมุด citation (ชั้น 3 · ครึ่งหลัง)

> เอกสารนี้คือ **บทสรุปรวบยอด (capstone)** ของชุดงานวิจัย — นำ **หลักการห้องสมุดกายภาพ**
> (จาก [03-physical-library-principles.md](03-physical-library-principles.md)) มารวมกับ **หลักการ llm-wiki**
> เพื่อสร้าง **llm-wiki-library**: ห้องสมุดที่ชั้นวาง **llm-wiki-book หลายเล่ม**
> โดยมี **citation เป็นหัวใจ** ที่ร้อยทุกเล่มเข้าด้วยกัน

> เอกสารนี้ผูกชั้น 1-2-3 ให้เป็นภาพเดียว จึง **ไม่อธิบายซ้ำ** ในรายละเอียดของ:
> ไปป์ไลน์ผลิตเล่ม → [01-book-pipeline.md](01-book-pipeline.md) ·
> หน่วยหนังสือหนึ่งเล่ม → [02-llm-wiki-book.md](02-llm-wiki-book.md) ·
> หลักการห้องสมุดที่ยืมมา → [03-physical-library-principles.md](03-physical-library-principles.md) ·
> โมเดลกลาง → [00-overview.md](00-overview.md)
> แต่จะโฟกัสที่ "ทุกอย่าง **รวมกัน** อย่างไร" ตามแรงบันดาลใจจาก *Karpathy LLM Wiki*

---

## 1. ภาพรวม: จาก "หลายเล่ม" สู่ "ห้องสมุดเดียว"

ชั้น 2 ให้เรา **หนังสือที่สมบูรณ์ในตัวเองหนึ่งเล่ม** (raw + wiki + schema) ที่มีขอบเขตหัวข้อชัด
แต่หนึ่งเล่มตอบได้แค่เรื่องของมันเอง โลกความรู้จริงต้องการ **หลายเล่ม** ที่:

- **ค้นเจอ** ว่ามีเล่มอะไรบ้าง และเล่มไหนตอบคำถามนี้ได้ (catalog)
- **จัดระเบียบ** ให้เล่มที่เกี่ยวข้องอยู่ใกล้กัน (classification)
- **อ้างถึงกันได้** เล่มหนึ่งพาดพิงข้อความในอีกเล่มได้อย่างแม่นยำ (cross-book citation)
- **ตรวจสอบย้อนกลับได้** ทุกคำตอบสาวกลับถึง raw ได้เสมอ (provenance)

> นี่คือสิ่งที่ "ศาสตร์ห้องสมุด" ทำมาเป็นร้อยปีกับหนังสือกระดาษ —
> llm-wiki-library คือการนำหลักการเดียวกันมาครอบ **คอลเลกชันของ Karpathy-wiki**

---

## 2. สถาปัตยกรรมรวม (Combined Architecture)

ภาพนี้ขยายจากโมเดล 3 ชั้นซ้อนใน [00-overview.md](00-overview.md) โดยเพิ่ม
**ชั้น catalog/metadata คร่อมด้านบน** และ **ชั้น citation ที่ร้อยทะลุทุกเล่ม**:

```
╔══════════════════════════════════════════════════════════════════════╗
║  ชั้น 3 · LLM-WIKI-LIBRARY                                             ║
║                                                                        ║
║  ┌──────────────────────────────────────────────────────────────┐    ║
║  │ CATALOG / METADATA LAYER  (บัตรรายการเหนือเล่ม)                │    ║
║  │ Dublin Core/MARC · classification (DDC/LCC) · PID ของแต่ละเล่ม │    ║
║  │ union index → ใช้ route คำถามไปยังเล่มที่ถูกต้อง               │    ║
║  └───────┬───────────────────┬───────────────────┬──────────────┘    ║
║          │                   │                   │                     ║
║   ┌──────▼──────┐     ┌──────▼──────┐     ┌──────▼──────┐              ║
║   │ BOOK A      │     │ BOOK B      │     │ BOOK C      │  ชั้น 2       ║
║   │ (llm-wiki)  │     │ (llm-wiki)  │     │ (llm-wiki)  │  หลายเล่ม     ║
║   │ raw/  wiki/ │     │ raw/  wiki/ │     │ raw/  wiki/ │              ║
║   │ schema      │     │ schema      │     │ schema      │              ║
║   │ ↑pipeline↑  │     │ ↑pipeline↑  │     │ ↑pipeline↑  │  ชั้น 1       ║
║   └──┬───────┬──┘     └──┬───────┬──┘     └──────┬──────┘              ║
║      │       └───────────┼──╴ ╶──┘               │                     ║
║      │  ╭────────────────▼──────────────────────▼─────────────╮       ║
║      └──▶ CITATION LAYER  (เส้นด้ายที่ร้อยทุกเล่ม)            │       ║
║         │ • ภายในเล่ม: page → raw (provenance)                │       ║
║         │ • ข้ามเล่ม: claim ใน A → page ใน B (biblio link)     │       ║
║         │ • PID ระดับ page เพื่อให้ลิงก์ไม่เน่า (link rot)      │       ║
║         ╰──────────────────────────────────────────────────────╯       ║
╚══════════════════════════════════════════════════════════════════════╝
```

อ่านจากบนลงล่าง: **catalog** ช่วยเลือกเล่ม → แต่ละ **เล่ม** คือ Karpathy-wiki อิสระ
ที่ผลิตด้วย **pipeline** ของตัวเอง → และ **citation layer** ร้อยทั้งภายในและข้ามเล่ม
ทำให้ทุกข้อความสาวกลับถึงต้นตอได้

---

## 3. CITATION — หัวใจของห้องสมุด

ถ้าถอด citation ออก llm-wiki-library ก็เหลือเพียง "กองไฟล์ wiki" ที่เชื่อถือไม่ได้
citation คือสิ่งที่ทำให้ความรู้ **traceable** และ **compound** ข้ามเล่มได้
มี 3 ระดับ:

### 3.1 Citation ภายในเล่ม (intra-book · page → raw)

ทุกหน้า `wiki/` ต้องผูกกลับไปยังชิ้น `raw/` ที่เป็นที่มา — นี่คือกฎพื้นฐานที่ชั้น 2 บังคับไว้แล้ว
(ดู [02-llm-wiki-book.md](02-llm-wiki-book.md)) ในมุมห้องสมุดคือ "เชิงอรรถ/บรรณานุกรมท้ายเล่ม"

```
wiki/concept-x.md  ──cites──▶  raw/paper-2401.pdf#p7
                   ──cites──▶  raw/notes/2026-05-meeting.md#L40-52
```

> raw เป็น **immutable** จึงเป็น "ความจริง" ที่ citation ชี้ไปได้อย่างเสถียร
> เทียบ Karpathy: `wiki/` (ไฟล์ที่คอมไพล์แล้ว) ต้อง map กลับไป `raw/` (ซอร์ส) ได้เสมอ

### 3.2 Citation ข้ามเล่ม (inter-book · bibliographic linking)

claim ในเล่ม A สามารถอ้างถึง **หน้าใดหน้าหนึ่ง** ในเล่ม B ได้ —
นี่คือ "บรรณานุกรม" ระหว่างหนังสือ เหมือนหนังสือเล่มหนึ่งอ้างอิงอีกเล่มในห้องสมุด

```
BOOK A · wiki/architecture.md
   "ใช้กลไก consensus แบบ Raft [ดู BOOK-B]"
        │
        ▼  cross-book citation (ผ่าน PID ของเล่ม + PID ของหน้า)
BOOK B (PID: lib:book/distributed-systems) · wiki/raft.md (PID: .../page/raft)
```

หลักสำคัญ: A **ไม่ก๊อปเนื้อหา** ของ B มาไว้ในตัว แต่ **อ้างถึง** เพื่อคง
**single source of truth** — B ยังเป็นเจ้าของและดูแลหน้านั้นเล่มเดียว

### 3.3 Persistent identifier — กันลิงก์เน่า (link rot)

ลิงก์ข้ามเล่มจะ "เน่า" ทันทีถ้าผูกกับ path ของไฟล์ตรง ๆ แล้วมีการย้าย/เปลี่ยนชื่อ
จึงต้องมี **PID 2 ระดับ** (ยืมแนวคิด DOI/Handle/ARK จากห้องสมุดดิจิทัล —
ดู [../how-to-build-digital-library.md](../how-to-build-digital-library.md) §6):

| ระดับ | ตัวอย่าง PID | ชี้ไปที่ | เทียบห้องสมุด |
|-------|-------------|---------|----------------|
| เล่ม (book) | `lib:book/distributed-systems` | หนังสือทั้งเล่ม | เลขเรียกหนังสือ (call number) |
| หน้า (page) | `lib:book/.../page/raft@v3` | หน้า wiki หนึ่งหน้า (+เวอร์ชัน) | เลขหน้า + ฉบับพิมพ์ |

> PID เป็น **indirection layer**: ลิงก์ชี้ที่ PID, ตาราง resolver แปลง PID → path ปัจจุบัน
> ย้ายไฟล์ได้โดยลิงก์ไม่พัง — แก้แค่ที่ resolver

---

## 4. Catalog เหนือเล่ม — เลือก จัดหมวด ค้นข้ามเล่ม

catalog คือ "บัตรรายการ/OPAC" ของห้องสมุดนี้ มัน**ไม่เก็บเนื้อหา** แต่เก็บ
**metadata ของแต่ละเล่ม** เพื่อให้เลือกและค้นได้ (รายละเอียดมาตรฐานดู [03-physical-library-principles.md](03-physical-library-principles.md)):

- **เลือกเล่ม (selection):** เก็บ metadata ระดับเล่ม (title, subject, ขอบเขต, owner, สถานะ lint)
- **จัดหมวด (classification):** ให้หัวข้อใกล้กันอยู่ใกล้กัน (DDC/LCC) → เดินสำรวจตามหมวดได้
- **ค้นข้ามเล่ม (cross-book discovery):** union index รวม metadata + หัวเรื่องหน้าเด่นจากทุกเล่ม
  เหมือน OAI-PMH เก็บเกี่ยว metadata ข้ามคลัง

**กฎทอง: catalog ต้องคงขอบเขตของแต่ละเล่ม (ขอบเขตความรู้ชัด)**
catalog บอกได้ว่า "เรื่องนี้อยู่เล่มไหน" แต่ **ไม่ละลายเล่มเข้าด้วยกัน** —
แต่ละเล่มยังมี schema และ lint ของตัวเอง ความรับผิดชอบจึงชัดเจน

---

## 5. Cross-book query — ถามครั้งเดียว ตอบจากหลายเล่ม

เมื่อมีคำถามที่คาบเกี่ยวหลายหัวข้อ ห้องสมุดต้อง **route แล้ว synthesize**:

```
คำถาม
  │
  ▼
[route]      catalog + union index หาว่า "เล่มไหนเกี่ยวข้อง"  → {A, C}
  │
  ▼
[query/เล่ม] ยิงคำถามย่อยเข้า query ของ A และ C แยกกัน
             (แต่ละเล่มตอบจาก wiki ของตัวเอง — ดู 01-book-pipeline.md)
  │
  ▼
[synthesize] รวมคำตอบ + แนบ citation ของแต่ละเล่ม
             "ตามเล่ม A หน้า x ... ส่วนเล่ม C หน้า y ระบุว่า ..."
  │
  ▼
คำตอบที่ traceable (อ้างหลายเล่ม) → ดี ๆ file กลับเป็นหน้าใหม่ในเล่มที่เหมาะสม
```

### ทำไมต้องคงขอบเขตเล่ม แทนที่จะรวมเป็น "วิกิยักษ์ใบเดียว"

| | วิกิยักษ์ใบเดียว (one giant wiki) | llm-wiki-library (หลายเล่มมี boundary) |
|---|---|---|
| ขอบเขตความรู้ | เบลอ ปนกันหมด | **ชัดต่อเล่ม** รู้ว่าใครรับผิดชอบอะไร |
| lint / สุขภาพ | ตรวจทั้งก้อน ช้า ขัดแย้งซ่อนลึก | ตรวจต่อเล่ม + ตรวจ "ข้ามเล่ม" เฉพาะจุด |
| provenance | สาวยาก เมื่อทุกอย่างปนกัน | สาวกลับ raw ของเล่มนั้นได้ตรง |
| สิทธิ์/ownership | ปนกัน แก้ทับกัน | กำหนดเจ้าของ/สิทธิ์ต่อเล่มได้ |
| สเกล | บวมจนคอมไพล์ใหม่ทั้งก้อนแพง | เพิ่ม/ลบเล่มได้อิสระ (modular) |

> เทียบวิทยาการคอมพิวเตอร์: เล่ม = **module/service ที่มี bounded context**,
> cross-book citation = **typed dependency** ระหว่าง module — ดีกว่า "monolith ก้อนเดียว"

---

## 6. ธรรมาภิบาลและความเชื่อถือ (Governance & Trust)

หลักการกำกับยืมเต็มจาก [03-physical-library-principles.md](03-physical-library-principles.md) — ที่นี่สรุปสั้น ๆ:

- **ต้นฉบับ (raw) = ความจริง:** wiki แก้ได้/คอมไพล์ใหม่ได้ แต่ raw immutable เป็นหลักยึด
- **lint รักษาสุขภาพเล่ม:** ตรวจขัดแย้ง/ล้าสมัย/หน้ากำพร้า/ลิงก์ขาด ทั้งในเล่มและ**ข้ามเล่ม**
- **ทุกคำตอบ traceable:** ไม่มี citation = ไม่ปล่อยออก (no-citation, no-answer)

| ความเสี่ยง | ผลกระทบ | การบรรเทา (สั้น) |
|-----------|---------|------------------|
| Provenance ขาด | คำตอบเชื่อไม่ได้ | บังคับ citation ทุกหน้า; lint จับหน้าไร้ที่มา |
| ขัดแย้งข้ามเล่ม | A กับ B พูดต่างกัน | cross-book lint รายงานข้อขัดแย้ง; เลือก/บันทึกเหตุผล |
| Link rot | ลิงก์ข้ามเล่มพัง | ใช้ PID + resolver (ดู §3.3) |
| ลิขสิทธิ์/สิทธิ์ raw | นำเข้า raw ที่ไม่มีสิทธิ์ | บันทึก license ใน metadata เล่ม; access control ต่อเล่ม |

> รายละเอียดความเสี่ยง+มาตรการเต็มอยู่ใน [03-physical-library-principles.md](03-physical-library-principles.md)

---

## 7. ตาราง: ชั้น/องค์ประกอบ → หน้าที่ → มาจากหลักการใด

| ชั้น/องค์ประกอบ | หน้าที่ | มาจากหลักการ |
|-----------------|---------|----------------|
| Pipeline (ingest→synthesis→lint→query) | ผลิต/ดูแลเนื้อหาหนึ่งเล่ม | **llm-wiki** |
| `raw/` immutable | ความจริง/ที่มา | **llm-wiki** (+citation ของห้องสมุด) |
| `wiki/` หน้าสังเคราะห์ | เนื้อหาที่อ่าน/ถามได้ | **llm-wiki** |
| `schema` ต่อเล่ม | กติกาบรรณาธิการของเล่ม | **llm-wiki** (เทียบ style guide ห้องสมุด) |
| Catalog/metadata layer | เลือก/จัดหมวด/ค้นข้ามเล่ม | **library** (Dublin Core/MARC, DDC/LCC, OAI-PMH) |
| Citation ภายในเล่ม | page → raw provenance | **library** ⊗ **llm-wiki** |
| Citation ข้ามเล่ม | claim A → page B | **library** (bibliographic linking) |
| PID (book/page) + resolver | กันลิงก์เน่า | **library** (DOI/Handle/ARK) |
| Cross-book query/synthesis | route + รวมคำตอบหลายเล่ม | **library** (union catalog) ⊗ **llm-wiki** (query) |
| Lint (ในเล่ม + ข้ามเล่ม) | รักษาสุขภาพ + จับขัดแย้ง | **llm-wiki** ⊗ **library** (collection maintenance) |
| Preservation (สำรอง+fixity) | รักษา raw/wiki ระยะยาว | **library** (LOCKSS/PREMIS) |

---

## 8. การเทียบเคียงกับวิทยาการคอมพิวเตอร์

| llm-wiki-library | แนวคิดวิทยาการคอมพิวเตอร์ |
|------------------|---------------------------|
| เล่มมีขอบเขตชัด (bounded scope) | **Bounded context / Microservice** |
| Cross-book citation ผ่าน PID | **Typed dependency / Linker symbol resolution** |
| PID + resolver (กันลิงก์เน่า) | **Indirection layer / Stable URI / Symlink** |
| Catalog/union index เหนือเล่ม | **Service registry / Federated index** |
| Route คำถาม → เล่มที่ใช่ | **Query router / Service discovery** |
| Synthesize คำตอบหลายเล่ม | **Scatter-gather / Fan-out aggregation** |
| raw immutable = ความจริง | **Append-only log / Source of truth** |
| synthesis = LLM คอมไพล์ raw→wiki | **Compiler (source → build artifact)** |
| lint (ในเล่ม+ข้ามเล่ม) | **Test suite / Static analysis + Link check** |
| filed-back answers | **Memoization / Incremental build cache** |
| Preservation (fixity/replication) | **Checksum / Replication** |

> แกนเดียวกับ Karpathy LLM Wiki: `raw/`=ซอร์ส, LLM=คอมไพเลอร์, `wiki/`=build artifact,
> `lint`=เทสต์, `query`=รันไทม์ — llm-wiki-library เพียงเพิ่ม "ตัวเชื่อมหลาย artifact" (linker + registry)

---

## 9. เช็กลิสต์: ประกอบ llm-wiki-library

- [ ] มี llm-wiki-book อย่างน้อยหนึ่งเล่มที่ผ่าน pipeline ครบ (ดู [01-book-pipeline.md](01-book-pipeline.md))
- [ ] แต่ละเล่มมีขอบเขตหัวข้อชัด + schema + lint ของตัวเอง
- [ ] บังคับ citation ภายในเล่ม: ทุกหน้า wiki ผูกกลับ raw (no-citation, no-answer)
- [ ] กำหนด **PID ระดับเล่ม** และ **PID ระดับหน้า** + ตั้ง resolver (กันลิงก์เน่า)
- [ ] สร้าง catalog/metadata layer (Dublin Core/MARC) + classification (DDC/LCC)
- [ ] สร้าง union index เพื่อ route คำถามข้ามเล่ม
- [ ] รองรับ cross-book citation (อ้างถึง ไม่ก๊อป) + cross-book lint จับข้อขัดแย้ง
- [ ] วาง cross-book query: route → query ต่อเล่ม → synthesize + แนบ citation ทุกเล่ม
- [ ] กำหนด ownership/สิทธิ์/license ต่อเล่ม + access control
- [ ] วางแผน preservation: สำรองหลายชุด + fixity/checksum สำหรับ raw และ wiki
- [ ] file-back: คำตอบ cross-book ที่ดีถูกเขียนกลับเป็นหน้าใหม่ในเล่มที่เหมาะสม

**แก่นสำคัญ:** llm-wiki-library = **หลายเล่ม Karpathy-wiki ที่มีขอบเขตชัด** + **catalog คร่อมด้านบน**
+ **citation ร้อยทุกเล่ม** โดยยึดหลักเดียว — **raw คือความจริง, ทุกคำตอบสาวกลับได้** —
ทำให้ความรู้ทั้งห้องสมุด **เชื่อถือได้และทบต้น (compound)** แทนที่จะเป็นวิกิยักษ์ที่เบลอและสาวกลับไม่ได้

> อ้างอิงแนวคิดต้นทาง: Andrej Karpathy, "LLM Wiki" (gist, 2025–2026)
