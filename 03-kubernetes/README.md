# ส่วนที่ 3: Kubernetes (Stateless vs. Stateful Architecture)

คู่มือการจัดการระบบบน Kubernetes โดยแบ่งแยกอย่างชัดเจนระหว่าง:
- **Stateful Application (Database):** ใช้ `StatefulSet` ร่วมกับ `Headless Service` เพื่อรักษาอัตลักษณ์และ State
- **Stateless Application (Web Application):** ใช้ `Deployment` ร่วมกับ `NodePort Service` เพื่อรองรับการขยายตัว (Scale)

---

## 📁 โครงสร้างไฟล์ Manifests

1. [`01-config-secret.yaml`](file:///C:/Users/kritt/IdeaProjects/MIDTERM-VM-DOCKER-Kubernetes/03-kubernetes/01-config-secret.yaml): จัดการ ConfigMap (`app-config`) และ Secret (`app-secret`)
2. [`02-stateful-db.yaml`](file:///C:/Users/kritt/IdeaProjects/MIDTERM-VM-DOCKER-Kubernetes/03-kubernetes/02-stateful-db.yaml): MySQL StatefulSet + Headless Service (`clusterIP: None`)
3. [`03-stateless-web.yaml`](file:///C:/Users/kritt/IdeaProjects/MIDTERM-VM-DOCKER-Kubernetes/03-kubernetes/03-stateless-web.yaml): Web Deployment (2 Replicas) + NodePort Service (`port 30080`)

> 📂 *หมายเหตุ: หากโจทย์ระบุให้ใช้แบบปกติ (Standard Deployment ทั้งหมด) สามารถดูตัวอย่างได้ที่โฟลเดอร์ [`standard-deployment-mode/`](file:///C:/Users/kritt/IdeaProjects/MIDTERM-VM-DOCKER-Kubernetes/03-kubernetes/standard-deployment-mode)*

---

## 🧠 การเปรียบเทียบสถาปัตยกรรม Stateless vs Stateful บน Kubernetes

| คุณลักษณะ | Stateless (Web Server) | Stateful (Database) |
| :--- | :--- | :--- |
| **K8s Controller** | `Deployment` | `StatefulSet` |
| **ชื่อของ Pod** | สุ่มแฮช เช่น `web-deployment-7b9c-xyz` | มีลำดับแน่นอน เช่น `mysql-0`, `mysql-1` |
| **Service Type** | `NodePort` หรือ `ClusterIP` ทั่วไป | `Headless Service` (`clusterIP: None`) |
| **DNS Resolution** | โหลดบาลานซ์สุ่มไปยัง Pod ใดก็ได้ | ชี้ตรงราย Pod เช่น `mysql-0.mysql-service` |
| **การ Scale** | ขยาย/ลดได้ทันที ไม่มีผลเสียต่อข้อมูล | มีลำดับในการสร้าง (0->1->2) และลบ (2->1->0) |

---

## 🚀 ลำดับขั้นตอนการ Deploy (How-To Apply)

### 1. เข้าสู่โฟลเดอร์ Kubernetes
```bash
cd 03-kubernetes
```

### 2. รันคำสั่งตามลำดับ (หรือรันรวดเดียว)
```bash
# 1. สร้าง ConfigMap และ Secret ก่อนเสมอ
kubectl apply -f 01-config-secret.yaml

# 2. สร้าง MySQL Stateful Database
kubectl apply -f 02-stateful-db.yaml

# 3. สร้าง Stateless Web Application
kubectl apply -f 03-stateless-web.yaml
```

> ⚡ **ทางลัด (รันรวดเดียว):**
> ```bash
> kubectl apply -f .
> ```

---

## 🔍 คำสั่งตรวจสอบสถานะและการทำงาน

```bash
# ตรวจสอบ Pod ทั้งหมดพร้อม Node และ IP
kubectl get pods -o wide

# ตรวจสอบ StatefulSet และ Deployment
kubectl get statefulset,deployment

# ตรวจสอบ Services ทั้งหมด (ดู Headless Service และ NodePort)
kubectl get svc

# ตรวจสอบ ConfigMap และ Secret
kubectl get configmap,secret
```

---

## 🌐 การเข้าถึง Web Application

1. **ผ่าน NodePort โดยตรง:**
   - URL: `http://localhost:30080` หรือ `http://<Node-IP>:30080`
2. **ผ่าน Minikube Service (กรณีใช้ Minikube):**
   ```bash
   minikube service web-service --url
   ```
3. **ผ่าน Port-Forwarding (ใช้ทดสอบด่วน):**
   ```bash
   kubectl port-forward svc/web-service 8080:80
   # เข้าใช้งานผ่าน: http://localhost:8080
   ```

---

## 🛠️ คำสั่ง Debug และ Scaling ระหว่างสอบ

```bash
# 1. ทดสอบขยาย Scale Stateless Web เป็น 4 Pods
kubectl scale deployment web-deployment --replicas=4

# 2. ดู Logs ของ Stateful Database
kubectl logs -f mysql-0

# 3. ดู Logs ของ Web Pods
kubectl logs -f -l app=web

# 4. ทดสอบ DNS ภายในคลัสเตอร์ (ทดสอบจาก Web Pod ไปหา DB)
kubectl exec -it <web-pod-name> -- nslookup mysql-0.mysql-service

# 5. ลบ Resources ทั้งหมดเพื่อ Reset
kubectl delete -f .
```
