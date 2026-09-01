# 📖 คำอธิบาย Kubernetes Manifests ทุกบรรทัด ทุกไฟล์ (Exam Study Guide)

เอกสารสรุปความหมายและหน้าที่ของแต่ละบรรทัดในไฟล์ YAML สำหรับอ่านเตรียมสอบและนำไปใช้เขียน Kubernetes Manifests จากศูนย์

---

## 🎯 โครงสร้างความสัมพันธ์ของระบบ

```text
[ ผู้ใช้ / Browser ] 
        │
        ▼ (Port 30080)
┌─────────────────────────────────────────────────────────────┐
│ 1. web-service (Service: NodePort)                          │
│    - nodePort: 30080 -> port: 80 -> targetPort: 80         │
│    - selector: app: web                                     │
└─────────────────────────────────────────────────────────────┘
        │
        ▼ (กระจายโหลดแบบ Round-Robin)
┌─────────────────────────────────────────────────────────────┐
│ 2. web-deployment (Deployment: 2 Replicas)                  │
│    - Pod 1: wordpress (label: app: web)                     │
│    - Pod 2: wordpress (label: app: web)                     │
│    - อ่าน DB_HOST, DB_NAME จาก ConfigMap                    │
│    - อ่าน Password จาก Secret                                │
└─────────────────────────────────────────────────────────────┘
        │
        ▼ (เชื่อมต่อผ่าน DNS: mysql-0.mysql-service:3306)
┌─────────────────────────────────────────────────────────────┐
│ 3. mysql-service (Service: Headless, clusterIP: None)       │
│    - selector: app: mysql                                   │
└─────────────────────────────────────────────────────────────┘
        │
        ▼ (ชี้ตรงไปยัง Pod เฉพาะตัว)
┌─────────────────────────────────────────────────────────────┐
│ 4. mysql (StatefulSet: 1 Replica)                           │
│    - Pod: mysql-0 (label: app: mysql)                       │
│    - อ่าน Password จาก Secret                                │
│    - อ่าน DB_NAME จาก ConfigMap                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 📑 สารบัญอธิบายไฟล์

1. [`01-config-secret.yaml` ➔ การจัดการตัวแปรและข้อมูลความลับ](#1-01-config-secretyaml)
2. [`02-stateful-db.yaml` ➔ Database (Headless Service + StatefulSet)](#2-02-stateful-dbyaml)
3. [`03-stateless-web.yaml` ➔ Web Application (NodePort Service + Deployment)](#3-03-stateless-webyaml)
4. [`standard-deployment-mode/` ➔ โหมดพื้นฐาน (ClusterIP + Deployment)](#4-standard-deployment-mode)

---

## 1. `01-config-secret.yaml`

```yaml
# ==============================================================
# ส่วนที่ 1: ConfigMap
# ==============================================================
apiVersion: v1              # API Version มาตรฐานของ ConfigMap คือ v1
kind: ConfigMap             # ประกาศว่าเป็น Object ชนิด ConfigMap
metadata:
  name: app-config          # ตั้งชื่อว่า app-config ไว้ให้ออบเจกต์อื่นมาเรียกใช้งาน
data:                       # ข้อมูล Key-Value ที่ต้องการเก็บ
  DB_HOST: "mysql-0.mysql-service:3306" # ที่อยู่ Host และ Port ของ Database
  DB_NAME: "exam_db"        # ชื่อฐานข้อมูล

---                         # ตัวแบ่ง Resource หลายตัวในไฟล์เดียว

# ==============================================================
# ส่วนที่ 2: Secret
# ==============================================================
apiVersion: v1              # API Version มาตรฐานของ Secret คือ v1
kind: Secret                # ประกาศว่าเป็น Object ชนิด Secret
metadata:
  name: app-secret          # ตั้งชื่อว่า app-secret
type: Opaque                # เป็น Secret ชนิดข้อมูลทั่วไป
stringData:                 # ข้อดีของ stringData คือพิมพ์รหัสผ่านเป็นข้อความธรรมดาได้เลย
  db-password: "rootpassword" # Kubernetes จะแปลงเป็น Base64 ให้อัตโนมัติในเบื้องหลัง
```

---

## 2. `02-stateful-db.yaml`

```yaml
---
# ==============================================================
# ส่วนที่ 1: Headless Service
# ==============================================================
apiVersion: v1
kind: Service
metadata:
  name: mysql-service       # ชื่อ Service (ใช้เป็นส่วนหนึ่งของ DNS Name)
spec:
  clusterIP: None           # กำหนดเป็น None เพื่อเป็น Headless Service (ไม่มี Virtual IP กลาง)
  ports:
    - port: 3306            # พอร์ตที่ Service รับฟัง
      targetPort: 3306      # พอร์ตใน Container ที่ส่งข้อมูลไป
  selector:
    app: mysql              # ส่องหา Pod ที่มีป้ายชื่อ "app: mysql"

---
# ==============================================================
# ส่วนที่ 2: StatefulSet (MySQL)
# ==============================================================
apiVersion: apps/v1         # StatefulSet อยู่ในกลุ่ม apps/v1
kind: StatefulSet           # ประกาศว่าเป็น StatefulSet สำหรับแอปพลิเคชันที่ต้องการรักษา State
metadata:
  name: mysql               # ชื่อของ StatefulSet
spec:
  serviceName: "mysql-service" # ระบุชื่อ Headless Service ที่จะสร้าง DNS ให้ Pod
  replicas: 1               # รัน MySQL จำนวน 1 Pod (จะได้ชื่อ Pod ว่า mysql-0)
  selector:
    matchLabels:
      app: mysql            # StatefulSet จะควมคุมเฉพาะ Pod ที่มี Label นี้
  template:                 # แม่แบบ Pod
    metadata:
      labels:
        app: mysql          # กำหนด Label ประจำตัว Pod เป็น "app: mysql"
    spec:
      containers:
        - name: mysql       # ชื่อ Container ภายใน Pod
          image: mysql:8.0  # Image ที่ใช้งาน
          env:
            - name: MYSQL_ROOT_PASSWORD     # ตัวแปรชื่อ MYSQL_ROOT_PASSWORD ใน Container
              valueFrom:                    # สั่งให้ดึงค่าจาก Resource อื่น
                secretKeyRef:
                  name: app-secret          # จาก Secret ชื่อ app-secret
                  key: db-password          # ดึงค่าจาก Key db-password
            - name: MYSQL_DATABASE          # ตัวแปรชื่อ MYSQL_DATABASE ใน Container
              valueFrom:
                configMapKeyRef:
                  name: app-config          # จาก ConfigMap ชื่อ app-config
                  key: DB_NAME              # ดึงค่าจาก Key DB_NAME
          ports:
            - containerPort: 3306           # เปิดพอร์ต 3306 ใน Container
```

---

## 3. `03-stateless-web.yaml`

```yaml
---
# ==============================================================
# ส่วนที่ 1: NodePort Service
# ==============================================================
apiVersion: v1
kind: Service
metadata:
  name: web-service         # ชื่อ Service
spec:
  type: NodePort            # ประเภท NodePort สำหรับเปิดให้คนภายนอก Cluster เข้ามาได้
  ports:
    - port: 80              # พอร์ตภายใน Service
      targetPort: 80        # พอร์ตของ Web Container ที่เปิดรับ HTTP (พอร์ต 80)
      nodePort: 30080       # พอร์ตบน Node จริง (เปิด Browser เข้า http://<Node-IP>:30080)
  selector:
    app: web                # ส่งทราฟฟิกไปยัง Pod ที่มี Label "app: web"

---
# ==============================================================
# ส่วนที่ 2: Deployment (WordPress Web Application)
# ==============================================================
apiVersion: apps/v1
kind: Deployment            # ใช้ Deployment สำหรับ Stateless App
metadata:
  name: web-deployment      # ชื่อ Deployment
spec:
  replicas: 2               # สร้าง Web Pod ขึ้นมา 2 ตัวทำงานคู่ขนานกัน
  selector:
    matchLabels:
      app: web              # ควบคุม Pod ที่มี Label "app: web"
  template:
    metadata:
      labels:
        app: web            # กำหนด Label ให้ Pod แต่ละตัวเป็น "app: web"
    spec:
      containers:
        - name: wordpress   # ชื่อ Container
          image: wordpress:latest # Image ของ WordPress
          env:
            - name: WORDPRESS_DB_HOST       # ตัวแปรชี้เป้าหมาย Database
              valueFrom:
                configMapKeyRef:
                  name: app-config
                  key: DB_HOST              # ได้ค่าเป็น "mysql-0.mysql-service:3306"
            - name: WORDPRESS_DB_NAME       # ตัวแปรชื่อ Database
              valueFrom:
                configMapKeyRef:
                  name: app-config
                  key: DB_NAME              # ได้ค่าเป็น "exam_db"
            - name: WORDPRESS_DB_USER       # ระบุ User ตรงๆ
              value: "root"
            - name: WORDPRESS_DB_PASSWORD   # ตัวแปร Password Database
              valueFrom:
                secretKeyRef:
                  name: app-secret
                  key: db-password          # ได้ค่ารหัสผ่านจาก Secret
          ports:
            - containerPort: 80             # เปิดพอร์ต 80 ใน Container
```

---

## 4. `standard-deployment-mode/`

ในกรณีที่โจทย์ต้องการให้ใช้ **Deployment ปกติ** ทั้งหมด:
* ตัวแปร `WORDPRESS_DB_HOST` จะระบุค่าเป็น `mysql-service:3306` (ชี้ไปที่ ClusterIP Service ตรงๆ ไม่ต้องมี `-0`)
* Service ของ MySQL จะเป็น `type: ClusterIP` (มี Virtual IP กลาง 1 ตัว)
* ฐานข้อมูล MySQL จะรันภายใต้ `kind: Deployment`
