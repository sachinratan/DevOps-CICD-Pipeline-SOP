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

#### Uncordon the worker node instances:
```
$ for node in $(kubectl get nodes --no-headers | "awk '{print $1}'); do kubectl uncordon $node; done
```

#### Patch deployment examples:
- To patch the individual deployment:
```
$ kubectl patch deployments <deployment_name> -n <namespace> -p '{"spec": {"template": {"spec": {"nodeSelector": {"<key>": "<value>"}}}}}' 
```
- To patch all the deployment from within the namespace:
```
$ for deploy in $(kubectl get deployments -o custom-columns=NAME:.metadata.name --no-headers); do kubectl patch deployment $deploy -p '{"spec": {"template": {"spec": {"nodeSelector": {"<key>": "<value>"}}}}}';  done
```
- Note: Replace the namespace, deployment name, and node selector label key value accordingly.

#### Following command measure the first byte response and check for slow DNS resolution that might cause latency:
```
$ curl -kso /dev/null -w "\n===============\n
| DNS lookup: %{time_namelookup}\n
| Connect: %{time_connect}\n
| App connect: %{time_appconnect}\n
| Pre-transfer: %{time_pretransfer}\n
| Start transfer: %{time_starttransfer}\n
| Total: %{time_total}\n
| HTTP Code: %{http_code}\n===============\n" https://<replace_with_destination_service_domain_name>/
```
#### Kubernetes node enable disable scheduling
- Get node name
```
$nodes=$(kubectl get node --no-headers -o custom-columns=NAME:.metadata.name)
```
- Marking the nodes in schedulingDisabled state
```
$for node in $nodes; do echo "Node: $node"; kubectl cordon "$node"; done
```
- Enabling the node for scheduling the pods
```
$for node in $nodes; do echo "Node: $node"; kubectl uncordon "$node"; done
```
