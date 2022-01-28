# kubernetes-jenkins-pipeline-on-AWS

Table of Content
=================

1. Setup an AWS EC2 Instance
2. Connect to EC2 Instance
3. Install JDK on AWS EC2 Instance
4. Install and Setup Jenkins
5. Update visudo and assign administrative privileges to Jenkins user
6. Install Docker
7. Install and Setup AWS CLI
8. Install and Setup Kubectl
9. Install and Setup eksctl
10. Create eks cluster using eksctl
11. Add Docker and GitHub Credentials into Jenkins
12. Add jenkins stages
13. Build, deploy and test CI CD pipeline

## 1. Setup an AWS EC2 Instance
The first step would be for us to set up an EC2 instance and on this instance, we will be installing -

* JDK
* Jenkins
* eksctl
* kubectl

### 2. Install JDK on AWS EC2 Instance

we need to install JAVA(JDK) on the EC2 instance.
Lets first update the package manager of the virtual machine -

```
sudo apt-get update  
```
install java by running the following command

```
sudo apt install openjdk-11-jre-headless
```

### 3. Install and Setup Jenkins

First, add the repository key to the system:

```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
```
Next, append the Debian package repository address to the server’s sources.list:

```
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
```
 Run update so that apt will use the new repository:

```
sudo apt update
```
Finally, install Jenkins and its dependencies:
```
sudo apt install jenkins
```
Let’s start Jenkins using systemctl:
```
sudo systemctl start jenkins
```
Since systemctl doesn’t display output, you can use its status command to verify that Jenkins started successfully:
```
sudo systemctl status jenkins
```
Now we know the public IP address of the EC2 machine, so now we can access the Jenkins from the browser using the public IP address followed by the port 8080

### 4. Update visudo and assign administration privileges to jenkins user

Now we have installed the Jenkins on the EC2 instance. To interact with the Kubernetes cluster Jenkins will be executing the shell script with the Jenkins user, so the Jenkins user should have an administration(superuser) role assigned forehand.

Let’s add jenkins user as an administrator and also ass NOPASSWD so that during the pipeline run it will not ask for root password.

Open the file /etc/sudoers in vi mode
```
sudo vi /etc/sudoers 
```
Add the following line at the end of the file
```
jenkins ALL=(ALL) NOPASSWD: ALL 
```
Now we can use Jenkins as root user and for that run the following command -
```
sudo su - jenkins  
```
### 5. Install Docker

Now we need to install the docker after installing the Jenkins.

The docker installation will be done by the Jenkins user because now it has root user privileges.

Use the following command for installing the docker -
```
sudo apt install docker.io
```

#### Add jenkins user to Docker group
Jenkins will be accessing the Docker for building the application Docker images, so we need to add the Jenkins user to the docker group.
```
sudo usermod -aG docker jenkins 
```
### 6. Install and Setup AWS CLI

Okay so now we have our EC2 machine and Jenkins installed. Now we need to set up the AWS CLI on the EC2 machine so that we can use eksctl in the later stages

```
 curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
Configure AWS CLI

To configure the AWS the first command we are going to run is -
```
aws configure 
```
Once you execute the above command it will ask for the following information -
```
AWS Access Key ID [None]:
AWS Secret Access Key [None]:
Default region name [None]:
Default output format [None]:
```

### 7. Install and Setup Kubectl

now we need to set up the kubectl also onto the EC2 instance where we set up the Jenkins in the previous steps.

Here is the command for installing kubectl

```
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
```
```
chmod +x ./kubectl 
sudo mv ./kubectl /usr/local/bin
```
### 8 Install and Setup eksctl

for non-Linux OS you can find a binary download here:

https://github.com/weaveworks/eksctl/releases

on Linux, you can just execute:
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/download/latest_release/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp  

sudo mv /tmp/eksctl /usr/local/bin
```
This utility will use the same credentials file as we explored for the AWS cli, located under '~/.aws/credentials'

Test
```
eksctl version
```
### 9. Create eks cluster using eksctl

Create a yml file

eg: my-first-eksctl.yml

```
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: EKS-course-cluster
  region: us-east-1

nodeGroups:
  - name: ng-1
    instanceType: t2.small
    desiredCapacity: 3
    ssh: # use existing EC2 key
      publicKeyName: eks-course
```


### 12. Add jenkins stages


```
pipeline
{
    agent any
    
    tools {
    maven 'maven-3.8.4'
    }
    
  

    
    stages
    {
        stage('scm')
        {
            steps
            {
                git credentialsId: 'tomcat-deployer',
                url: 'https://github.com/nikhil15041993/dockeransiblejenkins.git'
            }
            
        }
        
        stage('maven build')
        {
            steps
            {
                sh "mvn clean package"
            }
        }
        
        stage('Docker build')
        {
            steps
            {
            
                 
               sh 'docker build /var/lib/jenkins/workspace/my-ansible-maven-project -t nikhilnambiars/tomcat:${DOCKER_TAG} '
            }
        }
        
        
        stage('DockerHub Push'){
            steps{
              
               withCredentials([string(credentialsId: 'docker-hub2', variable: 'dockerHubPwd')]) {
                    sh "docker login -u nikhilnambiars -p ${dockerHubPwd}"
                }
                
                sh "docker push nikhilnambiars/tomcat:${DOCKER_TAG}"
            }
        }
        
        
        
        stage('Docker Deploy'){
            steps{
            ansiblePlaybook credentialsId: 'dev-server1', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=${DOCKER_TAG}", installation: 'my-ansible', inventory: 'dev.inv', playbook: 'deploy-docker.yml'     
            }
        }
        
        
        stage("kubernetes deployment"){
        sh 'kubectl apply -f deployment.yml'
    }
        
        
    }
}
```
