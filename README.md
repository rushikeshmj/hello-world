# Hello World Application

This is a simple Python application that prints "Hello, World!".

## Installation

```sh
git clone <repository-url>
cd hello_world_app
python -m venv venv
source venv/bin/activate  # On Windows use `venv\Scripts\activate`
pip install -r requirements.txt
###########################################################
#######################################################

Jenkins-POC
POC Overview
1. Setting up Jenkins: Continuous Integration server setup. 
2. Installing Docker: Containerization tool installation for environment consistency. 
3. Setting up SonarQube: Code quality and security analysis. 
4. Setting up EC2 for deployment 
5. Pipeline and Deployments: Automating builds, tests, and deployments.
Github repository link:
https://github.com/rushikeshmj/hello-world
Launch Virtual Machine using AWS EC2
Here is a detailed list of the basic requirements and setup for the EC2 instance i
have used for running Jenkins, including the specifics of the instance type, AMI,
and security groups.
EC2 Instance Requirements and Setup: 
1. Instance Type
- Instance Type: `t2.large` 
- vCPUs: 2
- Memory: 8 GB 
- Network Performance: Moderate
2. Amazon Machine Image (AMI)
- AMI: Ubuntu Server 20.04 LTS (Focal Fossa)
3. Security Groups 
Security groups act as a virtual firewall for your instance to control inbound
and outbound traffic.
After Launching your Virtual machine ,SSH into the Server.
Install below tools to EC2
1. Jenkins 
I am launching the container of Jenkins because 8080 port is not accessible on my 
machine
2. Docker
Sudo apt-get install docker.io 
Run this command to give access to docker
sudo chmod 666 /var/run/docker.sock
3. Trivy
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo
apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main
| sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
4. Git
5. Aws cli
Configure Jenkins
Access Jenkins Dashboard: 
Open a web browser and navigate to your Jenkins instance 
Log in with your Jenkins credentials. (cat address provided on Jenkins)
Install Plugins: 
-Go to Manage Jenkins > Manage Plugins.
-Click on the Available tab.
Search for and install the following plugins:
1. Docker: Enables Jenkins to use Docker containers.
2. Sonar Scanner: For Scanning Vulnerabilities.
3. Docker Pipeline: Allows Jenkins to use Docker containers in pipeline
jobs.
4. ECR
5. Cobertura
6. Code scanner plugin
7. Github
8. Pipeline stage view
9. Ssh
10. Suggested plugins
Configuring SonarQube Scanner Plugin
1. Manage Jenkins: Go to "Manage Jenkins".
2. Global Tool Configuration: Click on "Global Tool Configuration".
3. SonarQube Scanner: 
• Scroll down to the "SonarQube Scanner" section. 
• Click "Add SonarQube Scanner".
• Provide a name (e.g., Sonar scanner).
• Optionally, check "Install automatically" to let Jenkins handle the
installation.
• Save the configuration.
4. Manage Jenkins: Go back to "Manage Jenkins".
5. Configure System: 
• Scroll down to the "SonarQube servers" section. 
• Click "Add SonarQube".
• Provide a name for the server (e.g., SonarQube).
• Set the "Server URL" to the URL of your SonarQube instance.
• Add a "Server Authentication Token".
Creating a Token on SonarQube
1. Log in to SonarQube: Open your SonarQube instance in a web browser
and log in.
2. My Account: Click on your user profile at the top-right corner and select
"My Account".
3. Security: Navigate to the "Security" tab.
4. Generate Token: Under "Generate Tokens", provide a name for the
token (e.g., JenkinsToken).
5. Generate: Click on "Generate" and copy the token. 
Adding SonarQube Token to Jenkins
1. Manage Jenkins: Go to "Manage Jenkins".
2. Configure System: Scroll to the "SonarQube servers" section. 
3. Add Token: 
• Under the "Server Authentication Token" section, click "Add" next 
to "Credentials".
• Select "Jenkins" and then "Secret text". 
• Paste the token you copied from SonarQube.
• Provide an ID (e.g., sonarqube-token).
• Save the credentials.
• Select the newly added token from the dropdown list.
Configuring Docker Plugin
1. Manage Jenkins: Go to "Manage Jenkins".
2. Global Tool Configuration: Click on "Global Tool Configuration".
3. Docker: 
• Scroll down to the "Docker" section.
• Click "Add Docker Tool".
• Provide a name (e.g., docker).
• Optionally, check "Install automatically" to let Jenkins handle the
installation.
• Save the configuration.
Configure system with their setting and path
Save the credentials
Create Node and configure
Create Job and it to github repo and cred
Create IAM user
Create ECR repo
Create Pipeline
pipeline {
 agent {
 label 'ec2'
 }
 environment {
 DOCKER_REGISTRY = "584294315145.dkr.ecr.us-east-1.amazonaws.com"
 SONARQUBE_SERVER = "http://34.207.58.229:9000/"
 AWS_REGION = "us-east-1"
 PUBLIC_IP = "http://34.207.58.229:9000"
 AWS_ECR_ACCOUNT_ID = "533267249366"
 AWS_ECR_REGION = "us-east-1"
 AWS_ECR_REPOSITORY = "jenkins-poc"
 DOCKER_IMAGE_NAME = "hello-world-app"
 SONARQUBE_LOGIN_TOKEN = "squ_e758c27be837e60687142a751c8734fea3da9ca3"
 }
 stages {
 stage('Checkout') {
 steps {
 retry(3) {
 checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 
'https://github.com/shekharbo/hello-world.git']]])
 }
 }
 }
 stage('Unit Test') {
 steps {
 sh 'docker --version'
 sh 'docker pull python:3.9'
 sh 'pip install -r requirements.txt'
 sh 'python3 -m venv ~/myenv'
 sh """
 set +x
 . /home/ubuntu/myenv/bin/activate
 """
 sh """
 /home/ubuntu/.local/bin/pytest 
/home/ubuntu/workspace/hello_world_demo/tests/test_main.py
 """
 }
 }
 stage('Code Coverage') {
 steps {
 sh 'pip install coverage'
 sh """
 /home/ubuntu/.local/bin/coverage run -m pytest 
/home/ubuntu/workspace/hello_world_demo/tests/test_main.py
 """
 sh '/home/ubuntu/.local/bin/coverage report'
 sh '/home/ubuntu/.local/bin/coverage xml -o coverage.xml'
 cobertura coberturaReportFile: 'coverage.xml'
 }
 }
 stage('SCA and SonarQube') {
 steps {
 withSonarQubeEnv('SonarQubeServer') {
 script {
 def scannerHome = tool 'SonarQubeScanner'
 if (scannerHome) {
 sh "/home/hello-world-demo-python/hello-world/sonar-scanner-5.0.1.3006-
linux/bin/sonar-scanner \
 -Dsonar.projectKey=hello-world \
 -Dsonar.sources=src \
 -Dsonar.host.url=${SONARQUBE_SERVER} \
 -Dsonar.login=${SONARQUBE_LOGIN_TOKEN}"
 } else {
 error "SonarQube Scanner not configured."
 }
 }
 }
 }
 }
 stage('Build and tag image using Docker') {
 steps {
 script {
 dir('/home/ubuntu/hello-world-demo-python/hello-world') {
 sh 'pwd'
 sh 'ls -l Dockerfile'
 sh 'docker build -t hello-world-app .'
 sh "docker tag hello-world-app 
${AWS_ECR_ACCOUNT_ID}.dkr.ecr.${AWS_ECR_REGION}.amazonaws.com/${AWS_ECR_REPOSI
TORY}:latest"
 withCredentials([usernamePassword(credentialsId: 'aws-ecr-credentials', 
usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
 sh "aws ecr get-login-password --region ${AWS_ECR_REGION} | docker login --
username AWS --password-stdin 
${AWS_ECR_ACCOUNT_ID}.dkr.ecr.${AWS_ECR_REGION}.amazonaws.com"
 sh "docker push 
${AWS_ECR_ACCOUNT_ID}.dkr.ecr.${AWS_ECR_REGION}.amazonaws.com/${AWS_ECR_REPOSI
TORY}:latest"
 }
 }
 }
 }
 }
 stage('Image scan using trivy') {
 steps {
 sh "trivy image 
${AWS_ECR_ACCOUNT_ID}.dkr.ecr.${AWS_ECR_REGION}.amazonaws.com/${AWS_ECR_REPOSI
TORY}:latest"
 }
 }
 stage('Deploy to EC2') {
 steps {
 sshagent(['ssh_key']) {
 sh "ssh -o StrictHostKeyChecking=no ubuntu@34.207.58.229 aws ecr get-loginpassword --region ${AWS_ECR_REGION} | docker login --username AWS --password-stdin 
${DOCKER_REGISTRY}"
 sh "ssh -o StrictHostKeyChecking=no ubuntu@34.207.58.229 docker pull 
${AWS_ECR_ACCOUNT_ID}.dkr.ecr.${AWS_ECR_REGION}.amazonaws.com/${AWS_ECR_REPOSI
TORY}:latest"
 sh "ssh -o StrictHostKeyChecking=no ubuntu@34.207.58.229 docker run -d -p 
8081:8080 
${AWS_ECR_ACCOUNT_ID}.dkr.ecr.${AWS_ECR_REGION}.amazonaws.com/${AWS_ECR_REPOSI
TORY}:latest"
 }
 }
 }
 }
}
Successful Deployment
