# Amazon App Deployment: A DevSecOps Approach with Terraform and Jenkins CI/CD

# PROJECT ARCHITECTURE :-
![image](https://github.com/Pradyumna018/Amazon-App-Deployment-with-Terraform-and-Jenkins-CICD-Docker-SonarQube/assets/136186419/501b22ae-6dc1-4718-81b6-024532c4d47b)

# Step-1 : Lunch an EC2 instance and install Terraform & AWS CLI.

## To install Terraform :

```
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform
```
## To install AWS CLI :
```
sudo apt  install awscli
```
# Step-2 : Create IAM Role with Administrator access and also create Aws Access key and Secret Access key

## Aws Configure
```
aws configure
```
## Provide your Aws Access key and Secret Access key

# Step-3 : Clone my Repository
```
https://github.com/Pradyumna018/Amazon-App-Deployment-with-Terraform-and-Jenkins-CICD-Docker-SonarQube.git
```
## Now navigate to JENKINS-TF directory.

# Step-4 : Terraform commands to provision
```
terraform init
```
```
terraform validate
```
```
terraform plan
```
```
terraform apply
```
# Step-5 : Run Jenkins & SonarQube on their port
```
sudo chmod 777 /var/run/docker.sock
```
 ```
<instance-ip:8080> #jenkins
<instance-public-ip:9000> #sonarqube
```
# Step-6 : Install Plugins like JDK, Sonarqube Scanner, NodeJs, Docket, OWASP Dependency Check in Jenkins

Goto Manage Jenkins →Plugins → Available Plugins →

Install below plugins

1 → Eclipse Temurin Installer (Install without restart)

2 → SonarQube Scanner (Install without restart)

3 → NodeJs Plugin (Install Without restart)

4 → Docker

5 → Docker Commons

6 → Docker Pipeline

7 → Docker API

8 → docker-build-step

# Step-7 : Let's go to our Jenkins Pipeline and add the script in our Pipeline Script.
```
pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/Pradyumna018/Amazon-App-Deployment-with-Terraform-and-Jenkins-CICD-Docker-SonarQube.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Amazon \
                    -Dsonar.projectKey=Amazon '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
	stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'Dependency'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
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
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build -t amazon ."
                       sh "docker tag amazon pradyumnam/amazon:latest "
                       sh "docker push pradyumnam/amazon:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image pradyumnam/amazon:latest > trivyimage.txt" 
            }
        }
	stage('Deploy to container'){
            steps{
                sh 'docker run -d --name amazon -p 3000:3000 pradyumnam/amazon:latest'
            }
        }

    }
}
```

# OUTPUT 

![image](https://github.com/Pradyumna018/Amazon-App-Deployment-with-Terraform-and-Jenkins-CICD-Docker-SonarQube/assets/136186419/96e5b01c-565c-446b-b1b8-303e0362653c)

# Step-8 : Let's destroy everything
```
terraform destroy --auto-approve

```
In this age of digital transformation, safeguarding your applications is no longer an option but a necessity. The synergy of Terraform, Jenkins, SonarQube, and Trivy empowers us to not only deploy our applications with speed and efficiency but also to do so with an unwavering focus on security. We hope this guide has been a valuable resource in your journey towards embracing DevSecOps principles and securing your Amazon app deployments on AWS. Remember, the world of technology is ever-evolving, and so are the threats that come with it. Stay vigilant, stay informed, and continue to adapt your DevSecOps practices to stay ahead in the realm of secure and efficient application development.

Thank You ❤️


