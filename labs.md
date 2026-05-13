# Labs — Kubernetes 101

## Lab 1: Pod พื้นฐาน

**วัตถุประสงค์:** สร้าง Pod, ตรวจสอบ logs, exec เข้าไปใน container

```bash
# 1. สร้าง Pod
kubectl apply -f c01/1-pod.yaml

# 2. ดูสถานะ
kubectl get pods
kubectl get pods -o wide

# 3. ดู logs
kubectl logs nginx-pod

# 4. เข้าไปใน container
kubectl exec -it nginx-pod -- sh

# 5. ลบ Pod — สังเกตว่า Pod หายไปถาวร (ไม่สร้างใหม่)
kubectl delete pod nginx-pod
```

**คำถาม:** หลังจากลบ Pod แล้ว Pod จะกลับมาเองไหม? ทดลองรัน `kubectl get pods` อีกครั้ง

---

## Lab 2: Deployment + Service

**วัตถุประสงค์:** สร้าง Deployment, scale, expose ผ่าน Service

```bash
# 1. สร้าง Deployment
kubectl apply -f c01/2-deployment.yaml

# 2. ดูสถานะ
kubectl get deployments
kubectl get pods -l app=nginx

# 3. Scale
kubectl scale deployment nginx-deployment --replicas=5

# 4. สร้าง Service
kubectl apply -f c01/3-service.yaml

# 5. ทดสอบผ่าน port-forward
kubectl port-forward svc/nginx-service 8080:80
# แล้วเปิด http://localhost:8080

# 6. ดู endpoints
kubectl get endpoints
```

**คำถาม:** ถ้า Pod ตัวหนึ่งตาย (ลองลบ Pod ทิ้ง) — Deployment จะสร้างใหม่ให้ไหม?

---

## Lab 3: ConfigMap + Secret

**วัตถุประสงค์:** สร้าง ConfigMap และ Secret แล้ว mount เป็น environment variables

```bash
# 1. สร้าง ConfigMap และ Secret
kubectl apply -f c01/8-configmap.yaml
kubectl apply -f c01/9-secret.yaml

# 2. ดูค่าที่เก็บ
kubectl get cm app-config -o yaml
kubectl get secret app-secrets -o yaml

# 3. สร้าง Pod ที่ใช้ ConfigMap และ Secret
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: config-test-pod
spec:
  containers:
    - name: busybox
      image: busybox
      command: ["sleep", "3600"]
      envFrom:
        - configMapRef:
            name: app-config
        - secretRef:
            name: app-secrets
EOF

# 4. ตรวจสอบว่า env variables เข้ามาแล้ว
kubectl exec config-test-pod -- env | grep -E "APP_|DB_"
```

**คำถาม:** ทำไม Secret ถึงไม่ปลอดภัย 100%? (ดูค่าใน etcd และ base64 decode)

---

## Lab 4: DaemonSet

**วัตถุประสงค์:** สร้าง DaemonSet และสังเกตว่า Pod รันบนทุก Node

```bash
# 1. ตรวจสอบ Nodes ใน Cluster
kubectl get nodes

# 2. สร้าง DaemonSet
kubectl apply -f c02/2-daemonset.yaml

# 3. ดูว่า Pod รันบนทุก Node
kubectl get pods -o wide -l app=fluentd
# จำนวน Pod ควรเท่ากับจำนวน Nodes

# 4. ตรวจสอบว่าใช้ hostPath volume
kubectl describe pod -l app=fluentd | grep -A5 Mounts
```

**คำถาม:** ถ้ามี Node ใหม่ joins cluster — DaemonSet จะทำอะไร? ต้องกำหนด replicas ไหม?

---

## Lab 5: StatefulSet

**วัตถุประสงค์:** สร้าง StatefulSet และสังเกต stable identity + PVC

```bash
# 1. ต้องมี StorageClass ก่อน (ถ้าไม่มี — skipp ได้)
kubectl apply -f c01/4-storageclass.yaml

# 2. สร้าง StatefulSet
kubectl apply -f c02/3-statefulset.yaml

# 3. สังเกตชื่อ Pod — มีลำดับแน่นอน
kubectl get pods -l app=web
# web-statefulset-0, web-statefulset-1, web-statefulset-2

# 4. ดู PVC ที่สร้างอัตโนมัติ
kubectl get pvc
# www-web-statefulset-0, www-web-statefulset-1, www-web-statefulset-2

# 5. ทดสอบ stable network identity
kubectl run tmp --image=busybox -it --rm --restart=Never -- \
  nslookup web-statefulset-0.web-headless.default.svc.cluster.local
```

**คำถาม:** ถ้าลบ Pod web-statefulset-1 แล้ว Pod ใหม่จะชื่ออะไร? ลบ PVC ด้วยหรือเปล่า?

---

## Lab 6: HPA (Auto Scaling)

**วัตถุประสงค์:** สร้าง HPA และทดสอบ auto scaling

**ข้อกำหนด:** ต้องมี Metrics Server — `minikube addons enable metrics-server`

```bash
# 1. สร้าง Deployment (ต้องมี resource requests)
kubectl apply -f c01/2-deployment.yaml

# 2. สร้าง HPA
kubectl apply -f c01/10-hpa.yaml

# 3. ดูสถานะ HPA
kubectl get hpa
kubectl describe hpa hpa-cpu

# 4. สร้างโหลด — ดูว่า HPA scale หรือไม่
kubectl run -it load-generator --image=busybox -- sh
# ใน container: while true; do wget -q -O- http://nginx-service; done

# 5. ดู HPA แบบ real-time
kubectl get hpa -w
```

**คำถาม:** CPU utilization เท่าไหร่ถึงจะ trigger scale up? scale down ใช้เวลากี่นาที?

---

## Lab 7: เปรียบเทียบ Deployment vs DaemonSet vs StatefulSet

**วัตถุประสงค์:** รันทั้ง 3 ตัวพร้อมกันและเปรียบเทียบพฤติกรรม

```bash
# 1. สร้างทั้งหมด
kubectl apply -f c02/

# 2. ตรวจสอบความแตกต่าง
kubectl get pods -o wide

# 3. ลบ Pod ทีละตัวและสังเกต
PODS=$(kubectl get pods -o name)
for pod in $PODS; do
  echo "=== ลบ $pod ==="
  kubectl delete $pod --now
  sleep 2
  kubectl get pods
  echo ""
done
```

**คำถาม:** หลังจากลบ Pod — ตัวไหนสร้างใหม่? ตัวไหนไม่สร้าง? ตัวไหนชื่อเท่าเดิม?
