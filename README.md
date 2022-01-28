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

###  Install JDK on AWS EC2 Instance

we need to install JAVA(JDK) on the EC2 instance.
Lets first update the package manager of the virtual machine -

```
sudo apt-get update  
```
install java by running the following command

```
sudo apt install openjdk-11-jre-headless
```

### Install and Setup Jenkins

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
