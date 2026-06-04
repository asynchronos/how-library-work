# ชุดงานวิจัย: LLM-Wiki Library

> สารบัญและแผนที่ของชุดเอกสาร `docs/llm-wiki/` — งานค้นคว้าเชิงออกแบบว่าด้วย
> **ห้องสมุดของหนังสือที่เป็น LLM-Wiki**: ผลิตหนังสือแต่ละเล่มด้วยไปป์ไลน์
> `ingest → synthesis → lint → query` (แรงบันดาลใจจาก *Karpathy LLM Wiki*) แล้วนำ
> หลักการห้องสมุดกายภาพมาเก็บหนังสือเหล่านั้นแบบ **citation**

> เริ่มอ่านที่ [00-overview.md](00-overview.md) ซึ่งเป็น "โมเดลกลาง (canonical model)"
> ที่เอกสารทุกชิ้นยึดถือ

---

## โมเดล 3 ชั้น และตำแหน่งของเอกสารแต่ละชิ้น

```
╔══════════════════════════════════════════════════════════════════════╗
║                        00 · OVERVIEW (โมเดลกลาง)                       ║
║         raw = ความจริง · LLM = คอมไพเลอร์ · citation = หัวใจ            ║
╚══════════════════════════════════════════════════════════════════════╝
        │                       │                        │
        ▼                       ▼                        ▼
┌────────────────┐   ┌────────────────────┐   ┌──────────────────────────┐
│ ชั้น 1 · PIPELINE│   │ ชั้น 2 · BOOK        │   │ ชั้น 3 · LIBRARY          │
│ โรงงานผลิตเล่ม   │   │ หนังสือหนึ่งเล่ม     │   │ ห้องสมุดหลายเล่ม + citation│
│                │   │                    │   │                          │
│ 01 book-       │   │ 02 llm-wiki-book   │   │ 03 physical-library-      │
│    pipeline    │   │    (raw+wiki+schema)│  │    principles (สกัดหลักการ)│
│ ingest→synth   │   │                    │   │ 04 llm-wiki-library       │
│ →lint→query    │   │                    │   │    (รวมร่าง · capstone)    │
└───────┬────────┘   └─────────┬──────────┘   └────────────┬─────────────┘
        │                      │                           │
        └──────────────┬───────┴───────────────┬───────────┘
                       ▼                        ▼
        ┌──────────────────────────┐   ┌────────────────────────────────┐
        │ ลงมือทำจริง (Practice)     │   │ การนำไป implement (Delivery)    │
        │ 05 worked-example (Raft)  │   │ 07 implementation-roadmap       │
        │ 06 schema-example (CLAUDE)│   │ 08 expert-team-and-governance   │
        │                          │   │ 09 reference-stack-and-ops      │
        └──────────────────────────┘   └────────────────────────────────┘
```

---

## สารบัญเอกสาร

| # | เอกสาร | ชั้น/กลุ่ม | เนื้อหาโดยย่อ |
|---|--------|-----------|---------------|
| 00 | [overview](00-overview.md) | โมเดลกลาง | โมเดล 3 ชั้น, อภิธานศัพท์, การปรับวัตถุประสงค์จาก RAG-centric |
| 01 | [book-pipeline](01-book-pipeline.md) | ชั้น 1 | `ingest → synthesis → lint → query` + compiler analogy + ต่างจาก RAG |
| 02 | [llm-wiki-book](02-llm-wiki-book.md) | ชั้น 2 | หนึ่งเล่ม = `raw/`+`wiki/`+`schema`, ขอบเขต, versioning, citation ในเล่ม |
| 03 | [physical-library-principles](03-physical-library-principles.md) | ชั้น 3 | สกัดหลักห้องสมุด: catalog, DDC/LCC, authority, FRBR, preservation |
| 04 | [llm-wiki-library](04-llm-wiki-library.md) | ชั้น 3 (capstone) | รวมร่าง: catalog + citation ในเล่ม/ข้ามเล่ม + PID + cross-book query |
| 05 | [worked-example](05-worked-example.md) | ลงมือทำ | สร้างเล่ม Raft end-to-end ผ่านทุกสเตจ |
| 06 | [schema-example](06-schema-example.md) | ลงมือทำ | ตัวอย่าง `CLAUDE.md` เต็ม + การเวอร์ชัน schema |
| 07 | [implementation-roadmap](07-implementation-roadmap.md) | delivery | roadmap เฟส 0–5 + gate + timeline + DoD |
| 08 | [expert-team-and-governance](08-expert-team-and-governance.md) | delivery | ทีม 8 บทบาท + RACI + governance + quality gate |
| 09 | [reference-stack-and-ops](09-reference-stack-and-ops.md) | delivery | reference architecture + เครื่องมือ + ops + preservation |

---

## เส้นทางการอ่านแนะนำ (Reading Paths)

| ถ้าคุณคือ... | อ่านตามลำดับ |
|--------------|--------------|
| **อยากเข้าใจแนวคิด** | 00 → 01 → 02 → 04 |
| **วิศวกร/จะลงมือทำ** | 00 → 01 → 06 → 05 → 09 |
| **บรรณารักษ์/Information Architect** | 00 → 03 → 04 |
| **ผู้นำโครงการ/วางแผน** | 00 → 07 → 08 → 09 |

---

> แนวคิดต้นทาง: Andrej Karpathy, *"LLM Wiki"*
> ([gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f), 2025–2026)
>
> เอกสารรุ่นก่อน (กรอบ RAG-centric ที่ปรับแล้ว เก็บไว้เพื่ออ้างอิงเชิงประวัติ):
> [../llm-wiki-library.md](../llm-wiki-library.md) · [../llm-wiki-adoption-workflow.md](../llm-wiki-adoption-workflow.md)
