# Worked Example — สร้าง llm-wiki-book หนึ่งเล่มแบบ end-to-end

> เอกสารนี้คือ **ตัวอย่างลงมือทำจริง (worked example)**: เดินผ่านไปป์ไลน์
> `ingest → synthesis → lint → query` ทั้งวงจร เพื่อสร้าง **หนังสือหนึ่งเล่ม
> (llm-wiki-book)** จากศูนย์ แล้วเตรียมหยิบเข้าห้องสมุด — ใช้หัวข้อตัวอย่างที่จับต้องได้คือ
> **"Raft Consensus Algorithm"**

> หมายเหตุ: อ่านโมเดลกลางที่ [00-overview.md](00-overview.md) ก่อน · ทฤษฎีของแต่ละสเตจดู
> [01-book-pipeline.md](01-book-pipeline.md) · โครงสร้างเล่มดู [02-llm-wiki-book.md](02-llm-wiki-book.md)
> · กติกา (schema) ที่เล่มนี้ใช้ดู [06-schema-example.md](06-schema-example.md) ·
> การเก็บเข้าห้องสมุดดู [04-llm-wiki-library.md](04-llm-wiki-library.md)

---

## 1. โจทย์: นิยามเล่มและขอบเขต

| รายการ | ค่า |
|--------|-----|
| ชื่อเล่ม (Title) | Raft Consensus Algorithm |
| ขอบเขต (Scope) | อัลกอริทึมความเห็นพ้องแบบ Raft — leader election, log replication, safety. **ไม่ครอบ** Paxos/ZAB (อยู่เล่มอื่น) |
| ผู้ใช้เป้าหมาย | วิศวกรที่ต้องเข้าใจ Raft เพื่อนำไป implement/ตรวจรีวิว |
| PID เล่ม (ตั้งล่วงหน้า) | `lib:book/raft-consensus` |

> ยึดกฎชั้น 2: **หนึ่งเล่ม = หนึ่งหัวข้อ** — เราจงใจตัด Paxos ออกเพื่อให้ scope คม
> (เล่มอื่นใน [04-llm-wiki-library.md](04-llm-wiki-library.md) จะอ้างถึงเล่มนี้แบบ cross-book ได้)

---

## 2. โครงไฟล์เริ่มต้น (ตาม schema)

```
raft-consensus/                 ← หนึ่งเล่ม (PID: lib:book/raft-consensus)
├── CLAUDE.md                   ← schema (ดู 06-schema-example.md)
├── raw/                        ← IMMUTABLE: ที่มา
├── wiki/                       ← REGENERABLE: หน้าที่สังเคราะห์
│   ├── index.md
│   └── log.md
```

`raw/` และ `wiki/` ยังว่าง — เริ่มจาก ingest

---

## 3. สเตจ INGEST — นำเข้าและย่อย source

ทยอยนำเข้า 3 source หลัก เก็บลง `raw/` แบบ **immutable** + เขียน `log.md` ต่อท้าย:

| source id | ที่มา | ชนิด |
|-----------|------|------|
| `S1` | Ongaro & Ousterhout, *"In Search of an Understandable Consensus Algorithm"* (Raft paper, 2014) | paper |
| `S2` | บันทึกการบรรยาย MIT 6.824 เรื่อง Raft | lecture notes |
| `S3` | บทความบล็อก "Raft visualization" + คำอธิบาย leader election | web |

```
raw/
├── S1-raft-paper-2014.pdf
├── S2-mit-6824-raft-notes.md
└── S3-raft-visualization.html.md
```

`wiki/log.md` (append-only) หลัง ingest:

```markdown
## [2026-06-01] ingest | S1 raft-paper-2014
## [2026-06-01] ingest | S2 mit-6824-raft-notes
## [2026-06-02] ingest | S3 raft-visualization
```

> หลักเหล็ก: ตั้งแต่ตอนนี้ **ห้ามแก้ `raw/`** ถ้าผู้เขียนต้นทางออกฉบับแก้ ให้ใส่เป็น
> `S1b-...` ใหม่ (append) ไม่ทับของเดิม — เพื่อรักษา provenance (ดู [01-book-pipeline.md](01-book-pipeline.md) §2.1)

---

## 4. สเตจ SYNTHESIS — "คอมไพล์" raw เป็นหน้า wiki

LLM อ่าน `raw/` + ทำตาม schema เขียนหน้าหลายชนิด **1 source แตะหลายหน้า** ผลลัพธ์:

```
wiki/
├── index.md
├── log.md
├── entity/
│   ├── leader.md          ├── follower.md      └── candidate.md
├── concept/
│   ├── leader-election.md ├── log-replication.md └── safety.md
└── comparison/
    └── raft-vs-paxos.md
```

### ตัวอย่าง `wiki/index.md` (สารบัญ)

```markdown
# Raft Consensus — Index

## Concepts
- [[concept/leader-election]] — เลือก leader ด้วย term + คะแนนเสียงข้างมาก
- [[concept/log-replication]] — leader คัดลอก log ไปยัง follower
- [[concept/safety]] — การันตีว่า log ที่ commit แล้วไม่ย้อนกลับ

## Entities (states)
- [[entity/leader]] · [[entity/follower]] · [[entity/candidate]]

## Comparisons
- [[comparison/raft-vs-paxos]]
```

### ตัวอย่าง `wiki/concept/leader-election.md` (พร้อม citation กลับ raw)

```markdown
# Leader Election

Raft แบ่งเวลาเป็น **term** แต่ละ term มี leader ได้มากสุดหนึ่งตัว [^1].
เมื่อ follower ไม่ได้ยิน heartbeat ภายใน *election timeout* จะเลื่อนเป็น
[[entity/candidate]] เพิ่ม term แล้วขอคะแนนเสียง ผู้ได้เสียงข้างมากเป็น
[[entity/leader]] [^2]. ดูภาพรวมสถานะใน [[entity/follower]].

[^1]: raw/S1-raft-paper-2014.pdf#sec5.2
[^2]: raw/S2-mit-6824-raft-notes.md#L88-110
```

> สังเกต: ทุกข้ออ้างผูกกลับ `raw/<file>#<loc>` = **citation ภายในเล่ม** (ชั้น 2)
> และหน้าเชื่อมกันด้วย `[[wikilink]]` = โครงข่ายความรู้ในเล่ม

---

## 5. สเตจ LINT — ตรวจสุขภาพเล่ม (รอบแรก)

รัน lint พบปัญหาจริง 3 อย่าง แก้แล้วบันทึกผล:

| ที่พบ | ชนิด | การแก้ |
|-------|------|--------|
| `comparison/raft-vs-paxos.md` ไม่มีใครลิงก์ถึงจาก index | **orphan** | เพิ่มลิงก์ในส่วน Comparisons ของ index.md |
| `concept/safety.md` อ้าง "election timeout 100ms" แต่ S2 บอก "150–300ms" | **contradiction** | ยึด S2 (ชัด/อัปเดตกว่า), แก้ตัวเลข, บันทึกเหตุผลใน log |
| `entity/candidate.md` กล่าวถึง "pre-vote" แต่ไม่มี citation | **missing provenance** | ไม่มีใน S1–S3 → mark เป็น knowledge gap ป้อนเป็นโจทย์ ingest ต่อไป |

`wiki/log.md` เพิ่ม:

```markdown
## [2026-06-03] lint | แก้ orphan(raft-vs-paxos), contradiction(timeout→S2),
##                     gap(pre-vote: ไม่มีใน raw → รอ ingest source เพิ่ม)
```

> หลักจาก [01-book-pipeline.md](01-book-pipeline.md) §2.3: lint ที่แตะ "ข้อเท็จจริง"
> (เช่น แก้ตัวเลข timeout) ควรชี้ raw และให้คนรีวิวก่อน apply — ไม่ใช่เกลี่ยให้เนียนเอง

---

## 6. สเตจ QUERY — สนทนากับเล่ม + file-back

ผู้ใช้ถามจาก `wiki/` ที่สังเคราะห์ไว้ (ไม่ค้นสดจาก raw):

```
ถาม:  "ถ้ามี leader สองตัวพร้อมกันจะเกิดอะไร?"

ตอบ:  Raft กันด้วยกฎ "หนึ่ง term มี leader ได้ตัวเดียว" — leader จะ commit ได้ต่อเมื่อ
      ได้ ack จากเสียงข้างมาก ทำให้สอง leader ที่ term ต่างกันไม่สามารถ commit
      ทับกันได้ [ตาม [[concept/safety]] และ [[concept/leader-election]]]
      อ้างอิง: raw/S1-raft-paper-2014.pdf#sec5.2, #sec5.4
```

คำตอบนี้มีค่า → **file back** เป็นหน้าใหม่ `wiki/concept/split-brain.md` (ผ่าน lint รอบถัดไป)
→ ครั้งหน้าใครถามเรื่องนี้ เล่ม "ฉลามขึ้น" เพราะมีหน้าตรงประเด็นแล้ว = **ความรู้ compound**

```markdown
## [2026-06-03] query→file-back | สร้าง concept/split-brain.md จากคำถามผู้ใช้
```

---

## 7. ผลลัพธ์: ตรวจว่าเป็น "เล่มที่ฟอร์มดี"

เทียบกับเช็กลิสต์ใน [02-llm-wiki-book.md](02-llm-wiki-book.md) §8:

- [x] มี `raw/`, `wiki/`, `CLAUDE.md` ครบสามส่วน
- [x] `raw/` immutable (append-only — S1b ถ้ามีฉบับแก้)
- [x] `wiki/` regenerate ได้จาก `raw/` ทั้งหมด
- [x] มี `index.md` ครอบทุกหน้า + `log.md` append-only
- [x] ทุกข้ออ้างมี citation กลับ `raw/`
- [x] ไม่มี orphan ค้าง (lint ผ่าน)
- [x] ขอบเขตหนึ่งหัวข้อชัด (Raft เท่านั้น, ไม่ปน Paxos)
- [x] query อ่านจาก wiki + file-back ได้

> เล่มนี้ตอนนี้ "หยิบไปวางบนชั้นห้องสมุดได้" เพราะมี identity + boundary คม

---

## 8. เก็บเข้าห้องสมุด (ส่งต่อชั้น 3)

ขั้นถัดไปคือลงระเบียน catalog + ตั้ง PID ระดับหน้า เพื่อให้เล่มอื่นอ้างถึงได้:

```
catalog record (Dublin Core ย่อ):
  Title:      Raft Consensus Algorithm
  Subject:    Distributed Systems / Consensus     (classification: DDC 004.x)
  Identifier: lib:book/raft-consensus
  Source:     raw/S1, raw/S2, raw/S3
  Version:    edition-1 (Expression ตาม FRBR)
```

ตอนนี้เล่มอื่น เช่น `lib:book/distributed-systems` สามารถ **cross-book cite** มาที่
`lib:book/raft-consensus/page/leader-election` ได้ (ดู [04-llm-wiki-library.md](04-llm-wiki-library.md) §3.2)

---

## 9. สรุปบทเรียนจากตัวอย่างนี้

- ไปป์ไลน์ทำให้ความรู้ดิบ 3 แหล่งกลายเป็น **เล่มเดียวที่สอดคล้องและตรวจสอบได้**
- **citation ผูกทุกอย่างไว้กับ raw** ตั้งแต่หน้าแรก — ไม่ใช่มาเติมทีหลัง
- **lint จับของจริง** (orphan/contradiction/gap) ตั้งแต่เล่มยังเล็ก
- **query + file-back** ทำให้เล่มโตขึ้นจากการใช้งาน (compound)
- scope ที่คม = เล่มพร้อมเข้าห้องสมุดและถูกอ้างข้ามเล่ม

> ถัดไป: ดูกติกาที่ทำให้ทุกสเตจประพฤติสม่ำเสมอใน [06-schema-example.md](06-schema-example.md)
> · แรงบันดาลใจ: Andrej Karpathy, "LLM Wiki" ([gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f))
