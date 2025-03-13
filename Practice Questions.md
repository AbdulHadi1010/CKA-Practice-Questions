## Task 1:
Create a PVC my-pvc with the capacity 10Gi and storage class k8s-csi-plugin.

• Assign the PVC to the pod named nginx-pod with image nginx and mount to path /usr/share/html • Ensure the pod claims the volume as ReadWriteMany access mode • Use kubectl patch or kubectl edit to update the capacity of the PVC as 70Gi to record the change.

### Solution:

- Create a PVC.
- Create a pod manifest file and add volumes and volume mounts inside it.
Add the following in proper places:
```yaml
volumes:
  ...
 - name: task-pv-storage
    persistentVolumeClaim:
      claimName: my-pv-claim
containers:
  ...
  volumeMounts:
    - mountPath: /usr/share/html
      name: task-pv-storage
  ...
```

> [!NOTE]
> Volumes are immutable and cannot be changed while the POD is in running state.

- If a PV is not created automatically create a PV using manifest file.

### Resizing the volume:

- Delete the existing pod.
- Edit the `pvc.yaml` file and edit the ``spec->resource->request->storage`` value.
- Create the pod again.
#### Examples of imperative commands:
 
 Creating a PVC:
```bash
kubectl create pvc my-pvc --access-modes=ReadWriteMany --resources=requests.storage=10Gi --storage-class=manual
```

## Task 2:

Create a service account my-sa in new namespace my-ns.

Create a cluster role with the name new-cluster-role and ensure the role only can create and list below resources.

• DaemonSets • Deployments • Replicaset • Pods

Ensure only the newly created service account can use the role and it is effective with the name space my-ns.

```bash
To create a new SA:

$ kubectl create serviceaccount my-sa -n my-ns

Verification:
$ kubectl get sa -n my-ns

Verification:
$ kubectl create clusterrole new-cluster-role --verb=create,list --resource=daemonsets,deployments,replicaset,pods -n my-ns --dry-run -o yaml 

To create ClusterRole
$ kubectl create clusterrole new-cluster-role --verb=create,list --resource=daemonsets,deployments,replicaset,pods -n my-ns

Verification:
$ kubectl create clusterrolebinding new-cluster-role-binding --clusterrole=new-cluster-role --serviceaccount=default:my-sa  -n my-ns --dry-run -o yaml

To create ClusterRoleBinding
$  kubectl create clusterrolebinding new-cluster-role-binding --clusterrole=new-cluster-role --serviceaccount=default:my-sa -n my-ns
```

## Task 3:

From the pod label name=cpu-burner, find pods running high CPU workloads and Write the name of the pod consuming most CPU to the file /tmp/cpu.txt.


```bash
kubectl top pods -l name=cpu-burner -n kube-system --no-headers=true | head -n 1 | awk '(print $1)' > /tmp/cpu.txt

(or)

kubectl top pods -l name=cpu-burner -n kube-system 

echo '<podname>' >> /tmp/cpu.txt
```

## Task 4:


Schedule a pod as follows Name :- nginx01 image :- nginx Node Selector :- name=node.

### Solution:

- Create a nginx pod:
```bash
kubectl run nginx01 --image=nginx --port=80 --dry-run=client -o yaml > pod.yaml 
```
- Edit the pod manifest file:
```bash
vim pod.yaml
```
- The manifest file should look like this:
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
- Execute the pod.
```bash
kubectl apply -f pod.yaml
```


## Task 5:

Create a new nginx Ingress resource as follows: o Name: nginx-ingress o Namespace: ingress-ns o Exposing service me.html on path /me.html using service port 8080 o Exposing service test on path /test using service port 8080

#### Solution:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
spec:
  defaultBackend:
    resource:
      apiGroup: k8s.example.com
      kind: StorageBucket
      name: static-assets
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

## Task 6:

Create a NetworkPolicy named k8s-netpol in the namespace namespace-netpol in a way that pods running on namespace internal on port 9200 can only access pods running in namespace-netpol. a. Allow the pods to communicate if they are running on port 9200 within the namespace b. Ensure the NetworkPolicy doesn’t allow other pods that are running other than port 9200 c. The communication from and to the pods running on port 9200 d. No pods running on port 9200 from other name spaces to allowed

NB: There will be a default deny all from any namespace netpol will be already present.


## Task 7:

Monitor the logs of pod loggy and extract log lines issue-not-found. Write the output to 
/tmp/pod.txt. 

````bash
kubectl logs loggy | grep "issue-not-found" > /tmp/pod.txt
````


## Task 8:

Create a deployment and perform rollback.

Name: nginx Using container nginx with version 1.11-alpine The deployment should contain 3 replicas Next, deploy the app with new version 1.13-alpine by performing a rolling update and record that update. Finally, rollback that update to the previous version 1.11-alpine


```bash
# Create the deployment
kubectl create deployment nginx --image=nginx:1.11-alpine --replicas=3

# Update the image to version 1.13-alpine and record the update
kubectl set image deployment/nginx nginx=nginx:1.13-alpine --record

# Check the rollout status to ensure the rolling update is successful
kubectl rollout status deployment/nginx

# Rollback to the previous version if needed
kubectl rollout undo deployment/nginx

```

## Task 8.1:

Scale the deployment learning to 3 pods. 

```bash
kubectl scale deployment learning --replicas=3
```


## Task 9:

Create a persistent Volume with name my-vol, of capactiy 2Gi and access mode ReadWriteOnce. The type of volume is hostpath and its location is /path/to/file


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

## Task 10:

Upgrade master control plane components from version 1.20.0 to only version 1.20.1. 
• Drain the master before start the upgrade and uncordn once the upgrade is completed 
• Update the kubelet and kubeadm as well

```bash
To upgrade the kubeadm:

$ apt-get update && sudo apt-mark unhold kubeadm && apt-get install -y kubeadm=1.21.x-00
$ kubeadm version
$ kubeadm upgrade plan
$ kubectl drain controlplane
$ sudo kubeadm upgrade plan
$ sudo kubeadm upgrade apply v1.21.1
$ kubectl uncordon controlpane 

To update kubelet:

$ apt-get update && sudo apt-mark unhold kubelet && apt-get install -y kubelet=1.21.x-00
$ sudo systemctl daemon-reload
$ sudo systemctl restart kubelet
```

## Task 11:

Check to see how many nodes are ready (not including nodes tainted NoSchedule) and write the number to /path/to/node

```bash
kubectl describe nodes | grep Taints | grep NoSchedule | wc -l >>/path/to/node
```

## Task 12:

Set configuration context 

```bash
$ kubectl config use-context k8s 
```

Scale the deployment webserver to 6 pods

```bash
kubectl scale deployment webserver --replicas=6
```


## Task 13:

Create a pod named kucc8 with a single app container for each of the following images running inside (there may be between 1 and 4 images specified): nginx + redis + memcached.


```bash
kubectl run kucc4 --image=nginx --dry-run=client -o yaml > pod.yaml
vim pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: kucc4
  name: kucc4
spec:
  containers:
  - image: nginx # copy and add 2 times
    name: nginx  # change name, copy and add 2 times
  - image: redis
    name: redis
  - image: memcached
    name: memcached
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

```


## Task 14:

Set the node labelled with name:node-01 as unavailable and reschedule all the pods running on it.

```bash
kubectl get nodes
kubectl drain node-01 --ignore-daemonsets=true --delete-local-data=true --force=true
```

## Task 15:

Name: jenkins Using image: jenkins In a new Kubenetes namespace named tools

```bash
kubectl create ns tools
kubectl run jenkins --imagejenkins -n tools
```

## Task 16:

Create a Static Pod with:
Name: consul 
Using image: consul 
In a new Kubenetes namespace named tools

```bash
kubectl run consul --image=consul --dry-run=client -n tools -o yaml > pod.yaml
```

```bash
Move the pod in the Kuberenetes Manifest file
mv pod.yaml /etc/Kubernetes/Manifest
vim pod.yaml
# Save and exist the pod will be created based on the scheduleder automatically.
```

## Task 17:

A Kubernetes worker node, labelled with name "node-01" is in state NotReady. Investigate why this is the case, and perform any appropriate steps to bring the node to a Ready state, ensuring that any changes are made permanent.

```bash
kubectl get nodes
ssh node-01
sudo -i
systemctl status kubelet
systemctl start kubelet
systemctl enable kubelet
```

Here are all the ConfigMap-related questions and answers:

---

## Task 18:

You create a ConfigMap named `myconfigmap` with the following data:
- `APP_ENV`: production
- `DB_HOST`: db.example.com  
Verify that the ConfigMap was created and check its contents.

**Answer:**  
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap
data:
  APP_ENV: production
  DB_HOST: db.example.com
```

Commands:
```bash
kubectl apply -f configmap.yaml
kubectl describe configmap myconfigmap
```

---

## Task 19:

Create a Pod that uses the `myconfigmap` created earlier as environment variables. Ensure the environment variables from the ConfigMap are set correctly in the container.

**Answer:**  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
  - image: nginx
    name: configmap-pod
    envFrom:
      - configMapRef:
          name: myconfigmap
```

Commands:
```bash
kubectl exec configmap-pod -- env
```

---

## Task 20:

Create a ConfigMap named `web-config` containing two key-value pairs:  
- `index.html`: Basic HTML content  
- `error.html`: Simple error message  
Mount these values as files in the `/usr/share/nginx/html` directory of an NGINX container running in a Pod named `nginx-pod`. Verify the files.

**Answer:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: web-config
data:
  index.html: |
    <html><body><h1>Welcome</h1></body></html>
  error.html: |
    <html><body><h1>Error</h1></body></html>
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - image: nginx
    name: nginx
    volumeMounts:
    - name: html-config
      mountPath: /usr/share/nginx/html
  volumes:
  - name: html-config
    configMap:
      name: web-config
```

---

## Task 21:

Modify the `myconfigmap` ConfigMap to set `APP_ENV` to `staging` and verify that the Pod's environment variables reflect this change.

**Answer:**  
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap
data:
  APP_ENV: staging
  DB_HOST: db.example.com
```

Commands:
```bash
kubectl exec -it configmap-pod -- env
```

---

## Task 22:

Create a ConfigMap named `script-config` with a key `init-script.sh` containing a multi-line shell script. Mount it in a Pod and ensure the script runs when the container starts.

**Answer:**  
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: script-config
data:
  init-script.sh: |
    #!/bin/sh
    echo "Initializing app"
    export APP_READY=true
---
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - image: busybox
    name: app-container
    command: ["/bin/sh", "-c", "sh /etc/config/init-script.sh && tail -f /dev/null"]
    volumeMounts:
    - name: script-volume
      mountPath: /etc/config
  volumes:
  - name: script-volume
    configMap:
      name: script-config
```

---

## Task 23:

Use a Secret for sensitive data like passwords. Create a Secret and pass it to a Pod via environment variables.

**Answer:**  
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: dotfile-secret
data:
  password: $(echo -n "Abc@123" | base64)
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: mycm
data:
  APP_ENV: production-ready
---
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
  - image: nginx
    name: configmap-pod
    envFrom:
      - configMapRef:
          name: mycm
    env:
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: dotfile-secret
            key: password
```

Commands:
```bash
kubectl exec -it configmap-pod -- env
```

---

## Task 24:

Create a NetworkPolicy that allows only Pods with the label `app: frontend` to connect to Pods labeled `app: backend` on port 8080. All other traffic should be blocked.

**Answer:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: frontend
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: backend
    ports:
    - protocol: TCP
      port: 8080
```

---

## Task 25:

Create a NetworkPolicy that denies all ingress traffic to Pods labeled `app: database` in the `production` namespace. Ensure that egress is still allowed.

**Answer:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
  - Ingress
  # Egress is implicitly allowed if not restricted
```

---

## Task 25:

Create a NetworkPolicy that restricts all Pods in the `web` namespace to only send traffic to IP addresses in the range `10.0.0.0/16` on TCP port 80. All other egress traffic should be denied.

**Answer:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-egress
  namespace: web
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/16
    ports:
    - protocol: TCP
      port: 80
```

---

## Task 28:

Write a NetworkPolicy that ensures the Pod labeled `app: isolated` in the `dev` namespace is fully isolated and cannot communicate with any other Pods in the cluster, nor receive any inbound traffic.

**Answer:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: isolate-pod
  namespace: dev
spec:
  podSelector:
    matchLabels:
      app: isolated
  policyTypes:
  - Ingress
  - Egress
  # No ingress or egress rules are specified, so all traffic is denied
```

---

## Task 29:

Create a NetworkPolicy that allows all Pods in the `frontend` namespace to communicate with any Pod in the `backend` namespace on port 8080, while blocking all other traffic.

**Answer:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: frontend
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: backend
    ports:
    - protocol: TCP
      port: 8080
```

---

## Task 30:

What is the simplest way to define a NetworkPolicy that denies all ingress and egress traffic by default for all Pods in a specific namespace?

**Answer:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: specific-namespace
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

---

## Task 31:

You have created a NetworkPolicy to restrict ingress to certain Pods. How would you use a busybox Pod to test whether your policy is working as expected? What command would you run to test access between two Pods in different namespaces?

**Answer:**
1. **Create a busybox Pod in the testing namespace:**
   ```bash
   kubectl run busybox --image=busybox --namespace=testing -- sleep 3600
   ```
2. **Get the IP address of the busybox Pod:**
   ```bash
   kubectl get pod busybox -o wide --namespace=testing
   ```
3. **Test connectivity from the busybox Pod:**
   ```bash
   kubectl exec -it busybox --namespace=testing -- wget -qO- http://<target-pod-ip>:8080
   ```

---

## Task 31: 
Design a NetworkPolicy that allows DNS traffic (UDP on port 53) from any Pod to the DNS service in the `default` namespace but blocks all other traffic.

**Answer:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-traffic
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0
    ports:
    - protocol: UDP
      port: 53
  # Ingress policy is not defined, so all ingress traffic is blocked by default
```

---
