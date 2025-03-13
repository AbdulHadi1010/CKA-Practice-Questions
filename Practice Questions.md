# Kubernetes Tasks

## Task 1: Create and Assign a PVC

### Requirements
- Create a PVC `my-pvc` with capacity `10Gi` and storage class `k8s-csi-plugin`.
- Assign the PVC to a pod named `nginx-pod` with image `nginx` and mount to `/usr/share/html`.
- Ensure the pod claims the volume with `ReadWriteMany` access mode.
- Use `kubectl patch` or `kubectl edit` to update the PVC capacity to `70Gi`.

### Solution
#### Create a PVC
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
  storageClassName: k8s-csi-plugin
```

#### Assign PVC to a Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
        - mountPath: /usr/share/html
          name: task-pv-storage
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: my-pvc
```

> **Note:** Volumes are immutable and cannot be changed while the pod is running.

#### Resize the PVC
```sh
kubectl patch pvc my-pvc -p '{"spec":{"resources":{"requests":{"storage":"70Gi"}}}}'
```

---

## Task 2: Create a Service Account and Cluster Role

### Requirements
- Create a service account `my-sa` in namespace `my-ns`.
- Create a cluster role `new-cluster-role` that can create and list:
  - DaemonSets
  - Deployments
  - ReplicaSets
  - Pods
- Bind the role to `my-sa`.

### Solution
#### Create the Service Account
```sh
kubectl create serviceaccount my-sa -n my-ns
```

#### Create the Cluster Role
```sh
kubectl create clusterrole new-cluster-role --verb=create,list --resource=daemonsets,deployments,replicasets,pods
```

#### Bind the Cluster Role
```sh
kubectl create clusterrolebinding new-cluster-role-binding --clusterrole=new-cluster-role --serviceaccount=my-ns:my-sa
```

---

## Task 3: Monitor CPU Usage

### Requirement
Find the pod with the highest CPU usage (label: `name=cpu-burner`) and write its name to `/tmp/cpu.txt`.

### Solution
```sh
kubectl top pods -l name=cpu-burner --no-headers=true | head -n 1 | awk '{print $1}' > /tmp/cpu.txt
```

---

## Task 4: Schedule a Pod with Node Selector

### Requirement
- Create a pod named `nginx01` with image `nginx`.
- Schedule it on a node labeled `name=node`.

### Solution
#### Create the Pod Manifest
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx01
spec:
  containers:
    - name: nginx
      image: nginx
  nodeSelector:
    name: node
```

#### Apply the Manifest
```sh
kubectl apply -f pod.yaml
```

---

## Task 5: Create an Ingress Resource

### Requirement
- Create an `nginx-ingress` resource in `ingress-ns`.
- Expose `me.html` on `/me.html` using service port `8080`.
- Expose `test` service on `/test` using service port `8080`.

### Solution
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  namespace: ingress-ns
spec:
  rules:
    - http:
        paths:
          - path: /me.html
            pathType: Prefix
            backend:
              service:
                name: me
                port:
                  number: 8080
          - path: /test
            pathType: Prefix
            backend:
              service:
                name: test
                port:
                  number: 8080
```

---

## Task 6: Create a Network Policy

### Requirement
- Allow only pods in `internal` namespace on port `9200` to communicate with `namespace-netpol`.
- Block all other communication.

### Solution
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: k8s-netpol
  namespace: namespace-netpol
spec:
  podSelector:
    matchLabels: {}
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: internal
      ports:
        - protocol: TCP
          port: 9200
  policyTypes:
    - Ingress
```

---

## Task 7: Monitor Logs

### Requirement
Monitor logs of pod `loggy` and extract lines containing `issue-not-found`.

### Solution
```sh
kubectl logs loggy | grep "issue-not-found" > /tmp/pod.txt
```

---

## Task 8: Deployment and Rollback

### Requirement
- Create a `nginx` deployment with 3 replicas using `nginx:1.11-alpine`.
- Update the image to `nginx:1.13-alpine`.
- Rollback to the previous version.

### Solution
```sh
kubectl create deployment nginx --image=nginx:1.11-alpine --replicas=3
kubectl set image deployment/nginx nginx=nginx:1.13-alpine --record
kubectl rollout status deployment/nginx
kubectl rollout undo deployment/nginx
```

---

## Task 9: Create a Persistent Volume

### Requirement
- Create a PV `my-vol` with `2Gi` capacity.
- Access mode: `ReadWriteOnce`.
- Volume type: `hostPath` (`/path/to/file`).

### Solution
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-vol
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: '/path/to/file'
```

---

## Task 10: Upgrade Control Plane

### Requirement
- Upgrade from `1.20.0` to `1.20.1`.
- Drain the master before upgrading and uncordon after completion.

### Solution
```sh
kubectl drain controlplane --ignore-daemonsets
kubeadm upgrade apply v1.20.1
kubectl uncordon controlplane
```

---

## Task 11: Count Ready Nodes

### Requirement
Count the number of ready nodes excluding those tainted `NoSchedule`.

### Solution
```sh
kubectl get nodes --no-headers | grep -v 'NoSchedule' | wc -l > /path/to/node
```

---


