# Non-functional Requirements — VisionOps Reliability Platform

## วัตถุประสงค์

เอกสารนี้กำหนด Non-functional Requirements ของ VisionOps Reliability Platform

Non-functional Requirements คือข้อกำหนดด้านคุณภาพของระบบ เช่น Security, Reliability, Performance, Observability, Maintainability, Portability, Reproducibility, Cost และ Auditability

เอกสารนี้เป็นส่วนหนึ่งของ M0 — Safety & Design และใช้เป็นฐานก่อนออกแบบ Architecture, ADR, Threat Model, SLO และ Cost Guardrails

---

## Scope ของ Non-functional Requirements

โปรเจกต์นี้เป็น Production-Oriented Lab ไม่ใช่ Production-Grade Service

ดังนั้น NFR ในเอกสารนี้จะกำหนดเป้าหมายที่เหมาะกับ

- การเรียนรู้
- การทำ Portfolio
- การทดลองภายใต้งบ AWS ประมาณ $100
- การทำงานคนเดียว
- การแยก Lab Implementation ออกจาก Production Reference Architecture

---

## Non-functional Requirements Summary

| ID | หมวด | Requirement | Priority |
|---|---|---|---|
| NFR-001 | Security | ห้ามใช้ข้อมูลจริงหรือข้อมูลส่วนบุคคลจริงใน Core workload | Must |
| NFR-002 | Security | ห้าม commit secret, token, AWS key, kubeconfig หรือ Terraform state | Must |
| NFR-003 | Security | Root User ต้องเปิด MFA และไม่ใช้สำหรับงานประจำ | Must |
| NFR-004 | Security | Cloud access สำหรับ automation ต้องหลีกเลี่ยง long-lived credentials | Must |
| NFR-005 | Security | S3 bucket และ database ต้องไม่เปิด public โดยไม่จำเป็น | Must |
| NFR-006 | Reliability | ระบบต้องมี health checks แยก liveness และ readiness | Must |
| NFR-007 | Reliability | Worker ต้องรองรับ retry แบบมีขอบเขต | Must |
| NFR-008 | Reliability | Event ที่ process ไม่สำเร็จต้องมีสถานะตรวจสอบได้ | Must |
| NFR-009 | Reliability | Failed deployment ต้องถูก detect และ rollback ได้ใน phase ถัดไป | Should |
| NFR-010 | Performance | ต้องวัด latency และ throughput ก่อน claim performance | Must |
| NFR-011 | Performance | Initial target: API p95 latency ต่ำกว่า 500 ms ภายใต้ expected lab load | Should |
| NFR-012 | Performance | Initial target: Event processing p95 ต่ำกว่า 10 วินาที | Should |
| NFR-013 | Observability | ระบบต้องมี metrics, logs และ traces หรือ trace context ตาม phase | Must |
| NFR-014 | Observability | Logs ต้องเป็น structured logs | Must |
| NFR-015 | Observability | Metrics ต้องหลีกเลี่ยง high-cardinality labels | Must |
| NFR-016 | Operability | ต้องมี runbook สำหรับ failure สำคัญใน phase SRE | Should |
| NFR-017 | Maintainability | Code/config/docs ต้องจัดโครงสร้างให้อ่านง่ายและแยก responsibility | Must |
| NFR-018 | Portability | Core app ต้องรันได้ทั้ง local และ Kubernetes ในอนาคต | Should |
| NFR-019 | Reproducibility | Infrastructure และ deployment ต้องทำซ้ำได้ด้วย code/automation ใน phase ถัดไป | Must |
| NFR-020 | Cost | ต้องตั้ง AWS Budget ก่อนสร้าง resource ที่มีค่าใช้จ่าย | Must |
| NFR-021 | Cost | ต้องมี teardown checklist ก่อนเริ่ม cloud lab | Must |
| NFR-022 | Auditability | Deployment artifact ต้องเชื่อมกลับไปยัง commit ได้ในอนาคต | Should |
| NFR-023 | Accessibility | Dashboard ควรแสดงสถานะเป็นข้อความอ่านง่าย ไม่พึ่งสีอย่างเดียว | Should |
| NFR-024 | Documentation | ทุก claim สำคัญต้องมี evidence หรือระบุว่าเป็น design target | Must |
| NFR-025 | Scope Control | ห้ามเพิ่ม optional tool โดยไม่มี requirement และ trade-off | Must |

---

## Security Requirements

### NFR-SEC-001 — No Real Sensitive Data

ระบบต้องใช้ synthetic data เท่านั้นใน Core Scope

Acceptance criteria:

- [ ] ไม่มี real CCTV data
- [ ] ไม่มี personal image data
- [ ] ไม่มี customer data
- [ ] ไม่มีข้อมูลจากบริษัทที่เคยฝึกงาน
- [ ] ไม่มี secret ใน sample payload

---

### NFR-SEC-002 — No Secrets in Git

ห้าม commit ข้อมูลต่อไปนี้:

- AWS access key
- AWS secret access key
- session token
- `.env`
- private key
- kubeconfig
- Terraform state
- database password
- MFA secret
- account recovery detail

Acceptance criteria:

- [ ] `.gitignore` ป้องกันไฟล์เสี่ยง
- [ ] เอกสารไม่ใส่ secret
- [ ] evidence ไม่เปิดเผยข้อมูลลับ
- [ ] security scanner จะถูกเพิ่มใน phase ถัดไป

---

### NFR-SEC-003 — AWS Root Account Protection

Root User ต้องถูกใช้เฉพาะงานระดับ account/security เท่านั้น

Acceptance criteria:

- [ ] Root MFA enabled
- [ ] Root access keys = 0
- [ ] Root user daily usage prohibited
- [ ] ไม่มี root credential ใน GitHub

---

### NFR-SEC-004 — Least Privilege Direction

ระบบต้องออกแบบให้ IAM permission แคบที่สุดเท่าที่เหมาะสมกับ lab

Acceptance criteria:

- [ ] แยก human permission, pipeline permission และ workload permission
- [ ] ไม่ใช้ wildcard permission โดยไม่มีเหตุผล
- [ ] ใช้ OIDC/temporary credentials สำหรับ GitHub Actions ใน phase ที่เกี่ยวข้อง
- [ ] อธิบายข้อจำกัดของ lab permission หากยังไม่สมบูรณ์

---

## Reliability Requirements

### NFR-REL-001 — Health and Readiness

ระบบต้องรองรับ liveness และ readiness แยกกัน

Acceptance criteria:

- [ ] liveness ใช้ตรวจ process health
- [ ] readiness ใช้ตรวจความพร้อมรับ traffic
- [ ] readiness สามารถ fail เมื่อ dependency สำคัญไม่พร้อม
- [ ] endpoint ใช้ต่อกับ Kubernetes probes ได้

---

### NFR-REL-002 — Bounded Retry and DLQ

Queue/Worker ต้องจัดการ failure โดยไม่ retry แบบไม่มีที่สิ้นสุด

Acceptance criteria:

- [ ] transient failure ถูก retry
- [ ] retry มี max attempts หรือขอบเขต
- [ ] poison message ไป DLQ หรือ terminal failed state
- [ ] DLQ/message failure ตรวจสอบได้

---

### NFR-REL-003 — No Silent Data Loss

Event ที่ API รับสำเร็จต้องมีสถานะตรวจสอบได้เสมอ

Acceptance criteria:

- [ ] ทุก accepted event มี `event_id`
- [ ] event status ตรวจสอบได้
- [ ] failed event มี reason/error code
- [ ] ไม่มี event ที่หายโดยไม่มีสถานะ

---

### NFR-REL-004 — Recovery Direction

ระบบต้องออกแบบให้สามารถ rollback, restore และ rebuild ได้ใน phase ถัดไป

Acceptance criteria:

- [ ] มีแนวทาง rollback
- [ ] มีแนวทาง backup/restore
- [ ] มีแนวทาง rebuild environment จาก code
- [ ] ไม่ claim ว่า production-grade หากยังไม่มี evidence

---

## Performance Requirements

### NFR-PERF-001 — Measure Before Claim

ห้าม claim ว่าระบบเร็วหรือ scalable โดยไม่มีผลวัด

Acceptance criteria:

- [ ] มี metric สำหรับ latency
- [ ] มี metric สำหรับ throughput
- [ ] มี baseline load test ใน phase ถัดไป
- [ ] performance claim ต้องมี environment และ version กำกับ

---

### NFR-PERF-002 — Initial Lab Targets

Initial performance targets สำหรับ lab:

| Metric | Initial Target |
|---|---:|
| API successful request latency | p95 < 500 ms |
| Event processing latency | p95 < 10 วินาที |
| Event accounting | 100% ของ accepted events ต้องตรวจสอบสถานะได้ |

หมายเหตุ: ตัวเลขนี้เป็น design target ต้องปรับจากผลวัดจริงใน phase ถัดไป

---

## Observability Requirements

### NFR-OBS-001 — Metrics

ระบบต้อง expose metrics ที่จำเป็นต่อการดูแลระบบ

ตัวอย่าง metrics:

- request rate
- error rate
- latency
- queue depth
- processing time
- worker success/failure count
- build/version info

Acceptance criteria:

- [ ] มี `/metrics`
- [ ] metric labels ไม่ใช้ event ID หรือ request ID
- [ ] metrics ใช้สร้าง SLI/SLO ได้ในอนาคต

---

### NFR-OBS-002 — Logs

ระบบต้องสร้าง structured logs

Acceptance criteria:

- [ ] logs มี timestamp
- [ ] logs มี level
- [ ] logs มี service name
- [ ] logs มี correlation/request context
- [ ] logs ไม่บันทึก secret หรือข้อมูลอ่อนไหว

---

### NFR-OBS-003 — Traces / Correlation

ระบบต้องเตรียมแนวทางสำหรับ trace หรือ correlation ข้าม API → Queue → Worker

Acceptance criteria:

- [ ] request มี correlation ID
- [ ] queue message เก็บ correlation context
- [ ] worker log สามารถโยงกลับ event/request ได้
- [ ] OpenTelemetry จะถูกเพิ่มใน phase Observability

---

## Operability Requirements

### NFR-OPS-001 — Runbook Direction

Failure สำคัญต้องมี runbook ใน phase SRE

ตัวอย่าง:

- API unavailable
- worker down
- queue backlog
- DLQ not empty
- database unavailable
- failed deployment
- high latency
- high error rate

Acceptance criteria:

- [ ] ระบุ failure สำคัญที่ต้องมี runbook
- [ ] runbook จะถูกสร้างใน phase M7/M8
- [ ] alert ในอนาคตต้องชี้ไปยัง runbook

---

## Maintainability Requirements

### NFR-MAINT-001 — Clear Repository Boundary

ระบบต้องแยก repository ตาม responsibility

| Repository | Responsibility |
|---|---|
| visionops-app | application workload |
| visionops-infra | infrastructure, automation, security, observability, docs |
| visionops-gitops | Kubernetes desired state และ Argo CD configuration |

Acceptance criteria:

- [ ] เอกสารและไฟล์อยู่ใน repo ที่ถูกต้อง
- [ ] ไม่เอา application source ไปไว้ใน infra repo โดยไม่จำเป็น
- [ ] ไม่เอา Terraform state หรือ secret ไปไว้ใน GitOps repo

---

### NFR-MAINT-002 — Documentation First for M0

ใน M0 ต้องออกแบบและบันทึกก่อน implementation

Acceptance criteria:

- [ ] มี charter
- [ ] มี requirements
- [ ] มี architecture diagrams
- [ ] มี ADR สำคัญ
- [ ] มี threat model
- [ ] มี cost guardrails

---

## Portability and Reproducibility Requirements

### NFR-REP-001 — Local First

ระบบต้องออกแบบให้ทดลองส่วนใหญ่บน local ก่อนใช้ AWS

Acceptance criteria:

- [ ] local development path ถูกกำหนด
- [ ] cloud lab ใช้เพื่อ validate integration
- [ ] ไม่เปิด EKS/RDS/ALB ทิ้งไว้โดยไม่จำเป็น

---

### NFR-REP-002 — Reproducible Infrastructure Direction

Infrastructure ต้องสร้างซ้ำได้ด้วย Terraform ใน phase ถัดไป

Acceptance criteria:

- [ ] infrastructure ไม่ควรถูกสร้างด้วย console เป็นหลัก
- [ ] Terraform จะเป็น source of truth สำหรับ AWS resources
- [ ] manual console ใช้เพื่อ inspect/troubleshoot เท่านั้น

---

## Cost Requirements

### NFR-COST-001 — Budget Guardrail

ต้องตั้ง AWS Budget ก่อนเริ่มสร้าง resource ที่มีค่าใช้จ่าย

Acceptance criteria:

- [ ] Budget amount สอดคล้องกับ hard cap
- [ ] มี alert threshold หลายระดับ
- [ ] เข้าใจว่า AWS Budget เป็น notification ไม่ใช่ hard stop
- [ ] ไม่มี billing detail sensitive ถูกเผยแพร่

---

### NFR-COST-002 — Teardown Discipline

ทุก cloud session ต้องมี teardown หรือ resource review

Acceptance criteria:

- [ ] มี teardown checklist ใน phase นี้
- [ ] EKS/RDS/ALB/NAT Gateway ต้องเปิดเฉพาะเมื่อจำเป็น
- [ ] หลังใช้งานต้องตรวจ resource ค้าง
- [ ] ต้องมี cost evidence แบบ sanitized

---

## Auditability Requirements

### NFR-AUDIT-001 — Evidence-based Claims

ทุก claim สำคัญใน Portfolio ต้องมีหลักฐาน

Acceptance criteria:

- [ ] claim ด้าน security มี evidence
- [ ] claim ด้าน cost มี evidence
- [ ] claim ด้าน reliability มี evidence
- [ ] claim ด้าน deployment มี evidence
- [ ] ถ้ายังไม่มี evidence ต้องระบุว่าเป็น design target

---

### NFR-AUDIT-002 — Commit and Artifact Traceability

ใน phase ถัดไป deployment artifact ต้อง trace กลับไปที่ commit ได้

Acceptance criteria:

- [ ] image tag ใช้ git SHA หรือ digest
- [ ] deployment แสดง version ได้
- [ ] release evidence ระบุ commit/version/environment

---

## Accessibility Requirements

### NFR-ACC-001 — Dashboard Readability

Dashboard ต้องอ่านเข้าใจง่ายและไม่พึ่งสีอย่างเดียว

Acceptance criteria:

- [ ] status มีข้อความ เช่น `SUCCEEDED`, `FAILED`, `PROCESSING`
- [ ] ใช้สีได้ แต่ต้องมี text label เสมอ
- [ ] error message อ่านเข้าใจได้
- [ ] layout ไม่ซับซ้อนเกินไป

---

## Scope Control Requirements

### NFR-SCOPE-001 — No Tool Without Requirement

ห้ามเพิ่มเครื่องมือใหม่เพียงเพราะอยากให้ portfolio ดูเยอะ

Acceptance criteria:

- [ ] tool ใหม่ต้องมี requirement รองรับ
- [ ] tool ใหม่ต้องมี trade-off หรือ ADR เมื่อกระทบ architecture
- [ ] optional tool ต้องแยกจาก core scope
- [ ] ถ้า tool เพิ่ม complexity มากกว่าคุณค่า ต้อง defer

---

## Out of Scope สำหรับ NFR รอบนี้

- การรับประกัน SLA ทางกฎหมาย
- multi-region high availability จริง
- enterprise compliance certification
- production on-call rotation จริง
- 24/7 public production operation
- real customer data protection program
- external penetration test
- full disaster recovery แบบองค์กร

---

## Requirement Traceability

| Source | NFR Area |
|---|---|
| Project Charter | Security guardrails |
| Project Charter | AWS cost hard cap และ budget discipline |
| Project Charter | Local-first และ short-lived cloud lab |
| Project Charter | Observability ด้วย metrics, logs, traces |
| Project Charter | SLI/SLO, alert, incident response และ recovery |
| Project Charter | Lab vs Production Reference distinction |
| Project Charter | Evidence-based portfolio claims |

---

## Change Log

| Date | Change |
|---|---|
| 2026-09-07 | Created initial non-functional requirements |