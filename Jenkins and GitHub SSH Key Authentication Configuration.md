Step 1: Generate SSH Key Pair
First, generate an SSH key pair that will be used for authentication:

bash

# On your local machine or any system with ssh-keygen
ssh-keygen -t ed25519 -C "sachinshinde741@gmail.com" -f jenkins-github-key

# This creates two files:
# - jenkins-github-key (private key)
# - jenkins-github-key.pub (public key)

Note: If your system doesn't support ed25519, use RSA instead:

bash

ssh-keygen -t rsa -b 4096 -C "sachinshinde741@gmail.com" -f jenkins-github-key

Step 2: Add Public Key to GitHub
Copy the public key content:

bash

cat jenkins-github-key.pub

Add to GitHub:

Go to GitHub.com and log in
Click your profile picture → Settings
Click SSH and GPG keys (left sidebar)
Click New SSH key
Title: Jenkins Kubernetes Slave
Key type: Authentication Key
Paste the public key content
Click Add SSH key
Step 3: Add Private Key to Jenkins Credentials
Access Jenkins:

Go to Jenkins dashboard
Navigate to Manage Jenkins → Manage Credentials
Add SSH Credential:

Click on (global) domain
Click Add Credentials
Configure as follows:
Kind: SSH Username with private key
ID: github-ssh-key (you're already using this)
Description: GitHub SSH Key for Code Push
Username: git (important: use "git", not your GitHub username)
Private Key: Select Enter directly
Click Add and paste your private key content (from jenkins-github-key file)
Passphrase: Leave empty (unless you set one during key generation)
Click Create
Step 4: Configure Jenkins Pod Template
Ensure your Kubernetes pod template includes Git and SSH client:

yaml

apiVersion: v1
kind: Pod
spec:
  containers:
  - name: git
    image: alpine/git:latest
    command:
    - cat
    tty: true
    volumeMounts:
    - name: ssh-key
      mountPath: /root/.ssh
      readOnly: true
  volumes:
  - name: ssh-key
    secret:
      secretName: jenkins-ssh-key
      defaultMode: 0400

Step 5: Test SSH Connection
Create a test pipeline to verify the connection:

groovy

pipeline {
    agent {
        kubernetes {
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

    stages {
        stage('Test GitHub SSH Connection') {
            steps {
                container('git') {
                    sshagent(credentials: ['github-ssh-key']) {
                        sh '''
                            # Add GitHub to known hosts
                            mkdir -p ~/.ssh
                            ssh-keyscan github.com >> ~/.ssh/known_hosts

                            # Test connection
                            ssh -T git@github.com || true
                        '''
                    }
                }
            }
        }
    }
}

Expected output: "Hi sachinratan! You've successfully authenticated, but GitHub does not provide shell access."

Step 6: Complete Pipeline with Authentication
Here's a complete pipeline that authenticates and pushes code:

groovy

pipeline {
    agent {
        kubernetes {
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
        stage('Setup Git') {
            steps {
                container('git') {
                    sh '''
                        # Configure Git
                        git config --global user.name "sachinratan"
                        git config --global user.email "sachinshinde741@gmail.com"
                        git config --global --add safe.directory '*'
                    '''
                }
            }
        }

        stage('Clone Repository') {
            steps {
                container('git') {
                    sshagent(credentials: ['github-ssh-key']) {
                        sh '''
                            # Add GitHub to known hosts
                            mkdir -p ~/.ssh
                            ssh-keyscan github.com >> ~/.ssh/known_hosts

                            # Clone repository
                            git clone ${GIT_REPO} repo
                            cd repo
                            git checkout ${GIT_BRANCH}
                        '''

