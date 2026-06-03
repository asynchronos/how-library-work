# เอกสาร Schema ตัวอย่าง — `CLAUDE.md` ของหนึ่งเล่ม

> **schema** คือ "กติกาบรรณาธิการ" ที่อยู่ในแต่ละเล่ม (ไฟล์เดียว เช่น `CLAUDE.md`
> หรือ `AGENTS.md`) บอก LLM ว่าจะ **คอมไพล์ `raw/` เป็น `wiki/` อย่างไร** ในทุกสเตจ
> ของไปป์ไลน์ — เทียบ Karpathy คือ **"สเปก/config ของคอมไพเลอร์"**

> หมายเหตุ: schema ถูกอ้างถึงตลอดใน [01-book-pipeline.md](01-book-pipeline.md) และ
> [02-llm-wiki-book.md](02-llm-wiki-book.md) · ตัวอย่างเล่มที่ใช้ schema นี้คือ
> [05-worked-example.md](05-worked-example.md) (เล่ม Raft) · โมเดลกลาง [00-overview.md](00-overview.md)

---

## 1. ทำไมต้องมี schema

ถ้าไม่มี schema ทุกครั้งที่ LLM synthesis/lint จะ "ตีความโครงเล่มใหม่" → หน้าเพี้ยน,
ตั้งชื่อมั่ว, รูปแบบ citation ไม่นิ่ง schema ตรึงสิ่งเหล่านี้ให้ **deterministic** —
หนึ่งไฟล์ที่ **ทุกสเตจอ้างถึง** และเป็นสิ่งที่ทำให้ `wiki/` **regenerate ซ้ำได้ผลเหมือนเดิม**

| schema กำหนด | กันปัญหา |
|--------------|----------|
| โครงไดเรกทอรี + ชนิดหน้า | หน้าซ้ำ/วางผิดที่ |
| กฎตั้งชื่อ (slug) | ลิงก์ `[[...]]` ชี้ผิด |
| รูปแบบ citation | provenance ขาด/ไม่นิ่ง |
| ขอบเขต (scope) | scope creep (ปนหลายโดเมน) |
| กติกา lint | แก้วน/ลบของมีค่า |

---

## 2. ตัวอย่าง `CLAUDE.md` เต็ม (เล่ม Raft จาก worked example)

````markdown
# Schema: Raft Consensus Algorithm (llm-wiki-book)

PID: lib:book/raft-consensus

## 1. Scope (ขอบเขต — บังคับ)
- เล่มนี้ครอบ **เฉพาะ** อัลกอริทึม Raft: leader election, log replication, safety,
  membership change, log compaction
- **นอกขอบเขต (ห้ามสร้างหน้า):** Paxos, ZAB, Viewstamped Replication
  → ถ้าเนื้อหาแตะเรื่องเหล่านี้ ให้ "อ้างถึงเล่มอื่นแบบ cross-book" ไม่ใช่เขียนเอง
- เมื่อ source แตะเรื่องนอกขอบเขต ให้บันทึกใน log ว่าเป็น out-of-scope แล้วข้าม

## 2. Directory layout
- raw/        : ที่มา IMMUTABLE — อ่านอย่างเดียว ห้ามแก้/ลบ ห้าม synthesize ทับ
- wiki/       : หน้าที่สังเคราะห์ (regenerable)
  - wiki/index.md           : สารบัญ (ต้องครอบทุกหน้า)
  - wiki/log.md             : บันทึก append-only
  - wiki/entity/<slug>.md   : สถานะ/สิ่งเฉพาะ (1 หน้า/1 entity)
  - wiki/concept/<slug>.md  : แนวคิด/กลไก
  - wiki/comparison/<slug>.md : วิเคราะห์เทียบ

## 3. Naming (slug)
- slug = ตัวพิมพ์เล็ก คั่นด้วย `-` (kebab-case) เช่น `leader-election`
- หนึ่ง concept/entity = หนึ่งไฟล์ ห้ามมีหน้าซ้ำความหมาย → ถ้าซ้ำให้ **merge**
- wikilink อ้างด้วย path สัมพัทธ์: `[[concept/leader-election]]`

## 4. Citation (บังคับทุกข้ออ้าง)
- ทุกประโยคที่ยืนยันข้อเท็จจริงต้องมี footnote ชี้กลับ raw:
  `[^n]: raw/<source-id>-<file>#<locator>`
  - locator: `#secX.Y` (paper), `#Lxx-yy` (ไฟล์ข้อความ), `#p<n>` (เลขหน้า)
- ถ้าเขียนข้อความที่ **ไม่มี raw รองรับ** → ห้ามเขียน ให้ขึ้น "knowledge gap" ใน log แทน
- รูปแบบ cross-book (อ้างเล่มอื่น): `[ดู lib:book/<id>/page/<slug>]`

## 5. Ingest workflow
1. คัดลอก source ลง raw/ ตั้ง id `S<n>-<ชื่อ>` (immutable)
2. ตรวจ dedupe กับ raw เดิมก่อนเก็บ
3. append log: `## [YYYY-MM-DD] ingest | S<n> <ชื่อ>`
4. ส่งต่อให้ synthesis (อย่าข้ามไป query)

## 6. Synthesis rules
- เขียนจาก raw เท่านั้น (ห้ามเติมจากความจำโมเดล)
- 1 source ให้ปรับ **ทุกหน้าที่เกี่ยว** (summary/entity/concept/index) ไม่ใช่หน้าเดียว
- merge เข้าหน้าเดิมก่อนสร้างหน้าใหม่
- ร้อย wikilink + ปรับ index.md ทุกครั้งที่เพิ่ม/แก้หน้า (กัน orphan)

## 7. Lint rules (รันเป็นรอบ ไม่ใช่ทุก ingest)
ตรวจ: contradiction · stale · orphan · missing page · broken link ·
       missing provenance · knowledge gap
- การแก้ที่กระทบ "ข้อเท็จจริง" → เสนอ + ชี้ raw + รอคนรีวิว ห้าม apply เงียบ ๆ
- ห้ามลบหน้าที่มี backlink โดยไม่ย้าย/merge ก่อน

## 8. Query rules
- ตอบจาก wiki/ ที่สังเคราะห์ไว้ (ไม่ค้นสดจาก raw)
- ทุกคำตอบต้องแนบ citation; ถ้าวิกิไม่มีข้อมูล ให้บอก "ไม่พบ" + บันทึกเป็น gap
- คำตอบที่มีค่า → file back เป็นหน้าใหม่ (ผ่าน lint รอบถัดไป)

## 9. Do-NOT (สรุปข้อห้าม)
- ❌ แก้/ลบ raw/    ❌ เขียนข้อความไร้ citation
- ❌ สร้างหน้าซ้ำแทน merge    ❌ สร้างหน้านอก scope
- ❌ ตอบ query จากความจำโมเดล
````

---

## 3. คำอธิบายส่วนสำคัญของ schema

| ส่วน | ทำหน้าที่อะไรในไปป์ไลน์ | โยงหลักการ |
|------|--------------------------|-------------|
| **Scope** | กันไม่ให้เล่มบวมข้ามโดเมน (ชั้น 2) | one book = one topic |
| **Directory + page types** | ให้ synthesis รู้ว่าจะวางหน้าไหนที่ไหน | indexing granularity |
| **Naming/slug** | ให้ wikilink/citation ไม่พัง | stable identifier |
| **Citation format** | บังคับ provenance ทุกข้ออ้าง | citation ภายในเล่ม (ชั้น 3) |
| **Lint rules** | คุมไม่ให้ LLM "เกลี่ยข้อมูลเนียนแต่ผิด" | quality gate + human-in-the-loop |
| **Do-NOT** | กฎเหล็กเชิงลบ อ่านง่าย ตรวจง่าย | guardrails |

---

## 4. schema กับการทำเวอร์ชัน (FRBR)

schema เป็นส่วนหนึ่งของ **identity ของเล่ม** (หัวข้อ + ชุด raw + schema) ตาม [02-llm-wiki-book.md](02-llm-wiki-book.md) §4:

- แก้ **scope/structure ใน schema** → ถือเป็น **expression ใหม่** (FRBR) เพราะคอมไพล์ออกมาได้โครงต่าง
- จึงควร **เวอร์ชัน schema** ควบคู่กับเล่ม และบันทึกใน log เมื่อเปลี่ยน
- เพราะ `wiki/` regenerable: เปลี่ยน schema แล้ว **คอมไพล์ใหม่ทั้งเล่มจาก `raw/`** ได้เสมอ

> โยง classification/FRBR เต็ม ๆ ที่ [03-physical-library-principles.md](03-physical-library-principles.md) §6

---

## 5. เช็กลิสต์ schema ที่ดี

- [ ] ระบุ **PID เล่ม** และ **scope** (ทั้งในและนอกขอบเขต) ชัดเจน
- [ ] กำหนด **directory layout + ชนิดหน้า** ครบ พร้อมกฎ index.md/log.md
- [ ] มีกฎ **naming/slug** ที่ทำให้ wikilink เสถียร
- [ ] มี **รูปแบบ citation** ตายตัว (ในเล่ม + cross-book) และกฎ "ไม่มี raw = ไม่เขียน"
- [ ] เขียน workflow ของ **ingest / synthesis / lint / query** แยกชัด
- [ ] lint rules ระบุ **human-in-the-loop** สำหรับการแก้เชิงข้อเท็จจริง
- [ ] มีส่วน **Do-NOT** ที่อ่านแล้วตรวจตามได้ทันที
- [ ] ระบุแนวทาง **เวอร์ชัน schema** (โยง FRBR/expression)

> ถัดไป (ปิดท้ายชุด): roadmap การ implement โดยทีมผู้เชี่ยวชาญ —
> [07-implementation-roadmap.md](07-implementation-roadmap.md),
> [08-expert-team-and-governance.md](08-expert-team-and-governance.md),
> [09-reference-stack-and-ops.md](09-reference-stack-and-ops.md)
>
> แรงบันดาลใจ: Andrej Karpathy, "LLM Wiki" (schema = `CLAUDE.md`,
> [gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f))
