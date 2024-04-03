# ProjectApp - Step by step End-to-End Deployment process

![Project](https://github.com/joelwatch/registration-app/assets/106047236/923da58c-a7e6-421e-856f-eae8a4f71aab)

Deploy ec2 amazon-linux2 and Attach Admin role to give him Permission to Deploy Iac with Terraform

## Introduction
Terraform Scripts and manifest files have been provided in this repository. Feel free to edit as you desire.

## 1. Start a Terraform Server on AWS
amazon-linux2-22.04

## 2. Install Terraform as root user
```bash
sudo yum update –y
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
sudo yum -y install terraform
```

## 3. Start a Jenkins Server on AWS
change the host name
reboot the server
create an s3 bucket for backup for the terraform state files (project-registre)
create a Jenkins folder to store your terraform code to create Jenkins and cd into
```bash
sudo hostnamectl set-hostname Terraform-server
sudo init 6
mkdir Jenkins && cd Jenkins
```
 
* Create 5 tf files: provider.tf, main.tf, security.tf, variable.tf, data.tf
deploy your Iac with terraform.
add the flag --lock=false to destroy
```bash
terraform init
terraform fmt
terraform validate
terraform plan
terraform applay --auto-approve
```

## 4. Install Jenkins as root user

Connect jenkins server & Install jenkins:
reference: https://www.jenkins.io/doc/book/installing/linux/
reference: https://www.jenkins.io/doc/tutorials/tutorial-for-installing-jenkins-on-AWS/
```bash
$ sudo yum update –y
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum upgrade
sudo amazon-linux-extras install epel
sudo amazon-linux-extras install java-openjdk11 -y
sudo yum install java-11-amazon-corretto -y
sudo yum install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins
java -version
javac -version
systemctl status jenkins
sudo hostnamectl set-hostname jenkins
reboot: sudo init 6
```

## 5. Install and Configure Maven

* Refer: https://maven.apache.org/install.html
Copy the download link from https://maven.apache.org/download.cgi in jenkins server

```bash
sudo su  & cd ~
cd /opt
wget https://dlcdn.apache.org/maven/maven-3/3.9.5/binaries/apache-maven-3.9.5-bin.tar.gz
tar -xzvf apache-maven-3.9.5-bin.tar.gz
```
* Remove the tar file:
```bash
rm -rf apache-maven-3.9.5-bin.tar.gz
```

* Move apache-maven3.9.5 into a folder name maven:
```bash
mv apache-maven-3.9.5 maven
ls
cd maven
cd bin/
```

* Check the mvn version by typing:
```bash
./mvn -v
```
* Make some changes to type maven command everywhere (like env variable):
```bash
cd ~
ls -a      #It will show the hidden files also
find / -name java-11*   # to find all java directories 
```

* Copy this path: /usr/lib/jvm/java-11-openjdk-11.0.22.0.7-1.amzn2.0.1.x86_64
 
* Goto:
```bash
vim .bash_profile
```

* Enter below lines below fi:
```bash
M2_HOME=/opt/maven
M2=/opt/maven/bin
JAVA_HOME=/usr/lib/jvm/java-11-openjdk-11.0.22.0.7-1.amzn2.0.1.x86_64
PATH=$PATH:$HOME/bin:$JAVA_HOME:$M2_HOME:$M2
```
* Save and quit. type:

* Refresh the path:
```bash
source .bash_profile
echo $PATH
mvn -v   # now you can call mvn everywhere
```

## 6. Configure Jenkins User Interface
* Connect to GUI jenkins create a user:
manage jenkins —> plugins —> available plugins —> search for maven integration —> install —> return to the top page
* Go to: manage jenkins —> tools —> jdk installation —> add jdk —> name: java11 —> JAVA_HOME: <Paste $JAVA_HOME path here>
Java_Home: /usr/lib/jvm/java-11-openjdk-11.0.22.0.7-1.amzn2.0.1.x86_64
```bash
sudo echo $JAVA_HOME
```

* Goto:
maven installation —> add maven —> name: maven —> uncheck installation automatically —> MAVEN_HOME: <Paste $M2_HOME path here> —> apply and save
```bash
sudo echo $M2_HOME
```
MAVEN_HOME:/opt/maven     //You need to add this at Jenkins Job under Maven Installations

* Goto: plugins —> install plugins —> type github —> disable github branch source plugin and enable GitHub plugin —>restart

* Goto jenkins server and install git:
```bash
yum install git -y
```
## 7. Create a Test Job

* Create a test job: new item —> call it test1—> maven project—> source code management —> git —> https://github.com/joelwatch/MyProjectApp.git —> branch: main —> goals and options: clean install—> save and build
 Goto the project job we just build -> Workspace -> target, you will found a folder webapp.war which the final build file (important for the following purpuse).

## 8. Start Ansible Server on AWS

* Goto Terraform server, copy jenkins folder and call it ansible:
```bash
cp -r jenkins ansible
```

* Make some modification in the code (you have ansible Iac code in this repo, cd ansible and edit)
vi data.tf (no changes)—> vi main.tf (change Jenkins-server to Ansible-server)—> vi provider.tf(change jenkins to ansible) —>vi security.tf (change security1 to 2) —> vi variable(to ingressrules and egressrules add port 8089.
Type:
```bash
ls -a —> remove .terraform and .terraform.lock.hcl(rm -rf) (a new folder)
terraform init
terrafom fmt
terraform validate
terraform plan
terraform apply -—auto-approve
(flag -lock=false if any issue)
```

## 9. Install and Configure Ansible
ssh to ansible server and change the hostname:
```bash
hostnamectl set-hostname ansible-server
```

* Create a new user:
```bash
sudo su -
useradd ansadmin
```

* Create a password:
```bash
passwd <new passwd>
```

* Add the new user to the sudo group:
```bash
visudo
```
* Under #%wheel ALL=(ALL) NOPASSWD: ALL add ansadmin ALL=(ALL) NOPASSWD: ALL

* Goto:
```bash
cd /etc/ssh
vi sshd_config
```

* Change PasswordAuthentication yes —> save and quit
```bash
service sshd reload
```

* Switch to the user ansadmin:
```bash
su ansadmin
cd ~
```

* Generate a key to ansadmin to all the servver it will manage:
```bash
ssh-keygen
```
* Type enter until it complete (ls -a to see hidden files)
follow instructions...
```bash
cd .ssh/
ls
sudo su -
ls -a
ls .ssh/
```

* Install Ansible
```bash
cd ~
amazon-linux-extras install ansible2
ansible --version
```

## 10. Intregrate Ansible with Jenkins
Goto:
manage jenkins --> available plugins --> search for Plublish Over SSH and install it
click on restart jenkins when installation is complete.

log back in --> goto manage jenkins --> system --> SSH Servers --> add --> Name: ansible-server --> Hostname: public ip of your ansible-server --> username: ansadmin --> advance --> check the box: Use password authentication or use different --> Passphrase: password of ansadmin --> apply and test configuration (if it succes) save

## 11. Install Docker in Ansible Server

* Goto: 
ansadmin@Ansible-Server, cd into opt, create directory change the owner of the directory to ansadmin, 
```bash
cd /opt
sudo mkdir docker
ls
ll (to see permissions)
sudo chown ansadmin:ansadmin docker
ll (the folder have now ansadmin permission as owner)
```

* Goto jenkins platform do a test build and make some integrations:
click on test1 --> configure add a post build action--> choose send build artifatcs over SSH -> name: Ansible-Server -> Transfert Set -> Source files: webapp/target/*.war -> Remove prefix: webapp/target -> Remote directory: //opt//docker -> apply save.
build again
Goto ansible server and check if you see webapp.war:
```bash
date
ls docker
ll docker (to see time build was made)
```

install docker:
```bash
cd /opt/docker
sudo yum install docker
sudo usermod -aG docker ansadmin (add ansadmin in docker group to perform command)
id ansadmin  (to see ansadmin in the docker group)
sudo service docker start
sudo systemctl start docker
init 6
sudo systemctl start docker
sudo systemctl status docker
docker ps 
sudo su – ansadmin
cd /opt/docker/
ls
```

## 12. Create Project Dockerfile in Ansible Server
```bash
vi Dockerfile
```

## 13. Create Ansible Playbook for Docker Tasks
* See Manifests in Repo
```bash
ifconfig (take the private ip)
sudo vi /etc/ansible/hosts (at the first line: [ansible] under this paste the private ip save quit)
ssh-copy-id <private-IPAddress>  (to copy the key we generate previously, to be able to login to docker server.)
vi app-ci.yml 
ansible-playbook app-ci.yml -–check (to check if the playbook is good)
```

* login to docker hub: 
```bash
docker login -u beerussama2 put the passwd and build the image in your dockerhub repo.
ansible-playbook app-ci.yml
docker images
```

## 14. Create the CI Job
* Goto jenkins create a new job: "Myproject-CI" -> maven project -> Git: paste same git url -> branch: main -> goals and options: clean options -> add a post-build action: send build artifact over ssh (Source files: webapps/target/*.war -> Remove prefix: webapps/target -> Remote directory: //opt//docker).
* To run your playbook in Exec commad type:
```bash
ansible-playbook /opt/docker/app-ci.yml
```
apply, save and build.

## 15. Start the EKS Server on AWS
* Goto terraform server
```bash
cp -r ansible eks-server
cd eks-server
ls
```

* Delete; .terreform .terraform.lock.hcl, make some changes and deploy with terraform.
copy the public ip of eks-server and ssh, change the hostname:
```bash
sudo vi /etc/hostname (delete all and put EKS_Server)
init 6
```

## 16. Provision EKS cluster with eksctl
* Install Kubectl
Refer: https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html
```bash
sudo su -
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.28.1/2023-09-14/bin/linux/amd64/kubectl
chmod +x ./kubectl 
mv kubectl /bin
ls /bin | grep kubectl
kubectl --version
```

* Install eksctl
Refer: https://github.com/eksctl-io/eksctl/blob/main/README.md#installation
```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
cd /tmp
sudo mv /tmp/eksctl /bin
eksctl version
```

* Create and Attach roles to the EKS Server
```bash
AmazonEC2FullAccess
AWSCloudFormationFullAccess
IAMFullAccess
AdministratorAccess
```

* Provision EKS Cluster
```bash
eksctl create cluster --name myprojectapp-cluster \
--region us-east-2 \
--node-type t2.small
```

* Install AWS CLI
Refer: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

* Connect to your cluster
```bash
aws eks update-kubeconfig --region us-east-2 --name myprojectapp-cluster
kubectl get nodes
kubectl get pods
```
* Your cluster is up and no pods yet

* Now create a deployment manifest file, service
```bash
vi myprojectapp-deployment.yml
vi myprojectapp-service.yml
```

## 17. Integrate EKS Server with Ansible
```bash
vi /etc/ssh/sshd_config
PasswordAuthentication yes
passwd root (to assign passwd to root user to enable authentication from ansible server)
service sshd reload
```

* Now get the private IP for this bootstrap server (ipconfig or go to AWS Dashboard), goto to ansible server to configure eks server
switch to ansadmin, you can use private ip because they are in the same vpc.
[kubernetes]
<private ip eksctl>

* On the Ansible Server
```bash
sudo su - ansadmin
vi /etc/ansible/hosts
```
Put your eks private ip for your bootstrap server below [ansible]
[kubernetes]
<private IP>

To connet to eks-server from ansible-server, copy the public from ansible to bootstrap server.
```bash
ssh-copy-id root@EKS-Server-PrivateIP (the password is for the root user eks-server)
ssh roo@<privateIp of eksserver> to check if you can connect
exit  to back to ansible server
```

* On the Ansible Server, create a playbook for the deployment
```bash
vi kube_deploy.yml
ansible-playbook kube_deploy.yml --check
ansible-playbook kube_deploy.yml
kubctl get pods (to check if your pods are up and running)
kubectl get all (to all pods, loadbalancer, service...)
```

* On the EKS Server, create manifest files for the deployment
```bash
vi myapp-deployment.yml
vi myapp-service.yml

```

## 18. Create a CD Job on Jenkins
goto jenkins: create new item -> Myproject-CD -> freestyle project -> post-build action -> Send build actifact over SSH, Set the Exec command: 
```bash
ansible-playbook /opt/docker/kube_deploy.yml
```
apply and save, build now.

## 19. Intergrate the CI and the CD Jobs on Jenkins
Goto Myproject-CI -> configure -> Build Triggers check Poll SCM -> Schedule: * * * * * (to check a reclycle every minute if the is there's any update in git repo and trigger the build).
Goto: Post Steps -> Add post-build action: Build other projects -> Project to build: Myproject-CD -> check Trigger only if build is stable -> apply and save, build Myproject-CI

## 20. Deploy/Test the Application
In your repo in your local make some change in the Readme file and push it to see if the triggers works.
don't forget to: git status -> git add . -> git commit -m "commit msg" -> git push origin main.
Goto jenkins and check if Myproject-CI trigger changes -> and after CI finish Myproject-CD will triiger deployment -> after that check docker hub repo if a new image has been build and updated.
If you look in eks-server as a root
```bash
sudo -i
```

You check you pods
```bash
kubectl get pods
```

You will see pods has been deploy few seconds ago, if u check ur loadbalancer:
```bash
kubectl get svc
```

You will see your LB get deploy few seconds ago and you can copy the EXTERNAL-IP (ending with .com) of your LB to access your application paste it to the browser with :8080 (put: /myprojectapp to access the app).

## Happy you got to this point. Hope it worked!

# Congratulations!

# Kamgaing Tech Systems Solutions
# kamgaing.tech@gmail.com
# registration-app
