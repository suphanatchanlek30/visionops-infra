# AWS Teardown Checklist — VisionOps Reliability Platform

## วัตถุประสงค์

เอกสารนี้กำหนด **Teardown Checklist** สำหรับ AWS Cloud Lab ของ VisionOps Reliability Platform

Checklist นี้ใช้ก่อน ระหว่าง และหลังการเปิด AWS resources เพื่อป้องกันค่าใช้จ่ายเกินงบ ลดความเสี่ยงจาก resource ค้าง และสร้างหลักฐานว่าการใช้งาน cloud ถูกควบคุมอย่างเป็นระบบ

เอกสารนี้ต้องใช้ร่วมกับไฟล์:

- `docs/cost/aws-cost-envelope.md`
- `docs/project-charter.md`
- `docs/security/threat-model.md`

> หมายเหตุ: เอกสารนี้เป็น Operational Checklist Baseline เท่านั้น ยังไม่มีการสร้าง AWS Resource ใหม่จาก Issue นี้

---

## Checklist Status

| รายการ | ค่า |
|---|---|
| Version | v0 |
| Phase | M0 — Safety & Design |
| Status | Operational checklist baseline |
| Cloud resources created by this issue | No |
| Sensitive details recorded | No |
| Primary cloud | AWS |
| Primary region | ap-southeast-1 |
| Owner | Suphanat Chanlek |
| Last updated | 2026-09-07 |

---

## Core Rule

ทุกครั้งที่เปิด AWS resources สำหรับ VisionOps ต้องทำตาม flow นี้

```text
Plan
  → Open Cloud Session
  → Validate / Test
  → Collect Evidence
  → Teardown
  → Audit Orphan Resources
  → Record Cost Evidence
```

ถ้าไม่มีเวลาพอทำ **teardown** และ **orphan resource audit** ห้ามเริ่ม cloud session

---

## Before Cloud Session Checklist

ตรวจสิ่งต่อไปนี้ก่อนสร้าง resource ใด ๆ

### Account and Budget

- [ ] ใช้ AWS account ที่ถูกต้อง
- [ ] ใช้ region ที่ถูกต้อง เช่น `ap-southeast-1`
- [ ] AWS Budget ถูกตั้งค่าแล้ว
- [ ] ตรวจ current spend ก่อนเริ่ม
- [ ] ตรวจว่า spend ยังไม่ถึง stop condition
- [ ] ตรวจว่า credit / budget ยังพอ
- [ ] เข้าใจว่า Budget Alert ไม่ใช่ spending hard stop
- [ ] ตั้ง reminder เพื่อปิด resource เมื่อ session จบ

### Session Planning

- [ ] ระบุเป้าหมายของ session
- [ ] ระบุ resource ที่จะสร้าง
- [ ] ระบุ resource ที่อาจมีค่าใช้จ่ายรายชั่วโมง
- [ ] ระบุเวลาที่จะเริ่ม
- [ ] ระบุเวลาที่จะ teardown
- [ ] มีเวลาเหลือพอสำหรับตรวจ resource หลัง destroy
- [ ] ระบุ evidence ที่ต้องเก็บ
- [ ] ระบุว่า session นี้เกี่ยวกับ Issue / Phase ใด
- [ ] ระบุ rollback / cleanup plan หาก resource สร้างไม่สำเร็จ

### Security

- [ ] ไม่ใช้ Root User สำหรับงานประจำ
- [ ] ไม่ใช้ root access key
- [ ] ไม่ใช้ long-lived key ถ้ามีทางเลือกที่ปลอดภัยกว่า
- [ ] ไม่มี credential อยู่ใน command, screenshot หรือ log
- [ ] ไม่มี `.env`, `.pem`, kubeconfig หรือ Terraform state ถูกเตรียม commit
- [ ] ใช้ synthetic data เท่านั้น
- [ ] ไม่เปิด database public
- [ ] ไม่เปิด storage public
- [ ] ไม่เปิด security group กว้างโดยไม่มีเหตุผล

### Terraform / IaC Readiness

- [ ] ตรวจ `terraform plan` ก่อน apply หาก session ใช้ Terraform
- [ ] ตรวจว่า resource ทุกตัวมี tag ที่เหมาะสม
- [ ] ตรวจว่าไม่มี resource แพงที่ไม่ตั้งใจ เช่น NAT Gateway หรือ RDS
- [ ] ตรวจว่า destroy path มีอยู่และเข้าใจ
- [ ] ตรวจว่า backend/state ไม่ถูก commit
- [ ] ตรวจว่า variables ไม่มี secret
- [ ] ตรวจว่า plan ไม่มี output sensitive ที่จะถูกเผยแพร่

---

## During Cloud Session Checklist

ระหว่างใช้งาน cloud session ให้ตรวจสิ่งต่อไปนี้

- [ ] ใช้เฉพาะ resource ที่อยู่ใน scope ของ session
- [ ] ไม่สร้าง resource เพิ่มโดยไม่มีเหตุผล
- [ ] ถ้าจำเป็นต้องสร้าง resource เพิ่ม ต้องบันทึกเหตุผล
- [ ] ไม่เปิด service public โดยไม่จำเป็น
- [ ] ไม่เปิด database public
- [ ] ไม่เปิด security group กว้างโดยไม่มีเหตุผล
- [ ] ไม่ upload secret หรือข้อมูลจริง
- [ ] บันทึก command / evidence แบบ sanitized
- [ ] ตรวจเวลา session เป็นระยะ
- [ ] หยุด session ถ้าพบ cost หรือ security risk
- [ ] ถ้าเกิด error ให้บันทึกอย่างปลอดภัย ไม่ copy secret หรือ account detail ลงเอกสาร
- [ ] ถ้าใช้ console manual ต้องบันทึกว่าเกิด manual action อะไร และทำไม

---

## After Cloud Session Teardown Checklist

หลังใช้งาน ต้องตรวจและ cleanup

### Terraform Destroy

ถ้าใช้ Terraform:

- [ ] run `terraform destroy` สำหรับ environment ที่เกี่ยวข้อง
- [ ] ตรวจว่า destroy สำเร็จ
- [ ] บันทึกผลแบบ sanitized
- [ ] ถ้า destroy fail ต้องเปิด follow-up issue ทันที
- [ ] ห้ามปิด session ถ้ายังมี resource ค้างโดยไม่ทราบสาเหตุ
- [ ] ตรวจว่า state file ไม่ถูก commit
- [ ] ตรวจว่า plan file ไม่ถูก commit

### EKS / Kubernetes

- [ ] EKS cluster ถูกลบแล้ว
- [ ] Managed node groups ถูกลบแล้ว
- [ ] Fargate profiles ถ้ามี ถูกลบแล้ว
- [ ] Kubernetes load balancer resources ถูกลบแล้ว
- [ ] ไม่มี orphan ENI ที่เกี่ยวข้องค้างอยู่
- [ ] ไม่มี EBS volume จาก persistent volume ค้างโดยไม่ตั้งใจ
- [ ] ไม่มี kubeconfig ถูก commit
- [ ] ไม่มี cluster access credential ถูกบันทึกใน evidence

### EC2 / Compute

- [ ] EC2 instances ถูก terminate แล้ว
- [ ] ไม่มี stopped instance ที่ไม่ตั้งใจค้าง
- [ ] ไม่มี unattached EBS volume ค้าง
- [ ] ไม่มี AMI หรือ snapshot ที่ไม่ตั้งใจค้าง
- [ ] ไม่มี Elastic IP หรือ Public IPv4 resource ค้าง
- [ ] ไม่มี key pair หรือ private key ถูก commit
- [ ] ไม่มี security group ที่เปิด inbound กว้างโดยไม่ตั้งใจ

### Networking

- [ ] Load balancer ถูกลบแล้ว
- [ ] Target groups ถูกลบแล้ว
- [ ] NAT Gateway ถูกลบแล้ว ถ้ามี
- [ ] Internet Gateway / Route Table / Security Group ที่สร้างโดย lab ถูก cleanup ตามแผน
- [ ] VPC ที่สร้างสำหรับ lab ถูกลบแล้ว ถ้าเป็น disposable environment
- [ ] ไม่มี VPC endpoint ที่ไม่ได้ตั้งใจค้าง
- [ ] ไม่มี Route 53 record ที่ไม่ได้ตั้งใจค้าง
- [ ] ไม่มี private endpoint หรือ sensitive endpoint ถูกบันทึกในเอกสาร

### Database

- [ ] RDS instance ถูกลบแล้ว ถ้าเป็น short-lived lab
- [ ] ตรวจ final snapshot setting ว่าตั้งใจหรือไม่
- [ ] snapshot ที่ไม่จำเป็นถูกลบตาม policy
- [ ] database password หรือ dump ไม่ถูก commit
- [ ] backup ที่ต้องเก็บถูก sanitize และจัดเก็บในที่เหมาะสม
- [ ] database ไม่เคยเปิด public โดยไม่จำเป็น
- [ ] ไม่มี connection string หรือ credential อยู่ใน log/evidence

### Storage

- [ ] S3 temporary objects ถูกลบหรือมี lifecycle policy
- [ ] S3 bucket ที่ไม่ต้องใช้ถูกลบ
- [ ] S3 public access ยังถูก block
- [ ] ไม่มี object ที่มี secret หรือข้อมูลจริง
- [ ] backup objects ถูกตรวจตาม policy
- [ ] ไม่มี bucket policy ที่เปิด public โดยไม่ตั้งใจ
- [ ] ไม่มี object URI ที่เปิดเผยข้อมูลอ่อนไหวใน evidence

### Messaging

- [ ] SQS queues ที่สร้างเพื่อ lab ถูกลบ ถ้าเป็น disposable
- [ ] DLQ ถูกลบหรือ cleanup ตามแผน
- [ ] ไม่มี message ที่มีข้อมูลจริงหรือ secret
- [ ] redrive policy ถูกตรวจตาม design
- [ ] ไม่มี queue ที่รับ/ส่งข้อมูลนอก scope ของ project

### Container Registry

- [ ] ECR repository lifecycle policy เหมาะสม
- [ ] image เก่าที่ไม่จำเป็นถูกลบตาม policy
- [ ] ไม่มี image ที่มี secret build artifact
- [ ] ไม่มี image tag ที่ทำให้เข้าใจผิด เช่น `latest` สำหรับ release evidence
- [ ] image digest หรือ tag ที่ใช้ใน demo ถูกบันทึกแบบไม่ sensitive

### Logs and Monitoring

- [ ] CloudWatch log groups มี retention สั้นตาม lab policy
- [ ] log groups ที่ไม่จำเป็นถูกลบ
- [ ] ไม่มี log ที่เผย secret
- [ ] dashboard / alarm / metric ที่สร้างทดลองถูก cleanup ตามแผน
- [ ] ไม่มี log export ที่มีข้อมูลอ่อนไหวถูก commit
- [ ] ค่า log retention ถูกตั้งให้เหมาะกับ lab

### IAM and Security

- [ ] ไม่มี temporary IAM user / access key ค้าง
- [ ] IAM roles/policies ที่สร้างเพื่อ lab ยังจำเป็นหรือถูก cleanup แล้ว
- [ ] ไม่มี broad permission ที่ไม่ตั้งใจค้าง
- [ ] ไม่มี secret ถูกเก็บผิดที่
- [ ] Root User ไม่ถูกใช้สำหรับงานประจำ
- [ ] ไม่มี access key ถูกสร้างโดยไม่จำเป็น
- [ ] ไม่มี policy ที่ใช้ `Action:*` หรือ `Resource:*` โดยไม่มีเหตุผล/ADR
- [ ] ตรวจว่า GitHub Actions / OIDC role ไม่ได้ถูกสร้างผิด scope

---

## Orphan Resource Audit Checklist

หลัง teardown ให้ตรวจ resource ต่อไปนี้ใน AWS Console หรือ CLI

- [ ] EKS clusters
- [ ] EKS node groups
- [ ] Fargate profiles
- [ ] EC2 instances
- [ ] EC2 volumes
- [ ] EC2 snapshots
- [ ] Elastic IPs / Public IPv4
- [ ] Load balancers
- [ ] Target groups
- [ ] NAT Gateways
- [ ] VPCs
- [ ] Subnets
- [ ] Route tables
- [ ] Internet Gateways
- [ ] VPC endpoints
- [ ] Security groups
- [ ] RDS instances
- [ ] RDS snapshots
- [ ] S3 buckets
- [ ] SQS queues
- [ ] ECR repositories / images
- [ ] CloudWatch log groups
- [ ] CloudWatch alarms
- [ ] IAM users / access keys
- [ ] IAM roles / policies created for lab
- [ ] Route 53 records if used
- [ ] ACM certificates if used

---

## Stop Conditions

ต้องหยุด session และ cleanup ทันทีเมื่อเกิดเงื่อนไขใด ๆ ต่อไปนี้

- [ ] budget alert ถึง threshold สูง เช่น 70 USD หรือ 90 USD
- [ ] พบ resource ที่ไม่รู้ว่าถูกสร้างจากอะไร
- [ ] พบ resource แพง เช่น NAT Gateway, RDS หรือ ALB ที่ไม่ได้ตั้งใจสร้าง
- [ ] ไม่สามารถ teardown ได้ภายในเวลาที่วางแผน
- [ ] เจอ credential หรือ secret risk
- [ ] มีการเปิด public access โดยไม่ตั้งใจ
- [ ] Terraform state หรือ kubeconfig เสี่ยงถูก commit
- [ ] ไม่สามารถตรวจ Billing หรือ Resource Inventory ได้
- [ ] พบว่า resource ถูกสร้างผิด region
- [ ] พบว่าใช้ Root User ทำงานประจำโดยไม่ตั้งใจ
- [ ] พบว่า evidence มีข้อมูลอ่อนไหว

---

## Cost Evidence Template

หลังจบ session ให้บันทึก evidence แบบ sanitized

### Sanitized Cost Evidence — Session `<ID>`

| Field | Value |
|---|---|
| Session date |  |
| Region |  |
| Purpose |  |
| Related issue |  |
| Cloud resources created |  |
| Cloud resources destroyed |  |
| Orphan resources found |  |
| Orphan resources resolved |  |
| Budget threshold reached |  |
| Manual billing/resource review completed |  |
| Sensitive details recorded | No |

หมายเหตุ: ห้ามบันทึก AWS account ID, credentials, billing details, payment information, private endpoints, kubeconfig, Terraform state หรือ secret values

---

## Example Session Closeout

ตัวอย่างข้อความสำหรับปิด cloud session:

### Cloud Session Closeout

| Check | Result |
|---|---|
| Purpose | Validate short-lived AWS lab resource behavior |
| Region | ap-southeast-1 |
| Teardown completed | Yes |
| EKS clusters remaining | 0 |
| EC2 instances remaining for this project | 0 |
| Load balancers remaining for this project | 0 |
| RDS instances remaining for this project | 0 |
| NAT Gateways remaining for this project | 0 |
| Orphan resources found | 0 |
| Sensitive details recorded | No |

---

## What This Issue Does Not Do

Issue นี้ยังไม่ทำสิ่งต่อไปนี้:

- ไม่สร้าง AWS resources
- ไม่ run Terraform apply
- ไม่สร้าง EKS
- ไม่ deploy application
- ไม่ทดสอบ load จริง
- ไม่เก็บ billing screenshot ที่มีข้อมูลอ่อนไหว
- ไม่สร้าง IAM role หรือ access key ใหม่
- ไม่แก้ Billing setting นอกเหนือจากเอกสาร

---

## Related Documents

- `docs/cost/aws-cost-envelope.md`
- `docs/project-charter.md`
- `docs/security/threat-model.md`
- `docs/architecture/network-architecture.md`
- `docs/requirements/non-functional-requirements.md`

---

## Evidence

| Evidence | Status |
|---|---|
| Teardown checklist created | Yes |
| Before session checklist created | Yes |
| During session checklist created | Yes |
| After session checklist created | Yes |
| Orphan resource audit checklist created | Yes |
| Stop conditions defined | Yes |
| Cost evidence template created | Yes |
| Example session closeout created | Yes |
| Cloud resources created | No |
| Sensitive details recorded | No |

---

## Change Log

| Date | Change |
|---|---|
| 2026-09-07 | Created initial AWS teardown checklist |
