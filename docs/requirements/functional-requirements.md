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