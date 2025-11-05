# Netflix-Project-Documentation
## Create Instance 
- with t3.medium instance type
- Port allowed 1)Tomcat:8080 2)SonarQube:9000
- and 20gb storage capicity

## Step2: Connect To Instance
 - Install Jenkins
 - Install Docker
 - Install SonarQube    # used for code quality testing
 - Install Trivy        # used to scan docker images
      
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
**SonarQube**
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

**Go to manage jenkins and Install Required Plugins**



