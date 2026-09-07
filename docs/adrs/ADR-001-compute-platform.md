# ADR-001: เลือก Amazon EKS เป็น Compute / Orchestration Platform หลักสำหรับ VisionOps

## สถานะ

Accepted

## วันที่

2026-09-07

## เจ้าของการตัดสินใจ

Suphanat Chanlek

---

## บริบท

VisionOps Reliability Platform เป็นโปรเจกต์ Portfolio สำหรับเรียนรู้และแสดงความสามารถด้าน Cloud, DevOps, Platform Engineering และ Site Reliability Engineering หรือ SRE

เป้าหมายของโปรเจกต์นี้ไม่ได้หยุดแค่การ deploy application ให้รันได้ แต่ต้องการพิสูจน์ว่าเจ้าของโปรเจกต์สามารถทำสิ่งต่อไปนี้ได้จริง:

- ออกแบบระบบแบบ production-oriented
- สร้าง infrastructure แบบทำซ้ำได้
- deploy workload บน container platform
- เข้าใจ Kubernetes primitives
- ใช้ CI/CD และ GitOps
- วาง health checks, readiness checks และ resource controls
- ออกแบบ observability ด้วย metrics, logs และ traces
- ทดสอบ failure, rollback และ recovery
- บริหาร cloud cost ภายใต้งบจำกัด
- แยก Lab Implementation ออกจาก Production Reference Architecture

Workload กลางของระบบคือ **Synthetic Vision Event Processing Platform** ที่ประกอบด้วย API, Worker, Queue, Database, Object Storage, Dashboard และ Observability stack

ตัวเลือกหลักบน AWS สำหรับการรัน container workload ได้แก่:

- Amazon EKS
- Amazon ECS
- AWS Fargate
- EC2 แบบ manual
- Local Kubernetes เท่านั้น

เอกสารนี้บันทึกการตัดสินใจว่า VisionOps จะใช้ **Amazon EKS** เป็น cloud orchestration platform หลักสำหรับ Cloud Lab แต่จะใช้แบบ **short-lived lab environment** ไม่เปิดทิ้ง 24/7

---

## Decision Drivers

การตัดสินใจนี้พิจารณาจากปัจจัยต่อไปนี้:

| Driver | คำถามที่ต้องตอบ |
|---|---|
| Learning Value | เครื่องมือนี้ช่วยให้เรียน Cloud / DevOps / SRE ได้ลึกแค่ไหน |
| Target-role Signal | เหมาะกับ Portfolio สำหรับ Cloud, DevOps, Platform หรือ SRE internship หรือไม่ |
| Kubernetes Depth | ได้ฝึก Kubernetes primitives จริงหรือไม่ |
| GitOps Compatibility | รองรับ Argo CD, Helm และ desired state model ได้ดีหรือไม่ |
| Observability | รองรับ Prometheus, Grafana, OpenTelemetry และ SRE GameDay ได้ดีหรือไม่ |
| Portability | ความรู้ที่ได้ย้ายไปใช้ cloud หรือ Kubernetes environment อื่นได้หรือไม่ |
| Cost | อยู่ในงบ AWS credit ประมาณ 100 USD ได้หรือไม่ |
| Solo Feasibility | คนเดียวสามารถทำให้เสร็จและอธิบายได้จริงหรือไม่ |
| Operational Complexity | ความซับซ้อนช่วยให้เรียนรู้ หรือมากเกินจนทำให้โปรเจกต์ไม่เสร็จ |

---

## ตัวเลือกที่พิจารณา

### Option A — Amazon EKS

Amazon EKS เป็น managed Kubernetes service สำหรับรัน Kubernetes cluster บน AWS

ข้อดี:

- ได้เรียน Kubernetes ecosystem จริง
- ใช้กับ Helm, Argo CD, Gateway API, HPA, PDB, RBAC, NetworkPolicy และ Observability stack ได้ตรงเป้าหมาย
- เหมาะกับการแสดงทักษะ Cloud-Native, Platform Engineering และ SRE
- ความรู้ที่ได้ย้ายไปใช้ Kubernetes environment อื่นได้ง่ายกว่า ECS-specific workflow
- เหมาะกับ GameDay เช่น pod failure, broken release, node drain, autoscaling และ drift detection
- เชื่อมกับ AWS services เช่น ECR, S3, SQS, IAM, CloudWatch และ ALB ได้

ข้อเสีย:

- ซับซ้อนกว่า ECS/Fargate
- มีค่าใช้จ่าย control plane ของ EKS และค่าใช้จ่าย worker nodes / load balancer / storage แยกต่างหาก
- ต้องเข้าใจ Kubernetes หลายส่วน เช่น workloads, networking, storage, IAM integration และ cluster add-ons
- ถ้าเปิด cluster ทิ้งไว้จะใช้งบเร็ว
- อาจ overkill หากเป้าหมายจริงเป็นเพียงการ deploy service ขนาดเล็ก

เหมาะกับโปรเจกต์นี้เพราะเป้าหมายคือการเรียนรู้ Kubernetes, GitOps, SRE และ Cloud-Native Operations อย่างลึก

---

### Option B — Amazon ECS

Amazon ECS เป็น AWS container orchestration service ที่จัดการ container workload โดยไม่ต้องใช้ Kubernetes API โดยตรง

ข้อดี:

- ง่ายกว่า EKS ในหลายกรณี
- AWS-native มาก
- เหมาะกับการ deploy service บน AWS ด้วย operational overhead ต่ำกว่า
- ไม่ต้องดูแล Kubernetes control plane, add-ons หรือ Kubernetes API objects
- ใช้กับ Fargate ได้ดี
- เหมาะกับทีมที่ต้องการ run container บน AWS โดยไม่ต้องการ Kubernetes complexity

ข้อเสียสำหรับโปรเจกต์นี้:

- ไม่ได้ฝึก Kubernetes primitives โดยตรง เช่น Deployment, Service, Gateway, HPA, PDB, RBAC, Helm และ Argo CD
- GitOps แบบ Kubernetes-native จะไม่เด่นเท่ากับ EKS
- SRE GameDay ที่เกี่ยวกับ Kubernetes เช่น pod readiness, node drain, rollout, resource limit และ manifest drift จะทำได้น้อยกว่า
- Portfolio จะสื่อสารทักษะ Platform/Kubernetes/SRE ได้ไม่ชัดเท่า EKS

เหมาะกว่า EKS ในบาง production workload ที่ต้องการความเรียบง่าย ลด overhead และต้องการ AWS-native container service โดยไม่จำเป็นต้องใช้ Kubernetes

---

### Option C — AWS Fargate

AWS Fargate เป็น serverless compute engine สำหรับ containers ใช้ได้กับทั้ง ECS และ EKS โดยคิดค่าบริการตาม vCPU, memory และ storage ที่ใช้งานจริงจากช่วงเวลาที่ task หรือ pod เริ่มจนหยุดทำงาน :contentReference[oaicite:1]{index=1}

ข้อดี:

- ไม่ต้องดูแล server หรือ worker nodes โดยตรง
- ลดภาระ node patching, capacity planning บางส่วน และ instance management
- เหมาะกับ workload ที่ต้องการจ่ายตาม resource ที่ใช้
- ใช้กับ ECS ได้ง่ายและเป็นทางเลือกที่ดีสำหรับ service ขนาดเล็กหรือ event-driven workloads

ข้อเสียสำหรับโปรเจกต์นี้:

- ลดโอกาสเรียนรู้ node-level Kubernetes operations
- บาง GameDay เช่น node drain, node capacity, DaemonSet behavior หรือ node-level troubleshooting จะไม่ชัด
- Observability และ platform add-ons บางรูปแบบอาจมีข้อจำกัดเมื่อเทียบกับ managed node group
- ถ้าใช้ Fargate เป็น baseline อาจทำให้ Portfolio ด้าน Kubernetes operations ลึกน้อยลง

Fargate เหมาะกับ production workload หลายประเภท แต่สำหรับ VisionOps รอบนี้ยังไม่ใช่ baseline เพราะโปรเจกต์ต้องการเรียน node, cluster และ Kubernetes operations โดยตรง

---

### Option D — EC2 แบบ Manual

ใช้ EC2 ติดตั้ง application, Docker, reverse proxy และ monitoring เอง

ข้อดี:

- เหมาะกับการเรียน Linux, process, systemd, networking, firewall และ troubleshooting
- ค่าใช้จ่ายควบคุมง่ายถ้าเปิดเฉพาะช่วงสั้น
- ช่วยให้เข้าใจพื้นฐานก่อน Kubernetes

ข้อเสียสำหรับโปรเจกต์นี้:

- ไม่ได้ฝึก Kubernetes, Helm, Argo CD หรือ GitOps อย่างเต็มรูปแบบ
- ไม่เหมาะเป็น final platform สำหรับ Cloud-Native / SRE Portfolio
- ต้องดูแล scaling, deployment, rollback และ orchestration เองมากเกินไป
- ไม่สะท้อน target architecture ที่ต้องการฝึกใน Phase M4-M8

EC2 จะถูกใช้ใน Project 1 หรือ Linux/Network Foundation Lab แต่ไม่ใช้เป็น cloud orchestration platform หลักของ Portfolio

---

### Option E — Local Kubernetes Only

ใช้ kind หรือ local Kubernetes โดยไม่ใช้ AWS EKS

ข้อดี:

- ฟรีหรือเกือบฟรี
- เหมาะสำหรับเรียน Kubernetes, Helm, Argo CD และ Observability
- สร้างและลบ cluster ได้เร็ว
- ใช้ทดลอง failure ได้ปลอดภัยกว่า cloud

ข้อเสีย:

- ไม่พิสูจน์ cloud integration กับ AWS services จริง เช่น IAM, ECR, S3, SQS, ALB, CloudWatch และ AWS Budgets
- ไม่ได้เรียน cloud networking, workload identity และ managed Kubernetes integration จริง
- Portfolio สำหรับ Cloud/DevOps/SRE จะขาด evidence ฝั่ง cloud

Local Kubernetes จะเป็น environment หลักสำหรับการเรียนส่วนใหญ่ แต่ต้องมี short-lived AWS EKS validation เพื่อพิสูจน์ cloud integration

---

## Decision

เลือกใช้ **Amazon EKS with Managed Node Groups** เป็น cloud orchestration platform หลักสำหรับ VisionOps Cloud Lab

Decision นี้มีขอบเขตดังนี้:

- ใช้ EKS เพื่อเรียน Kubernetes, Cloud-Native Delivery, GitOps, Observability และ SRE
- ใช้ Managed Node Group เพื่อเรียน node, pod scheduling, resource usage, node drain และ cluster operations
- ใช้ local Kubernetes เช่น kind เป็น environment หลักสำหรับเรียนและ iterate ส่วนใหญ่
- ใช้ EKS เป็น short-lived AWS Lab เฉพาะช่วง validate integration หรือบันทึก demo
- ไม่เปิด EKS ทิ้งไว้ 24/7
- ไม่เรียกระบบนี้ว่า production-grade
- แยก Lab Implementation ออกจาก Production Reference Architecture
- บันทึกค่าใช้จ่ายและต้อง teardown หลังใช้งาน cloud session

---

## เหตุผลหลักของการเลือก EKS

### 1. ตรงกับเป้าหมายการเรียนรู้ของโปรเจกต์

VisionOps ต้องการพิสูจน์ทักษะ DevOps/SRE ที่ลึกกว่าแค่ deploy container ให้รันได้

EKS ทำให้ได้เรียน:

- Kubernetes Deployment
- Service
- Gateway / HTTPRoute
- ConfigMap
- Secret
- ServiceAccount
- RBAC
- Resource requests / limits
- Liveness probe
- Readiness probe
- Horizontal Pod Autoscaler
- PodDisruptionBudget
- Helm
- Argo CD
- GitOps drift detection
- Observability บน Kubernetes
- Failure และ recovery scenarios

สิ่งเหล่านี้เป็นหัวใจของ Cloud-Native และ SRE Portfolio

---

### 2. ให้ target-role signal สูงกว่า ECS สำหรับเป้าหมายนี้

ถ้าเป้าหมายหลักคือ Software Engineer ที่แค่ต้อง deploy app บน AWS, ECS/Fargate อาจเป็นทางเลือกที่ง่ายและ practical มาก

แต่สำหรับเป้าหมายของ VisionOps คือ:

- Cloud Engineer
- DevOps Engineer
- Platform Engineer
- SRE Intern

EKS ให้ signal ที่ชัดกว่า เพราะแสดงความเข้าใจ Kubernetes และ ecosystem รอบตัว เช่น Helm, Argo CD, Prometheus, Grafana และ workload identity

---

### 3. รองรับ GitOps และ SRE GameDay ได้ชัดกว่า

EKS ทำให้สามารถสร้าง evidence ที่ recruiter เห็นได้ชัด เช่น

- Argo CD แสดง desired state เทียบ live state
- ทดลอง manual drift แล้ว controller ตรวจพบ
- deploy broken version แล้ว readiness ไม่ให้ traffic เข้า
- delete pod แล้ว Kubernetes reconcile กลับ
- node drain แล้วดู PDB behavior
- ตั้ง HPA แล้วดู scaling
- ดู metrics/logs/traces จาก workload บน Kubernetes

สิ่งเหล่านี้เป็นหลักฐานเชิงปฏิบัติที่ ECS/Fargate อธิบายได้ยากกว่าในรูปแบบ Kubernetes-native

---

### 4. เชื่อมกับ AWS services ได้ครบ

EKS ยังสามารถเชื่อมกับ AWS services ที่ VisionOps ต้องใช้ เช่น

- Amazon ECR สำหรับ container registry
- Amazon S3 สำหรับ object storage / backup / Terraform state
- Amazon SQS สำหรับ asynchronous queue และ DLQ
- AWS IAM สำหรับ workload identity
- AWS Load Balancer Controller / ALB สำหรับ public entry
- CloudWatch สำหรับ cloud-level metrics/logs
- AWS Budgets สำหรับ cost guardrails

ดังนั้น EKS ตอบทั้ง Kubernetes learning และ AWS integration

---

## Consequences

### ผลดี

- ได้ Portfolio ที่สะท้อน Kubernetes / DevOps / Platform / SRE ชัดเจน
- มีพื้นที่สำหรับ GitOps, Helm, Observability และ GameDay จริง
- สามารถแยก local Kubernetes และ AWS EKS validation ได้เป็นระบบ
- สร้าง evidence สำหรับ resume ได้มากกว่าเพียง “deployed app to cloud”
- ทำให้ Architecture และ Operations discussion มีความลึก

### ผลเสีย

- ใช้เวลามากกว่า ECS/Fargate
- มีความเสี่ยงเรื่องค่าใช้จ่ายสูงกว่า local-only หรือ ECS บางรูปแบบ
- ต้องเรียนหลายส่วนพร้อมกัน เช่น VPC, IAM, EKS, Kubernetes และ add-ons
- คนเดียวทำอาจ scope ใหญ่เกินไปหากไม่คุม
- ต้องมี teardown discipline ทุกครั้ง

### Risk Mitigation

| Risk | Mitigation |
|---|---|
| EKS cost สูง | ใช้ short-lived EKS sessions, AWS Budget, teardown checklist |
| Complexity สูง | เริ่มจาก local kind ก่อน แล้วค่อยขึ้น EKS |
| Scope ใหญ่เกินไป | แยก Core และ Optional scope ชัดเจน |
| เปิด resource ค้าง | ใช้ tags, checklist และ resource audit |
| ยังไม่พร้อม Kubernetes | ทำ Project 1 Linux/Network และ local Kubernetes ก่อน |
| Claim เกินจริง | ใช้คำว่า production-oriented ไม่ใช้ production-grade |

---

## Cost Considerations

Amazon EKS มีค่า cluster control plane ตามชั่วโมง โดยราคาปัจจุบันของ EKS standard Kubernetes version support คือ 0.10 USD ต่อ cluster ต่อชั่วโมง และยังต้องจ่ายค่า resources อื่นที่สร้างเพื่อรัน worker nodes หรือส่วนประกอบอื่น ๆ แยกต่างหาก :contentReference[oaicite:2]{index=2}

ดังนั้น VisionOps จะใช้ EKS แบบ short-lived เท่านั้น:

```text
Create EKS
  → Deploy
  → Validate
  → Collect evidence
  → Destroy
  → Audit remaining resources
```

ข้อกำหนด cost discipline:

- ต้องมี AWS Budget ก่อนเริ่ม cloud lab
- ไม่เปิด EKS ข้ามคืนโดยไม่จำเป็น
- หลีกเลี่ยง NAT Gateway หรือ RDS หากยังไม่จำเป็น
- ตรวจ ALB, EC2, EBS, Public IPv4, NAT Gateway และ RDS หลังจบ session
- บันทึก cost evidence แบบ sanitized
- ห้ามเริ่ม EKS ก่อนมี teardown checklist

---

## Decision Summary

| Topic | Decision |
|---|---|
| Primary cloud orchestration platform | Amazon EKS |
| Compute mode | Managed Node Group |
| Local learning platform | kind หรือ local Kubernetes |
| Alternative considered | ECS, Fargate, EC2, Local-only |
| Reason | ต้องการเรียน Kubernetes, GitOps, Observability และ SRE GameDay อย่างลึก |
| Cost strategy | Short-lived cloud lab + AWS Budget + teardown checklist |
| Production claim | Production-oriented เท่านั้น ไม่ใช่ production-grade |
| Revisit date | หลังจบ M6 หรือเมื่อ cost/complexity เกินแผน |

---

## Rejected Alternatives Summary

| Alternative | เหตุผลที่ไม่เลือกเป็น baseline |
|---|---|
| ECS | ง่ายและ practical แต่ไม่ฝึก Kubernetes-native operations เท่าที่โปรเจกต์ต้องการ |
| ECS + Fargate | เหมาะกับ service ง่ายและลด node operations แต่ไม่ตอบเป้าหมายการเรียน node/Kubernetes operations |
| EKS + Fargate | ใช้ Kubernetes API ได้ แต่ลด node-level learning และบาง observability/add-on scenario |
| EC2 manual | ดีสำหรับ Linux foundation แต่ไม่ใช่ final cloud-native orchestration target |
| Local Kubernetes only | ประหยัดที่สุด แต่ไม่พิสูจน์ AWS integration จริง |

---

## Revisit Triggers

ต้องกลับมาทบทวน decision นี้หากเกิดเงื่อนไขใดเงื่อนไขหนึ่ง:

- ค่าใช้จ่าย EKS เกิน cost envelope ของโปรเจกต์
- ตั้งค่า EKS ใช้เวลามากจน M0-M6 ล่าช้าเกินแผน
- local Kubernetes สามารถพิสูจน์ learning objective ได้เพียงพอโดยไม่ต้องใช้ EKS
- AWS credit ไม่พอหรือใกล้หมด
- เป้าหมาย portfolio เปลี่ยนจาก DevOps/SRE/Kubernetes ไปเป็น AWS application deployment ที่เน้น simplicity
- ECS/Fargate กลายเป็น requirement ของตำแหน่งงานที่สมัครโดยตรง
- EKS version/support/pricing เปลี่ยนจนกระทบแผน
- ความซับซ้อนของ cluster ทำให้ project ไม่เสร็จตาม timebox

---

## Related Documents

- `docs/project-charter.md`
- `docs/requirements/functional-requirements.md`
- `docs/requirements/non-functional-requirements.md`
- `docs/architecture/system-architecture.md`
- `docs/architecture/network-architecture.md`

---

## Evidence

เอกสารนี้เป็น design decision baseline เท่านั้น ยังไม่มีการสร้าง EKS, ECS, Fargate หรือ EC2 resource จริงจาก ADR นี้

| Evidence | Status |
|---|---|
| Decision recorded | Yes |
| Cloud resources created | No |
| Sensitive details recorded | No |
| Cost risk acknowledged | Yes |
| Revisit triggers defined | Yes |

---

## Change Log

| Date | Change |
|---|---|
| 2026-09-07 | Created ADR-001 for compute platform decision |