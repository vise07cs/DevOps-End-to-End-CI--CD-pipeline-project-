# DevOps-End-to-End-CI--CD-pipeline-project-

<img width="2088" height="925" alt="diagram-export-12-3-2025-11_35_38-PM" src="https://github.com/user-attachments/assets/b9c37014-a2c8-4f79-bfc9-359ea5f1cc71" />


This Project is setup using the below tools 

- Maven
- Git Hub
- Jenkins
- Docker
- Kubernetes


# S1 : Jenkins Server setup using Linux VM 

1. Create Ubuntu VM using AWS EC2 (t2.medium)
2. Enable 8080 Port Number in Security Group Inbound Rules
3. Connect to VM using MobaXterm
4. Instal Java

sudo apt update
sudo apt install fontconfig openjdk-17-jre
java -version

5 . Install Jenkins 

sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkin

6. Start Jenkins

sudo systemctl enable jenkins
sudo systemctl start jenkins


7. Verify Jenkins

sudo systemctl status jenkins

8. Open jenkins server in browser using VM public ip

http://public-ip:8080/

9. Copy Jenkins Password

sudo cat /var/lib/jenkins/secrets/initialAdminPassword

10. Create Admin Account & Install Required Plugins in Jenkins

<img width="2009" height="1141" alt="Screenshot 2025-12-03 134153" src="https://github.com/user-attachments/assets/c680d6b0-455a-43e8-8981-e8b83b167f2f" />

<img width="2307" height="530" alt="Screenshot 2025-11-27 125700" src="https://github.com/user-attachments/assets/9a994d74-9830-4a7c-a52a-85fb891c7bf8" />


# S2 : Configure Maven as Global Tool in Jenkins

Manage Jenkins -> Tools -> Maven Installation -> Add maven

# S3 : Create EKS Management Host in AWS

1. Launch new Ubuntu VM using AWS EC2 ( t2.medium )
2. Connect to machine and install kubectl using below commands

curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client

3. Install AWS CLI latest version using below commands

sudo apt install unzip 
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version

4. Install eksctl using below commands

curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version


# S4 :  Create IAM role & attach to EKS Management Host & Jenkins Server

1. Create New Role using IAM service ( Select Usecase - ec2 )

2. Add below permissions for the role

3 . IAM FUlL ACCESS , EC2 FULL ACCESS , VPC FULL ACCESS , ADMINISTRATOR FULL ACCESS 


4. Attach created role to EKS Management Host (Select EC2 => Click on Security => Modify IAM Role => attach IAM role we have created)

5. Attach created role to Jenkins Machine (Select EC2 => Click on Security => Modify IAM Role => attach IAM role we have created)

<img width="2369" height="1159" alt="Screenshot 2025-12-03 135501" src="https://github.com/user-attachments/assets/a11e29a6-4ca2-4a8d-93bf-af2337e1be58" />


<img width="2369" height="1054" alt="Screenshot 2025-12-03 134952" src="https://github.com/user-attachments/assets/a6fdc651-bbeb-4d97-9aca-8c9ac2640359" />



#S5 : Create EKS Cluster using eksctl


eksctl create cluster --name dev-cluster --region us-west-2 --node-type t2.medium  --zones us-west-1a,us-west-1c

It may take 15-20 minutes of time 
Once done , verify using
kubectl get nodes  


<img width="2547" height="1167" alt="Screenshot 2025-11-27 172620" src="https://github.com/user-attachments/assets/45cb77c4-4b5d-4c60-8edc-d3eeeafdcf53" />



# S6 : Setup Docker in Jenkins

curl -fsSL get.docker.com | /bin/bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
sudo docker version


# S7 : Install AWS CLI in JENKINS Server

URL : https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

sudo apt install unzip 
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version


# S8 : Install Kubectl in JENKINS Server

curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client

S9:  Update EKS Cluster Config File in Jenkins Server

1. Execute below command in Eks Management host & copy kube config file data
$ cat .kube/config

2. Execute below commands in Jenkins Server and paste kube config file
$ cd /var/lib/jenkins
$ sudo mkdir .kube
$ sudo vi .kube/config

3. Execute below commands in Jenkins Server and paste kube config file for ubuntu user to check EKS Cluster info
 aws eks update-kubeconfig --region us-west-2 --name dev-cluster

 


4.  check eks nodes
$ kubectl get nodes
Note: We should be able to see EKS cluster nodes here.


<img width="2260" height="765" alt="Screenshot 2025-11-27 140846" src="https://github.com/user-attachments/assets/4f2e1863-6b5b-43dd-a4c9-3f0126a178ad" />

S10:  Create Jenkins CI CD Job

Stage-1 : Clone Git Repo
Stage-2 : Maven Build
Stage-3 : Create Docker Image
Stage-4 : Push Docker Image to Registry
Stage-5 : Deploy app in k8s eks cluster


pipeline {
    agent any
    
    tools{
        maven "Maven-3.9.9"
    }

    stages {
        stage('Clone Repo') {
            steps {
                git 'https://github.com/<repo-name>'
            }
        }
        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Docker Image') {
            steps {
                sh 'docker build -t vikit/mavenwebapp .'
            }
        }
        stage('k8s deployment') {
            steps {
                sh 'kubectl apply -f k8s-deploy.yml'
            }
        }
    }
}

<img width="2466" height="1036" alt="Screenshot 2025-12-03 135232" src="https://github.com/user-attachments/assets/53903763-5ed0-42dd-99d7-5b4baaf54d56" />



Step - 11 : Access Application in Browser via LoadBalancer DNS (we can further connect it to AWS Route 53)

We should be able to access our application








  












