# Functional Requirements — VisionOps Reliability Platform

## วัตถุประสงค์

เอกสารนี้กำหนด Functional Requirements ของระบบ VisionOps Reliability Platform

Functional Requirements คือข้อกำหนดว่า “ระบบต้องทำอะไรได้บ้าง” ในเชิงพฤติกรรมของระบบ เช่น การรับ Event, ตรวจสอบข้อมูล, สร้างสถานะ, ส่งงานเข้า Queue, ประมวลผลโดย Worker, แสดงผลผ่าน Dashboard และเปิด endpoint สำหรับตรวจสุขภาพระบบ

เอกสารนี้เป็นส่วนหนึ่งของ M0 — Safety & Design และอ้างอิงจาก `docs/project-charter.md`

---

## Scope ของ Functional Requirements

ระบบ VisionOps ใช้ **Synthetic Vision Event** เป็น workload กลางของโปรเจกต์ โดย Event อาจแทนเหตุการณ์จำลอง เช่น

- PPE compliance event
- vehicle detection event
- anomaly event
- generic image-processing event

ระบบนี้ไม่ได้มีเป้าหมายเพื่อสร้าง Product Feature ใหญ่ แต่มีเป้าหมายเพื่อใช้เป็น workload สำหรับเรียนรู้ Cloud, DevOps, Kubernetes, CI/CD, GitOps, Observability และ SRE

---

## Core Components

| Component | หน้าที่ |
|---|---|
| Event API | รับและตรวจสอบ Event จาก client |
| Worker | รับงานจาก Queue แล้วประมวลผลแบบ asynchronous |
| Database | เก็บ metadata และสถานะของ Event |
| Queue | แยก API ออกจาก Worker และรองรับ retry/DLQ |
| Object Storage | เก็บ object หรือ result ของ Event |
| Dashboard | แสดงสถานะ Event และภาพรวมระบบแบบง่าย |
| Event Generator | สร้าง traffic จำลองสำหรับ test/load/failure |
| Observability Endpoints | เปิด metrics, health และ version information |

---

## Priority Definition

| Priority | ความหมาย |
|---|---|
| Must | ต้องมีใน Core Scope หากไม่มีถือว่าระบบยังไม่ผ่านเป้าหมายหลัก |
| Should | ควรมีใน Core Scope แต่สามารถเลื่อนไปหลังจาก Must เสร็จได้ |
| Could | เป็น Optional หรือ Extension หลัง Core เสร็จ |
| Won't | ไม่ทำในรอบนี้ หรืออยู่นอก Scope ของ Portfolio |

---

## Event Status Definition

| Status | ความหมาย |
|---|---|
| `RECEIVED` | API รับ Event แล้วและกำลังเตรียมบันทึก |
| `QUEUED` | Event ถูกส่งเข้า Queue เพื่อรอ Worker ประมวลผล |
| `PROCESSING` | Worker กำลังประมวลผล Event |
| `RETRYING` | Event ประมวลผลไม่สำเร็จชั่วคราวและจะถูก retry |
| `SUCCEEDED` | Event ประมวลผลสำเร็จ |
| `FAILED` | Event ล้มเหลวแบบไม่ควร retry ต่อ |
| `DLQ` | Event หรือ message ถูกส่งไป Dead Letter Queue หลัง retry เกินขอบเขต |

---

## Functional Requirements

| ID | Requirement | Priority | Core/Optional |
|---|---|---|---|
| FR-001 | ระบบต้องรับ Vision Event ใหม่ผ่าน API ได้ | Must | Core |
| FR-002 | ระบบต้อง validate request payload ก่อนรับ Event | Must | Core |
| FR-003 | ระบบต้องสร้าง `event_id` ให้ Event ที่รับสำเร็จ | Must | Core |
| FR-004 | ระบบต้องรองรับ `idempotency_key` เพื่อลดปัญหา duplicate submission | Must | Core |
| FR-005 | ระบบต้องบันทึกสถานะเริ่มต้นของ Event ลงฐานข้อมูล | Must | Core |
| FR-006 | ระบบต้องส่ง Event ที่รับสำเร็จเข้า Queue เพื่อประมวลผลแบบ asynchronous | Must | Core |
| FR-007 | ระบบต้องตอบกลับ client โดยไม่ต้องรอให้ Worker ประมวลผลจนเสร็จ | Must | Core |
| FR-008 | ระบบต้องให้ client ตรวจสอบสถานะของ Event ได้ | Must | Core |
| FR-009 | ระบบต้องแสดงรายการ Event แบบพื้นฐานได้ | Should | Core |
| FR-010 | Worker ต้องรับงานจาก Queue ได้ | Must | Core |
| FR-011 | Worker ต้องเปลี่ยนสถานะ Event เป็น `PROCESSING` เมื่อเริ่มประมวลผล | Must | Core |
| FR-012 | Worker ต้องเปลี่ยนสถานะ Event เป็น `SUCCEEDED` เมื่อประมวลผลสำเร็จ | Must | Core |
| FR-013 | Worker ต้องเปลี่ยนสถานะ Event เป็น `FAILED` เมื่อเกิดความผิดพลาดที่ไม่ควร retry | Must | Core |
| FR-014 | Worker ต้องรองรับ retry สำหรับ transient failure | Must | Core |
| FR-015 | Event ที่ retry เกินขอบเขตต้องถูกส่งไปยัง DLQ หรือมีสถานะที่ตรวจสอบได้ | Must | Core |
| FR-016 | ระบบต้องมี endpoint สำหรับ liveness check | Must | Core |
| FR-017 | ระบบต้องมี endpoint สำหรับ readiness check | Must | Core |
| FR-018 | ระบบต้องมี endpoint สำหรับ metrics | Must | Core |
| FR-019 | ระบบต้องมี endpoint สำหรับ version/build information | Must | Core |
| FR-020 | ระบบต้องสร้าง structured logs ที่มี correlation/request context | Must | Core |
| FR-021 | ระบบต้องส่งหรือ expose telemetry ที่จำเป็นต่อ Observability | Must | Core |
| FR-022 | Dashboard ต้องแสดงสถานะ Event ได้แบบพื้นฐาน | Should | Core |
| FR-023 | Dashboard ต้องแสดงสถานะระบบหรือ service health แบบพื้นฐานได้ | Should | Core |
| FR-024 | Event Generator ต้องสร้าง normal traffic ได้ | Should | Core |
| FR-025 | Event Generator ต้องสร้าง invalid payload เพื่อทดสอบ validation ได้ | Should | Core |
| FR-026 | Event Generator ต้องสร้าง traffic spike เพื่อใช้กับ load test ได้ | Should | Core |
| FR-027 | ระบบต้องรองรับ lab-only failure mode สำหรับ GameDay | Should | Core |
| FR-028 | ระบบต้องปิดหรือป้องกัน lab-only failure mode นอก environment ที่ตั้งใจ | Must | Core |
| FR-029 | ระบบต้องรองรับ backup และ restore ของข้อมูลสำคัญใน lab | Should | Core |
| FR-030 | ระบบต้องรองรับการเชื่อมต่อกับ AI/CV inference worker ในอนาคต | Could | Optional |

---

## API Requirements

### FR-API-001 — Create Event

ระบบต้องมี API สำหรับสร้าง Event ใหม่

ตัวอย่าง endpoint:

```text
POST /api/v1/events
```

Expected behavior:

- รับ payload ตาม schema ที่กำหนด
- validate required fields
- reject invalid payload ด้วย error response ที่เข้าใจได้
- สร้าง `event_id`
- บันทึก Event ลง database
- ส่ง message เข้า Queue
- return `event_id` และสถานะเริ่มต้น

Acceptance criteria:

- [ ] valid payload ได้ HTTP success response
- [ ] invalid payload ถูก reject
- [ ] Event ที่รับสำเร็จมี `event_id`
- [ ] Event ถูกบันทึกใน database
- [ ] Event ถูกส่งเข้า Queue
- [ ] response ไม่รอ Worker ประมวลผลเสร็จ
- [ ] response มี correlation หรือ request context ที่ใช้ตรวจสอบย้อนหลังได้

---

### FR-API-002 — Get Event Status

ระบบต้องมี API สำหรับดูสถานะ Event

ตัวอย่าง endpoint:

```text
GET /api/v1/events/{eventId}
```

Expected behavior:

- รับ `eventId`
- ค้นหาสถานะใน database
- return สถานะปัจจุบัน เช่น `RECEIVED`, `QUEUED`, `PROCESSING`, `SUCCEEDED`, `FAILED`, `DLQ`
- ถ้าไม่พบ Event ต้องตอบ not found อย่างเหมาะสม

Acceptance criteria:

- [ ] Event ที่มีอยู่สามารถ query ได้
- [ ] Event ที่ไม่มีอยู่ตอบ not found
- [ ] response ไม่เปิดเผยข้อมูลภายในที่ไม่จำเป็น
- [ ] response มีสถานะที่อ่านเข้าใจง่าย

---

### FR-API-003 — List Events

ระบบควรมี API สำหรับดูรายการ Event แบบพื้นฐาน

ตัวอย่าง endpoint:

```text
GET /api/v1/events
```

Expected behavior:

- แสดงรายการ Event ล่าสุด
- รองรับ pagination ขั้นพื้นฐาน
- สามารถกรองตามสถานะได้ในอนาคต

Acceptance criteria:

- [ ] list events ได้
- [ ] มี pagination หรือ limit เพื่อป้องกัน query ใหญ่เกินไป
- [ ] ไม่ดึงข้อมูลทั้งหมดแบบไม่มีขอบเขต
- [ ] response แสดงข้อมูลเท่าที่จำเป็นต่อ dashboard และ debugging

---

### FR-API-004 — Health Endpoints

ระบบต้องมี health endpoints แยกตามหน้าที่

ตัวอย่าง endpoint:

```text
GET /health/live
GET /health/ready
```

Expected behavior:

- `/health/live` ใช้ตรวจว่า process ยังทำงาน
- `/health/ready` ใช้ตรวจว่า service พร้อมรับ traffic
- readiness สามารถ fail ได้เมื่อ dependency สำคัญยังไม่พร้อม

Acceptance criteria:

- [ ] liveness endpoint ตอบได้เมื่อ process ยังทำงาน
- [ ] readiness endpoint สะท้อนความพร้อมรับ traffic
- [ ] endpoint เหมาะสำหรับใช้กับ Kubernetes probes ในอนาคต
- [ ] readiness failure ไม่ควรถูกออกแบบให้ restart process โดยไม่จำเป็น

---

### FR-API-005 — Metrics Endpoint

ระบบต้องมี endpoint สำหรับ metrics

ตัวอย่าง endpoint:

```text
GET /metrics
```

Expected behavior:

- expose metrics ที่ Prometheus scrape ได้
- มี HTTP request metrics
- มี application metrics ที่จำเป็นต่อ SLI/SLO ในอนาคต

Acceptance criteria:

- [ ] `/metrics` เปิดข้อมูล metrics ที่จำเป็น
- [ ] ไม่ expose sensitive data
- [ ] metric labels ไม่ใช้ high-cardinality field เช่น raw event ID
- [ ] metrics สามารถนำไปใช้สร้าง dashboard และ alert ใน phase ถัดไปได้

---

### FR-API-006 — Version Endpoint

ระบบต้องมี endpoint สำหรับระบุ version/build

ตัวอย่าง endpoint:

```text
GET /version
```

Expected behavior:

- แสดง service name
- แสดง git commit SHA หรือ build version
- แสดง build time ตามความเหมาะสม

Acceptance criteria:

- [ ] ตรวจสอบได้ว่า deployment ปัจจุบันมาจาก version ใด
- [ ] ข้อมูล version สามารถใช้เชื่อมกับ CI/CD และ GitOps evidence ได้
- [ ] ข้อมูล version ไม่เปิดเผยข้อมูลลับ เช่น token, internal path หรือ credential

---

## Event Payload Requirements

ตัวอย่างโครงสร้าง Event ขั้นต้น:

```json
{
  "idempotency_key": "client-generated-key",
  "event_type": "ppe_compliance",
  "source": "synthetic-camera-01",
  "object_uri": "s3://bucket/key-or-local-placeholder",
  "captured_at": "2026-09-07T10:00:00Z"
}
```

Required fields:

| Field | Required | คำอธิบาย |
|---|---|---|
| `idempotency_key` | Yes | key จาก client เพื่อช่วยป้องกัน duplicate submission |
| `event_type` | Yes | ประเภทของ event เช่น `ppe_compliance`, `vehicle_detection`, `anomaly` |
| `source` | Yes | แหล่งกำเนิด event แบบ synthetic เช่น `synthetic-camera-01` |
| `object_uri` | No | path หรือ URI ของ object ที่ใช้ทดสอบ |
| `captured_at` | No | เวลาจำลองที่ event ถูก capture |

Validation rules:

- [ ] `event_type` ต้องอยู่ในรายการที่ระบบรองรับ
- [ ] `idempotency_key` ต้องไม่ว่าง
- [ ] `source` ต้องไม่ว่าง
- [ ] request body ต้องไม่ใหญ่เกิน limit ที่กำหนดในอนาคต
- [ ] ห้ามส่งข้อมูลจริงหรือข้อมูลส่วนบุคคลจริงใน payload

---

## Event Lifecycle Requirements

ระบบต้องจัดการสถานะ Event ตาม lifecycle ต่อไปนี้

```text
RECEIVED
  → QUEUED
  → PROCESSING
  → SUCCEEDED

หรือ

PROCESSING
  → RETRYING
  → PROCESSING
  → SUCCEEDED

หรือ

PROCESSING / RETRYING
  → FAILED / DLQ
```

Acceptance criteria:

- [ ] ทุก Event ที่รับสำเร็จต้องมีสถานะตรวจสอบได้
- [ ] ไม่มี Event ที่ “หายเงียบ” โดยไม่มีสถานะ
- [ ] failure ต้องมี error code หรือ reason ที่เหมาะสม
- [ ] retry ต้องมีขอบเขต
- [ ] DLQ หรือ terminal failed state ต้องตรวจสอบได้
- [ ] state transition ต้องไม่ข้ามขั้นตอนสำคัญโดยไม่มีเหตุผล

---

## Queue and Worker Requirements

Worker เป็นส่วนที่ทำให้ระบบแยก API ออกจากการประมวลผลจริง

Requirements:

- [ ] Worker ต้องรับ message จาก Queue ได้
- [ ] Worker ต้องประมวลผล message แบบ asynchronous
- [ ] Worker ต้อง update สถานะ Event ก่อนและหลังประมวลผล
- [ ] Worker ต้อง delete message จาก Queue หลังจากบันทึกผลสำเร็จแล้วเท่านั้น
- [ ] Worker ต้องรองรับ transient failure ด้วย retry ที่มีขอบเขต
- [ ] Worker ต้องไม่ retry แบบไม่มีที่สิ้นสุด
- [ ] Worker ต้องรองรับ poison message โดยส่งไป DLQ หรือ terminal failed state
- [ ] Worker ต้องสร้าง logs/metrics ที่ช่วย debug ได้

---

## Dashboard Requirements

Dashboard เป็น UI แบบ minimal สำหรับแสดง operational state ไม่ใช่ product UI ขนาดใหญ่

Requirements:

- [ ] แสดงรายการ Event ล่าสุด
- [ ] แสดงสถานะ Event
- [ ] แสดงเวลาที่รับ Event และเวลาที่ประมวลผลสำเร็จถ้ามี
- [ ] แสดง service health แบบพื้นฐาน
- [ ] แสดงข้อความสถานะ ไม่พึ่งสีอย่างเดียว
- [ ] ไม่จำเป็นต้องมี authentication complex ใน Core Scope
- [ ] ไม่จำเป็นต้องมี design system ขนาดใหญ่

---

## Event Generator Requirements

Event Generator ใช้สำหรับสร้าง traffic เพื่อทดสอบระบบ

Requirements:

- [ ] สร้าง valid events ได้
- [ ] สร้าง invalid events ได้
- [ ] สร้าง traffic spike ได้
- [ ] กำหนดจำนวน event และ rate ได้
- [ ] ไม่ยิง traffic ไปยังระบบภายนอกที่ไม่ใช่ environment ของโปรเจกต์
- [ ] ใช้ข้อมูล synthetic เท่านั้น

---

## Lab-only Failure Requirements

ระบบควรมี failure mode สำหรับ GameDay เช่น

- simulate slow processing
- simulate worker failure
- simulate invalid dependency
- simulate memory pressure เฉพาะ lab
- simulate poison message

Rules:

- [ ] ต้องเปิดเฉพาะ environment ที่ระบุว่าเป็น lab/local
- [ ] ต้องปิดโดย default
- [ ] ต้องไม่เปิดใน public/demo environment โดยไม่ได้ตั้งใจ
- [ ] ต้องมี documentation ว่าใช้เพื่อ GameDay เท่านั้น
- [ ] ต้องไม่สามารถถูกเรียกใช้งานโดย accident ใน environment ที่ไม่ใช่ lab

---

## Out of Scope สำหรับ Functional Requirements รอบนี้

สิ่งต่อไปนี้ไม่อยู่ใน Core Scope:

- ระบบ login ผู้ใช้จริง
- payment
- real CCTV หรือ real personal image data
- model training pipeline ขนาดใหญ่
- multi-tenant management
- complex admin portal
- mobile application
- notification system สำหรับ user จริง
- production customer workflow
- multi-region business workflow

---

## Requirement Traceability

| Source | Requirement |
|---|---|
| Project Charter | Workload กลางใช้ Synthetic Vision Event |
| Project Charter | Event API, Worker, Queue, Database, Object Storage และ Dashboard |
| Project Charter | Health, Readiness, Metrics และ Version endpoint |
| Project Charter | Queue retry, DLQ, Observability และ SRE GameDay |
| Project Charter | Optional AI/CV Operations Extension |

---

## Change Log

| Date | Change |
|---|---|
| 2026-09-07 | Created initial functional requirements |
