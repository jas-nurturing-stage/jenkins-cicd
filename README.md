# 🛠️ Jenkins Setup & Pipeline Guide

You can [click here](https://github.com/psagar-dev/cheatsheet/blob/main/Jenkins/install.md) for more details on how to install Jenkins on Ubuntu.

---

## 🔧 Install Docker Engine

After opening the terminal, follow these steps:

- Install Docker
- Add your user to the Docker group
- Add the Jenkins user to the Docker group

```bash
sudo curl -fsSL https://get.docker.com | sudo sh

# Add Your User to the Docker Group
sudo usermod -aG docker $USER
newgrp docker

# Add Jenkins User to the Docker Group
sudo usermod -aG docker jenkins
newgrp docker

# Restart Jenkins if needed:
sudo systemctl restart jenkins
```

---

## 📦 Prerequisite Plugin Installation

Before proceeding with the pipeline setup or deployment process, ensure the following plugins are installed in your CI/CD environment (e.g., Jenkins).

### ✅ Required Plugins

| Plugin Name        | Purpose |
|--------------------|---------|
| **SSH Agent**      | Allows you to use SSH credentials to connect to remote servers during pipeline execution. |
| **Docker Pipeline**| Provides steps to build, run, and manage Docker containers as part of your pipeline. |

### 🔧 Installation Steps (Jenkins Example)

1. **Open Jenkins Dashboard**
2. Navigate to **Manage Jenkins > Manage Plugins**
3. Go to the **Available** tab
4. Search for the following plugins:
   - `SSH Agent`
   - `Docker Pipeline`
5. Select both plugins and click **Install without restart**
6. (Optional) Restart Jenkins if required

### 🧪 Verify Installation

1. Go to **Manage Jenkins > System Information**
2. Search for the plugin names to confirm they are listed and active.

---

## 🛠️ Shared Library Setup

To configure the global library in Jenkins:

1. Navigate to your Jenkins **Dashboard**.
2. Click on **Manage Jenkins**.
3. Select **System**.
4. Scroll down to the section labeled **Global Trusted Pipeline Libraries**.
5. Click **Add** and fill out the fields as follows:

### Library Configuration

| Field                        | Value                                                                 |
|-----------------------------|-----------------------------------------------------------------------|
| **Name**                    | `Shared`                                                              |
| **Default Version**         | `main` **(Branch name)**                                              |
| **Retrieval Method**        | `Modern SCM`                                                          |
| **Source Code Management**  | `Git`                                                                 |
| **Project Repository**      | `https://github.com/psagar-dev/jenkins-shared-libraries.git`          |
| **Credentials**             | `psagar-dev/******` *(Select your GitHub credentials if the repo is private)* |

Once the library is configured, you can load it in your `Jenkinsfile` using the following syntax:

```groovy
@Library('Shared') _
```

This will enable you to use the shared functions and utilities defined in the library across your Jenkins pipelines.

---

## 📨 Configure SMTP in Jenkins for Email Notifications

#### Jenkins Extended E-mail Notification Configuration

This configuration enables Jenkins to send emails using Gmail SMTP.

1. Navigate to your Jenkins **Dashboard**.
2. Click **Manage Jenkins**.
3. Select **System**.
4. Scroll down to **Extended E-mail Notification**.
5. Fill out the fields as follows:

#### SMTP Settings

![User Service Build](https://raw.githubusercontent.com/psagar-dev/cheatsheet/refs/heads/main/Jenkins/images/Extended-E-mail-Notification.png)

| Setting      | Value               |
|--------------|---------------------|
| SMTP Server  | `smtp.gmail.com`    |
| SMTP Port    | `587`               |

### Advanced Settings

- **Credentials**: Set to a Gmail-based user with an app-specific password or a valid credential ID in Jenkins.
- **Use TLS**: ✅ *(Enabled)*

---

## 🔐 Credentials Setup (Required)

Please add the following credentials in Jenkins:

| Store  | Domain   | ID                | Name          | Functionality                      |
|--------|----------|-------------------|---------------|------------------------------------|
| System | (global) | Github            | ***** / ****** | Used for accessing GitHub repos in pipelines – [How to add this](https://github.com/psagar-dev/cheatsheet/blob/main/Jenkins/credentials-username-password.md) |
| System | (global) | gmail-smtp        | ***** / ****** | Sends notification emails via Gmail SMTP – [How to add this](https://github.com/psagar-dev/cheatsheet/blob/main/Jenkins/credentials-username-password.md) |
| System | (global) | docker-hub-creds  | ***** / ****** | Authenticates Docker Hub for image push/pull – [How to add this](https://github.com/psagar-dev/cheatsheet/blob/main/Jenkins/credentials-username-password.md) |
| System | (global) | ssh-ec2           | ubuntu         | SSH key to connect to EC2 instance – [How to add this](https://github.com/psagar-dev/cheatsheet/blob/main/Jenkins/credentials-ssh.md) |

---
### EC2 install Create

![EC2](./images/jenkins/ec2.png)

Then cloudflare add domain

![cloudflare Add Domain](./images/jenkins/adddomain-cloudflare.png)

---

## 🌍 Jenkins Global Environment Variables

This document outlines the global environment variables configured in Jenkins.

1. Navigate to your Jenkins **Dashboard**.
2. Click **Manage Jenkins**.
3. Select **System**.
4. Scroll down to **Global properties**.
5. Then click **Environment variables**.

### ✅ Global Properties Configuration

Environment variables are enabled globally.

### 📋 List of Environment Variables

| Name                    | Value               | Description                        |
|-------------------------|---------------------|------------------------------------|
| `FLASK_CICD_EC2_HOST`   | `52.66.137.64`      | Public IP of the EC2 instance      |
| `FLASK_CICD_EC2_USER`   | `ubuntu`            | Default SSH user for EC2 login     |
| `FLASK_EMAIL_RECIPIENTS`| `xxxxxxx@gmail.com` | Email recipient for notifications  |

---

## 🔧 Webhook Configuration in GitHub

### ✅ Step-by-Step: How to Add a GitHub Webhook

1. **Go to Your Repository**
   - Open your GitHub repository (e.g., `https://github.com/psagar-dev/flask-cicd`)
2. **Navigate to Webhooks**
   - Click on **Settings** (top bar)
   - In the left sidebar, click **Webhooks**
3. **Click “Add webhook”**
   - Button is located at the top-right of the Webhooks section
4. **Configure Webhook URL**
   - In the **Payload URL** field, enter the URL of your server that will receive the webhook (e.g., your Jenkins server or custom endpoint):
     ```
     http://your-server-ip-or-domain/github-webhook/
     ```
5. **Set Content Type**
   - Choose:
     ```
     application/json
     ```
6. **Select Events to Trigger**
   - Choose:
     - **Just the push event** *(common for CI/CD)*
7. **Activate the Webhook**
   - Make sure **"Active"** is checked
   - Click the **"Add webhook"** button
8. **Test the Webhook**
   - Push a commit or perform the selected event
   - Monitor the target server (Jenkins, etc.) or GitHub's webhook **Recent Deliveries** section

![GitHub Webhook](./images/jenkins/github-webhook.png)

---

## 🚀 Jenkins Pipeline Configuration for `flask-cicd`

1. **Log in to Jenkins**
2. **Click on “New Item”**
   - This is usually located on the left-hand side of the Jenkins dashboard
3. **Enter a name for your job**
   - Example: `flask-cicd`
4. **Select “Pipeline” as the project type**
5. **Click “OK”**
   - This will take you to the configuration page for the new pipeline job

### 📁 Pipeline Definition

- **Definition**: Pipeline script from SCM
- **SCM**: Git
- **Repository URL**: `https://github.com/psagar-dev/flask-cicd.git`
- **Credentials**: `psagar-dev/******`
- **Branch Specifier**: `main`
- **Script Path**: `Jenkinsfile`

### ⚡ Trigger

- [x] GitHub hook trigger for GITScm polling 

### 📝 Notes

- This configuration uses a declarative pipeline stored in the `main` branch under the file `Jenkinsfile`.
- Ensure that the **GitHub webhook** is properly configured in your GitHub repository settings to trigger Jenkins jobs automatically.

![Jenkins Configuration](./images/jenkins/flask-cicd-configration.png)

---

# 📄 Jenkinsfile

📌 The shared library used in this pipeline:  
🔗 [jenkins-shared-libraries](https://github.com/psagar-dev/jenkins-shared-libraries)

```groovy
@Library('Shared') _
def config = securityConfig("securelooper/flask-cicd:${BUILD_NUMBER}",'flask-cicd-container')

pipeline {
    agent any
    
    stages {
        
        stage('Python Dependency Install') {
            agent {
                docker {
                    image 'python:3.13-slim'
                }
            }
            steps {
                installPythonDepsVm()
            }
        }

        stage("Security Scans") {
            steps {
                script {
                    securityScan()
                }
            }
        }
        
        stage('Unit Test') {
            agent {
                docker {
                    image 'python:3.13-slim'
                }
            }
            steps {
                unitTest()
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${config.DOCKER_IMAGE}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    pushDockerImage("${config.DOCKER_IMAGE}", "${config.DOCKER_CREDENTIALS_ID}")
                }
            }
        }

        stage('Deploy On Deploying') {
            steps {
                 sshagent (credentials: ['ssh-ec2']) {
                    script {
                        remoteDockerDeploy(
                            "${config.DOCKER_IMAGE}",
                            "${config.CONTAINER_NAME}",
                            "80:5000",
                            "ssh-ec2"
                        )
                    }
                 }
            }
        }
    }

    post {
        success {
            script {
                sendSuccessEmailNotification("${env.FLASK_EMAIL_RECIPIENTS}")
            }
        }
        
        failure {
           script {
                sendFailureEmailNotification("${env.FLASK_EMAIL_RECIPIENTS}")
            }
        }
    }
}
```

---

### 🚀 Flask CI/CD Pipeline Summary

#### 🔧 Key Components

- **Shared Library**: `@Library('Shared')` — contains reusable functions
- **Docker Image**: `securelooper/flask-cicd:${BUILD_NUMBER}`
- **Container Name**: `flask-cicd-container`
- **Base Image**: `python:3.13-slim`
- **Deployment Target**: Remote EC2 instance via SSH

---

### 📦 Pipeline Stages

| Stage                  | Purpose                                           |
|------------------------|---------------------------------------------------|
| **Python Dependency Install** | Installs dependencies inside Python Docker container |
| **Security Scans**     | Runs security checks on code and dependencies     |
| **Unit Test**          | Executes unit tests                               |
| **Build Docker Image** | Builds Docker image with dynamic tag              |
| **Push Docker Image**  | Pushes image to registry using secure credentials |
| **Deploy On Deploying**| Deploys to EC2 using SSH and Docker               |

---

## 📬 Notifications

- ✅ **Success**: Sends email to `${env.FLASK_EMAIL_RECIPIENTS}`
- ❌ **Failure**: Sends alert email on pipeline failure

![Success](./images/jenkins/success-mail.png)
![Fail](./images/jenkins/fail-mail-job.png)

---

## 🖥️ Deployment Details

- Uses `sshagent` with credential ID: `ssh-ec2`
- Maps port: `80 (host)` → `5000 (container)`
- Custom function: `remoteDockerDeploy(...)`

---

## 🚀 Pipeline Overview

📷 ![Pipeline Overview](./images/jenkins/pipline-overview.png)

![Deploy](./images/jenkins/deploy.png)

🚀 Project - 2: Flask CI/CD Pipeline for GitHub Actions AWS ECS Fargate
Automate the build, test, security scan, and deployment of your Flask app using GitHub Actions, AWS ECR/ECS (Fargate), Cloudflare DNS, and Slack notifications.

📝 Brief Description
This CI/CD setup enables:

Automated build, test, and deployment for every code change (staging) and release (production).
Security checks using Trivy and Gitleaks before every deployment.
Seamless Docker image storage in AWS ECR.
Serverless deployment with AWS ECS Fargate.
Custom domains managed by Cloudflare.
Slack notifications for pipeline status (success/failure).
Result: Every code change is built, tested, security-checked, and deployed automatically—making your app secure, up-to-date, and reliably available on your custom domain.

🖼️ Architecture Overview

📸 Visual Guide
1. 🐳 AWS ECR (Elastic Container Registry)
🔒 Create ECR Private Repo: Create a private repository in AWS ECR to store your Docker images securely. ECR Private Repo

📂 ECR Repo List: View all existing repositories in ECR, including your new Flask app repo. ECR Private Repo List

🏷️ ECR Image Tags: Shows the list of images and tags that have been pushed to your private ECR repository. Private register images

2.🚢 AWS ECS Fargate
🛠️ Create ECS Cluster: Start by creating a new ECS cluster to run your containers in Fargate mode. Create Cluster

📝 Create Task Definition: Shows the interface for creating a new ECS task definition for your Flask app. ECS Task Create

📜 Task Definition List: Displays all ECS task definitions which describe how containers are run. ECS Task List

📝 Create Service: This screen is used to set up a new service in Amazon ECS. Here, you choose your task definition, configure deployment settings, and connect your containers to an Application Load Balancer. This process makes sure your application runs smoothly, scales easily, and is accessible through a reliable load balancer. ECS Create Service

ECS Service List

🚦 Application Load Balancer
Load Balancer

3.🌐 Cloudflare DNS
🏷️ Production Domain Setup: Example of a CNAME(Load Balancer) record in Cloudflare for routing traffic to your production environment. cloudflare Production Setup

🧪 Staging Domain Setup: Example of an A record in Cloudflare to route staging traffic for QA/testing. cloudflare Stage Setup

Here's a concise and professional deployment module document for setting up an EC2 instance for GitHub Actions self-hosted runner and deploying to a staging environment.

🛠️ EC2 Setup for GitHub Actions Runner & Staging Deployment
📍 Overview
This guide covers:

Provisioning an EC2 instance for a GitHub Actions runner
Installing the runner & dependencies
Connecting it to GitHub
Deploying to a staging environment
1️⃣ Launch EC2 for GitHub Actions Runner
Ec2

2️⃣ Install Docker & Add Permissions
sudo curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
newgrp docker
🚀 Step-by-Step: Set Up GitHub Actions Runner (Ubuntu/Linux)
✅ 1. Go to GitHub Repository Settings
Open your repository on GitHub.

Click on Settings > Actions > Runners.

Click "New self-hosted runner".

Select:

OS: Linux
Architecture: x64
GitHub will generate a setup script like this:

# Create a folder
mkdir actions-runner && cd actions-runner

# Download latest runner package
curl -o actions-runner-linux-x64-2.314.1.tar.gz -L https://github.com/actions/runner/releases/download/v2.314.1/actions-runner-linux-x64-2.314.1.tar.gz

# Extract installer
tar xzf ./actions-runner-linux-x64-2.314.1.tar.gz

# Configure the runner
./config.sh --url https://github.com/<your-username>/<your-repo> --token <your-token>

# Start the runner
./run.sh
⚠️ Replace <your-username>/<your-repo> and <your-token> with your actual repository info.

🧪 3. Verify Runner
Go back to Settings > Actions > Runners.
You should see the runner status as Idle.
Self Host

🧩 Notes:
Ensure the machine has access to Docker (if your workflows need it).
Add your runner to specific labels for easier workflow filtering.
Use runs-on: self-hosted or runs-on: [self-hosted, linux] in your workflow.
⚙️ Environment Variables
Set these as GitHub Secrets for security:

Variable Name	Example	Usage
AWS_ACCESS_KEY_ID	AKIAIOSXXXXXXXXXXXX	AWS authentication
AWS_SECRET_ACCESS_KEY	wJalrXXXXXXXX/K7MDENG/bPxRfiCXXXXXXXX	AWS authentication
AWS_REGION	ap-south-1	AWS region
ECR_REPOSITORY	flask-app	ECR repository name
ECS_CLUSTER	flask-app-cluster	ECS cluster name
ECS_SERVICE	flask-app-service	ECS service name
ECS_TASK_DEFINITION	flask-task-def:1	ECS task definition
SSH_HOST	ec2-12-34-56-78.compute.amazonaws.com	SSH (if used)
SSH_PRIVATE_KEY	-----BEGIN RSA PRIVATE KEY-----...	SSH key (if used)
✅ How to Add Environment Variables in GitHub Actions
Go to your GitHub repository.
Click on Settings.
Select Secrets and variables > Actions.
Click New repository secret.
Add the secret with the Name and Value.
🚦 Pipeline Overview
🏁 Main Branch: production.yml
on:
  release:
    types: [published]
  workflow_dispatch:
🚀 Deploys to Production when a release is published.
🧪 Staging Branch: staging.yml
on:
  push:
    branches: [staging]
  workflow_dispatch:
🧪 Deploys to Staging on every push to staging branch.
🧩 Key Pipeline Jobs
📦 Install Dependencies
Common to both staging.ymlandproduction.yml` — checks out code, sets up Python, and installs requirements.

install-dependencies:
  runs-on: self-hosted
  steps:
    - name: Checkout code
      uses: actions/checkout@v4
    - name: Set up Python
      uses: actions/setup-python@v5
      with:
        python-version: '3.12'
    - name: Install dependencies
      run: |
        pip install --upgrade pip
        pip install -r requirements.txt
📝 Explanation:
Step	Description
actions/checkout	Checks out the code from the GitHub repository into the runner.
setup-python@v5	Sets up Python 3.12 environment on the GitHub self-hosted runner.
Install dependencies	Upgrades pip and installs Python packages from requirements.txt.
🔐 Security Check
✅ Used in both staging.yml and production.yml Scans for vulnerabilities (Trivy) and secrets (Gitleaks).

security-check:
  runs-on: self-hosted
  needs: install-dependencies
  steps:
    - name: Checkout code
      uses: actions/checkout@v4
    - name: Trivy Installation
      run: |
        sudo apt-get update -y
        sudo apt-get install -y wget apt-transport-https gnupg lsb-release
        wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
        echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
        sudo apt-get update -y
        sudo apt-get install -y trivy
    - name: Trivy FS Scan
      run: trivy fs --format table -o fs-report.json .
    - name: Gitleaks Installation
      run: sudo apt install gitleaks -y
    - name: Gitleaks Code Scan
      run: gitleaks detect --source . -r gitleaks-report.json -f json
📝 Explanation
Step	Description
needs: install-dependencies	Ensures this job runs after install-dependencies is complete.
actions/checkout@v4	Checks out the repo code again in this job (required for scanning tools).
Trivy Installation	Installs Trivy, a tool to scan for OS/package/code vulnerabilities.
Trivy FS Scan	Scans the full file system (.) and writes the output to fs-report.json.
Gitleaks Installation	Installs Gitleaks, a secrets detection tool.
Gitleaks Code Scan	Scans the codebase for hardcoded secrets and writes a report in gitleaks-report.json.
🛡️ Purpose
This job helps catch:

Vulnerabilities in dependencies or file system content (Trivy).
Secrets accidentally committed to the codebase (Gitleaks).
✅ Ideal for CI/CD pipelines in production-grade environments where security checks are mandatory before deployment.

🧪 Run Tests
Executes Python tests to verify code before deployment. ✅ Used in both staging.yml and production.yml

run-tests:
  runs-on: self-hosted
  needs: security-check
  steps:
    - name: Checkout code
      uses: actions/checkout@v4
    - name: Set up Python
      uses: actions/setup-python@v5
      with:
        python-version: '3.12'
    - name: Install dependencies
      run: |
        pip install --upgrade pip
        pip install -r requirements.txt
    - name: Run tests
      run: python -m pytest tests/
📝 Explanation
Step	Description
needs: security-check	Ensures this job runs after security-check completes.
actions/checkout@v4	Retrieves the repository code to the runner.
setup-python@v5	Sets up Python 3.12 for running the tests.
Install dependencies	Installs all required Python packages listed in requirements.txt.
python -m pytest tests/	Executes all test cases inside the tests/ directory using pytest.
🧠 Purpose
This job:

Ensures your application code is functionally correct before deployment.
Catches errors early by automatically running your unit or integration tests.
✅ Best practice to run this before any deployment to staging or production.

🚀 deploy-staging
This job automatically deploys the latest code to the staging server whenever changes are pushed to the staging branch.

It performs the following:

Builds and tags a Docker image using the latest code.
Pushes the image to Amazon ECR.
Connects via SSH to a remote staging server.
Pulls the new image, removes the old container, and runs the updated container using the new image.
This ensures your staging environment is always up to date with the latest code changes for testing and review.

deploy-staging:
        runs-on: self-hosted
        needs: run-tests
        if: github.ref == 'refs/heads/staging'
        env:
            ECR_REPOSITORY: ${{ secrets.ECR_REPOSITORY }}
            IMAGE_TAG: staging-${{ github.run_number }}
        steps:
          - name: Checkout code
            uses: actions/checkout@v4
          
          - name: Configure AWS credentials
            uses: aws-actions/configure-aws-credentials@v4
            with:
                aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
                aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
                aws-region: ${{ secrets.AWS_REGION }}
          
          - name: Log in to Amazon ECR
            id: login-ecr
            uses: aws-actions/amazon-ecr-login@v2

          - name: Build, tag, and push Docker image to ECR
            env:
                ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
            run: |
                  docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
                  docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG

          - name: Set up SSH key
            run: |
              mkdir -p ~/.ssh
              echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_ed25519
              chmod 600 ~/.ssh/id_ed25519
              ssh-keyscan -H ${{ secrets.SSH_HOST }} >> ~/.ssh/known_hosts
          
          - name: Deploy to remote server via SSH
            uses: appleboy/ssh-action@v1
            with:
              host: ${{ secrets.SSH_HOST }}
              username: ubuntu
              key: ${{ secrets.SSH_PRIVATE_KEY }}
              port: 22
              script: |
                export AWS_ACCESS_KEY_ID=${{ secrets.AWS_ACCESS_KEY_ID }}
                export AWS_SECRET_ACCESS_KEY=${{ secrets.AWS_SECRET_ACCESS_KEY }}
                export AWS_REGION=${{ secrets.AWS_REGION }}

                if ! command -v aws &> /dev/null; then
                  echo "Installing AWS CLI..."
                  sudo apt update
                  sudo apt install unzip curl -y
                  curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
                  unzip -o awscliv2.zip
                  sudo ./aws/install
                  export PATH=$PATH:/usr/local/bin
                fi

                echo "🔐 Logging into ECR..."
                aws ecr get-login-password --region ${{ secrets.AWS_REGION }} | docker login --username AWS --password-stdin ${{ steps.login-ecr.outputs.registry }}

                IMAGE_URI=${{ steps.login-ecr.outputs.registry }}/${{ secrets.ECR_REPOSITORY }}:staging-${{ github.run_number }}

                echo "📦 Pulling Image: $IMAGE_URI"
                docker pull $IMAGE_URI

                echo "🧹 Removing old container"
                docker rm -f python-cicd-container || true

                echo "🚀 Running new container"
                docker run -d --name python-cicd-container -p 80:5000 $IMAGE_URI
🚀 deploy-production
This job automatically deploys the app to AWS ECS when a new GitHub release is published.

It performs the following:

Builds and tags a Docker image from the latest release code.
Pushes the image to Amazon ECR.
Fetches the ECS task definition, updates it with the new image.
Registers a new task definition revision.
Deploys the new version to the ECS service by updating it.
✅ This ensures production always runs the latest release, with a safe and controlled deployment to ECS.

deploy-production:
        runs-on: self-hosted
        needs: run-tests
        env:
          ECR_REPOSITORY: ${{ secrets.ECR_REPOSITORY }}
          IMAGE_TAG: ${{ github.run_number }}
        if: github.event_name == 'release' && github.event.action == 'published'
        steps:
            - name: Checkout code
              uses: actions/checkout@v4

            - name: Configure AWS credentials
              uses: aws-actions/configure-aws-credentials@v4
              with:
                aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
                aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
                aws-region: ${{ secrets.AWS_REGION }}

            - name: Log in to Amazon ECR
              id: login-ecr
              uses: aws-actions/amazon-ecr-login@v2

            - name: Build, tag, and push Docker image to ECR
              env:
                ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
              run: |
                  docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
                  docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG

            - name: Deploy to remote server via SSH
              uses: appleboy/ssh-action@v1
              with:
                host: ${{ secrets.SSH_HOST }}
                username: ubuntu
                key: ${{ secrets.SSH_PRIVATE_KEY }}
                port: 22
                script: |
                  export AWS_ACCESS_KEY_ID=${{ secrets.AWS_ACCESS_KEY_ID }}
                  export AWS_SECRET_ACCESS_KEY=${{ secrets.AWS_SECRET_ACCESS_KEY }}
                  export AWS_REGION=${{ secrets.AWS_REGION }}

                  if ! command -v aws &> /dev/null; then
                    echo "Installing AWS CLI..."
                    sudo apt update
                    sudo apt install unzip curl -y
                    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
                    unzip -o awscliv2.zip
                    sudo ./aws/install
                    export PATH=$PATH:/usr/local/bin
                  fi

                  echo "🔐 Logging into ECR..."
                  aws ecr get-login-password --region ${{ secrets.AWS_REGION }} | docker login --username AWS --password-stdin ${{ steps.login-ecr.outputs.registry }}

            - name: Fetch Current Task Definition
              run: |
                aws ecs describe-task-definition \
                  --task-definition ${{ secrets.ECS_TASK_DEFINITION }} \
                  --query "taskDefinition" > task-def.json

            - name: Update Image in Task Definition
              id: update-task
              run: |
                IMAGE_URI=${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY:$IMAGE_TAG
                jq --arg IMAGE "$IMAGE_URI" '
                  del(
                    .taskDefinitionArn,
                    .revision,
                    .status,
                    .requiresAttributes,
                    .compatibilities,
                    .registeredAt,
                    .registeredBy
                  )
                  | .containerDefinitions[0].image = $IMAGE
                ' task-def.json > new-task-def.json

            - name: Register New Task Definition Revision
              id: register-task
              run: |
                TASK_DEF_ARN=$(aws ecs register-task-definition \
                  --cli-input-json file://new-task-def.json \
                  --query "taskDefinition.taskDefinitionArn" \
                  --output text)
                echo "TASK_DEF_ARN=$TASK_DEF_ARN" >> $GITHUB_ENV

            - name: Deploy to ECS
              run: |
                aws ecs update-service \
                  --cluster ${{ secrets.ECS_CLUSTER }} \
                  --service ${{ secrets.ECS_SERVICE }} \
                  --task-definition $TASK_DEF_ARN \
                  --force-new-deployment
Includes:
Repository name
Branch
Commit SHA
✅ Confirms successful deployment after a published release.
Includes:
Repository name
Branch
Commit SHA
🚨 Helps the team stay informed of broken builds or deployment issues immediately.
4.⚙️ CI/CD Pipeline Status (GitHub Actions )
🚀 Production Pipeline Success: Shows a successful production pipeline run on GitHub Actions, triggered by a release. Production Pipline Success

🧪 Staging Pipeline Success: Shows a successful staging pipeline run for new code pushes to the staging branch. Stgaing Pipline Success

📣 Production Slack Notification: Slack notification for production pipeline status (success or failure). Prduction Fail Success

❌ Staging Pipeline Failed Notification: Example Slack alert when the staging pipeline fails. Staging Faild

✅ Staging Pipeline Success Notification: Example Slack alert when the staging pipeline succeeds. Stgaing Success

5.🌟 Application Result
🌍 Production App Live: The deployed Flask app running on the production domain. Production View Success

🧪 Staging App Live: The deployed Flask app available on the staging domain for testing. Stage View Success
