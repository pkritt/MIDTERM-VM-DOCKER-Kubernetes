# 🚀 ชุดคู่มือและไฟล์เตรียมสอบ Midterm
## Selected Topic in Computer Engineering: Virtualization, Container, and Kubernetes

คลังไฟล์ Manifests, Dockerfile, Compose, Netplan, คู่มือสอบปฏิบัติ และสรุปทฤษฎีเนื้อหาบรรยาย (Lecture Notes) ครบวงจร

---

## 📂 โครงสร้างโฟลเดอร์ของโปรเจกต์

```text
MIDTERM-VM-DOCKER-Kubernetes/
├── 📖 ReadmeLecture.md                # 🌟 สรุปทฤษฎีเตรียมสอบ Midterm ฉบับละเอียด (Module 0 - 3)
├── 01-virtualization/                 # ส่วนปฏิบัติ 1: VirtualBox & Linux Setup
│   ├── 50-cloud-init-template.yaml   # Template Static IP (รองรับ Single & Dual Adapters)
│   └── README.md                     # คู่มือ NAT Network, Host-Only & Port Forwarding
├── 02-docker/                         # ส่วนปฏิบัติ 2: Container (Docker & Compose)
│   ├── Dockerfile                    # Image Nginx สำหรับ Stateless Web
│   ├── website-files/
│   │   └── index.html                # ไฟล์ Static Web สำหรับทดสอบ Build Image
│   ├── docker-compose.yml            # Multi-tier Compose (Stateful DB + Stateless Web)
│   ├── .env / .env.example           # Environment variables
│   └── README.md                     # คู่มือ Docker build, compose & lifecycle
├── 03-kubernetes/                     # ส่วนปฏิบัติ 3: Kubernetes (Stateless vs Stateful)
│   ├── 01-config-secret.yaml         # ConfigMap (DB_HOST, DB_NAME) + Secret (db-password)
│   ├── 02-stateful-db.yaml           # MySQL StatefulSet + Headless Service (clusterIP: None)
│   ├── 03-stateless-web.yaml         # Web Deployment (2 Replicas) + NodePort (Port 30080)
│   ├── standard-deployment-mode/     # (ทางเลือก) รูปแบบ Deployment ล้วนแบบดั้งเดิม
│   └── README.md                     # คู่มือ kubectl, troubleshooting & scaling
└── README.md                          # หน้าสารบัญและ Quick Cheatsheet รวมทุกส่วน
```

---

## ⚡ Quick Navigation (ลิงก์ด่วนไปยังแต่ละส่วน)

0. [📖 **สรุปทฤษฎีเตรียมสอบ Midterm (Lecture Summary)**](file:///C:/Users/kritt/IdeaProjects/MIDTERM-VM-DOCKER-Kubernetes/ReadmeLecture.md)
   - **Module 0:** CAP Theorem, Scaling, Storage/RAID, Message Queue (RabbitMQ vs Kafka), Microservices, API Gateway & Proxies
   - **Module 1:** Linux Path, Hardware Virtualization (VT-x/AMD-V, VM Exit/Entry), Hypervisor Type 1 vs 2, VirtualBox Network Modes
   - **Module 2:** Container vs VM, Docker Architecture, Copy-on-Write (CoW)
   - **Module 3:** K8s Control Plane & Worker Node Processes, Pods/ConfigMap/Secret, Deployment vs StatefulSet, ClusterIP/Headless/NodePort/LoadBalancer/Ingress

1. [📘 **ส่วนปฏิบัติ 1: Virtualization (VirtualBox & Linux Setup)**](file:///C:/Users/kritt/IdeaProjects/MIDTERM-VM-DOCKER-Kubernetes/01-virtualization/README.md)
   - **NAT Network:** วง `ExamNet: 192.168.1xx.0/24` (สำหรับ VM คุยกันเองและออกเน็ต)
   - **Host-Only Adapter:** ให้ Host OS เข้าถึง VM ได้โดยตรงโดยไม่ต้องเซ็ต Port Forwarding
   - **Netplan Config:** [`50-cloud-init-template.yaml`](file:///C:/Users/kritt/IdeaProjects/MIDTERM-VM-DOCKER-Kubernetes/01-virtualization/50-cloud-init-template.yaml) (`VM1: .10`, `VM2: .20`)

2. [🐳 **ส่วนปฏิบัติ 2: Container (Docker & Docker Compose)**](file:///C:/Users/kritt/IdeaProjects/MIDTERM-VM-DOCKER-Kubernetes/02-docker/README.md)
   - **Stateless Web:** [`Dockerfile`](file:///C:/Users/kritt/IdeaProjects/MIDTERM-VM-DOCKER-Kubernetes/02-docker/Dockerfile) + [`website-files/index.html`](file:///C:/Users/kritt/IdeaProjects/MIDTERM-VM-DOCKER-Kubernetes/02-docker/website-files/index.html)
   - **Multi-tier Compose:** [`docker-compose.yml`](file:///C:/Users/kritt/IdeaProjects/MIDTERM-VM-DOCKER-Kubernetes/02-docker/docker-compose.yml) (Stateful DB + Named Volume `db_data` + Web)
   - **คำสั่งรัน:** `docker compose up -d` หรือ `docker build -t exam-myweb:1.0 .`

3. [☸️ **ส่วนปฏิบัติ 3: Kubernetes (Stateless vs Stateful)**](file:///C:/Users/kritt/IdeaProjects/MIDTERM-VM-DOCKER-Kubernetes/03-kubernetes/README.md)
   - **Config & Secret:** [`01-config-secret.yaml`](file:///C:/Users/kritt/IdeaProjects/MIDTERM-VM-DOCKER-Kubernetes/03-kubernetes/01-config-secret.yaml) (`app-config` + `app-secret`)
   - **StatefulSet (Database):** [`02-stateful-db.yaml`](file:///C:/Users/kritt/IdeaProjects/MIDTERM-VM-DOCKER-Kubernetes/03-kubernetes/02-stateful-db.yaml) (Headless Service: `mysql-0.mysql-service`)
   - **Deployment (Web):** [`03-stateless-web.yaml`](file:///C:/Users/kritt/IdeaProjects/MIDTERM-VM-DOCKER-Kubernetes/03-kubernetes/03-stateless-web.yaml) (NodePort Service: `port 30080`, 2 Replicas)
   - **คำสั่งรัน:** `kubectl apply -f .`

---

## 📌 รวมตาราง Port & URL สำหรับการทดสอบ (Test Matrix)

| ส่วน | บทบาท / สถาปัตยกรรม | Host / Target | Port | วิธีเข้าใช้งาน / ทดสอบ |
| :--- | :--- | :--- | :--- | :--- |
| **Part 1 (VM)** | Stateless Web | `127.0.0.1` (Forward) / `192.168.56.10` (Host-Only) | `8080 -> 80` | Browser: `http://localhost:8080` |
| **Part 1 (VM)** | Stateful DB | `127.0.0.1` (Forward) / `192.168.56.20` (Host-Only) | `3306 -> 3306` | DBeaver: `localhost:3306` |
| **Part 2 (Docker)** | Stateless Web (WordPress/Nginx) | `localhost` | `8080:80` | Browser: `http://localhost:8080` |
| **Part 2 (Docker)** | Stateful DB (MySQL) | `localhost` | `3306:3306` | DBeaver: `localhost:3306` (`root` / `rootpassword`) |
| **Part 3 (K8s)** | Stateful DB (Headless) | `mysql-0.mysql-service` | `3306` (Headless) | เรียกใช้งานผ่าน Pod ภายในคลัสเตอร์ |
| **Part 3 (K8s)** | Stateless Web (NodePort) | `localhost` หรือ `<Node-IP>` | `30080` (NodePort) | Browser: `http://localhost:30080` |

---

## 💡 สรุปความแตกต่างสำคัญ: Stateless vs. Stateful

```text
┌───────────────────────────────────────────────┐
│                 STATELESS                     │
│  - Web / API / Nginx / WordPress App          │
│  - ไม่มีข้อมูลผูกติดในเครื่อง                 │
│  - Scale เพิ่ม/ลดได้ทันที                     │
│  - K8s: ใช้ Deployment + NodePort/ClusterIP   │
└───────────────────────────────────────────────┘
                        ▲
                        │ (ต่อ Database ผ่าน Network / DNS)
                        ▼
┌───────────────────────────────────────────────┐
│                  STATEFUL                     │
│  - Database (MySQL, PostgreSQL, MongoDB)      │
│  - มีข้อมูลสำคัญ ต้องผูกกับ Persistent Volume │
│  - ต้องมีลำดับการสร้างและคงอัตลักษณ์ไว้       │
│  - K8s: ใช้ StatefulSet + Headless Service    │
└───────────────────────────────────────────────┘
```