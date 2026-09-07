# VisionOps Tool and Platform Version Matrix

## วัตถุประสงค์

ไฟล์นี้ใช้บันทึกเวอร์ชันของเครื่องมือและสภาพแวดล้อมที่ใช้พัฒนาโปรเจกต์ VisionOps Reliability Platform

การบันทึกเวอร์ชันตั้งแต่ต้นช่วยให้โปรเจกต์สามารถทำซ้ำ ตรวจสอบ และ debug ได้ง่ายขึ้น โดยเฉพาะเมื่อโปรเจกต์เริ่มใช้เครื่องมือหลายกลุ่ม เช่น AWS CLI, Terraform, Docker, Kubernetes, Helm, CI/CD และ Observability tools

---

## Metadata

| รายการ | ค่า |
|---|---|
| Project | VisionOps Reliability Platform |
| Repository | visionops-infra |
| Phase | M0 — Safety & Design |
| Owner | Suphanat Chanlek |
| Checked date | 2026-09-07 |
| Primary cloud | AWS |
| Primary region | ap-southeast-1 |
| Notes | บันทึกเฉพาะข้อมูลทั่วไป ไม่บันทึก credential, account ID, secret, token หรือข้อมูลอ่อนไหว |

---

## Development Environment

| รายการ | ค่า |
|---|---|
| Operating System | Windows 11 |
| Terminal | Git Bash / PowerShell |
| Editor | Visual Studio Code |
| Sensitive details recorded | No |

---

## Core Development Tools

| Tool | Version / Status | Command used | Notes |
|---|---|---|---|
| Git | 2.50.0.windows.1 | `git --version` | ใช้สำหรับ version control |
| VS Code | Installed | `code --version` | ใช้เป็น editor หลัก |
| Git Bash / Bash | Installed | `bash --version` | ใช้กับ shell scripts และ Git Bash |
| PowerShell | 5.1.26100.9278 | `$PSVersionTable.PSVersion` | ตรวจผ่าน PowerShell จาก Git Bash |
| Python | 3.11.4 | `python --version` | ใช้สำหรับ automation / tooling |
| pip | 26.1.2 | `pip --version` | Python package manager; ไม่บันทึก local full path |
| Node.js | 22.15.1 | `node --version` | ใช้กับ frontend / tooling |
| npm | 11.7.0 | `npm --version` | Node package manager |
| Go | go1.25.0 windows/amd64 | `go version` | ใช้กับ API / Worker |
| jq | Not installed yet | `jq --version` | ยังไม่ได้ติดตั้ง; จะใช้สำหรับอ่าน/แปลง JSON ใน scripts |
| OpenSSL | 3.2.4 | `openssl version` | ใช้ทดสอบ TLS / certificate |

---

## Cloud and Infrastructure Tools

| Tool | Version / Status | Command used | Notes |
|---|---|---|---|
| AWS CLI | 2.34.24 | `aws --version` | ใช้จัดการ AWS ผ่าน CLI |
| Terraform | Not installed yet | `terraform version` | จะติดตั้งก่อนเริ่ม Terraform / AWS Foundation |
| Docker | 28.5.1, build e180ab8 | `docker --version` | ใช้ build/run container |
| Docker Compose | v2.40.3-desktop.1 | `docker compose version` | ใช้ local integration environment |
| kubectl | Client v1.34.1 | `kubectl version --client` | ใช้จัดการ Kubernetes |
| Kustomize | v5.7.1 | `kubectl version --client` | bundled with kubectl |
| kind | Not installed yet | `kind version` | จะใช้สร้าง local Kubernetes cluster |
| Helm | Not installed yet | `helm version` | จะใช้ package Kubernetes manifests |
| Argo CD CLI | Not installed yet | `argocd version --client` | จะใช้ใน GitOps phase |
| k6 | Not installed yet | `k6 version` | จะใช้ใน load testing phase |
| Ansible | Not installed yet | `ansible --version` | จะใช้ใน Linux / configuration management lab |

---

## Security and Quality Tools

| Tool | Version / Status | Command used | Notes |
|---|---|---|---|
| Gitleaks | Not installed yet | `gitleaks version` | ใช้ตรวจ secret leak |
| Trivy | Not installed yet | `trivy --version` | ใช้ scan image / filesystem |
| Checkov | Not installed yet | `checkov --version` | ใช้ scan IaC |
| TFLint | Not installed yet | `tflint --version` | ใช้ lint Terraform |
| ShellCheck | Not installed yet | `shellcheck --version` | ใช้ตรวจ Bash script |
| Hadolint | Not installed yet | `hadolint --version` | ใช้ตรวจ Dockerfile |
| kubeconform | Not installed yet | `kubeconform -v` | ใช้ validate Kubernetes manifests |

---

## Version Collection Commands

### Git Bash

```bash
git --version
bash --version
python --version
pip --version
node --version
npm --version
go version
aws --version
terraform version
docker --version
docker compose version
kubectl version --client
kind version
helm version
jq --version
openssl version
```

### PowerShell

```powershell
$PSVersionTable.PSVersion
git --version
python --version
pip --version
node --version
npm --version
go version
aws --version
terraform version
docker --version
docker compose version
kubectl version --client
kind version
helm version
```

---

## Current Missing Tools

| Tool | Required Phase | Priority |
|---|---|---|
| Terraform | M3 — Terraform & AWS Foundation | High |
| kind | M4 — Kubernetes Platform | High |
| Helm | M4 — Kubernetes Platform | High |
| jq | M1/M3 — Scripting / AWS JSON Processing | Medium |
| Gitleaks | M5 — CI/CD & GitOps | Medium |
| Trivy | M5 — CI/CD & GitOps | Medium |
| Checkov | M3/M5 — IaC Security | Medium |
| TFLint | M3/M5 — Terraform Quality | Medium |
| ShellCheck | M1 — Linux & Network | Medium |
| k6 | M7 — Observability & SRE | Medium |
| Argo CD CLI | M5 — CI/CD & GitOps | Medium |

---

## Security Notes

ห้ามบันทึกข้อมูลต่อไปนี้ลงไฟล์นี้

- AWS account ID
- AWS access key
- AWS secret access key
- session token
- root email
- billing information
- private key
- kubeconfig
- Terraform state
- absolute path ที่เปิดเผยข้อมูลส่วนตัวเกินจำเป็น

ไฟล์นี้ควรบันทึกเฉพาะ tool name, version, command และ notes ที่ปลอดภัยต่อการเผยแพร่ใน public repository

---

## Change Log

| Date | Change |
|---|---|
| 2026-09-07 | Created initial tool and platform version matrix |