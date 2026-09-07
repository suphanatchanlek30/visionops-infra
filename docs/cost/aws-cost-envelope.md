# AWS Cost Envelope — VisionOps Reliability Platform

## วัตถุประสงค์

เอกสารนี้กำหนด **AWS Cost Envelope** สำหรับโปรเจกต์ VisionOps Reliability Platform

Cost Envelope คือกรอบงบประมาณที่ใช้ควบคุมว่าโปรเจกต์นี้ควรใช้ AWS Credit อย่างไร Resource ประเภทใดเสี่ยงทำให้เกิดค่าใช้จ่ายสูง และเมื่อไรควรหยุด cloud work เพื่อป้องกันค่าใช้จ่ายเกินงบ

เอกสารนี้เป็นส่วนหนึ่งของ **M0 — Safety & Design** และต้องใช้ร่วมกับไฟล์:

- `docs/cost/teardown-checklist.md`
- `docs/project-charter.md`
- `docs/security/threat-model.md`
- `docs/requirements/non-functional-requirements.md`

> หมายเหตุ: เอกสารนี้เป็น **Design Baseline** เท่านั้น ยังไม่มีการสร้าง AWS Resource ใหม่จาก Issue นี้

---

## Cost Envelope Status

| รายการ | ค่า |
|---|---|
| Version | v0 |
| Phase | M0 — Safety & Design |
| Status | Design baseline |
| Cloud resources created by this issue | No |
| Sensitive details recorded | No |
| Primary cloud | AWS |
| Primary region | ap-southeast-1 |
| Owner | Suphanat Chanlek |
| Last updated | 2026-09-07 |

---

## Budget Policy

| รายการ | ค่า |
|---|---:|
| Project hard cap | 100 USD |
| Operating target | 90 USD |
| Safety reserve | 10 USD |
| Default strategy | Local-first |
| Cloud usage pattern | Short-lived cloud sessions |
| EKS usage | Create → Validate → Collect evidence → Destroy |
| RDS / ALB / NAT Gateway | ใช้เฉพาะเมื่อมีเหตุผล และต้อง teardown หลังใช้งาน |

---

## Important Cost Principle

AWS Budget และ Cost Alerts เป็น **notification guardrail** เท่านั้น ไม่ใช่ **spending hard stop**

แปลว่า:

- Budget alert ช่วยแจ้งเตือนเมื่อค่าใช้จ่ายถึง threshold
- Budget alert ไม่ได้หยุด resource ให้อัตโนมัติ
- ค่าใช้จ่ายอาจเพิ่มต่อได้หลัง alert ถูกส่ง
- เจ้าของโปรเจกต์ต้องตรวจ resource และ billing เองหลังจบ session ทุกครั้ง
- ถ้าไม่แน่ใจว่า resource ใดคิดเงิน ต้องหยุดและตรวจ pricing ก่อนใช้งาน
- ถ้าไม่มีเวลาพอสำหรับ teardown และ resource audit ห้ามเริ่ม cloud session

---

## Spending Envelope

ตารางนี้เป็นกรอบการใช้เงินโดยประมาณสำหรับการเรียนรู้ ไม่ใช่ใบเสนอราคาที่รับประกัน

| Category | Target / Cap | ตัวอย่าง Resource | Intent |
|---|---:|---|---|
| Project 1 EC2 / Network Lab | 8 USD | EC2, EBS, Public IPv4 | ทดลอง Linux, SSH, systemd, Nginx, firewall |
| EKS Control Plane Sessions | 18 USD | Short-lived EKS clusters | Validate Kubernetes + AWS integration |
| Worker Nodes / EBS / IPv4 | 22 USD | Managed node group, EBS volume, public IPv4 | รัน workload บน EKS ช่วงสั้น |
| ALB / Network Experiments | 10 USD | Application Load Balancer, target groups, data transfer | ทดสอบ public entry และ routing |
| RDS Short-lived Lab | 10 USD | RDS PostgreSQL | ทดลอง managed database แบบ short-lived |
| ECR / S3 / SQS / Logs / Data Transfer | 7 USD | ECR images, S3 objects, SQS messages, CloudWatch logs | Registry, object storage, queue, logs |
| Final Demo Sessions | 10 USD | End-to-end demo environment | เก็บ evidence / demo recording |
| Emergency Reserve | 15 USD | Cleanup delay, repeated test, unexpected resource | Safety margin |
| Total Hard Cap | 100 USD | ทั้งโปรเจกต์ | ห้ามเกิน |

---

## Budget Alert Thresholds

Budget alerts ที่ตั้งไว้สำหรับโปรเจกต์:

| Threshold | Type | Intent | Action |
|---:|---|---|---|
| 5 USD | Actual | Early warning ว่ามีค่าใช้จ่ายเริ่มเกิดขึ้น | ตรวจว่า resource ที่สร้างตั้งใจหรือไม่ |
| 20 USD | Actual | เริ่มเข้าสู่ cloud lab cost จริง | ตรวจ spending เทียบกับ plan |
| 40 USD | Actual | เริ่มมี risk จาก EKS / ALB / RDS | ทบทวนว่า lab ใหญ่จำเป็นหรือไม่ |
| 70 USD | Actual | เข้าเขตอันตราย | หยุดทดลองใหญ่และ review cost |
| 90 USD | Forecasted / Actual | ใกล้ operating target | Freeze cloud work ที่ไม่จำเป็น |
| 100 USD | Hard cap | เกินกรอบโปรเจกต์ | Stop cloud work และ cleanup ทันที |

---

## Expensive Resources Watchlist

Resource ต่อไปนี้ต้องตรวจเป็นพิเศษ เพราะอาจคิดค่าใช้จ่ายตามชั่วโมงหรือสะสมต่อเนื่อง

| Resource | Cost Risk | Rule |
|---|---|---|
| EKS Cluster | มีค่า control plane ต่อชั่วโมง | เปิดเฉพาะ session ที่จำเป็น |
| EC2 Worker Nodes | คิดค่า compute / storage / IPv4 | ใช้ขนาดเล็กและ destroy หลัง lab |
| RDS | คิดค่า instance / storage / backup | ใช้ short-lived เท่านั้นใน lab |
| NAT Gateway | มี hourly cost และ data processing cost | หลีกเลี่ยงใน core lab ถ้าไม่จำเป็น |
| Application Load Balancer | มี hourly และ LCU cost | เปิดเฉพาะ demo / validation |
| EBS Volumes | อาจค้างหลัง instance ถูกลบ | audit หลังจบ session |
| Elastic IP / Public IPv4 | อาจคิดค่าใช้จ่ายถ้าค้าง | ตรวจทุกครั้งหลัง destroy |
| CloudWatch Logs | log volume และ retention อาจสะสม | ตั้ง retention สั้นสำหรับ lab |
| Snapshots / Backups | storage cost สะสม | เก็บเฉพาะที่จำเป็นและกำหนด cleanup |
| Data Transfer | อาจเกิดจาก load test หรือ cross-AZ | ใช้ synthetic load อย่างจำกัด |
| Route 53 / Hosted Zone | อาจมีค่าใช้จ่ายรายเดือน | ใช้เฉพาะเมื่อจำเป็นต่อ demo |
| ACM | Certificate ฟรีในหลายกรณี แต่ผูกกับ domain/resource | ใช้เฉพาะเมื่อมี domain จริงและจำเป็น |

---

## Cost Control Rules

1. ใช้ local environment เป็นหลักก่อนขึ้น AWS
2. ห้ามเปิด EKS, RDS, ALB หรือ NAT Gateway ทิ้งไว้โดยไม่จำเป็น
3. ทุก cloud session ต้องมีเป้าหมายชัดเจนก่อนเริ่ม
4. ทุก cloud session ต้องมี planned start และ planned end
5. หลังจบ session ต้อง run teardown หรือ resource review
6. ห้ามสร้าง resource ด้วย console แล้วลืมบันทึก
7. Resource ทุกตัวในอนาคตควรมี tag เช่น `Project=visionops`
8. Evidence ด้าน cost ต้อง sanitize ก่อน commit
9. ถ้า AWS spend ถึง 70 USD ต้องหยุดทดลองใหญ่และ review
10. ถ้า AWS spend ถึง 90 USD ต้อง freeze cloud work ที่ไม่จำเป็น
11. ห้ามเริ่ม EKS session ถ้ายังไม่มี teardown checklist
12. ห้าม claim ว่าคุม cost ได้ ถ้าไม่มี evidence
13. ห้ามใช้ root user สำหรับงาน cloud session ปกติ
14. ห้ามใช้ resource ที่ไม่รู้ cost model โดยไม่ตรวจ pricing ก่อน
15. ถ้าไม่แน่ใจว่า resource ใดถูกลบครบหรือยัง ให้ถือว่ายังไม่ปิด session

---

## Required Tags for Future AWS Resources

เมื่อต้องสร้าง AWS resources ใน phase ถัดไป ให้ใช้ tag มาตรฐานดังนี้

| Tag | Example | Purpose |
|---|---|---|
| Project | visionops | ระบุว่า resource เป็นของโปรเจกต์นี้ |
| Environment | lab | แยก local/lab/demo |
| Owner | suphanat | ระบุผู้รับผิดชอบ |
| ManagedBy | terraform | ระบุว่า resource ควรถูกจัดการด้วยอะไร |
| CostCenter | portfolio | ใช้แยกค่าใช้จ่ายสำหรับ portfolio |
| ExpiresAt | YYYY-MM-DD | ระบุวันหมดอายุของ resource |
| DataClassification | synthetic | ระบุว่าใช้ข้อมูลสังเคราะห์ |

---

## Stop Conditions

ต้องหยุด cloud work ทันทีเมื่อเกิดเงื่อนไขใดเงื่อนไขหนึ่ง:

- AWS spend ถึงหรือเกิน 70 USD โดยยังไม่จบ Core AWS validation
- Forecasted cost เข้าใกล้หรือเกิน 90 USD
- ไม่แน่ใจว่า resource ใดกำลังคิดเงิน
- EKS / RDS / ALB / NAT Gateway เปิดค้างโดยไม่มี active test
- Terraform destroy ล้มเหลวและยังมี resource ค้าง
- พบ resource ที่ไม่ได้ tag หรือไม่รู้ว่าถูกสร้างจากอะไร
- พบ credential หรือ secret risk
- ไม่สามารถตรวจ Billing / Cost Explorer ได้
- Cloud lab ไม่มีเป้าหมายหรือไม่มีเวลาพอ teardown
- มี budget alert เข้ามาแต่ยังไม่สามารถระบุสาเหตุได้
- มี resource ถูกสร้างนอก region ที่ตั้งใจโดยไม่ทราบสาเหตุ

---

## Cloud Session Cost Evidence Template

ใช้ template นี้ทุกครั้งเมื่อมีการเปิด AWS resource ที่อาจคิดเงิน

### Cost Evidence — Session `<ID>`

| Field | Value |
|---|---|
| Date |  |
| Start time |  |
| End time |  |
| Region |  |
| Purpose |  |
| Related issue |  |
| Resources created |  |
| Estimated cost before session |  |
| Observed cost after session |  |
| Budget threshold affected |  |
| Resources destroyed |  |
| Orphan resources found |  |
| Follow-up action |  |
| Sensitive details recorded | No |

หมายเหตุ: ห้ามบันทึก AWS account ID, billing detail, payment information, credentials, private endpoints, kubeconfig, Terraform state หรือ secret values

---

## Example Cost Closeout

### Cost Evidence — Session `EXAMPLE`

| Field | Value |
|---|---|
| Date | 2026-09-07 |
| Start time | 13:00 |
| End time | 15:00 |
| Region | ap-southeast-1 |
| Purpose | Validate short-lived AWS lab resource behavior |
| Related issue | Example only |
| Resources created | Example resources only |
| Estimated cost before session | Sanitized summary only |
| Observed cost after session | Sanitized summary only |
| Budget threshold affected | None |
| Resources destroyed | Yes |
| Orphan resources found | 0 |
| Follow-up action | None |
| Sensitive details recorded | No |

---

## What This Issue Does Not Do

Issue นี้ยังไม่ทำสิ่งต่อไปนี้:

- ไม่สร้าง AWS resource ใหม่
- ไม่เขียน Terraform
- ไม่สร้าง EKS
- ไม่สร้าง RDS
- ไม่สร้าง ALB
- ไม่สร้าง NAT Gateway
- ไม่สร้าง S3/SQS/ECR
- ไม่ทดสอบ cost จริงด้วย load test
- ไม่เก็บ billing screenshot ที่มีข้อมูลอ่อนไหว

Issue นี้เป็นเอกสาร baseline เพื่อใช้ก่อนทำ Cloud Lab ใน phase ถัดไป

---

## Related Documents

- `docs/project-charter.md`
- `docs/security/threat-model.md`
- `docs/requirements/non-functional-requirements.md`
- `docs/architecture/system-architecture.md`
- `docs/architecture/network-architecture.md`
- `docs/cost/teardown-checklist.md`

---

## Evidence

| Evidence | Status |
|---|---|
| Cost envelope defined | Yes |
| Hard cap documented | Yes |
| Operating target documented | Yes |
| Budget limitation acknowledged | Yes |
| Expensive resources listed | Yes |
| Stop conditions defined | Yes |
| Cost evidence template created | Yes |
| Cloud resources created | No |
| Sensitive details recorded | No |

---

## Change Log

| Date | Change |
|---|---|
| 2026-09-07 | Created initial AWS cost envelope |
