# ส่วนที่ 2: Container (Docker & Docker Compose)

คู่มือและโครงสร้างไฟล์สำหรับการรันระบบ Multi-tier ที่แบ่งแยก **Stateless (Web Server)** และ **Stateful (Database)** ชัดเจน

---

## 📁 โครงสร้างไฟล์ในโฟลเดอร์นี้

- [`Dockerfile`](file:///C:/Users/kritt/IdeaProjects/MIDTERM-VM-DOCKER-Kubernetes/02-docker/Dockerfile): ไฟล์สำหรับ Build Image Nginx (Stateless Web) ด้วยตัวเอง
- [`website-files/index.html`](file:///C:/Users/kritt/IdeaProjects/MIDTERM-VM-DOCKER-Kubernetes/02-docker/website-files/index.html): ตัวอย่าง Static Web Files ที่จะ Copy เข้าไปใน Image
- [`docker-compose.yml`](file:///C:/Users/kritt/IdeaProjects/MIDTERM-VM-DOCKER-Kubernetes/02-docker/docker-compose.yml): ไฟล์ Compose รันทั้งระบบ (DB Stateful + Web Stateless)
- [`.env`](file:///C:/Users/kritt/IdeaProjects/MIDTERM-VM-DOCKER-Kubernetes/02-docker/.env): ไฟล์เก็บตัวแปร Environment (ทางเลือก)

---

## 🧠 ทำความเข้าใจ Stateless vs. Stateful ใน Container

| ประเภท | ตัวอย่างในระบบ | ลักษณะสำคัญ | การจัดการข้อมูล |
| :--- | :--- | :--- | :--- |
| **Stateless** | Web / WordPress / Nginx | ไม่มีข้อมูลผูกติดใน Container สั่งลบ/สร้างใหม่/สเกลได้ทันที | ไม่ต้องใช้ Volume (อ่านไฟล์จาก Image หรือต่อ DB) |
| **Stateful** | MySQL Database | มีข้อมูลที่ต้องคงอยู่ (State) หากลบ Container ข้อมูลต้องไม่หาย | ต้องผูกกับ **Named Volume** (`db_data:/var/lib/mysql`) |

---

## 🚀 วิธีการรันระบบ (How-To Run)

### รูปแบบที่ 1: รันผ่าน Docker Compose (แนะนำ)
```bash
cd 02-docker

# สั่งรันในโหมด Background
docker compose up -d

# ตรวจสอบสถานะ Container
docker compose ps
```

### รูปแบบที่ 2: Build และรันเฉพาะ Custom Dockerfile (Stateless Web)
```bash
# 1. สั่ง Build Image จาก Dockerfile
docker build -t exam-myweb:1.0 .

# 2. รัน Container จาก Image ที่ Build
docker run -d -p 8080:80 --name exam_myweb_app exam-myweb:1.0

# 3. ทดสอบเปิด Browser เข้าที่: http://localhost:8080
```

---

## 🌐 ตารางการเข้าใช้งาน

| บริการ | URL / Host | Port | บัญชีผู้ใช้ |
| :--- | :--- | :--- | :--- |
| **Web Application** | `http://localhost:8080` | `8080` | - |
| **MySQL Database** | `localhost` | `3306` | `root` / `rootpassword` (DB: `exam_db`) |

---

## 🛠️ คำสั่งตรวจเช็คและ Debug ระหว่างสอบ

```bash
# ดู Log การทำงานแบบ real-time
docker compose logs -f

# ดู Log เฉพาะ Database
docker compose logs -f db

# เข้าไปทดสอบ Query ใน MySQL Container
docker exec -it exam_db mysql -u root -prootpassword exam_db

# เข้าไปดูไฟล์ใน Container Web
docker exec -it exam_web bash

# หยุดและลบ Container (ข้อมูลใน Volume ยังอยู่)
docker compose down

# ล้างข้อมูลทั้งหมดรวม Volume (Reset State)
docker compose down -v
```
