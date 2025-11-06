## Following are the step-by-step guide to deploy Jenkins master on an Amazon EKS cluster

### Step 1: Create a Namespace for Jenkins:
```
$ kubectl create namespace jenkins
 ```   
### Step 2: Create Jenkins ServiceAccount, ClusterRole, and ClusterRoleBinding:
```
$ kubectl apply -f - <<EOF
# jenkins-sa.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: jenkins
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: jenkins-admin
rules:
  - apiGroups: [""]
    resources: ["*"]
    verbs: ["*"]
  - apiGroups: ["apps"]
    resources: ["*"]
    verbs: ["*"]
  - apiGroups: ["batch"]
    resources: ["*"]
    verbs: ["*"]
  - apiGroups: ["extensions"]
    resources: ["*"]
    verbs: ["*"]
  - apiGroups: ["networking.k8s.io"]
    resources: ["*"]
    verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: jenkins-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: jenkins-admin
subjects:
- kind: ServiceAccount
  name: jenkins
  namespace: jenkins
EOF
```
## Step 3: Create a PersistentVolumeClaim for Jenkins:
```
$ kubectl apply -f - <<EOF
# jenkins-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jenkins-pvc
  namespace: jenkins
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: gp2
  resources:
    requests:
      storage: 20Gi
EOF
```
### Step 4: Create Jenkins Deployment:
```
$ kubectl apply -f - <<EOF
# jenkins-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
  namespace: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      serviceAccountName: jenkins
      securityContext:     # Add security context at pod level
        fsGroup: 1000     # Jenkins default group ID
        runAsUser: 1000   # Jenkins default user ID
      containers:
      - name: jenkins
        image: jenkins/jenkins:lts
        securityContext:   # Add security context at container level
          runAsUser: 1000
          allowPrivilegeEscalation: false
        ports:
        - containerPort: 8080
          name: http
        - containerPort: 50000
          name: jnlp
        volumeMounts:
        - name: jenkins-home
          mountPath: /var/jenkins_home
        livenessProbe:
          httpGet:
            path: /login
            port: 8080
          initialDelaySeconds: 90
          timeoutSeconds: 5
        readinessProbe:
          httpGet:
            path: /login
            port: 8080
          initialDelaySeconds: 60
          timeoutSeconds: 5
        resources:
          requests:
            cpu: "500m"
            memory: "1Gi"
          limits:
            cpu: "2000m"
            memory: "4Gi"
      volumes:
      - name: jenkins-home
        persistentVolumeClaim:
          claimName: jenkins-pvc
EOF
```
### Step 5: Create Jenkins Service:
```
$ kubectl apply -f - <<EOF
# jenkins-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: jenkins
  namespace: jenkins
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: 8080
      protocol: TCP
      name: http
    - port: 50000
      targetPort: 50000
      protocol: TCP
      name: jnlp
  selector:
    app: jenkins
EOF
```
## Step 6: For ALB setup, create Ingress:
```
$ kubectl apply -f - <<EOF
# jenkins-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: jenkins-ingress
  namespace: jenkins
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: jenkins
                port:
                  number: 80
EOF
```
### Step 7: Replace with jenkins manager pod name and fetch Jenkins login password
```
$ kubectl exec -it pod/<JENKINS-MANAGER-POD-NAME> -n jenkins -- cat /var/jenkins_home/secrets/initialAdminPassword

OR

$ kubectl exec -it $(kubectl get pods -n jenkins -l app=jenkins -o jsonpath='{.items[0].metadata.name}') -n jenkins -- cat /var/jenkins_home/secrets/initialAdminPassword
```
#### Note: Given that the Jenkins uses the Persistent Volume Claim for dynamic storage and ingress to expose the jenkins over public internet, you would require to install the Amazon EBS CSI driver add-on and AWS Load Balancer Controller to provision the EBS volume and Application or Network Load Balancer.

#### References:
- https://github.com/aws-samples/jenkins-cloudformation-deployment-example/blob/main/k8s/manifest.yaml - Jenkins deployment manifest
- https://docs.aws.amazon.com/eks/latest/userguide/lbc-helm.html - Install AWS Load Balancer Controller with Helm
- https://docs.aws.amazon.com/eks/latest/eksctl/installation.html - Installation options for Eksctl
- https://helm.sh/docs/intro/install/ - Installing Helm
