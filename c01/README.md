# Kubernetes 101 - ไฟล์ YAML พื้นฐาน

เอกสารนี้มีตัวอย่างไฟล์ YAML สำหรับการเรียนรู้พื้นฐาน Kubernetes

## 📁 ไฟล์ต่างๆ

### 1. Pod (1-pod.yaml)
- หน่วยเล็กที่สุดใน Kubernetes
- ประกอบด้วย container 1 ตัวหรือมากกว่า
- ใช้สำหรับ ad-hoc tasks หรือ development

**คำสั่ง:**
```bash
kubectl apply -f 1-pod.yaml
kubectl get pods
kubectl logs nginx-pod
kubectl delete pod nginx-pod
```

### 2. Deployment (2-deployment.yaml)
- จัดการ Replicas ของ Pod โดยอัตโนมัติ
- ใช้สำหรับ production applications
- รองรับ Rolling Updates และ Rollback

**คำสั่ง:**
```bash
kubectl apply -f 2-deployment.yaml
kubectl get deployments
kubectl get pods -l app=nginx
kubectl scale deployment nginx-deployment --replicas=5
kubectl set image deployment/nginx-deployment nginx=nginx:1.16
kubectl rollout undo deployment/nginx-deployment
```

### 3. Service (3-service.yaml)
- Expose Pod ให้สามารถเข้าถึงได้
- 3 ประเภท: ClusterIP, NodePort, LoadBalancer

**ประเภท Service:**
- **ClusterIP**: ใช้ภายใน Cluster เท่านั้น (Default)
- **NodePort**: expose ผ่าน Node IP:Port (30000-32767)
- **LoadBalancer**: ใช้ cloud provider's load balancer

**คำสั่ง:**
```bash
kubectl apply -f 3-service.yaml
kubectl get services
kubectl describe svc nginx-service
kubectl port-forward svc/nginx-service 8080:80
```

### 4. StorageClass (4-storageclass.yaml)
- กำหนดวิธี dynamic provisioning ของ PersistentVolume
- ต่างๆ provisioner เช่น AWS EBS, GCE PD, Azure Disk

**คำสั่ง:**
```bash
kubectl apply -f 4-storageclass.yaml
kubectl get storageclass
```

### 5. PersistentVolume (5-persistentvolume.yaml)
- Resource storage ในระดับ Cluster
- 3 Access Modes: ReadWriteOnce, ReadWriteMany, ReadOnlyMany
- Reclaim Policy: Retain, Delete, Recycle

**คำสั่ง:**
```bash
kubectl apply -f 5-persistentvolume.yaml
kubectl get pv
kubectl describe pv pv-data
```

### 6. PersistentVolumeClaim (6-persistentvolumeclaim.yaml)
- Pod ขอ storage ผ่าน PVC
- Kubernetes จับคู่ PVC กับ PV

**คำสั่ง:**
```bash
kubectl apply -f 6-persistentvolumeclaim.yaml
kubectl get pvc
kubectl get pv,pvc
```

### 7. Ingress (7-ingress.yaml)
- จัดการ HTTP/HTTPS routing เข้า Service
- ใช้ Ingress Controller เช่น nginx-ingress, istio

**คำสั่ง:**
```bash
kubectl apply -f 7-ingress.yaml
kubectl get ingress
kubectl describe ingress nginx-ingress
```

### 8. ConfigMap (8-configmap.yaml)
- เก็บ configuration data ที่ไม่ลับ
- สามารถใช้เป็น environment variables หรือ files
- ใช้สำหรับ app configuration, config files

**ประเภทการใช้:**
- Environment variables จาก ConfigMap
- Mount ConfigMap เป็น volume
- envFrom - ใช้ทั้ง ConfigMap

**คำสั่ง:**
```bash
kubectl apply -f 8-configmap.yaml
kubectl get configmap
kubectl describe cm app-config
kubectl get cm app-config -o yaml
```

### 9. Secret (9-secret.yaml)
- เก็บ sensitive data เช่น password, token, API keys
- ค่า base64 encoded (ไม่ใช่ encryption)
- 6 ประเภท: Opaque, docker-cfg, TLS, SSH, basic-auth, service-account-token

**ประเภท Secret:**
- **Opaque**: arbitrary data (default)
- **kubernetes.io/dockercfg**: Docker config
- **kubernetes.io/tls**: TLS certificates
- **kubernetes.io/ssh-auth**: SSH private key
- **kubernetes.io/basic-auth**: Basic authentication

**คำสั่ง:**
```bash
kubectl apply -f 9-secret.yaml
kubectl get secrets
kubectl describe secret app-secrets
# ดูค่า (base64 decoded)
kubectl get secret app-secrets -o jsonpath='{.data.password}' | base64 -d
# สร้าง Secret จาก CLI
kubectl create secret generic my-secret --from-literal=key=value
```

### 10. HPA (10-hpa.yaml)
- HorizontalPodAutoscaler - auto-scaling จำนวน Pod
- ปรับ replica ตามปริมาณ resource usage หรือ custom metrics
- ต้องติดตั้ง Metrics Server ก่อน

**ประเภท Scaling:**
- **Resource Metrics**: CPU, Memory utilization
- **Custom Metrics**: Application-specific metrics
- **External Metrics**: สำหรับ external systems เช่น queue depth

**ความต้องการ:**
- Pod ต้องมี Resource Requests
- Metrics Server ต้องติดตั้งใน cluster
- หากใช้ custom metrics ต้องติดตั้ง Custom Metrics API

**คำสั่ง:**
```bash
# ติดตั้ง Metrics Server (minikube)
minikube addons enable metrics-server

kubectl apply -f 10-hpa.yaml
kubectl get hpa
kubectl describe hpa hpa-cpu
kubectl get hpa -w  # watch HPA status
```

## 🚀 วิธีใช้ทั้งหมด

```bash
# ใช้ทั้งหมดให้ครั้งเดียว
kubectl apply -f .

# ดูทุก resources
kubectl get all
kubectl get pv,pvc,sc
kubectl get cm,secret
kubectl get hpa
```

## 📊 ความสัมพันธ์

```
┌─ StorageClass (SC)
│   ↓
├─ PersistentVolume (PV) ← Dynamic Provisioning
│   ↓
├─ PersistentVolumeClaim (PVC)
│   ↓
├─ Pod / Deployment ← ConfigMap, Secret
│   ↓
├─ HPA (auto-scaling)
│   ↓
├─ Service
│   ↓
└─ Ingress
```

## 📝 บันทึก

- แต่ละไฟล์มีความเป็นอิสระและสามารถใช้ได้แยกกัน
- ปรับเปลี่ยน namespace, names, images ตามต้องการ
- เสมอใช้ `kubectl apply` แทน `kubectl create` เพื่อให้ใช้ได้ซ้ำๆ
- Secret ต้องจัดการด้วยความระมัดระวัง - ไม่ควร commit ใน git

## 🔗 ลิงก์อ้างอิง
- [Kubernetes Official Docs](https://kubernetes.io/docs/)
- [API Reference](https://kubernetes.io/docs/reference/kubernetes-api/)
- [ConfigMap](https://kubernetes.io/docs/concepts/configuration/configmap/)
- [Secret](https://kubernetes.io/docs/concepts/configuration/secret/)
- [HPA](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
