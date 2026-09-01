# 📖 สรุปทฤษฎีเตรียมสอบ Midterm: Selected Topic in Computer Engineering

> [!TIP]
> **คำแนะนำ:** เพื่อประสบการณ์การอ่านที่ดีที่สุดและเห็นขนาดตัวอักษร สี และตารางที่จัดรูปแบบไว้ กรุณาเปิดไฟล์นี้ด้วยโปรแกรมที่รองรับ **Markdown Preview** (เช่น VS Code กด `Ctrl + Shift + V` หรือ `Ctrl + K, V`, Obsidian) หรือเปิดอ่านบน GitHub

---

## 📑 สารบัญเนื้อหา (Table of Contents)

1. [🖥️ Module 0: System Design Concepts (แนวคิดการออกแบบระบบ)](#️-module-0-system-design-concepts-แนวคิดการออกแบบระบบ)
   - [1. ทฤษฎีบท CAP (CAP Theorem)](#1-ทฤษฎีบท-cap-cap-theorem)
   - [2. กลยุทธ์การขยายขนาดระบบ (Scaling Strategies)](#2-กลยุทธ์การขยายขนาดระบบ-scaling-strategies)
   - [3. ประเภทของพื้นที่จัดเก็บข้อมูล (Storage Types) และ RAID](#3-ประเภทของพื้นที่จัดเก็บข้อมูล-storage-types-และ-raid)
   - [4. การจัดการคิวข้อความ (Message Queues)](#4-การจัดการคิวข้อความ-message-queues)
   - [5. สถาปัตยกรรมไมโครเซอร์วิส (Microservices Architecture)](#5-สถาปัตยกรรมไมโครเซอร์วิส-microservices-architecture)
   - [6. API Gateway และ Proxies](#6-api-gateway-และ-proxies)
2. [🛠️ Module 1: Virtualization (การจำลองเสมือน) และ Linux](#️-module-1-virtualization-การจำลองเสมือน-และ-linux)
   - [1. โครงสร้างระบบไฟล์ลินุกซ์ (Linux File System Hierarchy)](#1-โครงสร้างระบบไฟล์ลินุกซ์-linux-file-system-hierarchy)
   - [2. การจำลองเสมือนระดับฮาร์ดแวร์ (Hardware-assisted Virtualization)](#2-การจำลองเสมือนระดับฮาร์ดแวร์-hardware-assisted-virtualization)
   - [3. สถาปัตยกรรมไฮเปอร์ไวเซอร์ (Hypervisor Architectures)](#3-สถาปัตยกรรมไฮเปอร์ไวเซอร์-hypervisor-architectures)
   - [4. โหมดเครือข่ายของ VirtualBox (VirtualBox Network Modes)](#4-โหมดเครือข่ายของ-virtualbox-virtualbox-network-modes)
3. [🐳 Module 2: Container Technology (เทคโนโลยีคอนเทนเนอร์)](#-module-2-container-technology-เทคโนโลยีคอนเทนเนอร์)
   - [1. Containers vs. Virtual Machines](#1-containers-vs-virtual-machines)
   - [2. สถาปัตยกรรมด็อกเกอร์และอิมเมจ (Docker Architecture)](#2-สถาปัตยกรรมด็อกเกอร์และอิมเมจ-docker-architecture)
4. [☸️ Module 3: Kubernetes](#️-module-3-kubernetes)
   - [1. โปรเซสการควบคุมคลัสเตอร์ (Control Plane / Master Processes)](#1-โปรเซสการควบคุมคลัสเตอร์-control-plane--master-processes)
   - [2. โปรเซสการทำงานของเวิร์กเกอร์โหนด (Worker Node Processes)](#2-โปรเซสการทำงานของเวิร์กเกอร์โหนด-worker-node-processes)
   - [3. Pods, ConfigMaps และ Secrets](#3-pods-configmaps-และ-secrets)
   - [4. Deployments เทียบกับ StatefulSets (Stateless vs. Stateful)](#4-deployments-เทียบกับ-statefulsets-stateless-vs-stateful)
   - [5. บริการภายในคลัสเตอร์ (Internal Services)](#5-บริการภายในคลัสเตอร์-internal-services)
   - [6. บริการภายนอกคลัสเตอร์ (External Services)](#6-บริการภายนอกคลัสเตอร์-external-services)

---

# 🖥️ Module 0: System Design Concepts (แนวคิดการออกแบบระบบ)

---

### 1. ทฤษฎีบท CAP (CAP Theorem)

ทฤษฎีบท CAP ระบุว่า **ระบบฐานข้อมูลแบบกระจายตัว (Distributed Systems)** ไม่สามารถรับประกันคุณสมบัติทั้ง 3 ประการพร้อมกันได้ทั้งหมด โดยระบบสามารถการันตีได้ **สูงสุดเพียง 2 ใน 3 ประการ** เท่านั้น:

```text
               Consistency (ความสม่ำเสมอ)
                     /        \
                    /   เลือก  \
                   /   ได้เพียง \
                  /    2 จาก 3   \
                 /                \
Availability (ความพร้อม) ───────── Partition Tolerance (ทนต่อเน็ตหลุด)
```

* **Consistency (ความสม่ำเสมอ):** ข้อมูลทุกโหนดในระบบจะมีค่าเท่ากันเสมอ ณ เวลาใดเวลาหนึ่ง เมื่อเขียนข้อมูลลงไป โหนดอื่นทั้งหมดต้องเห็นข้อมูลอัปเดตทันที
* **Availability (ความพร้อมใช้งาน):** ทุกคำขอ (Request) ที่ส่งไปยังโหนดที่ยังทำงานอยู่จะต้องได้รับการตอบกลับ (Non-error Response) เสมอ แม้ข้อมูลอาจไม่ใช่เวอร์ชันล่าสุด
* **Partition Tolerance (ความทนทานต่อการถูกแบ่งส่วน):** ระบบยังคงทำงานต่อไปได้ แม้เครือข่ายจะเกิดปัญหาจนทำให้บางโหนดขาดการสื่อสารจากกัน

> [!IMPORTANT]
> **การตัดสินใจเลือกในระบบจริง:**
> - **CP (Consistency + Partition Tolerance):** เลือกรักษาความถูกต้องของข้อมูลเป็นหลัก โดยยอมปฏิเสธคำขอหรือปิดบริการบางส่วนเมื่อเกิดปัญหาเน็ตหลุด (เช่น ฐานข้อมูลการเงิน, Banking)
> - **AP (Availability + Partition Tolerance):** เลือกรองรับคำขอและให้บริการต่อไปเสมอ แม้ข้อมูลที่ตอบกลับอาจยังไม่อัปเดตล่าสุดชั่วคราว (เช่น Social Media Feed, DNS)

---

### 2. กลยุทธ์การขยายขนาดระบบ (Scaling Strategies)

| คุณลักษณะ | Vertical Scaling (Scale Up) | Horizontal Scaling (Scale Out) |
| :--- | :--- | :--- |
| **วิธีการ** | เพิ่ม CPU, RAM, Storage ในเซิร์ฟเวอร์เครื่องเดิม | เพิ่มจำนวนเซิร์ฟเวอร์เข้ามาช่วยกันทำงาน |
| **ข้อดี** | จัดการง่าย ไม่ต้องแก้โค้ดหรือสถาปัตยกรรม | ขยายได้ไม่จำกัด (Unlimited), ลดความเสี่ยง Single Point of Failure |
| **ข้อจำกัด** | มีเพดานจำกัดตามฮาร์ดแวร์, เสี่ยงเป็น Single Point of Failure | ระบบมีความซับซ้อนขึ้น ต้องมี Load Balancer |

---

### 3. ประเภทของพื้นที่จัดเก็บข้อมูล (Storage Types) และ RAID

#### ประเภท Storage:
* **📁 File Storage:** จัดเก็บข้อมูลในรูปแบบลำดับชั้น (Hierarchical Directory/Folders) เข้าถึงผ่าน File Path เช่น NFS, SMB
* **🧱 Block Storage:** แบ่งข้อมูลออกเป็นบล็อกย่อยๆ ไม่มี Metadata มากนัก ให้ความเร็วสูงและมีความหน่วงต่ำมาก (Low Latency) เหมาะสำหรับ **Database** และ **OS Disk**
* **📦 Object Storage:** จัดเก็บข้อมูลเป็น Object คู่กับ Metadata ละเอียด ไม่มีโครงสร้างโฟลเดอร์ เข้าถึงผ่าน HTTP/HTTPS REST API (เช่น AWS S3, MinIO)

#### เทคโนโลยี RAID:
* **RAID 0 (Striping):** แบ่งข้อมูลเขียนกระจายลงดิสก์หลายลูกพร้อมกัน **เน้นความเร็วสูงสุด** แต่**ไม่มีความทนทาน (No Fault Tolerance)** หากดิสก์พัง 1 ลูก ข้อมูลเสียหายทั้งหมด
* **RAID 1 (Mirroring):** คัดลอกข้อมูลชุดเดียวกันลงในดิสก์ 2 ลูกคู่ขนาน **เน้นความปลอดภัยสูง** กู้คืนได้ทันที แต่สูญเสียความจุไป 50%

---

### 4. การจัดการคิวข้อความ (Message Queues)

ช่วยให้ระบบสามารถทำงานร่วมกันแบบไม่ประสานเวลา (**Asynchronous Processing**) และช่วยลดการผูกมัด (Decoupling):

* **ผู้ส่ง (Producer):** ส่งข้อความเข้า Queue แล้วไปทำงานอื่นต่อได้ทันทีโดยไม่ต้องรอผลลัพธ์
* **ผู้รับ (Consumer):** ดึงข้อความจาก Queue ไปประมวลผลเมื่อทรัพยากรพร้อม

```text
[ Producer ] ──( ส่งงาน )──> [ Message Queue ] ──( ดึงงานไปทำ )──> [ Consumer ]
```

| เครื่องมือ | รูปแบบการทำงาน | พฤติกรรมการเก็บข้อความ |
| :--- | :--- | :--- |
| **RabbitMQ** | Message Broker ดั้งเดิม | ลบข้อความทิ้งทันทีเมื่อ Consumer ประมวลผลและส่ง Acknowledge เสร็จ |
| **Apache Kafka** | Distributed Event Streaming | บันทึกเป็น Log ถาวร เก็บข้อความไว้ตามระยะเวลาที่กำหนด แม้จะถูกอ่านไปแล้ว |

---

### 5. สถาปัตยกรรมไมโครเซอร์วิส (Microservices Architecture)

* **การแบ่งขอบเขต:** แบ่งเซอร์วิสย่อยตาม **"โดเมนธุรกิจ" (Business Domain)**
* **ความเป็นอิสระ (Autonomy):** แต่ละเซอร์วิสมีฐานข้อมูลของตัวเอง (Database-per-Service)
* **การผูกพันต่ำ (Loose Coupling):** สื่อสารกันผ่าน Lightweight API (เช่น REST HTTP, gRPC)
* **ข้อดี:** สามารถเลือกขยายขนาด (Scale) เฉพาะจุดที่มีภาระงานสูงได้ และเลือกใช้ภาษา/เทคโนโลยีที่ต่างกันได้ (**Polyglot**)
* **ข้อท้าทาย:** ความซับซ้อนด้านเครือข่าย, การติดตามทราฟฟิก (Distributed Tracing), และการรักษาความสอดคล้องของข้อมูล (Eventual Consistency)

---

### 6. API Gateway และ Proxies

#### 🚪 API Gateway
ทำหน้าที่เป็นประตูหน้าด่านรวมศูนย์ (Single Entry Point) คอยรับ Request ทั้งหมดจาก Client ก่อนส่งต่อให้ Microservices ภายใน:
* **Authentication & Authorization:** ยืนยันตัวตนและสิทธิ์การเข้าถึง
* **Load Balancing:** กระจายโหลดไปยัง Instance ต่างๆ
* **Rate Limiting & Throttling:** จำกัดอัตราการยิง Request เพื่อป้องกันระบบล่มหรือถูกโจมตี DDoS

#### 🔄 Forward Proxy vs. Reverse Proxy
* **Forward Proxy:** อยู่หน้า **Client** ใช้ส่งคำขอออกไปข้างนอกเพื่อซ่อนตัวตน / IP ของ Client หรือใช้ทำ Content Filtering
* **Reverse Proxy:** อยู่หน้า **Server** รับคำขอจากโลกภายนอกแล้วส่งต่อให้ Server ภายใน ช่วยซ่อนโครงสร้างและรักษาความปลอดภัยของ Server

---

# 🛠️ Module 1: Virtualization (การจำลองเสมือน) และ Linux

---

### 1. โครงสร้างระบบไฟล์ลินุกซ์ (Linux File System Hierarchy)

ในระบบ Linux ทุกอย่างจะถูกมองเป็นไฟล์ และเริ่มต้นที่จุดสูงสุดคือ **Root Directory (`/`)**

* **Absolute Path (พาธสัมบูรณ์):** เริ่มต้นระบุจาก Root Directory เสมอ และต้องขึ้นต้นด้วยเครื่องหมาย `/` เสมอ เช่น:
  ```bash
  /etc/netplan/50-cloud-init.yaml
  /var/log/syslog
  ```
* **Relative Path (พาธสัมพัทธ์):** เริ่มต้นนับจากไดเรกทอรีปัจจุบันที่ทำงานอยู่ (Current Working Directory) โดย**ไม่ขึ้นต้น**ด้วย `/` เช่น:
  ```bash
  ./website-files/index.html
  ../02-docker/docker-compose.yml
  ```

---

### 2. การจำลองเสมือนระดับฮาร์ดแวร์ (Hardware-assisted Virtualization)

การทำ Virtualization ด้วยซอฟต์แวร์แบบดั้งเดิมต้องใช้เทคนิค Binary Translation ซึ่งส่งผลให้สูญเสียประสิทธิภาพอย่างมาก ผู้ผลิต CPU จึงได้เพิ่มฟีเจอร์ระดับฮาร์ดแวร์เข้ามาช่วยโดยเฉพาะ (**Intel VT-x** และ **AMD-V**):

* **VMX Root Operation:** โหมดสิทธิ์สำหรับ **Hypervisor** ทำงาน
* **VMX Non-root Operation:** โหมดสำหรับ **Guest OS** ทำงาน
* **VM Exit:** สภาวะที่ฮาร์ดแวร์ตรวจพบคำสั่งที่มีผลกระทบ และสลับการควบคุมจาก Guest OS กลับมายัง Hypervisor (Root Operation)
* **VM Entry:** สภาวะที่ Hypervisor จัดการทรัพยากรเสร็จ และส่งการควบคุมกลับไปรัน Guest OS ต่อ

---

### 3. สถาปัตยกรรมไฮเปอร์ไวเซอร์ (Hypervisor Architectures)

```text
┌─────────────────────────┐       ┌─────────────────────────┐
│     Type 1 (Bare-Metal) │       │     Type 2 (Hosted)     │
├─────────────────────────┤       ├─────────────────────────┤
│    VM 1   │    VM 2     │       │    VM 1   │    VM 2     │
├─────────────────────────┤       ├─────────────────────────┤
│   Hypervisor (Direct)   │       │   Hypervisor (VirtualBox)│
├─────────────────────────┤       ├─────────────────────────┤
│     Hardware (Server)   │       │         Host OS         │
│                         │       ├─────────────────────────┤
│                         │       │     Hardware (PC/Mac)   │
└─────────────────────────┘       └─────────────────────────┘
```

* **Type 1 Hypervisor (Bare-metal):** ติดตั้งลงบนฮาร์ดแวร์ของเซิร์ฟเวอร์โดยตรง ไม่มี Host OS คั่นกลาง ให้ประสิทธิภาพ ความเสถียร และความปลอดภัยสูงสุด (เช่น VMware ESXi, Proxmox, Microsoft Hyper-V)
* **Type 2 Hypervisor (Hosted):** ติดตั้งเป็นโปรแกรมซอฟต์แวร์บนระบบปฏิบัติการของเครื่อง Host อีกชั้นหนึ่ง ใช้งานสะดวก เหมาะสำหรับการพัฒนาและทดสอบ (เช่น Oracle VirtualBox, VMware Workstation)

---

### 4. โหมดเครือข่ายของ VirtualBox (VirtualBox Network Modes)

| โหมดเครือข่าย | VM ออกเน็ตได้? | VM คุยกันเองได้? | Host คุยกับ VM ได้? | โลกภายนอกคุยกับ VM ได้? |
| :--- | :---: | :---: | :---: | :---: |
| **NAT** | ✅ | ❌ | ❌ (ต้อง Port Forward) | ❌ |
| **NAT Network** | ✅ | ✅ | ❌ (ต้อง Port Forward) | ❌ |
| **Bridged Adapter** | ✅ | ✅ | ✅ (เสมือนเครื่องจริงในวง) | ✅ (ได้ IP จาก Router จริง) |
| **Internal Network** | ❌ | ✅ | ❌ | ❌ (วงปิดสนิท) |
| **Host-Only Adapter**| ❌ | ✅ | ✅ (ต่อตรงผ่านวงจำลอง) | ❌ |

---

# 🐳 Module 2: Container Technology (เทคโนโลยีคอนเทนเนอร์)

---

### 1. Containers vs. Virtual Machines

```text
┌───────────────────────────────┐       ┌───────────────────────────────┐
│       Virtual Machines        │       │          Containers           │
├───────────────────────────────┤       ├───────────────────────────────┤
│  App A   │  App B   │  App C  │       │  App A   │  App B   │  App C  │
│  Bins    │  Bins    │  Bins   │       │  Bins    │  Bins    │  Bins   │
│ Guest OS │ Guest OS │ Guest OS│       ├───────────────────────────────┤
├───────────────────────────────┤       │        Container Engine       │
│          Hypervisor           │       ├───────────────────────────────┤
├───────────────────────────────┤       │     Host OS Kernel (แชร์ร่วม)  │
│            Hardware           │       ├───────────────────────────────┤
│                               │       │            Hardware           │
└───────────────────────────────┘       └───────────────────────────────┘
```

* **Virtual Machines:** ทำงานผ่าน Hypervisor แต่ละ VM จำลองระบบเสมือนทั้งตัวและมี **Guest OS ของตัวเอง** ทำให้มีความปลอดภัยและการแยกส่วนสูงมาก แต่สิ้นเปลือง RAM/CPU และบูตช้า
* **Containers:** ทำงานผ่าน Container Engine (เช่น Docker) บรรจุเฉพาะแอปพลิเคชันและ Library ที่จำเป็น โดย**แชร์ Kernel ของ Host OS ร่วมกัน** ทำให้มีขนาดเล็ก บูตได้ในเสี้ยววินาที และใช้ทรัพยากรอย่างคุ้มค่า

---

### 2. สถาปัตยกรรมด็อกเกอร์และอิมเมจ (Docker Architecture)

* **Docker Client & Daemon:** ใช้รูปแบบ Client-Server โดย CLI ส่งคำสั่ง REST API ไปยัง Docker Host (dockerd) เพื่อจัดการ Container
* **Docker Image:** เป็นแม่แบบแบบ **Read-only** ที่สร้างขึ้นจาก Layer ซ้อนทับกันหลายชั้นตามคำสั่งใน `Dockerfile`
* **Copy-on-Write (CoW):** เมื่อรัน Container ระบบจะเพิ่ม **Writable Container Layer** ไว้บนสุดเพียงชั้นเดียว หากมีการแก้ไขไฟล์ ระบบจะคัดลอกไฟล์จาก Image Layer ด้านล่างขึ้นมาแก้ไขที่ Layer บนสุด ทำให้ Base Image ไม่ถูกแก้ไขและสามารถแชร์ใช้งานร่วมกันได้

---

# ☸️ Module 3: Kubernetes

---

### 1. โปรเซสการควบคุมคลัสเตอร์ (Control Plane / Master Processes)

* **`kube-apiserver`:** ประตูหลักและศูนย์กลางการสื่อสารของคลัสเตอร์ ทำหน้าที่ตรวจสอบความถูกต้องและรับคำสั่ง REST API จากผู้ใช้งานหรือคอมโพเนนต์อื่น
* **`etcd`:** ฐานข้อมูลแบบ Distributed Key-Value ที่มีความสอดคล้องสูง ใช้บันทึกสถานะทั้งหมดของคลัสเตอร์ (ถือเป็น Single Source of Truth)
* **`kube-scheduler`:** คอยตรวจสอบ Pod ที่เพิ่งสร้างใหม่และยังไม่มี Node ประจำ แล้วเลือก Node ที่มีทรัพยากรเหมาะสมที่สุดให้ Pod นั้นไปทำงาน
* **`kube-controller-manager`:** รันลูปการควบคุม คอยตรวจจับสถานะปัจจุบันของคลัสเตอร์ และปรับปรุงให้ตรงกับสถานะที่ผู้ใช้ต้องการ (Desired State) เสมอ

---

### 2. โปรเซสการทำงานของเวิร์กเกอร์โหนด (Worker Node Processes)

* **`Container Runtime`:** ซอฟต์แวร์ที่ใช้รันคอนเทนเนอร์จริงบนโหนด (เช่น containerd, CRI-O)
* **`kubelet`:** Agent หลักประจำแต่ละโหนด คอยรับคำสั่ง PodSpec จาก Control Plane เพื่อสั่งให้ Container Runtime รันคอนเทนเนอร์ และรายงานสถานะกลับ
* **`kube-proxy`:** ดูแลการจัดการ Network Rules และ Routing บนแต่ละโหนด เพื่อให้ Service สามารถส่งทราฟฟิกไปยัง Pod ปลายทางได้อย่างถูกต้อง

---

### 3. Pods, ConfigMaps และ Secrets

* **Pod:** หน่วยที่เล็กที่สุดในการ Deploy บน Kubernetes ทำหน้าที่ห่อหุ้มคอนเทนเนอร์ตั้งแต่ 1 ตัวขึ้นไป โดยมีวงจรชีวิตแบบชั่วคราว (Ephemeral)
* **ConfigMap:** จัดเก็บข้อมูลการตั้งค่าระบบทั่วไปในรูปแบบ Key-Value เพื่อแยก Configuration ออกจาก Container Image
* **Secret:** จัดเก็บข้อมูลที่มีความอ่อนไหว เช่น Password, Token, SSH Key โดยจะถูกเข้ารหัสในรูปแบบ **Base64**

---

### 4. Deployments เทียบกับ StatefulSets (Stateless vs. Stateful)

| คุณสมบัติ | Deployment (Stateless) | StatefulSet (Stateful) |
| :--- | :--- | :--- |
| **ประเภทงาน** | Web App, API, Frontend, Nginx | Database (MySQL, PostgreSQL, MongoDB, Redis) |
| **สถานะข้อมูล** | ไม่บันทึกสถานะลงโหนด สั่งลบ/สร้างใหม่ได้ตลอด | มีข้อมูลสำคัญที่ต้องผูกติดกับ Storage |
| **ชื่อและอัตลักษณ์ของ Pod** | สุ่มแฮช (เช่น `web-67df9-a1b2c`) | เรียงลำดับคงที่และคาดเดาได้ (เช่น `mysql-0`, `mysql-1`) |
| **การเชื่อมต่อเครือข่าย** | ใช้ Service ทั่วไปกระจายโหลด | ใช้ร่วมกับ **Headless Service** เพื่อชี้ตรงราย Pod |
| **ฟีเจอร์เด่น** | Rolling Update, Rollback, Scale ได้รวดเร็ว | สร้าง/ลบ Pod ตามลำดับก่อน-หลังอย่างเข้มงวด |

---

### 5. บริการภายในคลัสเตอร์ (Internal Services)

* **`ClusterIP` (Default):** สร้าง Virtual IP ภายในระบบสำหรับเป็น Load Balancer กระจายโหลดไปยัง Pods ต่างๆ ภายในคลัสเตอร์เท่านั้น
* **`Headless Service` (`clusterIP: None`):** ไม่สร้าง Virtual IP กลาง แต่ระบบ DNS ของ Kubernetes จะคืนค่ารายการ IP ของ Pods ที่ตรงกับ Selector ทั้งหมดโดยตรง ทำให้ Client สามารถระบุโหนดได้เจาะจง เช่น `mysql-0.mysql-service`

---

### 6. บริการภายนอกคลัสเตอร์ (External Services)

```text
               [ ผู้ใช้งานภายนอก (Internet) ]
                              │
               ┌──────────────┼──────────────┐
               │              │              │
               ▼              ▼              ▼
         [ NodePort ]  [ LoadBalancer ]  [ Ingress ]
         (Port 30000-   (ได้ Public IP   (กำหนด Routing
            32767)       จาก Cloud)       ด้วย Path/Domain)
               │              │              │
               └──────────────┼──────────────┘
                              ▼
                      [ K8s Service ]
                              │
                      [ Backend Pods ]
```

* **`NodePort`:** เปิดพอร์ตเฉพาะ (ช่วง `30000-32767`) บนทุก Worker Node เพื่อให้เครื่องภายนอกเข้าถึงผ่าน `http://<Node-IP>:<NodePort>`
* **`LoadBalancer`:** สั่งขอสร้าง Load Balancer ภายนอกโดยอัตโนมัติจาก Cloud Provider (เช่น AWS NLB, GCP Load Balancer) เพื่อให้ได้ Public IP คงที่
* **`Ingress`:** ออบเจกต์ที่ทำหน้าที่เป็น HTTP/HTTPS Router อัจฉริยะ (Layer 7 Routing) ช่วยรวมจุดเข้าใช้งาน และกระจายทราฟฟิกไปยัง Services ต่างๆ ตาม Hostname หรือ URL Path (เช่น `/api` -> api-service, `/web` -> web-service)