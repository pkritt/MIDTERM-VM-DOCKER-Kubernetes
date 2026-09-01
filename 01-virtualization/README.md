# ส่วนที่ 1: Virtualization (การตั้งค่า VirtualBox & Linux Setup)

คู่มือการตั้งค่า Virtual Machine 2 เครื่อง (Web Server & Database) ให้อยู่ในวง Network เดียวกัน รองรับทั้งการเข้าถึงผ่าน **Port Forwarding** และ **Host-Only Adapter**

---

## 🎯 สรุป Topology และ IP Plan

| VM Name | บทบาท | Adapter 1 (NAT Network) | Adapter 2 (Host-Only - ทางเลือก) | บริการที่รัน |
| :--- | :--- | :--- | :--- | :--- |
| **VM1-Web** | Stateless (Web Server) | `192.168.1xx.10` | `192.168.56.10` | Nginx / Apache / WordPress (Port 80) |
| **VM2-DB** | Stateful (Database) | `192.168.1xx.20` | `192.168.56.20` | MySQL / MariaDB (Port 3306) |

> 💡 **หมายเหตุ:** แทนค่า `1xx` ด้วยรหัสนักศึกษาตามที่โจทย์กำหนด (เช่น `192.168.150.0/24`)

---

## 🛠️ ขั้นตอนที่ 1: สร้าง NAT Network บน VirtualBox

NAT Network ช่วยให้ VM ภายในคุยกันได้เอง และสามารถแชร์อินเทอร์เน็ตจากเครื่อง Host เพื่อติดตั้งแพ็กเกจได้พร้อมๆ กัน

1. เปิด **VirtualBox**
2. ไปที่เมนู **File** -> **Tools** -> **Network Manager** (หรือกด `Ctrl + H`)
3. เลือกแท็บ **NAT Networks**
4. กดปุ่ม **Create** (ปุ่ม +)
5. ตั้งค่าดังนี้:
   - **Network Name:** `ExamNet`
   - **IPv4 Prefix:** `192.168.1xx.0/24` (เปลี่ยน `xx` เป็นรหัสนักศึกษา เช่น `192.168.150.0/24`)
   - **Enable DHCP:** `[✔] ติ๊กถูก`
6. กด **Apply**

---

## 🛠️ ขั้นตอนที่ 2: ตั้งค่า Network Adapter ให้ VM ทั้ง 2 เครื่อง

ทำกับทั้ง **VM1 (Web)** และ **VM2 (DB)**:

1. คลิกขวาที่ชื่อ VM -> เลือก **Settings** (หรือกด `Ctrl + S`)
2. ไปที่เมนู **Network**
3. **Adapter 1 (จำเป็น):**
   - **Enable Network Adapter:** `[✔] ติ๊กถูก`
   - **Attached to:** เลือก `NAT Network`
   - **Name:** เลือก `ExamNet`
4. **Adapter 2 (ทางเลือก - Host-Only):**
   > *ใช้เมื่อต้องการให้ Host OS คุยกับ VM ได้โดยตรงด้วย IP โดยไม่ต้องตั้ง Port Forwarding*
   - **Enable Network Adapter:** `[✔] ติ๊กถูก`
   - **Attached to:** เลือก `Host-Only Adapter`
   - **Name:** เลือก `VirtualBox Host-Only Ethernet Adapter` (วงปกติคือ `192.168.56.x`)
5. กด **OK**

---

## 🛠️ ขั้นตอนที่ 3: ตั้งค่า Static IP บน Ubuntu (Netplan)

บูต VM แต่ละตัวขึ้นมา แล้วล็อกอินเข้าไปตั้งค่า IP แบบคงที่

### 1. ตรวจสอบชื่อ Network Interface
```bash
ip a
```
*(ปกติ Adapter 1 จะเป็น `enp0s3` และ Adapter 2 จะเป็น `enp0s8`)*

### 2. แก้ไขไฟล์ Netplan
```bash
sudo nano /etc/netplan/50-cloud-init.yaml
# (หรือไฟล์ .yaml ใดๆ ที่อยู่ใน /etc/netplan/)
```

### 3. เลือกรูปแบบการตั้งค่า:

#### แบบที่ A: ใช้เฉพาะ NAT Network (Adapter 1 ตัวเดียว)
```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 192.168.1xx.10/24 # VM1 ใช้ .10, VM2 ใช้ .20
      routes:
        - to: default
          via: 192.168.1xx.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

#### แบบที่ B: ใช้คู่กับ Host-Only Adapter (Adapter 2 ตัว)
```yaml
network:
  version: 2
  ethernets:
    enp0s3: # NAT Network ออกเน็ต
      dhcp4: no
      addresses:
        - 192.168.1xx.10/24
      routes:
        - to: default
          via: 192.168.1xx.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
    enp0s8: # Host-Only สำหรับ Host OS เข้าตรง
      dhcp4: no
      addresses:
        - 192.168.56.10/24 # VM1 ใช้ .10, VM2 ใช้ .20
```

> ⚠️ **ข้อควรระวังใน YAML:** ห้ามใช้ปุ่ม Tab เด็ดขาด ให้ใช้การเคาะเว้นวรรค (Space) 2 เคาะตามระดับ Level

### 4. บันทึกและทดสอบใช้งาน
```bash
# ตรวจสอบไวยากรณ์
sudo netplan try

# ใช้งานจริง
sudo netplan apply

# ตรวจสอบ IP ที่ได้
ip a
```

---

## 🛠️ ขั้นตอนที่ 4: ตั้งค่า Port Forwarding (กรณีใช้ NAT Network อย่างเดียว)

1. ไปที่ **File** -> **Tools** -> **Network Manager** -> **NAT Networks**
2. ดับเบิ้ลคลิกที่ `ExamNet` -> เลือกแถบ **Port Forwarding**
3. เพิ่ม Rules ดังนี้:

| Rule Name | Protocol | Host IP | Host Port | Guest IP | Guest Port |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Web-HTTP** | TCP | `127.0.0.1` | `8080` | `192.168.1xx.10` | `80` |
| **DB-MySQL** | TCP | `127.0.0.1` | `3306` | `192.168.1xx.20` | `3306` |
| **SSH-Web** | TCP | `127.0.0.1` | `2201` | `192.168.1xx.10` | `22` |
| **SSH-DB** | TCP | `127.0.0.1` | `2202` | `192.168.1xx.20` | `22` |

---

## 🔍 เช็คลิสต์ตรวจสอบผลลัพธ์ (Verification)

```bash
# 1. ทดสอบปิงระหว่าง VM1 <-> VM2
ping -c 4 192.168.1xx.20

# 2. ทดสอบการออกอินเทอร์เน็ต
ping -c 4 8.8.8.8

# 3. ทดสอบเข้าถึงจากเครื่อง Host:
# - ผ่าน Port Forwarding: http://localhost:8080 หรือต่อ DB ที่ localhost:3306
# - ผ่าน Host-Only IP: http://192.168.56.10 หรือต่อ DB ที่ 192.168.56.20:3306
```
