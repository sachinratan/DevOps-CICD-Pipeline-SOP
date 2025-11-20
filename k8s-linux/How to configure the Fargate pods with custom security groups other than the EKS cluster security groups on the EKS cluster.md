## This guide will help you with 'How to configure the Fargate pods with custom security groups other than the EKS cluster security groups on the EKS cluster'

### In order to configure the Fargate pods with the custom security group along with the EKS cluster default security groups. Please refer to the following steps of instructions:.

#### Pre-requisite: Active EKS cluster

#### Create the Fargate profile, if you have existing you can skip this step.
```
$  aws eks create-fargate-profile --fargate-profile-name "custom-sg-pod-profile" --cluster-name "sandbox-eks-cluster" --pod-execution-role-arn "arn:aws:iam::<AWS_ACCOUNT_NO>:role/eksctl-eks-fargate-cluster-FargatePodExecutionRole-5N4ASBDE5AU6" --subnets "subnet-xxxxxxxxxxx" "subnet-xxxxxxxxxx" --selectors "namespace=custom-sg"
```

#### Following the Fargate profile creation, create the custom security group for Fargate pods.
```
$ aws ec2 create-security-group --group-name "custom-fargate-pod-sg" --description "Security group for eks Fargate pod" --vpc-id "vpc-xxxxxxxxxx"
```

#### Create the namespace which is configured in the Fargate profile selector, you can skip this step, if the Fargate profile selector namespace already been created.
```
$ kubectl create ns custom-sg
```

#### In order to attach the custom security group to EKS Fargate pods, we will need the Security Group Policy CRD [3]. Let's deploy the Security Group Policy CRD.
```
$ kubectl apply -f - <<EOF
apiVersion: vpcresources.k8s.aws/v1beta1
kind: SecurityGroupPolicy
metadata:
   name: pod-sg-policy
   namespace: custom-sg
spec:
   podSelector: {}
   securityGroups:
     groupIds:
       - sg-xxxxxxxxxxxx    # Pod security group we created earlier
       - sg-xxxxxxxxxxxx    # Cluster security group
EOF
```
Note: When using a security group with a Fargate pod, it is necessary that the pod must communicate with the EKS control plane; hence, you must attach the cluster default security group along with custom security group. Also, leaving the podSelector blank, as done in the above example, attaches the custom security group(s) to all the pods launched in that namespace. Moreover, you can also attach security group(s) to the pod within the namespace based on label(s).

#### Lastly, deploy the application pod in the Fargate profile namespace where we have the Security Group Policy CRD deployed.
```
$ kubectl create deployment nginx --image=nginx --replicas=2 -n custom-sg
```

#### Once Fargate pods become healthy, with use of Fargate pod IP address you can check the Fargate pod ENI associted security groups in EC2 console.
```
- Amazon EC2 console >> Network Interfaces >> Search the Fargate Pod IP and select the network interface >> Review the details of attached Security Groups.
```
