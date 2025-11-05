# Netflix-Project-Documentation
## Create Instance 
- with t3.medium instance type
- Port allowed 1)Tomcat:8080 2)SonarQube:9000
- and 20gb storage capicity

## Step2: Connect To Instance
 - Install Jenkins
 - Install Docker
 - Install SonarQube    **used for code quality testing**
 - Install Trivy        **used to scan docker images**
      
**Jenkins**
````
sudo apt update
sudo apt install fontconfig openjdk-21-jre  -y
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins -y
````
**Docker**
````

sudo apt-get update
sudo apt-get install docker.io -y
sudo systemctl start docker
sudo usermod -aG docker ubuntu
sudo usermod -aG docker jenkins
newgrp docker
sudo chmod 777 /var/run/docker.sock
````
**SonarQube Container**
````
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
````

**Trivy**
````
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y
````
**Copy jenkins password from this location**
````
/var/lib/jenkins/secrets/initialAdminPassword
````
**Login into jenkins**

<img width="1915" height="930" alt="image" src="https://github.com/user-attachments/assets/a8e6f2d9-d9ec-4f87-baac-d4ddc4cbea80" />

**Go to manage jenkins then Plugins and Install Required Plugins**

<img width="1918" height="930" alt="image" src="https://github.com/user-attachments/assets/a9c8abfe-dec9-4621-a606-8afb42da84c0" />
---
**Go to manage jenkins then tools and add tools**

### Jdk

<img width="1911" height="867" alt="image" src="https://github.com/user-attachments/assets/61ddbebd-8ad6-4c3a-bbb2-802b5265cb64" />

### SonarQube Scanner

<img width="1918" height="927" alt="image" src="https://github.com/user-attachments/assets/0a192651-6ac0-4077-90e9-fce707e5b50f" />

### NodeJs

<img width="1901" height="923" alt="image" src="https://github.com/user-attachments/assets/4ff1b930-4c98-43be-92d8-4e39cb544090" />

**after this click on apply and save**
---

## Now Create the SonarQube token by login in with Port Number 9000
Username and Password: **admin**

**go to administrator->security->create token**

<img width="1918" height="680" alt="image" src="https://github.com/user-attachments/assets/5c80d1f4-5a22-4c2f-8181-e2a58534c70a" />

---
**Go to manage jenkins then Credentials and add creds of SQ and DockerHub-Uname&Pass**

<img width="1917" height="930" alt="image" src="https://github.com/user-attachments/assets/46fe1b06-e993-4ad5-b4db-41c2ec741edf" />

<img width="1916" height="930" alt="image" src="https://github.com/user-attachments/assets/045b5e5a-7833-4f67-a746-5041aa5be972" />

<img width="1918" height="437" alt="image" src="https://github.com/user-attachments/assets/60d21bbf-ca37-4ada-9f71-b675d4c1230b" />

**Go to manage jenkins then System and add SQ server**

<img width="1917" height="930" alt="image" src="https://github.com/user-attachments/assets/a8211338-814e-42f9-aea4-a1fffa0e508c" />

### restart the jenkins

# Now create pipline and write the script 

````
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {

        stage('Code-Pull') {
            steps {
                git branch: 'main', url: 'https://github.com/abhipraydhoble/Project-Netflix-Clone.git'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix'''
                }
            }
        }
        stage("quality gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
       stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker'){   
                       sh "docker build --build-arg TMDB_V3_API_KEY=020581a34f3ab93b1360a55bea864bd9 -t abhipraydh96/moviesite ."
                       sh "docker push abhipraydh96/moviesite "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image abhipraydh96/moviesite > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name netflix -p 8081:80 abhipraydh96/moviesite'
            }
        }
        
    }
}
````

**then build the pipeline**
# Output
<img width="1037" height="543" alt="image" src="https://github.com/user-attachments/assets/18cdd46a-8907-47b0-b42b-74bde553cf01" />










