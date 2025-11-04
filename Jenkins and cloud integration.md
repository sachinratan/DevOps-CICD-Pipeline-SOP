Following are steps to integrate the Jenkins master with cloud provider e.g. Amazon EKS

Step 1 : Install required plugin

Go to path `Jenkins >> Manage Jenkins >> Plugins` and install following plugin which will be required for cloud, pipeline, and docker cliemt up.

- Kubernetes plugin
- Pipeline plugin
- Git plugin
- Docker plugin

Step 2 : Configure Cloud:

Go to path `Jenkins >> Manage Jenkins >> Clouds >> New cloud` and give the name to cloud:

![alt text](https://github.com/sachinratan/k8s-dkr-cicd/blob/main/jnks_add_cloud.png)

Configure the cloud with following kubernetes setting:
```
Kubernetes settings:
- Name: kubernetes
- Kubernetes URL: https://kubernetes.default.svc
- Kubernetes Namespace: jenkins
- Jenkins URL: http://jenkins.jenkins.svc.cluster.local
- Jenkins tunnel: jenkins.jenkins.svc.cluster.local:50000
```

![alt text](https://github.com/sachinratan/k8s-dkr-cicd/blob/main/jnks_add_cloud_1.png)

![alt text](https://github.com/sachinratan/k8s-dkr-cicd/blob/main/jnks_add_cloud_3.png)
