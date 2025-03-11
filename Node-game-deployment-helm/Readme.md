# Kubernetes Setup with Helm, MongoDB, Redis, and EBS CSI

## Prerequisites
- AWS CLI installed and configured
- kubectl installed and configured for your EKS cluster
- Helm installed
- eksctl installed

---

## 1️⃣ Install EBS CSI Driver for Dynamic Provisioning

### Step 1: Create an IAM Role for EBS CSI
```sh
eksctl create iamserviceaccount \
  --name ebs-csi-controller-sa \
  --namespace kube-system \
  --cluster your-cluster-name \
  --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
  --approve \
  --role-only \
  --role-name AmazonEKS_EBS_CSI_DriverRole
```

### Step 2: Install EBS CSI Driver using Helm
```sh
helm repo add aws-ebs-csi-driver https://kubernetes-sigs.github.io/aws-ebs-csi-driver/
helm repo update
helm install aws-ebs-csi-driver aws-ebs-csi-driver/aws-ebs-csi-driver \
  --namespace kube-system \
  --set serviceAccount.controller.create=false \
  --set serviceAccount.controller.name=ebs-csi-controller-sa
```

---

## 2️⃣ Install MongoDB using Helm
```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

helm install my-mongo bitnami/mongodb \
  --set persistence.storageClass=ebs-sc \
  --set persistence.size=2Gi \
  --set auth.rootPassword=Yudiz@321
```

Check MongoDB pods:
```sh
kubectl get pods
```
Expected output:
```plaintext
NAME                                    READY   STATUS    AGE
my-mongo-mongodb-5f8f65675c-4k59r       1/1     Running   2m
```

MongoDB service name:
```sh
my-mongo-mongodb.default.svc.cluster.local
```

---

## 3️⃣ Install Redis using Helm
```sh
helm install redis-cluster stable/redis \
  --set cluster.slaveCount=3 \
  --set password=password \
  --set securityContext.enabled=true \
  --set securityContext.fsGroup=2000 \
  --set securityContext.runAsUser=1000 \
  --set volumePermissions.enabled=true \
  --set master.persistence.enabled=true \
  --set master.persistence.path=/data \
  --set master.persistence.size=8Gi \
  --set master.persistence.storageClass=ebs-sc \
  --set slave.persistence.enabled=true \
  --set slave.persistence.path=/data \
  --set slave.persistence.size=8Gi \
  --set slave.persistence.storageClass=ebs-sc
```

Check Redis pods:
```sh
kubectl get pods
```
Expected output:
```plaintext
NAME                               READY   STATUS    AGE
redis-cluster-master-0             1/1     Running   2m
redis-cluster-replicas-0           1/1     Running   2m
redis-cluster-replicas-1           1/1     Running   2m
redis-cluster-replicas-2           1/1     Running   2m
```

Redis service names:
- `redis-cluster-master.default.svc.cluster.local` (Read/Write operations, port 6379)
- `redis-cluster-replicas.default.svc.cluster.local` (Read-only operations, port 6379)

---

## 4️⃣ Backend Configuration



Apply these configurations:
```sh
kubectl apply -k .
```

---

## 5️⃣ Debugging
Check if MongoDB is reachable from backend:
```sh
kubectl exec -it backend-XXXX -- sh
nc -zv my-mongo-mongodb 27017
```

Check backend logs:
```sh
kubectl logs deployment/backend -f
```

Check MongoDB logs:
```sh
kubectl logs deployment/my-mongo-mongodb -f
```

---

## ✅ Summary
- Installed EBS CSI for dynamic volume provisioning
- Installed MongoDB using Helm with persistent storage
- Installed Redis with master-replica setup and persistence
- Created necessary Kubernetes Secrets and ConfigMaps
- Verified services and restarted backend for correct configuration

🚀 Your Kubernetes setup is now complete!