In continuation to document ![Jenkins Github SSH Authentication Configuration](https://github.com/sachinratan/k8s-dkr-cicd/blob/main/Jenkins%20and%20GitHub%20SSH%20Key%20Authentication%20Configuration.md) following pipeline will help to pull and push the code to git repository while using SSH authentication method.

Declarative Jenkins Pipeline:

```
pipeline {
    agent {
        kubernetes {
            cloud 'Amazon_EKS'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: git
    image: alpine/git:latest
    command:
    - cat
    tty: true
"""
        }
    }

    environment {
        GIT_REPO = 'git@github.com:sachinratan/jnks-k8s-integration-repo.git'
        GIT_BRANCH = 'main'
    }

    stages {
        stage('Setup Git and SSH') {
            steps {
                container('git') {
                    withCredentials([sshUserPrivateKey(credentialsId: 'jnks-github-ssh-auth-creds', keyFileVariable: 'SSH_KEY')]) {
                        sh '''
                            # Configure Git
                            git config --global user.name "sachinratan"
                            git config --global user.email "sachinshinde741@gmail.com"
                            git config --global --add safe.directory '*'

                            # Setup SSH
                            mkdir -p ~/.ssh
                            cp $SSH_KEY ~/.ssh/id_rsa
                            chmod 600 ~/.ssh/id_rsa
                            ssh-keyscan github.com >> ~/.ssh/known_hosts
                        '''
                    }
                }
            }
        }

        stage('Clone Repository') {
            steps {
                container('git') {
                    sh '''
                        git clone ${GIT_REPO} repo
                        cd repo
                        git checkout ${GIT_BRANCH}
                    '''
                }
            }
        }

        stage('Make Changes') {
            steps {
                container('git') {
                    sh '''
                        cd repo
                        echo "Updated at $(date)" >> update.log
                    '''
                }
            }
        }

        stage('Commit and Push') {
            steps {
                container('git') {
                    sh '''
                        cd repo
                        git add .
                        git commit -m "Automated update from Jenkins - $(date)" || echo "No changes"
                        git push origin ${GIT_BRANCH}
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Code pushed successfully to GitHub!'
        }
        failure {
            echo 'Failed to push code. Check logs.'
        }
    }
}

```
