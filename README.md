# 🚀 **CI/CD Pipeline: Deploying Applications to Kubernetes using Jenkins**

![Jenkins to Kubernetes Pipeline](https://cdn.hashnode.com/res/hashnode/image/upload/v1699693572375/16323e8c-15d8-4cee-aa98-d4f4f5657533.gif)

---

## 📋 **Overview**

This is an **end-to-end DevOps CI/CD pipeline project** that demonstrates how to automate the complete software delivery lifecycle using Jenkins, Maven, Docker, and Elastic Kubernetes Services (EKS). 

The pipeline automates everything from code changes in GitHub to deployment on Kubernetes, with multiple quality checks and security scans in between.

By the end of this project, you'll have a fully automated **CI/CD pipeline** that:

✅ **Detects code changes** from GitHub automatically  
✅ **Builds and tests** applications using Maven  
✅ **Analyzes code quality** using SonarQube  
✅ **Scans Docker images** for vulnerabilities using Trivy  
✅ **Pushes container images** to Docker Hub  
✅ **Deploys to Kubernetes** on AWS EKS  
✅ **Monitors deployments** automatically  

---

## 🎯 **Project Goals**

✓ Automate the complete build, test, and deployment process  
✓ Reduce manual errors and deployment time  
✓ Implement multiple quality gates before production  
✓ Scan for security vulnerabilities automatically  
✓ Enable fast and reliable application releases  
✓ Create a professional DevOps workflow  

---

## 📁 **Project Structure**

```
Jenkins-Kubernetes-CICD/
│
├── .github/
│   └── workflows/              # GitHub Actions workflows
│
├── kubernetes/
│   ├── deployment.yaml         # Kubernetes Deployment
│   ├── service.yaml            # Service configuration
│   ├── configmap.yaml          # Configuration data
│   └── secrets.yaml            # Sensitive data
│
├── helm/
│   ├── Chart.yaml              # Helm chart metadata
│   ├── values.yaml             # Default values
│   ├── values-dev.yaml         # Dev environment
│   ├── values-staging.yaml     # Staging environment
│   ├── values-prod.yaml        # Production environment
│   └── templates/
│       ├── deployment.yaml
│       ├── service.yaml
│       └── configmap.yaml
│
├── src/
│   ├── main/                   # Application source code
│   └── test/                   # Test cases
│
├── docker/
│   ├── Dockerfile              # Container image definition
│   └── .dockerignore           # Exclude unnecessary files
│
├── Jenkinsfile                 # Pipeline as Code
├── pom.xml                     # Maven configuration
├── requirements.txt            # Dependencies
├── README.md                   # Documentation
└── TROUBLESHOOTING.md          # Common issues & fixes
```

---

## 🔧 **Prerequisites**

Make sure you have these installed and running:

| Tool | Version | Purpose |
|------|---------|---------|
| **Jenkins** | 2.350+ | CI/CD automation server |
| **Kubernetes (EKS)** | 1.24+ | Container orchestration on AWS |
| **Docker** | 20.10+ | Container platform |
| **Maven** | 3.8+ | Java build tool |
| **kubectl** | Latest | Kubernetes CLI |
| **Helm** | 3.0+ | Kubernetes package manager |
| **Git** | 2.35+ | Version control |
| **SonarQube** | 9.0+ | Code quality analysis |
| **Trivy** | Latest | Container security scanning |

---

## 📦 **Infrastructure Setup**

### **Step 1: Create AWS EC2 Instances**

You'll need 2-3 EC2 instances:

1. **Jenkins-Master** - Runs Jenkins server
2. **Jenkins-Agent** - Runs build and deployment jobs
3. **SonarQube-Server** (Optional) - Code quality analysis

```bash
# Create Jenkins Master
aws ec2 run-instances --image-id ami-0c55b159cbfafe1f0 --instance-type t2.medium --key-name my-key

# Create Jenkins Agent
aws ec2 run-instances --image-id ami-0c55b159cbfafe1f0 --instance-type t2.medium --key-name my-key
```

### **Step 2: Install Jenkins on Master**

```bash
# Update system
sudo apt-get update && sudo apt-get upgrade -y

# Add Jenkins repository
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

# Install Jenkins
sudo apt-get update
sudo apt-get install jenkins -y
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

Access Jenkins at: `http://<Jenkins-Master-IP>:8080`

### **Step 3: Setup Jenkins Agent**

On Jenkins Agent server:

```bash
# Install Java
sudo apt-get install openjdk-11-jdk -y

# Create jenkins-slave folder
sudo mkdir /opt/jenkins-slave
sudo chown ubuntu:ubuntu /opt/jenkins-slave

# Add Jenkins user to docker group (for Docker builds)
sudo usermod -aG docker ubuntu
```

In Jenkins UI:
1. Go to **Manage Jenkins** → **Manage Nodes and Clouds** → **New Node**
2. Name it `Jenkins-Agent-1`
3. Set Remote Root Directory: `/opt/jenkins-slave`
4. Set Launch method: **SSH**
5. Add SSH key and agent IP

### **Step 4: Install Required Packages on Agent**

```bash
# Install Docker
sudo apt-get install docker.io -y

# Install Maven
sudo apt-get install maven -y

# Install kubectl
sudo apt-get install kubectl -y

# Install Trivy (for image scanning)
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update && sudo apt-get install trivy -y

# Install Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### **Step 5: Setup EKS Cluster**

```bash
# Create EKS cluster (replace cluster name)
eksctl create cluster --name my-cluster --region us-east-1 --nodegroup-name my-nodes --nodes 3 --node-type t3.medium

# Verify cluster
kubectl cluster-info
kubectl get nodes
```

---

## 🔌 **Jenkins Plugin Installation**

Go to **Manage Jenkins** → **Manage Plugins** → **Available** and install these:

**Core Plugins:**
- **Git** - GitHub integration
- **Pipeline** - Jenkins Pipeline support
- **Docker Pipeline** - Docker integration

**Build & Test:**
- **Maven Integration** - Maven build support
- **Pipeline Maven Integration** - Maven in pipelines

**Code Quality:**
- **SonarQube Scanner** - Code quality analysis

**Security:**
- **Docker** - Docker operations
- **Trivy** - Container image scanning

**Kubernetes:**
- **Kubernetes** - Kubernetes cluster operations
- **Kubernetes CLI** - kubectl integration

**Notifications:**
- **Email Extension** - Email notifications
- **Slack Notification** (Optional)

---

## 🔄 **CI/CD Pipeline Flow**

The complete pipeline flow is as follows:

```
Developer Push → GitHub Webhook → Jenkins Trigger
    ↓
Checkout Code → Build with Maven → Unit Tests
    ↓
SonarQube Analysis → Code Quality Gate
    ↓
Build Docker Image → Scan with Trivy
    ↓
Push to Docker Hub → Update EKS Deployment
    ↓
Verify Deployment → Health Checks
```

---

## 🔄 **Detailed Pipeline Stages**

### **Stage 1️⃣: Code Checkout**

Jenkins clones the code from GitHub when a push is detected.

```groovy
stage('Checkout Code') {
    steps {
        checkout scmGit(
            branches: [[name: '*/main']],
            userRemoteConfigs: [[url: 'https://github.com/username/repo.git']]
        )
    }
}
```

### **Stage 2️⃣: Build with Maven**

Compiles the code and creates JAR/WAR artifact.

```groovy
stage('Build Application') {
    steps {
        sh 'mvn clean package -DskipTests'
    }
}
```

### **Stage 3️⃣: Unit Testing**

Runs all unit tests to verify code quality.

```groovy
stage('Unit Testing') {
    steps {
        sh 'mvn test'
    }
    post {
        always {
            junit 'target/surefire-reports/*.xml'
        }
    }
}
```

### **Stage 4️⃣: SonarQube Code Analysis**

Performs static code analysis and enforces quality gates.

```groovy
stage('Code Quality Analysis') {
    steps {
        withSonarQubeEnv('SonarQube') {
            sh 'mvn sonar:sonar'
        }
        waitForQualityGate abortPipeline: true
    }
}
```

**Quality Gate Criteria:**
- Code Coverage > 80%
- No Critical Bugs
- No Security Vulnerabilities
- Code Duplication < 5%

### **Stage 5️⃣: Build Docker Image**

Creates a Docker image of the application.

```groovy
stage('Build Docker Image') {
    steps {
        script {
            sh '''
                docker build -t myrepo/myapp:${BUILD_NUMBER} .
                docker tag myrepo/myapp:${BUILD_NUMBER} myrepo/myapp:latest
            '''
        }
    }
}
```

Example `Dockerfile`:
```dockerfile
FROM openjdk:11
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

### **Stage 6️⃣: Trivy Security Scan**

Scans Docker image for vulnerabilities.

```groovy
stage('Scan with Trivy') {
    steps {
        script {
            sh '''
                trivy image --exit-code 0 --severity HIGH,CRITICAL \
                myrepo/myapp:${BUILD_NUMBER}
            '''
        }
    }
}
```

### **Stage 7️⃣: Push to Docker Hub**

Authenticates and pushes image to Docker Hub.

```groovy
stage('Push to Docker Hub') {
    steps {
        script {
            withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                sh '''
                    docker login -u ${USER} -p ${PASS}
                    docker push myrepo/myapp:${BUILD_NUMBER}
                    docker push myrepo/myapp:latest
                '''
            }
        }
    }
}
```

### **Stage 8️⃣: Deploy to Kubernetes (EKS)**

Deploys the application to EKS cluster using Helm.

```groovy
stage('Deploy to EKS') {
    steps {
        script {
            sh '''
                aws eks update-kubeconfig --name my-cluster --region us-east-1
                helm repo add myrepo https://helm.example.com
                helm repo update
                helm upgrade --install my-release myrepo/mychart \
                    --set image.tag=${BUILD_NUMBER} \
                    -f helm/values.yaml
            '''
        }
    }
}
```

### **Stage 9️⃣: Verify Deployment**

Checks if pods are running and healthy.

```groovy
stage('Verify Deployment') {
    steps {
        sh '''
            kubectl rollout status deployment/my-app -n default
            kubectl get pods -n default
            sleep 30
            kubectl exec -it $(kubectl get pods -o name | head -1) -- curl localhost:8080/health
        '''
    }
}
```

---

## 🔐 **Credentials Configuration**

Add these credentials in Jenkins:

1. **GitHub Credentials**
   - Type: Username and password
   - Go to: Manage Jenkins → Credentials → System → Global Credentials
   - ID: `github-credentials`

2. **Docker Hub Credentials**
   - Type: Username and password
   - ID: `docker-hub-credentials`

3. **AWS Credentials**
   - Type: Secret text
   - ID: `aws-credentials`
   - Contains: AWS Access Key and Secret Key

4. **Kubernetes Config**
   - Type: Kubernetes configuration
   - Add your kubeconfig content
   - ID: `kubernetes-config`

5. **SonarQube Token**
   - Type: Secret text
   - ID: `sonar-token`

---

## 🐛 **Common Issues & Solutions**

### **Issue 1: Jenkins Cannot Connect to Agent**

**Error:** `Agent offline`

**Solution:**
```bash
# Check SSH connectivity
ssh -i your-key.pem ubuntu@agent-ip

# Verify Jenkins can reach agent
telnet agent-ip 22

# Check agent logs
tail -f /opt/jenkins-slave/remoting/logs/agent.log
```

### **Issue 2: Docker Build Fails**

**Error:** `Cannot connect to Docker daemon`

**Solution:**
```bash
# Add Jenkins user to docker group
sudo usermod -aG docker jenkins

# Restart Jenkins
sudo systemctl restart jenkins

# Verify docker access
sudo -u jenkins docker ps
```

### **Issue 3: Maven Build Fails**

**Error:** `[ERROR] No POM found`

**Solution:**
```bash
# Check if pom.xml exists in repo
git ls -la | grep pom.xml

# Verify Maven installation
mvn -version

# Clean Maven cache
rm -rf ~/.m2/repository
```

### **Issue 4: SonarQube Quality Gate Failure**

**Error:** `QUALITY GATE FAILED`

**Solution:**
```bash
# Check SonarQube server connectivity
curl http://sonarqube-ip:9000/api/system/health

# Verify project key in pom.xml
# Set lower thresholds initially to pass

# Check quality gate rules
http://sonarqube-ip:9000/admin/quality_gates
```

### **Issue 5: EKS Deployment Fails**

**Error:** `ImagePullBackOff`

**Solution:**
```bash
# Verify image exists in Docker Hub
docker search myrepo/myapp

# Check kubeconfig
cat ~/.kube/config

# Create image pull secret
kubectl create secret docker-registry regcred \
  --docker-server=docker.io \
  --docker-username=<username> \
  --docker-password=<token>

# Add to deployment
kubectl patch serviceaccount default \
  -p '{"imagePullSecrets": [{"name": "regcred"}]}'
```

### **Issue 6: Trivy Scan Fails**

**Error:** `Database is locked`

**Solution:**
```bash
# Update Trivy database
trivy image --download-db-only

# Clean cache
rm -rf ~/.cache/trivy

# Run scan with single thread
trivy image --single-thread myrepo/myapp:latest
```

### **Issue 7: Helm Deployment Fails**

**Error:** `Release already exists`

**Solution:**
```bash
# Use upgrade with install
helm upgrade --install my-release ./helm-chart

# Or delete and recreate
helm delete my-release
helm install my-release ./helm-chart

# Check release status
helm status my-release
```

---

## 📊 **Monitoring & Troubleshooting**

### **View Jenkins Logs**
```bash
sudo tail -f /var/log/jenkins/jenkins.log
```

### **Check Pipeline Execution**
- Open Jenkins UI → Job Name → Build History → Click on build number → Console Output

### **Monitor EKS Deployment**
```bash
# Check deployment status
kubectl get deployment -n default
kubectl get pods -n default
kubectl describe pod <pod-name> -n default

# View pod logs
kubectl logs -f deployment/my-app -n default

# Check service endpoints
kubectl get svc -n default
kubectl describe svc my-app -n default
```

### **Verify SonarQube Analysis**
```bash
# Access SonarQube
http://sonarqube-ip:9000

# Check project quality metrics
http://sonarqube-ip:9000/dashboard?id=com.example:my-app
```

---

## 🔐 **Security Best Practices**

✅ Never commit secrets to GitHub - use Jenkins credentials  
✅ Use IAM roles instead of hardcoded AWS keys  
✅ Store Docker credentials in Jenkins Credentials Store  
✅ Implement network policies in Kubernetes  
✅ Use RBAC for Kubernetes access  
✅ Scan all Docker images with Trivy  
✅ Keep Jenkins and plugins updated  
✅ Enable Jenkins authentication and authorization  

---

## 📚 **Example Jenkinsfile**

```groovy
pipeline {
    agent {
        label 'Jenkins-Agent-1'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/username/repo.git']]
                )
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('SonarQube') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
                waitForQualityGate abortPipeline: true
            }
        }
        
        stage('Build Docker') {
            steps {
                sh '''
                    docker build -t myrepo/myapp:${BUILD_NUMBER} .
                    docker tag myrepo/myapp:${BUILD_NUMBER} myrepo/myapp:latest
                '''
            }
        }
        
        stage('Scan with Trivy') {
            steps {
                sh 'trivy image myrepo/myapp:${BUILD_NUMBER}'
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                        docker login -u ${USER} -p ${PASS}
                        docker push myrepo/myapp:${BUILD_NUMBER}
                    '''
                }
            }
        }
        
        stage('Deploy to EKS') {
            steps {
                sh '''
                    aws eks update-kubeconfig --name my-cluster --region us-east-1
                    helm upgrade --install my-release ./helm --set image.tag=${BUILD_NUMBER}
                '''
            }
        }
        
        stage('Verify') {
            steps {
                sh '''
                    kubectl rollout status deployment/my-app
                    kubectl get pods
                '''
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
```

---

## 📚 **Useful Resources**

- [Jenkins Official Docs](https://jenkins.io/doc/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [AWS EKS Setup Guide](https://docs.aws.amazon.com/eks/)
- [Helm Documentation](https://helm.sh/docs/)
- [SonarQube Docs](https://docs.sonarqube.org/)
- [Trivy Documentation](https://aquasecurity.github.io/trivy/)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)

---

## ✨ **Summary**

You've now built a professional **end-to-end CI/CD pipeline** that:

🎯 **Automates everything** from code push to Kubernetes deployment  
🔒 **Ensures quality** with Maven builds, unit tests, and SonarQube analysis  
🔍 **Scans for security** vulnerabilities using Trivy  
⚡ **Deploys reliably** to AWS EKS with Helm  
📊 **Monitors continuously** for issues and failures  

This pipeline handles the complete DevOps workflow professionally and is ready for production use!

---

## 📧 **Need Help?**

- Review Jenkins console output for error messages
- Check EKS pod logs: `kubectl logs -f deployment/my-app`
- Verify SonarQube quality gates: `http://sonarqube-ip:9000`

---
![Want to connect](https://camo.githubusercontent.com/baed803db45aec39ebbfcce722a51f771e3c356be81374351ee4fd0e8d96a88b/68747470733a2f2f696d6775722e636f6d2f6d65564a6e6d642e706e67)
