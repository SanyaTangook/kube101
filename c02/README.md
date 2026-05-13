# Kubernetes 101 - Workload Resources

เอกสารนี้เปรียบเทียบ Workload Resource หลักใน Kubernetes: Deployment, DaemonSet, StatefulSet

## 📁 ไฟล์ต่างๆ

### 1. Deployment (1-deployment.yaml)
- ใช้สำหรับจัดการ Replicas ของ Pod แบบ stateless
- รองรับ Rolling Updates และ Rollback
- Pod ทุกตัวไม่แตกต่างกัน (interchangeable)

**คำสั่ง:**
```bash
kubectl apply -f 1-deployment.yaml
kubectl get deployments
kubectl scale deployment web-deployment --replicas=5
kubectl rollout status deployment/web-deployment
kubectl rollout undo deployment/web-deployment
```

### 2. DaemonSet (2-daemonset.yaml)
- รัน Pod หนึ่งตัวบนทุก Node ใน Cluster
- เมื่อมี Node ใหม่ DaemonSet จะรัน Pod บน Node นั้นโดยอัตโนมัติ
- ใช้สำหรับ logging agent (fluentd), monitoring agent, network plugin

**คำสั่ง:**
```bash
kubectl apply -f 2-daemonset.yaml
kubectl get daemonsets
kubectl get pods -o wide -l app=fluentd  # ดู Pod ที่รันบนแต่ละ Node
kubectl describe daemonset fluentd-daemonset
```

### 3. StatefulSet (3-statefulset.yaml)
- ใช้สำหรับ stateful applications ที่ต้องการ stable network identity
- Pod แต่ละตัวมี identity ถาวร (web-statefulset-0, web-statefulset-1, ...)
- รองรับ PVC ที่สร้างอัตโนมัติผ่าน volumeClaimTemplates
- ลำดับการ start/stop Pod เป็นไปตามลำดับ (ordered)

**คำสั่ง:**
```bash
kubectl apply -f 3-statefulset.yaml
kubectl get statefulsets
kubectl get pods -l app=web
kubectl get pvc  # PVC ที่สร้างอัตโนมัติจาก volumeClaimTemplates
```

## 🔍 เปรียบเทียบ Workload Resources

| คุณสมบัติ | Deployment | DaemonSet | StatefulSet |
|-----------|-----------|-----------|-------------|
| จำนวน Pod ต่อ Node | ตาม replicas | 1 ตัวต่อ Node | ตาม replicas |
| Identity | ไม่มี (random) | ตามชื่อ Node | ถาวร (ordinal index) |
| Storage | ไม่มี (หรือ share volume) | มักใช้ hostPath | PVC เฉพาะต่อ Pod |
| Use Case | Web app, API server | Logging, monitoring | Database, message queue |
| Update Strategy | RollingUpdate | RollingUpdate | RollingUpdate (ordered) |
| Scaling | เพิ่ม/ลด replicas | อัตโนมัติตาม Node | เพิ่ม/ลดแบบ ordered |

## 🚀 วิธีใช้ทั้งหมด

```bash
# ใช้ทั้งหมด
kubectl apply -f .

# ลบทั้งหมด
kubectl delete -f .

# ดู resources ทั้งหมดใน c02
kubectl get deployments,daemonsets,statefulsets
```

## 📝 บันทึก

- Deployment และ StatefulSet ใช้ `spec.replicas` เพื่อกำหนดจำนวน Pod
- DaemonSet ไม่ใช้ replicas — จำนวน Pod เท่ากับจำนวน Node ใน Cluster
- StatefulSet ต้องมี Headless Service (serviceName) เพื่อ stable network identity
- volumeClaimTemplates ใน StatefulSet จะสร้าง PVC ให้อัตโนมัติ (dynamic provisioning)

## 🔗 ลิงก์อ้างอิง

- [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
- [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
