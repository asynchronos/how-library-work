# Reference Tech Stack & Operations — แพลตฟอร์มอ้างอิงสำหรับ llm-wiki-library

> เอกสารนี้เป็น **คู่มือ tech stack อ้างอิง + การปฏิบัติการ (operations)** สำหรับ
> *implement* llm-wiki-library ตามโมเดล 3 ชั้นใน [00-overview.md](00-overview.md):
> แต่ละ **องค์ประกอบ/สเตจ** ควร map ลงเป็น **building block** อะไรบนแพลตฟอร์ม,
> มี trade-off แบบไหน, และดูแล (monitor/preserve/scale) อย่างไร
> หลักการนี้ได้แรงบันดาลใจจาก *Andrej Karpathy, "LLM Wiki"*

> เอกสารนี้เป็น **vendor-neutral**: ให้ "ทางเลือก + เกณฑ์เลือก" ไม่บังคับผลิตภัณฑ์เดียว ·
> **ไม่อธิบายซ้ำ** แนวคิดที่เอกสารอื่นเป็นเจ้าของ แต่ **cross-link**:
> ไปป์ไลน์ → [01-book-pipeline.md](01-book-pipeline.md) ·
> หน่วยเล่ม → [02-llm-wiki-book.md](02-llm-wiki-book.md) ·
> หลักห้องสมุด → [03-physical-library-principles.md](03-physical-library-principles.md) ·
> citation/PID/cross-book → [04-llm-wiki-library.md](04-llm-wiki-library.md) ·
> schema → [06-schema-example.md](06-schema-example.md) ·
> เทคโนโลยี/preservation ของห้องสมุดดิจิทัล → [../how-to-build-digital-library.md](../how-to-build-digital-library.md)
>
> **เลื่อน (defer) ให้เอกสารอื่น:** เฟส/ลำดับการทำ → [07-implementation-roadmap.md](07-implementation-roadmap.md) ·
> บทบาททีม/ธรรมาภิบาล/นโยบาย eval + human-in-the-loop → [08-expert-team-and-governance.md](08-expert-team-and-governance.md)

---

## 1. สถาปัตยกรรมอ้างอิง (Reference Architecture)

แพลตฟอร์มต้องรองรับทั้ง 3 ชั้น โดย **`raw/` คือความจริง (immutable), `wiki/` คือ artifact
ที่ regenerate ได้, ส่วน vector index เป็น *ทางเลือก* สำหรับ retrieval เท่านั้น**
(ไม่ใช่แหล่งความรู้ — ดู [00-overview.md](00-overview.md) §0 และ [01-book-pipeline.md](01-book-pipeline.md) §4)

```
┌──────────────────────────────────────────────────────────────────────────┐
│  CLIENTS                                                                   │
│  ผู้ดูแล (ingest/lint) · ผู้ถาม (query) · CI/บอท (lint, link/citation check)│
└───────────────┬───────────────────────────────────────┬──────────────────┘
                │                                         │
        ┌───────▼────────┐                       ┌────────▼─────────┐
        │ CROSS-BOOK     │  route → fan-out       │ PID RESOLVER     │
        │ QUERY ROUTER   │◀──────────────────────▶│ (DOI/Handle/ARK) │
        │ (scatter-gather)│  PID → path ปัจจุบัน    │ indirection table │
        └──┬─────────┬────┘                       └────────┬─────────┘
           │         │                                     │
   ┌───────▼──┐  ┌───▼─────────┐  ┌──────────────┐  ┌──────▼───────────┐
   │ CATALOG/ │  │ SEARCH/INDEX│  │ VECTOR INDEX │  │  ORCHESTRATOR     │
   │ METADATA │  │ (full-text) │  │  (OPTIONAL,   │  │  คิวงาน ingest/   │
   │ DB (PG)  │  │ ES/Solr     │  │  retrieval)   │  │  synthesis/lint   │
   │ Dublin   │  │ union index │  │ pgvector/FAISS│  │  scheduler + HITL │
   │ Core/MARC│  └─────────────┘  │ Qdrant/Milvus │  │  hooks            │
   └────┬─────┘                   └──────┬───────┘  └──────┬───────────┘
        │                                │                  │ อ่าน/เขียนเล่ม
        │   ┌────────────────────────────▼──────────────────▼──────────────┐
        │   │  PER-BOOK STORE (ชั้น 2) — หนึ่งเล่ม = หนึ่ง bounded context  │
        │   │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐ │
        └──▶│  │ raw/         │  │ wiki/        │  │ schema (CLAUDE.md)    │ │
            │  │ OBJECT STORE │  │ GIT + MD     │  │ versioned ใน git      │ │
            │  │ immutable    │  │ regenerable  │  │ (สเปกคอมไพเลอร์)       │ │
            │  │ + checksum   │  │ + history    │  └──────────────────────┘ │
            │  │ + append log │  └──────────────┘                           │
            │  └──────┬───────┘                                             │
            └─────────┼─────────────────────────────────────────────────────┘
                      ▼
            ┌──────────────────────────────────────────────────────────────┐
            │  PRESERVATION (ชั้นล่างสุด) — ใช้ร่วมทุกเล่ม                    │
            │  LOCKSS (สำเนาหลายที่) · Fixity/checksum · format migration ·   │
            │  PREMIS log · backup/restore  → ดู how-to-build-digital-library│
            └──────────────────────────────────────────────────────────────┘
```

> อ่านจากล่างขึ้นบน: **preservation** ปกป้อง `raw/`+`wiki/` → แต่ละ **เล่ม** เก็บใน
> object store (raw) + git (wiki) → **catalog/search/vector** ทำให้ค้น/route ได้ →
> **router + resolver** ทำ cross-book query โดยไม่ทำลายขอบเขตเล่ม

---

## 2. ทางเลือกองค์ประกอบ + Trade-off (Component Choices)

### 2.1 ที่เก็บ `wiki/` — git/markdown vs CMS

| ทางเลือก | จุดเด่น | จุดอ่อน | เหมาะเมื่อ |
|----------|---------|---------|-----------|
| **Git + Markdown** (แนะนำ) | versioning ฟรี, diff/PR review = HITL hook, regenerate ได้, plain-text ตรงกับ "artifact ที่คอมไพล์" | ค้น full-text ต้องมี index แยก, ไม่มี UI ในตัว | ทีมเล็ก-กลาง, ต้องการ provenance/diff, คอมไพล์ซ้ำได้ |
| **Wiki/CMS** (MediaWiki, Wiki.js) | UI แก้ไข + ค้นในตัว, สิทธิ์ผู้ใช้ | versioning/diff ด้อยกว่า git, ผูกกับ DB | ผู้ใช้ non-technical แก้ wiki เอง |
| **DB-backed docs** | query เชิงโครงสร้างง่าย | เสีย "ไฟล์ = artifact", regenerate ยุ่ง | ต้องการ field คิวรีหนัก |

> เกณฑ์เลือก: **`wiki/` ต้อง regenerate จาก `raw/`+schema ได้เสมอ** (ดู [06-schema-example.md](06-schema-example.md) §4)
> git/markdown รักษาคุณสมบัตินี้ดีที่สุด และ PR = จุดแทรก human review ตามธรรมชาติ

### 2.2 ที่เก็บ `raw/` — object store + checksum (immutable)

| ทางเลือก | จุดเด่น | จุดอ่อน |
|----------|---------|---------|
| **S3-compatible object store** (S3/MinIO/Ceph) | object lock = immutability บังคับได้, versioning, scale | ต้องตั้ง lifecycle/lock policy เอง |
| **Content-addressed store** (git-annex, IPFS, OCFL) | checksum = address → tamper-evident โดยธรรมชาติ | learning curve, tooling น้อยกว่า |
| **WORM filesystem / append-only** | ง่าย, ตรง append-only log | scale/replication จำกัด |

> เกณฑ์: เปิด **object-lock/WORM + เก็บ checksum (SHA-256)** ทุกชิ้น และ **append-only log**
> ของการ ingest (ดู [01-book-pipeline.md](01-book-pipeline.md) §2.1) — กฎเหล็ก: `raw/` ห้ามแก้/ลบ

### 2.3 Vector index — **ทางเลือก** สำหรับ retrieval (ไม่ใช่แหล่งความรู้)

> RAG ที่นี่คือ *กลไก retrieval* เพื่อหา "หน้า `wiki/` ที่สังเคราะห์ไว้แล้ว" ให้ query อ่าน —
> **ไม่ใช่** ที่เก็บความรู้ และไม่ใช่การค้นสดจาก `raw/` (ดู [00-overview.md](00-overview.md) §0,
> [01-book-pipeline.md](01-book-pipeline.md) §4). ถ้าเล่มเล็ก/หน้าน้อย **อาจไม่ต้องมีเลย** —
> ให้ query เดินตาม index.md + wikilink + full-text ก็พอ

| ตัวเลือก | โมเดล deploy | จุดเด่น | เหมาะเมื่อ |
|----------|--------------|---------|-----------|
| **pgvector** | ส่วนขยายของ Postgres | รวมกับ metadata DB ที่มีอยู่, ops ชิ้นเดียว | จำนวน chunk ปานกลาง, อยากลด moving parts |
| **FAISS** | ไลบรารีฝังในแอป | เร็ว, ฟรี, คุมเอง | งานทดลอง/in-process, ไม่ต้องการ service แยก |
| **Qdrant** | service | filter + payload ดี, ใช้ง่าย | ต้อง metadata filter ต่อเล่ม, scale ปานกลาง-ใหญ่ |
| **Milvus** | cluster | scale แนวนอนสูง | corpus ใหญ่มากหลายล้านเวกเตอร์ |

เกณฑ์: เริ่มที่ **pgvector** (ไม่เพิ่ม service) → ย้ายไป Qdrant/Milvus เมื่อ latency/ขนาดโต ·
**ฝัง PID ของหน้า (book/page) เป็น payload** เพื่อให้ retrieval → citation ได้ทันที

### 2.4 Embedding model & LLM hosting

| มิติ | Cloud API | On-prem / self-host |
|------|-----------|---------------------|
| **Privacy** | raw ออกนอกองค์กร (ระวัง license/PII ของ `raw/`) | ข้อมูลอยู่ในบ้าน เหมาะ raw ที่ sensitive |
| **Cost** | จ่ายตามใช้ (synthesis = ภาระหลัก, ดู §6) | ลงทุน GPU คงที่ คุ้มเมื่อ throughput สูง |
| **คุณภาพ/บำรุงรักษา** | โมเดลแรงสุด, ไม่ต้องดูแล | คุมเวอร์ชันได้, ต้องดูแล/อัปเกรดเอง |
| **เกณฑ์เลือก** | prototype, โหลดไม่สม่ำเสมอ | raw ลิขสิทธิ์/ลับ (ดู [04](04-llm-wiki-library.md) §6) หรือ batch synthesis ปริมาณมาก |

> Embedding: ตรึงเวอร์ชันโมเดล + **เก็บ model id/เวอร์ชันใน metadata** เพราะเปลี่ยนโมเดล
> = ต้อง re-embed ทั้งเล่ม (เหมือน format migration ของ index). โมเดลใหญ่ใช้กับงานยาก,
> โมเดลเล็กกับ lint/embedding/งานง่าย (ดู §6)

### 2.5 Metadata DB + Search + PID resolver

| building block | ตัวเลือกอ้างอิง | บทบาท |
|----------------|------------------|--------|
| **Metadata DB** | PostgreSQL | catalog ระดับเล่ม (Dublin Core/MARC), สถานะ lint, ownership/license, ตาราง PID |
| **Full-text search** | Elasticsearch / Solr | union index ข้ามเล่มเพื่อ route + ค้นหน้า wiki (ดู [04](04-llm-wiki-library.md) §4) |
| **PID resolver** | บริการเล็ก ๆ (table PID→path) สไตล์ DOI/Handle/ARK | indirection: ลิงก์ชี้ PID, resolver แปลงเป็น path ปัจจุบัน → กัน link rot (ดู [04](04-llm-wiki-library.md) §3.3) |

> ทั้ง Postgres/ES/PID มาตรฐานเดียวกับห้องสมุดดิจิทัล — **reuse** เกณฑ์/มาตรฐานจาก
> [../how-to-build-digital-library.md](../how-to-build-digital-library.md) §4–6 (Dublin Core, OAI-PMH, DOI/Handle/ARK)

---

## 3. การ Automate/Orchestrate ไปป์ไลน์

แต่ละสเตจของ [01-book-pipeline.md](01-book-pipeline.md) map เป็น **งาน (job)** ที่ orchestrator คุม:

```
[ingest]   on-demand/event-driven : มี source ใหม่ → คัด raw ลง object store (immutable)
                                     + checksum + append log + enqueue synthesis
   │
[synthesis] queue worker (per-book) : LLM อ่าน raw+wiki+schema → เขียน/อัปเดต ~10–15 หน้า
                                       → commit เป็น branch → เปิด PR (จุด HITL)
   │
[lint]     scheduled (เช่น รายสัปดาห์) : ตรวจ orphan/broken link/missing provenance/
                                          contradiction → ออกรายงาน + เสนอ PR (ไม่ apply เงียบ)
   │
[query]    online service          : route → อ่าน wiki (ทางเลือก: ใช้ vector retrieve) →
                                      ตอบ + citation → file-back ดี ๆ เป็น PR หน้าใหม่
   │
[CI checks] on every PR/commit     : link health, citation present, schema conformance,
                                      cross-book PID resolvable → block merge ถ้าไม่ผ่าน
```

ทางเลือก orchestrator: **cron/systemd-timers** (เริ่มต้น), **CI runner** (GitHub Actions/GitLab CI
สำหรับ lint+link/citation check บนทุก PR), **workflow engine** (Airflow/Temporal/Prefect)
เมื่อ DAG ของ ingest→synthesis ซับซ้อนหรือต้อง retry/observability ระดับงาน

> **Human-in-the-loop:** synthesis/lint ที่กระทบข้อเท็จจริงต้อง "เสนอผ่าน PR ให้คนรีวิว"
> ไม่ apply เงียบ (ดู [01](01-book-pipeline.md) §2.3, [06](06-schema-example.md) §7).
> **นโยบายรีวิว/ใครอนุมัติ/เกณฑ์ eval = defer ให้ [08-expert-team-and-governance.md](08-expert-team-and-governance.md)**

---

## 4. Observability & Ops

### 4.1 Health metrics ที่ต้อง emit

| metric | สเตจ/ชั้นที่เกี่ยว | สัญญาณเตือนเมื่อ |
|--------|---------------------|------------------|
| **orphan page count** | lint (ในเล่ม) | > 0 และโตขึ้น = synthesis ลืมร้อยลิงก์ |
| **broken wikilink count** | lint (ในเล่ม) | > 0 = หน้าปลายทางหาย/slug เพี้ยน |
| **pages without citation** | lint / governance | > 0 = ละเมิด no-citation-no-answer |
| **cross-book link failures** | router + resolver | PID resolve ไม่ได้ = link rot ข้ามเล่ม |
| **query latency (p50/p95)** | query service | p95 พุ่ง = index/retrieval/route ช้า |
| **cost per query / per synthesis** | LLM hosting | พุ่ง = ต้อง cache/ใช้โมเดลเล็ก (ดู §6) |
| **lint backlog / stale pages** | lint scheduler | คั่งค้าง = รอบ lint ไม่ทันการ ingest |

> metric กลุ่ม orphan/broken-link/no-citation คือ **เป้า "ใกล้ศูนย์"** ตามเช็กลิสต์
> [01](01-book-pipeline.md) §6 — ทำให้เป็น dashboard + alert ต่อเล่ม

### 4.2 Logging / Audit ของ provenance chain

- **append-only audit log** ต่อเล่ม: ทุก ingest/synthesis/lint/file-back บันทึก
  ใคร/เมื่อไร/แตะหน้าใด/อ้าง raw ใด (git history + log.md ทำหน้าที่นี้ได้)
- **ผูก trace**: source raw-id → หน้า wiki ที่เกิด → citation → คำตอบ query —
  ตรวจสอบย้อนได้ทั้งสาย (provenance chain ตาม [04](04-llm-wiki-library.md) §3)
- เก็บ **model id/prompt/schema version** ที่ใช้ในแต่ละ synthesis เพื่อ reproduce

### 4.3 Backups

- `raw/` (object store): versioning + cross-region copy + checksum manifest
- `wiki/` (git): mirror remote หลายที่ + tag snapshot ต่อรอบ lint
- catalog/metadata DB + PID table: snapshot + WAL/PITR, ทดสอบ restore เป็นรอบ

---

## 5. Preservation & Reliability (reuse ห้องสมุดดิจิทัล)

หลักการสงวนรักษา **reuse เต็มจาก** [../how-to-build-digital-library.md](../how-to-build-digital-library.md) §7
(LOCKSS / Fixity-Checksum / Format Migration / Open Formats / PREMIS) — สรุปการนำมาใช้:

| มาตรการ (จาก digital-library) | ใช้กับ `raw/` (immutable truth) | ใช้กับ `wiki/` snapshot (regenerable) |
|-------------------------------|--------------------------------|----------------------------------------|
| **LOCKSS** (สำเนาหลายที่) | บังคับ — raw หายคือเสียความจริง | mirror git หลายที่ |
| **Fixity / checksum** | SHA ทุกชิ้น + ตรวจเป็นรอบ (จับ bit rot) | git hash ทำ fixity ให้แล้ว |
| **Format migration** | migrate ฟอร์แมต raw เก่าก่อนตาย | wiki = markdown (open format) เสถียร |
| **PREMIS log** | บันทึกประวัติทุกการแปลง/ตรวจ | log.md + git history |

> หมายเหตุความสำคัญ: `wiki/` **regenerate ได้จาก `raw/`+schema** จึงสำคัญรองจาก `raw/` —
> ถ้า wiki เสีย คอมไพล์ใหม่ได้ แต่ถ้า `raw/` เสีย **กู้ไม่ได้** จึงต้องลงทุน preservation
> กับ `raw/` หนักที่สุด (ดู [06](06-schema-example.md) §4 เรื่อง regenerable)

---

## 6. Cost & Scaling

งานหนักของระบบนี้อยู่ที่ **synthesis** (เขียน wiki) ไม่ใช่ retrieval ตอนถาม (ดู [01](01-book-pipeline.md) §4)
จึงคุมต้นทุนคนละจุดกับ RAG ทั่วไป:

| เทคนิค | ทำอะไร | เทียบวิทยาการคอมพิวเตอร์ |
|--------|--------|---------------------------|
| **Cache/memoize คำตอบ** | คำตอบ query ที่ดี file-back เป็นหน้า → ถามซ้ำอ่านจากหน้า ไม่เรียก LLM ใหม่ | memoization / incremental build cache |
| **Small-model-for-easy-tasks** | lint โครงสร้าง/embedding/classify ใช้โมเดลเล็ก-ถูก; synthesis ยากใช้โมเดลใหญ่ | tiered compute |
| **Batch synthesis** | รวม source แล้วคอมไพล์เป็นชุด/นอกเวลาพีค (off-peak) | batch job / build farm |
| **Per-book modular scaling** | แต่ละเล่ม = bounded context สเกล/คอมไพล์อิสระ ไม่ต้อง rebuild ทั้งห้องสมุด | microservice / per-module build |

> เพราะเล่มแยกขอบเขต (ดู [04](04-llm-wiki-library.md) §5) จึงเพิ่มเล่ม = เพิ่ม worker/คิวของเล่มนั้น
> โดยไม่กระทบเล่มอื่น — ต่างจาก "วิกิยักษ์ใบเดียว" ที่ต้องคอมไพล์ใหม่ทั้งก้อน (แพง)

---

## 7. ตาราง map: สเตจ/องค์ประกอบ → building block

| สเตจ/องค์ประกอบ (ชั้น) | building block บนแพลตฟอร์ม | ตัวเลือกอ้างอิง |
|--------------------------|-----------------------------|------------------|
| `raw/` immutable (ชั้น 2) | object store + lock + checksum + append log | S3/MinIO, OCFL, git-annex |
| `wiki/` regenerable (ชั้น 2) | git repo + markdown + PR review | Git host (PR = HITL) |
| `schema` (ชั้น 2) | ไฟล์ versioned ใน git | `CLAUDE.md`/`AGENTS.md` |
| **ingest** (ชั้น 1) | event/on-demand job → store + log | orchestrator worker |
| **synthesis** (ชั้น 1) | per-book queue worker + LLM → PR | LLM API/on-prem + queue |
| **lint** (ชั้น 1) | scheduled job + CI check → รายงาน/PR | cron + CI runner |
| **query** (ชั้น 1) | online service (+ optional retrieval) | API + (pgvector/Qdrant) |
| Vector index (retrieval, ทางเลือก) | vector DB ฝัง PID เป็น payload | pgvector/FAISS/Qdrant/Milvus |
| Catalog/metadata (ชั้น 3) | relational DB | PostgreSQL |
| Union index/ค้นข้ามเล่ม (ชั้น 3) | full-text search engine | Elasticsearch/Solr |
| PID + resolver (ชั้น 3) | indirection service (PID→path) | DOI/Handle/ARK-style |
| Cross-book query router (ชั้น 3) | route → fan-out → synthesize | query router service |
| Preservation (ทุกชั้น) | LOCKSS + fixity + PREMIS + backup | ดู digital-library §7 |

---

## 8. เช็กลิสต์ความพร้อมด้านปฏิบัติการ (Ops Readiness)

- [ ] `raw/` อยู่บน object store แบบ **immutable (object-lock/WORM)** + **SHA-256 ทุกชิ้น** + append-only ingest log
- [ ] `wiki/` อยู่บน **git** และ **regenerate จาก raw+schema ได้จริง** (พิสูจน์ด้วยการคอมไพล์ซ้ำ)
- [ ] ตัดสินใจชัดว่า **ใช้ vector index หรือไม่** — ถ้าใช้ ฝัง **PID ของหน้า** เป็น payload และเข้าใจว่าเป็น *retrieval ทางเลือก* ไม่ใช่แหล่งความรู้
- [ ] เลือก **LLM hosting** ตาม privacy/cost ของ `raw/` (cloud vs on-prem) + ตรึงเวอร์ชัน embedding
- [ ] ตั้ง **catalog DB (Postgres)** + **union search (ES/Solr)** + **PID resolver** พร้อมตาราง PID→path
- [ ] orchestrate ไปป์ไลน์: ingest (event), synthesis (queue→PR), **lint เป็นรอบ**, query (online)
- [ ] **CI checks** ทุก PR: link health · citation present · schema conformance · cross-book PID resolvable
- [ ] มี **HITL hook** ที่ PR สำหรับการแก้เชิงข้อเท็จจริง (นโยบาย → [08](08-expert-team-and-governance.md))
- [ ] emit **health metrics**: orphan, broken link, no-citation, cross-book link fail, query latency, cost/query → dashboard + alert
- [ ] **audit/provenance log** ครบสาย (raw-id → หน้า → citation → คำตอบ) + เก็บ model/schema version
- [ ] **backup**: raw versioning+cross-region, git mirror+snapshot, DB PITR — และ **ทดสอบ restore**
- [ ] **preservation**: LOCKSS + fixity รอบ + format migration ของ raw + PREMIS (reuse [digital-library §7](../how-to-build-digital-library.md))
- [ ] คุมต้นทุน: cache/file-back, small-model-for-easy-tasks, batch synthesis, **per-book modular scaling**

---

> **แก่นสำคัญ:** แพลตฟอร์มนี้คือ "คอมไพเลอร์ความรู้ที่ดูแลได้" — `raw/` (object store, immutable)
> = ซอร์สที่ต้องสงวนรักษาสุด, LLM+orchestrator = คอมไพเลอร์, `wiki/` (git) = artifact ที่
> regenerate/observe ได้, vector = retrieval ทางเลือก, และ catalog+resolver+router = "ตัวเชื่อม
> หลาย artifact" ระดับห้องสมุด ทั้งหมดวัดสุขภาพด้วย metric ที่ผูกกับ citation/provenance
>
> อ้างอิงแนวคิดต้นทาง: Andrej Karpathy, *"LLM Wiki"*
> ([gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f), 2025–2026)
> · โมเดลกลาง [00-overview.md](00-overview.md) · เฟสการทำ [07-implementation-roadmap.md](07-implementation-roadmap.md)
> · ทีม/ธรรมาภิบาล [08-expert-team-and-governance.md](08-expert-team-and-governance.md)
