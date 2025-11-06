## Following are steps to integrate the Jenkins master with cloud provider e.g. Amazon EKS

#### Pre-requisites : Install required plugin

Go to path `Jenkins >> Manage Jenkins >> Plugins` and install following plugin which will be required for cloud, pipeline, and docker client set up.

- Kubernetes plugin
- Pipeline plugin
- Git plugin
- Docker plugin

### Scenario 1: Jenkins master running on the same k8s cluster:

#### Step 1 : Configure Cloud:

Go to path `Jenkins >> Manage Jenkins >> Clouds >> New cloud` and give the name to cloud:

![alt text](https://github.com/sachinratan/k8s-dkr-cicd/blob/main/miscellaneous-data/jnks_add_cloud.png)

2.a: Configure the cloud with following kubernetes setting:
```
Kubernetes settings:
- Name: kubernetes
- Kubernetes URL: https://kubernetes.default.svc
- Kubernetes Namespace: jenkins
- Jenkins URL: http://jenkins.jenkins.svc.cluster.local
- Jenkins tunnel: jenkins.jenkins.svc.cluster.local:50000
```

![alt text](https://github.com/sachinratan/k8s-dkr-cicd/blob/main/miscellaneous-data/jnks_add_cloud_1.png)

![alt text](https://github.com/sachinratan/k8s-dkr-cicd/blob/main/miscellaneous-data/jnks_add_cloud_3.png)

#### Step 2 : Configure the pod template (optional):

Go to `Jenkins >> Manage Jenkins >> Clouds >> select the configured cloud name (Amazon_EKS) >> Select pod template >> New pod template`

![alt text](https://github.com/sachinratan/k8s-dkr-cicd/blob/main/miscellaneous-data/jnks_add_pod_template.png)

You can configure jenkins agent pod template with following pod configurations

![alt text](https://github.com/sachinratan/k8s-dkr-cicd/blob/main/miscellaneous-data/jnks_add_pod_template_config.png)
![alt text](https://github.com/sachinratan/k8s-dkr-cicd/blob/main/miscellaneous-data/jnks_add_template_config_1.png)
![alt text](https://github.com/sachinratan/k8s-dkr-cicd/blob/main/miscellaneous-data/jnks_add_pod_template_config_2.png)

## Scenario 2: Jenkins master running outside of the k8s cluster:

#### Step 1: Create the clusterrole and clusterrolebinding on target k8s cluster.
```
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: jenkins-role
rules:
- apiGroups: [""]
  resources: ["pods", "services", "deployments", "configmaps", "secrets"]
  verbs: ["create", "delete", "get", "list", "patch", "update", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments", "daemonsets", "replicasets", "statefulsets"]
  verbs: ["create", "delete", "get", "list", "patch", "update", "watch"]
EOF

# Create ClusterRoleBinding
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: jenkins-role-binding
subjects:
- kind: ServiceAccount
  name: jenkins-sa
  namespace: jenkins
roleRef:
  kind: ClusterRole
  name: jenkins-role
  apiGroup: rbac.authorization.k8s.io
EOF
```

#### Step 2 : Create a Secret, which creates a token for the service account.
```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: jnks-sa-token-secret
  namespace: jenkins
  annotations:
    kubernetes.io/service-account.name: jenkins-sa
type: kubernetes.io/service-account-token
```

#### Step 3 : Retrieve the service account secret token.
```
kubectl get secret jnks-sa-token-secret -n jenkins -o jsonpath='{.data.token}' | base64 --decode
```

#### Step 4: Create credential with type `secret text` for cluster accessibility.

![alt text](https://github.com/sachinratan/k8s-dkr-cicd/blob/main/miscellaneous-data/jnks-sa-secret-creds.png)

#### Step 5 : Configure Cloud:

Go to path `Jenkins >> Manage Jenkins >> Clouds >> New cloud` and give the name to the cloud:

![alt text](https://github.com/sachinratan/k8s-dkr-cicd/blob/main/miscellaneous-data/jnks_add_cloud.png)

Next, configure the following details from the target k8s cluster.

```
Name: target-eks-cloud
Kubernetes URL: https://[YOUR-CLUSTER-ENDPOINT]
Kubernetes Namespace: jenkins
Credentials: [Select the new kubeconfig credential]
Jenkins URL: http://jenkins-service.jenkins.svc.cluster.local:8080

Pod Labels:
  jenkins: slave

Pod Template:
  Name: eks-agent
  Namespace: jenkins
  Service Account: jenkins-sa
```

![alt text](https://github.com/sachinratan/k8s-dkr-cicd/blob/main/miscellaneous-data/jnks-add-ext-cloud.png)

![alt text](https://github.com/sachinratan/k8s-dkr-cicd/blob/main/miscellaneous-data/jnks-add-ext-cloud-2.png)

Test the connectivity to cloud (should see `test successful` meesage)

![alt text](https://github.com/sachinratan/k8s-dkr-cicd/blob/main/miscellaneous-data/jnks-cloud-test-success.png)

#### Step 5 : 

With all above configuration, you should be able to deploy to external k8s cluster environment.

References:
- https://devops.stackexchange.com/questions/9074/connect-external-jenkins-master-to-gke-with-kubernetes-plugin
- https://devopscube.com/jenkins-build-agents-kubernetes/
