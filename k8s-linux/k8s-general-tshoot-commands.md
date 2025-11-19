## This guide will provide the useful Kubernetes general troubleshooting related commands.

#### Debug Pod Approach (Create a debug pod with network access to the node): 
```
$ kubectl debug node/<node-name> -it --image=<debug-image> -- /bin/sh
```
#### To verify the IRSA set up and configuration and confirm that the IRSA set up works perfectly without any issue.
```
$ kubectl apply -f - << EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: irsa-lab-sa
  namespace: irsa-lab
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::${AWS_ACCOUNT_ID}:role/${IRSA_LAB_ROLE}

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: irsa-lab-deployment
  namespace: irsa-lab
spec:
  selector:
    matchLabels:
      app: irsa-lab-deployment
  replicas: 1
  template:
    metadata:
      labels:
        app: irsa-lab-deployment
    spec:
      serviceAccountName: irsa-lab-sa
      containers:
      - name: eks-iam-test
        image: amazon/aws-cli:latest
        command: [ "/bin/sh", "-c", "--" ]
        args: [ "while true; do sleep infinity; done;" ]
EOF
```

#### Attach the debug container and run the troubleshooting command:
```
$ kubectl debug -it -n kube-system --image=aws_cli:latest <controller_pod_name> -n kube-system -- bash
```
