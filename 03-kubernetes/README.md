# ☸️ คู่มือ Kubernetes ฉบับเจาะลึก (Line-by-Line Guide & Architecture)

คู่มืออธิบายการทำงานของไฟล์ Kubernetes Manifests อย่างละเอียดแบบ **บรรทัดต่อบรรทัด** ครอบคลุมทั้ง **Stateful Architecture Mode** และ **Standard Deployment Mode** พร้อมสูตรสำเร็จในการแปลงจาก Docker Compose สู่ Kubernetes สำหรับการสอบปฏิบัติ

---

## 📑 สารบัญเนื้อหา (Table of Contents)

1. [🧠 โครงสร้าง 4 บรรทัดทองคำของ Kubernetes (Foundation)](#-โครงสร้าง-4-บรรทัดทองคำของ-kubernetes-foundation)
2. [🌟 โหมดที่ 1: Stateful Architecture Mode (StatefulSet + Headless Service)](#-โหมดที่-1-stateful-architecture-mode-statefulset--headless-service)
   - [01-config-secret.yaml (อธิบายทีละบรรทัด)](#-ไฟล์ที่-1-01-config-secretyaml)
   - [02-stateful-db.yaml (อธิบายทีละบรรทัด)](#-ไฟล์ที่-2-02-stateful-dbyaml)
   - [03-stateless-web.yaml (อธิบายทีละบรรทัด)](#-ไฟล์ที่-3-03-stateless-webyaml)
3. [📦 โหมดที่ 2: Standard Deployment Mode (Pure Deployment + ClusterIP)](#-โหมดที่-2-standard-deployment-mode-pure-deployment--clusterip)
4. [📊 ตารางเปรียบเทียบ: Standard Deployment vs. StatefulSet](#-ตารางเปรียบเทียบ-standard-deployment-vs-statefulset)
5. [🛠️ สูตรแปลงข้อสอบ: Docker Compose ➔ Kubernetes Manifests](#️-สูตรแปลงข้อสอบ-docker-compose--kubernetes-manifests)
6. [⚠️ กฎเหล็กและข้อควรระวังในการเขียน YAML สอบ](#️-กฎเหล็กและข้อควรระวังในการเขียน-yaml-สอบ)
7. [🚀 คำสั่ง Deploy, ตรวจสอบ และ Debug ในห้องสอบ](#-คำสั่ง-deploy-ตรวจสอบ-และ-debug-ในห้องสอบ)

---

# 🧠 โครงสร้าง 4 บรรทัดทองคำของ Kubernetes (Foundation)

ทุก Resource ใน Kubernetes จะต้องมี 4 ฟิลด์หลักนี้เสมอ:

```yaml
apiVersion: ...    # 1. เวอร์ชัน API (เช่น v1 หรือ apps/v1)
kind: ...          # 2. ประเภท Resource (เช่น Pod, Service, Deployment, StatefulSet, Secret, ConfigMap)
metadata:          # 3. ข้อมูลประจำตัว เช่น name (ชื่อของ Resource นั้น) และ labels
spec:              # 4. Specification (ข้อกำหนดการทำงาน เช่น จำนวน Replicas, Ports, Container Image)
```

---

# 🌟 โหมดที่ 1: Stateful Architecture Mode (StatefulSet + Headless Service)

โหมดนี้เป็นโครงสร้างระดับมืออาชีพ (Best Practice) แยก ConfigMap, Secret, StatefulSet และ Deployment ออกจากกันชัดเจน

---

### 📄 ไฟล์ที่ 1: [`01-config-secret.yaml`](file:///C:/Users/kritt/IdeaProjects/MIDTERM-VM-DOCKER-Kubernetes/03-kubernetes/01-config-secret.yaml)

ใช้สำหรับจัดการตัวแปรสภาพแวดล้อม (Environment Variables) โดยแยกข้อมูลทั่วไปออกจากข้อมูลความลับ

```yaml
# ==============================================================
# ส่วนที่ 1: ConfigMap (สำหรับตัวแปรทั่วไปที่ไม่ใช่ความลับ)
# ==============================================================
apiVersion: v1              # API Version ของ ConfigMap คือ v1
kind: ConfigMap             # กำหนดประเภทเป็น ConfigMap
metadata:
  name: app-config          # ตั้งชื่อ Resource ว่า "app-config" เพื่อให้ Pod อื่นอ้างถึง
data:
  DB_HOST: "mysql-0.mysql-service:3306" # ชี้ไปยัง Pod แรกของ StatefulSet ผ่าน Headless Service
  DB_NAME: "exam_db"        # กำหนดชื่อฐานข้อมูล

---                         # เครื่องหมายคั่นระหว่าง Resource ในไฟล์เดียวกัน

# ==============================================================
# ส่วนที่ 2: Secret (สำหรับข้อมูลสำคัญ/รหัสผ่าน)
# ==============================================================
apiVersion: v1              # API Version ของ Secret คือ v1
kind: Secret                # กำหนดประเภทเป็น Secret
metadata:
  name: app-secret          # ตั้งชื่อ Resource ว่า "app-secret"
type: Opaque                # ชนิดของ Secret แบบ Key-Value ทั่วไป
stringData:                 # ใช้ stringData เพื่อเขียนค่า Plaintext โดยตรง (K8s จะ Encode Base64 ให้เอง)
  db-password: "rootpassword" # รหัสผ่าน root ของ Database
```

---

### 📄 ไฟล์ที่ 2: [`02-stateful-db.yaml`](file:///C:/Users/kritt/IdeaProjects/MIDTERM-VM-DOCKER-Kubernetes/03-kubernetes/02-stateful-db.yaml)

ใช้สร้าง **Headless Service** และ **StatefulSet** สำหรับจัดการฐานข้อมูล MySQL

```yaml
---
# ==============================================================
# ส่วนที่ 1: Headless Service (ทางเข้า DNS ประจำตัว Pod)
# ==============================================================
apiVersion: v1
kind: Service
metadata:
  name: mysql-service       # ชื่อ Service (จะถูกใช้เป็นโดเมน DNS ภายในคลัสเตอร์)
spec:
  clusterIP: None           # ⭐️ หัวใจสำคัญ: None = Headless Service ไม่สร้าง IP กลาง แต่คืน IP ตรงของ Pod
  ports:
    - port: 3306            # พอร์ตของ Service
      targetPort: 3306      # พอร์ตปลายทางภายใน Container ที่ MySQL กำลังรันอยู่
  selector:
    app: mysql              # คัดกรองส่งทราฟฟิกไปหา Pod ที่มีป้ายกำกับ "app: mysql"

---
# ==============================================================
# ส่วนที่ 2: StatefulSet (ตัวจัดการ Database Pod แบบ Stateful)
# ==============================================================
apiVersion: apps/v1         # StatefulSet อยู่ในกลุ่ม API apps/v1
kind: StatefulSet           # กำหนดประเภทเป็น StatefulSet
metadata:
  name: mysql               # ชื่อ StatefulSet
spec:
  serviceName: "mysql-service" # ⭐️ ผูกกับ Headless Service ที่สร้างไว้ด้านบนเพื่อกำหนด Domain Name
  replicas: 1               # จำนวน Pod Database ที่ต้องการรัน (ปกติเริ่มที่ 1 ตัว)
  selector:
    matchLabels:
      app: mysql            # StatefulSet จะควบคุม Pod ที่มี Label "app: mysql"
  template:                 # แม่แบบ (Template) สำหรับใช้ปั๊ม Pod ออกมา
    metadata:
      labels:
        app: mysql          # แปะป้าย Label "app: mysql" ให้กับ Pod (ต้องตรงกับ selector)
    spec:
      containers:           # กำหนดคุณสมบัติของ Container ภายใน Pod
        - name: mysql       # ตั้งชื่อ Container
          image: mysql:8.0  # Image ที่ดึงมาจาก Docker Hub
          env:              # รายการตัวแปร Environment ที่จะส่งให้ Container
            - name: MYSQL_ROOT_PASSWORD     # ตัวแปรที่ MySQL บังคับใช้
              valueFrom:                    # สั่งให้ดึงค่ามาจาก Secret
                secretKeyRef:
                  name: app-secret          # อ้างถึง Secret ชื่อ "app-secret"
                  key: db-password          # ดึงค่าจาก Key "db-password"
            - name: MYSQL_DATABASE          # ตัวแปรสำหรับสร้าง Database เริ่มต้น
              valueFrom:                    # สั่งให้ดึงค่ามาจาก ConfigMap
                configMapRef:
                  name: app-config          # อ้างถึง ConfigMap ชื่อ "app-config"
                  key: DB_NAME              # ดึงค่าจาก Key "DB_NAME"
          ports:
            - containerPort: 3306           # ประกาศพอร์ตที่ Container เปิดให้บริการ
```

---

### 📄 ไฟล์ที่ 3: [`03-stateless-web.yaml`](file:///C:/Users/kritt/IdeaProjects/MIDTERM-VM-DOCKER-Kubernetes/03-kubernetes/03-stateless-web.yaml)

ใช้สร้าง **NodePort Service** และ **Deployment** สำหรับ Web Application (WordPress)

```yaml
---
# ==============================================================
# ส่วนที่ 1: Service แบบ NodePort (เปิดพอร์ตให้ภายนอกเข้าใช้งาน)
# ==============================================================
apiVersion: v1
kind: Service
metadata:
  name: web-service         # ชื่อ Service
spec:
  type: NodePort            # ⭐️ ประเภท NodePort = เปิดพอร์ตบน Node จริงให้เข้าผ่าน Browser ได้
  ports:
    - port: 80              # พอร์ตของ Service ภายใน
      targetPort: 80        # พอร์ตของ Container เว็บปลายทาง (HTTP Port 80)
      nodePort: 30080       # ⭐️ พอร์ตที่เปิดบนเครื่องโฮสต์ (เข้าผ่าน http://<Node-IP>:30080)
  selector:
    app: web                # ส่งทราฟฟิกไปหา Pod ที่มี Label "app: web"

---
# ==============================================================
# ส่วนที่ 2: Deployment (ตัวจัดการ Web Application แบบ Stateless)
# ==============================================================
apiVersion: apps/v1
kind: Deployment            # ใช้ Deployment สำหรับ Stateless App เพราะ Scale เพิ่ม/ลดได้ทันที
metadata:
  name: web-deployment      # ชื่อ Deployment
spec:
  replicas: 2               # ⭐️ รัน 2 Pods เพื่อกระจายโหลดและทำ High Availability
  selector:
    matchLabels:
      app: web              # ควบคุม Pod ที่มี Label "app: web"
  template:
    metadata:
      labels:
        app: web            # แปะป้าย Label "app: web" ให้กับ Pod
    spec:
      containers:
        - name: wordpress   # ชื่อ Container
          image: wordpress:latest # Image ของ Web Application
          env:
            - name: WORDPRESS_DB_HOST       # ตัวแปรระบุที่อยู่ของ MySQL Server
              valueFrom:
                configMapRef:
                  name: app-config
                  key: DB_HOST              # ได้ค่าเป็น "mysql-0.mysql-service:3306"
            - name: WORDPRESS_DB_NAME       # ตัวแปรระบุชื่อ Database
              valueFrom:
                configMapRef:
                  name: app-config
                  key: DB_NAME              # ได้ค่าเป็น "exam_db"
            - name: WORDPRESS_DB_USER       # ระบุ User ที่ใช้เชื่อมต่อ
              value: "root"                 # สามารถใส่ค่าคงที่ตรงๆ ได้โดยใช้คำว่า value:
            - name: WORDPRESS_DB_PASSWORD   # ระบุรหัสผ่านเชื่อมต่อ Database
              valueFrom:
                secretKeyRef:
                  name: app-secret
                  key: db-password          # ได้ค่ารหัสผ่านจาก Secret
          ports:
            - containerPort: 80             # พอร์ต HTTP ภายใน Container
```

---

# 📦 โหมดที่ 2: Standard Deployment Mode (Pure Deployment + ClusterIP)

อยู่ในโฟลเดอร์ [`standard-deployment-mode/`](file:///C:/Users/kritt/IdeaProjects/MIDTERM-VM-DOCKER-Kubernetes/03-kubernetes/standard-deployment-mode) เป็นรูปแบบที่ใช้ **Deployment สำหรับทุกอย่าง** และใช้ **ClusterIP Service ปกติ**:

1. **[`01-secrets.yaml`](file:///C:/Users/kritt/IdeaProjects/MIDTERM-VM-DOCKER-Kubernetes/03-kubernetes/standard-deployment-mode/01-secrets.yaml):**
   - รวบรวมตัวแปร Credentials ทั้งหมด (`root-password`, `wp-user`, `wp-password`) ไว้ใน Secret ก้อนเดียว
2. **[`02-mysql-deployment.yaml`](file:///C:/Users/kritt/IdeaProjects/MIDTERM-VM-DOCKER-Kubernetes/03-kubernetes/standard-deployment-mode/02-mysql-deployment.yaml):**
   - **Service:** ใช้ `type: ClusterIP` (หรือเว้นว่างไว้) เพื่อสร้าง Virtual IP กลางสำหรับกระจายทราฟฟิก
   - **Controller:** ใช้ `kind: Deployment` สำหรับ MySQL แทน StatefulSet
3. **[`03-wordpress-deployment.yaml`](file:///C:/Users/kritt/IdeaProjects/MIDTERM-VM-DOCKER-Kubernetes/03-kubernetes/standard-deployment-mode/03-wordpress-deployment.yaml):**
   - ชี้ `WORDPRESS_DB_HOST` ตรงไปยังชื่อ Service กลาง: `mysql-service:3306` (ไม่ต้องระบุ index `-0` เหมือน Headless Service)

---

# 📊 ตารางเปรียบเทียบ: Standard Deployment vs. StatefulSet

| หัวข้อเปรียบเทียบ | Standard Deployment Mode | Stateful Architecture Mode |
| :--- | :--- | :--- |
| **Controller สำหรับ DB** | `Deployment` | `StatefulSet` |
| **Service สำหรับ DB** | `ClusterIP` (มี Virtual IP กลาง) | `Headless Service` (`clusterIP: None`) |
| **รูปแบบชื่อ Pod ของ DB** | สุ่มแฮช เช่น `mysql-deployment-7b9d-xyz` | คงที่และเรียงลำดับ เช่น `mysql-0`, `mysql-1` |
| **DNS ในการเชื่อมต่อ** | `mysql-service:3306` | `mysql-0.mysql-service:3306` |
| **การจัดการ State** | เหมาะกับข้อมูลที่ไม่เน้นการจำอัตลักษณ์ | เหมาะกับ Database ที่ต้องการผูกติดกับ Storage ถาวร |
| **การจัดการตัวแปร** | ใช้ Secret ร่วมกันทั้งหมด | แยก **ConfigMap** (ตัวแปรทั่วไป) + **Secret** (รหัสผ่าน) |

---

# 🛠️ สูตรแปลงข้อสอบ: Docker Compose ➔ Kubernetes Manifests

เมื่อได้โจทย์ที่เป็น `docker-compose.yml` ให้แปลงตามคู่มือนี้:

```text
┌───────────────────────────────────────────────┐          ┌───────────────────────────────────────────────┐
│              ใน Docker Compose                │          │                 ใน Kubernetes                 │
├───────────────────────────────────────────────┤          ├───────────────────────────────────────────────┤
│ service: web (แอปที่ไม่เก็บข้อมูล)            │  ───►    │ kind: Deployment (replicas >= 2)              │
│ service: db (ฐานข้อมูล/มี Volume)             │  ───►    │ kind: StatefulSet (serviceName: "...")        │
│ ports: "8080:80" (ต้องการเปิดให้ภายนอกเข้า)    │  ───►    │ kind: Service (type: NodePort, nodePort: ...) │
│ ports: "3306:3306" (สื่อสารภายในระบบ)         │  ───►    │ kind: Service (type: ClusterIP / Headless)    │
│ environment: (รหัสผ่าน/Token)                 │  ───►    │ kind: Secret (stringData)                     │
│ environment: (ชื่อ DB, URL, Hostname)         │  ───►    │ kind: ConfigMap (data)                        │
└───────────────────────────────────────────────┘          └───────────────────────────────────────────────┘
```

---

# ⚠️ กฎเหล็กและข้อควรระวังในการเขียน YAML สอบ

1. **ห้ามใช้ปุ่ม Tab เด็ดขาด:** ไวยากรณ์ YAML บังคับใช้การเคาะ Spacebar 2 เคาะตามระดับ Level เท่านั้น
2. **การจับคู่ Label กับ Selector (ต้องตรงกัน 100%):**
   * ใน Controller: `spec.selector.matchLabels.app: XXX` ⟷ `spec.template.metadata.labels.app: XXX`
   * ใน Service: `spec.selector.app: XXX` ⟷ Pod Label `app: XXX`
3. **การดึงค่าจาก Secret และ ConfigMap:**
   * ดึงจาก Secret: ใช้บล็อก `secretKeyRef` ระบุ `name` และ `key`
   * ดึงจาก ConfigMap: ใช้บล็อก `configMapRef` ระบุ `name` และ `key`
4. **ช่วงพอร์ตของ NodePort:** พอร์ตภายนอกที่กำหนดใน `nodePort` จะต้องอยู่ในช่วง **`30000 - 32767`** เสมอ (เช่น `30080`)
5. **ลำดับการ Apply:** ต้องสร้าง `Secret / ConfigMap` ก่อนสร้าง `Pod / Deployment` เสมอ

---

# 🚀 คำสั่ง Deploy, ตรวจสอบ และ Debug ในห้องสอบ

```bash
# 1. สั่งรัน Resources ทั้งหมดในโฟลเดอร์
kubectl apply -f .

# 2. ตรวจสอบสถานะ Pods แบบละเอียด (ดูว่าขึ้น Running ครบไหม)
kubectl get pods -o wide

# 3. ตรวจสอบ Services ทั้งหมด (ดูพอร์ต NodePort และ ClusterIP)
kubectl get svc

# 4. ดู Logs เมื่อ Pod มีปัญหาเปิดไม่ติด
kubectl logs -f -l app=web
kubectl logs -f mysql-0

# 5. ดูรายละเอียดและ Event สาเหตุข้อผิดพลาด (เช่น CrashLoopBackOff, ImagePullBackOff)
kubectl describe pod <pod-name>

# 6. คำสั่ง Port-Forward สำรอง (เมื่อ NodePort เข้าไม่ได้)
kubectl port-forward svc/web-service 8080:80

# 7. สั่งลบ Resources ทั้งหมดเพื่อเริ่มต้นใหม่
kubectl delete -f .
```
