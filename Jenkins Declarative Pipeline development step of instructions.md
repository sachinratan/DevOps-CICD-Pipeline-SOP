# Step of instructions to create the Jenkins declarative pipeline

Create the clusterrole for Jenkins dynamic slave to deploy the resources on k8s cluster:

```
kubectl apply -f - <<EOF
> ---
> apiVersion: v1
> kind: ServiceAccount
> metadata:
>   name: jenkins-deployer
>   namespace: jenkins
> ---
> apiVersion: rbac.authorization.k8s.io/v1
> kind: ClusterRole
> metadata:
>   name: jenkins-deployer-cluster-role
> rules:
> - apiGroups: [""]
>   resources: ["pods", "services", "secrets", "configmaps", "namespaces"]
>   verbs: ["create", "delete", "get", "list", "patch", "update", "watch"]
> - apiGroups: ["apps"]
>   resources: ["deployments", "replicasets", "statefulsets"]
>   verbs: ["create", "delete", "get", "list", "patch", "update", "watch"]
> - apiGroups: ["extensions"]
>   resources: ["deployments"]
>   verbs: ["create", "delete", "get", "list", "patch", "update", "watch"]
> ---
> apiVersion: rbac.authorization.k8s.io/v1
> kind: ClusterRoleBinding
> metadata:
>   name: jenkins-deployer-cluster-role-binding
> subjects:
> - kind: ServiceAccount
>   name: jenkins-deployer
>   namespace: jenkins
> roleRef:
>   kind: ClusterRole
>   name: jenkins-deployer-cluster-role
>   apiGroup: rbac.authorization.k8s.io
> EOF
serviceaccount/jenkins-deployer created
clusterrole.rbac.authorization.k8s.io/jenkins-deployer-cluster-role created
clusterrolebinding.rbac.authorization.k8s.io/jenkins-deployer-cluster-role-binding created
```

Create the pipeline with following Jenkinsfile:
```
pipeline {
    agent {
         kubernetes {
            cloud 'Amazon_EKS'
            yaml '''
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins: agent
spec:
  serviceAccountName: jenkins-deployer
  containers:
  - name: dind
    image: docker:24.0.7-dind    # Specific version of Docker daemon
    securityContext:
      privileged: true
    env:
    - name: DOCKER_TLS_CERTDIR
      value: ""
    - name: DOCKER_HOST
      value: "tcp://localhost:2375"
    ports:
    - containerPort: 2375
    volumeMounts:
    - name: dind-storage
      mountPath: /var/lib/docker
  - name: docker-client
    image: docker:24.0.7          # Same version as dind
    env:
    - name: DOCKER_HOST
      value: "tcp://localhost:2375"
    command:
    - cat
    tty: true
  volumes:
  - name: dind-storage
    emptyDir: {}
'''
    }
  }
    environment {
        REPO_URL = '<AWS_ACC_NO>.dkr.ecr.eu-central-1.amazonaws.com'
        BRANCH = 'main' // Or your desired branch
        ECR_REPO_NAME = 'git-jenkins-pipeline-repo'
        AWS_ACCOUNT_ID = '<AWS_ACC_NO>'
        AWS_DEFAULT_REGION = 'eu-central-1'
        K8S_CLUSTER_NAME = 'sandbox-env-cluster'
        K8S_NAMESPACE = 'default'
        K8S_DEPLOYMENT_NAME = 'app-deployment'
        IMAGE_NAME = 'my-app'
        IMAGE_TAG = 'latest'
        KUBECTL_VERSION = 'v1.34.1'
        AWS_REGION = 'eu-central-1'
    }

    parameters {
        choice(name: 'CLOUD_ENVIRONMENT', choices: ['aws-agent', 'gcp-agent', 'azure-agent'], description: 'Select the cloud environment for the Jenkins agent')
        string(name: 'IMAGE_TAG', defaultValue: "${env.BUILD_NUMBER}", description: 'Docker image tag (defaults to build number)')
    }
    stages {
        stage('Wait for Docker Daemon') {
            steps {
                container('docker-client') {
                    sh '''
                    # Wait for Docker daemon to be ready
                    echo "Waiting for Docker daemon..."
                    until docker info > /dev/null 2>&1; do
                      sleep 1
                    done
                    echo "Docker daemon is ready"
                    '''
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                container('docker-client') {
                    sh '''
                    # Install required packages
                    apk add --no-cache \
                        curl \
                        aws-cli \
                        bash \
                        jq

                    # Install kubectl
                    curl -LO "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl"
                    chmod +x kubectl
                    mv kubectl /usr/local/bin/

                    # Verify installations
                    echo "Docker version:"
                    docker --version
                    echo "AWS CLI version:"
                    aws --version
                    echo "Kubectl version:"
                    kubectl version --client
                    '''
                }
            }
        }
        stage('Clone GitHub Repo') {
            steps {
                container('docker-client') {
                  git url: 'https://github.com/sachinratan/SLRS-Admin-DevOps-Integration-Website.git', branch: 'main'
                  sh 'ls -l'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                container('docker-client') {
                    script {
                        sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                    }
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                container('docker-client') {
                    script {
                        withAWS(credentials: 'srs-cli-creds', region: env.AWS_DEFAULT_REGION) {
                            sh """
                            aws ecr get-login-password --region ${env.AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_DEFAULT_REGION}.amazonaws.com
                            docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_DEFAULT_REGION}.amazonaws.com/${env.ECR_REPO_NAME}:${params.IMAGE_TAG}
                            docker push ${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_DEFAULT_REGION}.amazonaws.com/${env.ECR_REPO_NAME}:${params.IMAGE_TAG}
                            """
                        }
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                container('docker-client') {
                    script {
                        withAWS(credentials: 'srs-cli-creds', region: env.AWS_REGION) {
                            try {
                                timeout(time: 10, unit: 'MINUTES') {
                                    retry(1) {
                                        sh '''
                                        aws eks update-kubeconfig --name ${env.K8S_CLUSTER_NAME} -region ${env.AWS_DEFAULT_REGION}
                                        kubectl apply -f k8s-deployment.yaml
                                        kubectl apply -f k8s-service.yaml
                                        '''
                                    }
                                }
                            } catch (Exception e) {
                                sh 'kubectl rollout undo deployment/myapp-deployment -n default'
                                throw e
                            }
                        }
                    }
                }
            }
        }
    }
        //stage('Deploy to Kubernetes') {
        //    steps {
        //        container('docker-client') {
        //            script {
        //                sh "kubectl set image deployment/${env.K8S_DEPLOYMENT_NAME} ${env.K8S_DEPLOYMENT_NAME}=${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_DEFAULT_REGION}.amazonaws.com/${env.ECR_REPO_NAME}:${params.IMAGE_TAG} -n ${env.K8S_NAMESPACE}"
        //                sh "kubectl rollout status deployment/${env.K8S_DEPLOYMENT_NAME} -n ${env.K8S_NAMESPACE}"
        //            }
        //        }
        //    }
        //}
       //}

    post {
        success {
            echo 'Repository cloned successfully on Node1.'
        }
        failure {
            echo 'Failed to clone repository.'
        }
    }
}
```
