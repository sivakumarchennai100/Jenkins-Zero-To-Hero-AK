# Jenkins Pipeline to build Java based application using Github,Maven,SonarQube,Shell Script, Argo CD and Kubernetes

![CI-CD End-To-End flow diagram ](https://github.com/user-attachments/assets/c55457db-f48d-4dfd-b9bd-cec6d4b2deff)

Note: 
      
      We have chosen to shell script rather than Image Updater in this deployment since it is widely used in the current market.
      Using the shell script, we can automatically update the build changes with Manifest repo or source code repo as mentined in the above diagram.
      
      Ansible is not a very good tool for the deploying the application, if we are dealing with configuration management, then we can go ahead with Ansible.
      
      Also it is always good to go with ArgoCD for CD (Continous Deployment/Delivery) part rather than using Jenkins Pipeline since ArgoCD is a very good Gitops 
      friendly tool to deploy the manifest repo with the K8s cluster.
      

**Here are the step-by-step details to set up an end-to-end Jenkins pipeline for a Java application using SonarQube, Argo CD, Helm, and Kubernetes:**

# Spring Boot based Java web application

For application related code docs, refer https://github.com/sivakumarchennai100/Jenkins-Zero-To-Hero-AK/tree/main/java-maven-sonar-argocd-helm-k8s/spring-boot-app
 
This is a simple Sprint Boot based Java application that can be built using Maven. Sprint Boot dependencies are handled using the pom.xml (Project Object Model file used by Maven) at the root directory of the repository.

This is a MVC architecture based application where controller returns a page with title and message attributes to the view.


## To Execute the application locally and access it using your browser

**Checkout the repo and move to the directory**
```
git clone https://github.com/iam-veeramalla/Jenkins-Zero-To-Hero/java-maven-sonar-argocd-helm-k8s/sprint-boot-app
cd java-maven-sonar-argocd-helm-k8s/sprint-boot-app
```

**Execute the Maven targets to generate the artifacts**

```
mvn clean package
```

The above maven target stores the artifacts to the `target` directory. You can either execute the artifact on your local machine
(or) run it as a Docker container.

** Note: To avoid issues with local setup, Java versions and other dependencies, I would recommend the docker way. **

- mvn clean package or mvn clean install will go to the specific folder where the maven application is stored and try to execute the mvn clean package.
- mvn clean package or mvn clean install will find the pom.xml file (written by devops/developer)
- pom.xml is responsible for getting the dependencies runtime and building the application
- In pom.xml we mention the list of dependencies required to run the application when executed locally or remotely by someone or other applications.
- Whenever mvn clean package or mvn clean install is executed, then the dependencies are installed as required for building the java application.
- 

### Execute locally (Java 11 needed) and access the application on http://localhost:8080

```
java -jar target/spring-boot-web.jar
```

### The Docker way

Build the Docker Image

```
docker build -t ultimate-cicd-pipeline:v1 .
```

```
docker run -d -p 8010:8080 -t ultimate-cicd-pipeline:v1
```

Hurray !! Access the application on `http://<ip-address>:8010`


## Next Steps

### Configure a Sonar Server locally

```
System Requirements
Java 17+ (Oracle JDK, OpenJDK, or AdoptOpenJDK)
Hardware Recommendations:
   Minimum 2 GB RAM
   2 CPU cores
sudo apt update && sudo apt install unzip -y
adduser sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.4.1.88267.zip
unzip *
chown -R sonarqube:sonarqube /opt/sonarqube
chmod -R 775 /opt/sonarqube
cd /opt/sonarqube/bin/linux-x86-64
./sonar.sh start
```

Hurray !! Now you can access the `SonarQube Server` on `http://<ip-address>:9000` 


****************************************************************************************************************************************
## To Execute and access the application using your browser using using Github,Maven,SonarQube,Shell Script, Argo CD and Kubernetes

## First Install an EC2 Instance:

## AWS EC2 Instance

- Go to AWS Console
- Create Instances
- Select Ubuntu image
- Instance type (T2-Large) ( Note: Terminate the EC2 instance once the application is deployed and tested, since it is chargeble, no-free tier )
- Select the existing Key-pair or create a new key-pair for SSH access
- Launch instances

<img width="994" alt="Screenshot 2023-02-01 at 12 37 45 PM" src="https://user-images.githubusercontent.com/43399466/215974891-196abfe9-ace0-407b-abd2-adcffe218e3f.png">

### Install Jenkins.

Pre-Requisites:
 - Java (JDK)

### Run the below commands to install Java and Jenkins

Install Java

```
sudo apt update
sudo apt install openjdk-17-jre
```

Verify Java is Installed

```
java -version
```

Now, you can proceed with installing Jenkins

```
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

**Note: ** By default, Jenkins will not be accessible to the external world due to the inbound traffic restriction by AWS. Open port 8080 in the inbound traffic rules as show below.

- EC2 > Instances > Click on <Instance-ID>
- In the bottom tabs -> Click on Security
- Security groups
- Add inbound traffic rules as shown in the image (you can just allow TCP 8080 as well, in my case, I allowed `All traffic`).

<img width="1187" alt="Screenshot 2023-02-01 at 12 42 01 PM" src="https://user-images.githubusercontent.com/43399466/215975712-2fc569cb-9d76-49b4-9345-d8b62187aa22.png">


### Login to Jenkins using the below URL:

http://<ec2-instance-public-ip-address>:8080    [You can get the ec2-instance-public-ip-address from your AWS EC2 console page]

Note: If you are not interested in allowing `All Traffic` to your EC2 instance
      1. Delete the inbound traffic rule for your instance
      2. Edit the inbound traffic rule to only allow custom TCP port `8080`
  
After you login to Jenkins, 
      - Run the command to copy the Jenkins Admin Password - `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`
      - Enter the Administrator password
      
<img width="1291" alt="Screenshot 2023-02-01 at 10 56 25 AM" src="https://user-images.githubusercontent.com/43399466/215959008-3ebca431-1f14-4d81-9f12-6bb232bfbee3.png">

### Click on Install suggested plugins

<img width="1291" alt="Screenshot 2023-02-01 at 10 58 40 AM" src="https://user-images.githubusercontent.com/43399466/215959294-047eadef-7e64-4795-bd3b-b1efb0375988.png">

Wait for the Jenkins to Install suggested plugins

<img width="1291" alt="Screenshot 2023-02-01 at 10 59 31 AM" src="https://user-images.githubusercontent.com/43399466/215959398-344b5721-28ec-47a5-8908-b698e435608d.png">

Create First Admin User or Skip the step [If you want to use this Jenkins instance for future use-cases as well, better to create admin user]

<img width="990" alt="Screenshot 2023-02-01 at 11 02 09 AM" src="https://user-images.githubusercontent.com/43399466/215959757-403246c8-e739-4103-9265-6bdab418013e.png">

Jenkins Installation is Successful. You can now starting using the Jenkins 

<img width="990" alt="Screenshot 2023-02-01 at 11 14 13 AM" src="https://user-images.githubusercontent.com/43399466/215961440-3f13f82b-61a2-4117-88bc-0da265a67fa7.png">

## Install the Docker Pipeline plugin in Jenkins:

   - Log in to Jenkins.
   - Go to Manage Jenkins > Manage Plugins.
   - In the Available tab, search for "Docker Pipeline".
   - Select the plugin and click the Install button.
   - Then in the Available tab, search for "SonarQube scanner".
   - Restart Jenkins after the plugin is installed.
   
<img width="1392" alt="Screenshot 2023-02-01 at 12 17 02 PM" src="https://user-images.githubusercontent.com/43399466/215973898-7c366525-15db-4876-bd71-49522ecb267d.png">

Wait for the Jenkins to be restarted.


http://<ec2-instance-public-ip>:8080/restart
```

The docker agent configuration is now successful.

****************************************************************************************************************************************
### Next step is to write a Jenkins file 

*** Url:https://github.com/sivakumarchennai100/Jenkins-Zero-To-Hero-AK/blob/main/java-maven-sonar-argocd-helm-k8s/spring-boot-app/JenkinsFile
    Path: java-maven-sonar-argocd-helm-k8s/spring-boot-app/JenkinsFile
    - Stages in Jenkins pipeline is nothing but what are the differnet blocks you are trying to build using the Jenkins pipeline
    - Checkout Stage is needed when the source code is placed at the Jenking pipeline itself and not on SCM repo


    Note: It is advisable to use docker as an agent in the Jenkins pipeline to save the maintenance cost overhead.
          Once the jenkins pipeline is triggered, docker agent executes the stages and steps involved in the Jenkins file and after executing the container is 
          destroyed.

          It is the responsibility of the devops engineer to build the docker image and place it in the Jenkins file. Jenkins will then trigger the pipeline.
          
    

### CREATE YOUR FIRST JENKINS PIPELINE:
======================================

when we create a jenkins pipeline using freestyle project, it is not in declarative way, whereas in pipeline approach, jenkins pipeline is written in declarative way.

Goto Jenkins UI --> Dashboard --> first-jenkins-job --> configure --> Pipeline --> select 
Definition: Pipeline script from SCM
SCM: GIT
Branch Specifier: main 
Script Path: my-first-pipeline/Jenkinsfile  --> Save

Goto Dashboard --> first-jenkins-job ---> Build now

- Once we have written the Jenkinsfile, then we need to put the docker credentials and Github credentials in Jenkins
  Goto Jenkins UI ---> Manage Jenkins ---> Manage Credentials ---> System ---> Global credentials ---> Add credentials: Kind- Username and Password , Scope-Global , Username- <dockerhub username> , Password - <Docker hub pw>, Create.
- Now the docker credentials is created.

- Then we need to put the github credentials in Jenkins
  Goto Jenkins UI ---> Manage Jenkins ---> Manage Credentials ---> System ---> Global credentials ---> Add credentials: Kind- Secret text , Scope-Global , Secret - Paste the token copied from github --> settings --> Developer settings --> Personel access token --> Create new token --> Provide the required details to create a new github token , ID-github , Create.

- Now the github credentials is created.
- Now restart the Jenkins everytime you do the configuration

****************************************************************************************************************************************
### Configure a Sonar Server locally or on a EC2 instance 

- Sonarquke needs to be installed within the same VPC where Jenkins and other CICD components were installed
- System Requirements
Java 17+ (Oracle JDK, OpenJDK, or AdoptOpenJDK)
Hardware Recommendations:
   Minimum 2 GB RAM
   2 CPU cores

# Commands:
$sudo apt update && sudo apt install unzip -y
$ sudo su -
#adduser sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.4.1.88267.zip
apt install unzip
sudo su - somarqube
unzip *
chmod -R 775 /home/sonarqube/sonarqube-9.4.0-54424
chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-9.4.0-54424
cd /sonarqube-9.4.0-54424/bin/linux-x86-64/
ls
./sonar.sh start

Note: By default Sonar server will run on port 9000

Check if Sonarqube is now accessible bu http://<EC2_ip address:9000>
```
Hurray !! Now you can access the `SonarQube Server` on `http://<ip-address>:9000` 
Will ask to change the defaultpassword at the first login

Now we have installed Sonarqube and installed the Sonarqube plugin on the Jenkins server, but inorder to authenticate Sonar with Jenkins
Goto Sonarqube UI --> My account --> Security --> Tokens ( Generate Tokens: Jenkins --> Generate - Copy the token )
Goto Jenkins UI ---> Manage Jenkins ---> Manage Credentials ---> System ---> Global credentials ---> Add credentials: Kind- Secret text , Scope-Global , Secret - Paste the token copied from Sonarqube , ID-sonarqube , Create.

Now the Sonarqube configuration is done.


************************************************************************

# IQ Difference between adduser and useradd command**

adduser
High-level command (actually a Perl or shell script).

More user-friendly and interactive.

Automatically:

Creates home directory (/home/username)

Sets default shell (/bin/bash)

Adds user to a group

Prompts to set a password

Ideal for manual user creation.

Example:
sudo adduser alice
➡️ Then it asks you questions like password, full name, etc.

🛠️ useradd
Low-level binary.

Not interactive — doesn’t do much unless you give it specific options.

You have to manually:

Create home directory (-m)

Set shell (-s /bin/bash)

Set password separately with passwd

Example:
sudo useradd -m -s /bin/bash alice
sudo passwd alice


**********************************************************************************************************************************
## Install Docker on the EC2 Instance:

## Docker Slave Configuration

Run the below command to Install Docker

```
sudo apt update
sudo apt install docker.io
```

 
### Grant Jenkins user and Ubuntu user permission to docker deamon.

```
sudo su - 
usermod -aG docker jenkins
usermod -aG docker ubuntu
systemctl restart docker
```

Once you are done with the above steps, it is better to restart Jenkins.
http://<ec2-instance-public-ip>:8080/restart  --> To restart jenkins

```
**********************************************************************************************************************************
## Install Kubernetes Cluster on your windows laptop using Minikube:

Install minikube on Windows machine:
----------------------------------

Reference Link: https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2F.exe+download


PS C:\Users\smohan> New-Item -Path 'c:\' -Name 'minikube' -ItemType Directory -Force


    Directory: C:\


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        04-09-2024     18:47                minikube


PS C:\Users\smohan> Invoke-WebRequest -OutFile 'c:\minikube\minikube.exe' -Uri 'https://github.com/kubernetes/minikube/releases/latest/download/minikube-windows-amd64.exe' -UseBasicParsing



step2:

PS C:\WINDOWS\system32> $oldPath = [Environment]::GetEnvironmentVariable('Path', [EnvironmentVariableTarget]::Machine)
PS C:\WINDOWS\system32> if ($oldPath.Split(';') -inotcontains 'C:\minikube'){
>>   [Environment]::SetEnvironmentVariable('Path', $('{0};C:\minikube' -f $oldPath), [EnvironmentVariableTarget]::Machine)
>> }
PS C:\WINDOWS\system32>



PS C:\WINDOWS\system32> minikube start

SUCCESS:

PS C:\WINDOWS\system32> minikube start
* minikube v1.35.0 on Microsoft Windows 11 Home Single Language 10.0.26100.3775 Build 26100.3775
* Automatically selected the docker driver. Other choices: hyperv, virtualbox, ssh
* Using Docker Desktop driver with root privileges
* Starting "minikube" primary control-plane node in "minikube" cluster
* Pulling base image v0.0.46 ...
* Downloading Kubernetes v1.32.0 preload ...
    > gcr.io/k8s-minikube/kicbase...:  500.31 MiB / 500.31 MiB  100.00% 5.48 Mi
    > preloaded-images-k8s-v18-v1...:  333.57 MiB / 333.57 MiB  100.00% 3.18 Mi
* Creating docker container (CPUs=2, Memory=2200MB) ...
! Failing to connect to https://registry.k8s.io/ from inside the minikube container
* To pull new external images, you may need to configure a proxy: https://minikube.sigs.k8s.io/docs/reference/networking/proxy/
* Preparing Kubernetes v1.32.0 on Docker 27.4.1 ...
  - Generating certificates and keys ...
  - Booting up control plane ...
  - Configuring RBAC rules ...
* Configuring bridge CNI (Container Networking Interface) ...
* Verifying Kubernetes components...
  - Using image gcr.io/k8s-minikube/storage-provisioner:v5
* Enabled addons: storage-provisioner, default-storageclass
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default


---------------------------------------
ERROR:

PS C:\WINDOWS\system32> minikube start
* minikube v1.33.1 on Microsoft Windows 10 Enterprise 10.0.19045.4780 Build 19045.4780
* Automatically selected the docker driver. Other choices: hyperv, ssh
* Using Docker Desktop driver with root privileges
* Starting "minikube" primary control-plane node in "minikube" cluster
* Pulling base image v0.0.44 ...
* Downloading Kubernetes v1.30.0 preload ...
    > preloaded-images-k8s-v18-v1...:  342.90 MiB / 342.90 MiB  100.00% 7.44 Mi
    > gcr.io/k8s-minikube/kicbase...:  481.58 MiB / 481.58 MiB  100.00% 6.62 Mi
* Creating docker container (CPUs=2, Memory=8100MB) ...
* Stopping node "minikube"  ...
* Powering off "minikube" via SSH ...
* Deleting "minikube" in docker ...
! StartHost failed, but will try again: creating host: create: provisioning: get ssh host-port: get port 22 for "minikube": docker container inspect -f "'{{(index (index .NetworkSettings.Ports "22/tcp") 0).HostPort}}'" minikube: exit status 1
stdout:


stderr:
template parsing error: template: :1:4: executing "" at <index (index .NetworkSettings.Ports "22/tcp") 0>: error calling index: reflect: slice index out of range

* Failed to start docker container. Running "minikube delete" may fix it: error loading existing host. Please try running [minikube delete], then run [minikube start] again: filestore "minikube": open C:\Users\skmohan\.minikube\machines\minikube\config.json: The system cannot find the file specified.
! Startup with docker driver failed, trying with alternate driver hyperv: Failed to start host: error loading existing host. Please try running [minikube delete], then run [minikube start] again: filestore "minikube": open C:\Users\skmohan\.minikube\machines\minikube\config.json: The system cannot find the file specified.
* Deleting "minikube" in docker ...
* Removing C:\Users\skmohan\.minikube\machines\minikube ...

X Exiting due to GUEST_FILE_IN_USE: remove C:\Users\skmohan\.minikube\machines\minikube\id_rsa: The process cannot access the file because it is being used by another process.
* Suggestion: Another program is using a file required by minikube. If you are using Hyper-V, try stopping the minikube VM from within the Hyper-V manager
* Documentation: https://minikube.sigs.k8s.io/docs/reference/drivers/hyperv/
* Related issue: https://github.com/kubernetes/minikube/issues/7300

PS C:\WINDOWS\system32>

-------------------------------------------------------

Note:

If minikube is not working:

    Recreate the cluster by running:
    minikube delete --profile=minikube
    minikube start --profile=minikube



PS C:\WINDOWS\system32> minikube
minikube provisions and manages local Kubernetes clusters optimized for development workflows.


minikube kubectl -- get po -A

-----------------------------------------------------


PS C:\WINDOWS\system32> minikube start --driver=hyperv >>>>>>>>>>>>>>>>>>>>>>>>>> To start the minikube virtual env

ERROR:

PS C:\WINDOWS\system32> minikube start --memory=4096 --driver=hyperkit
* minikube v1.33.1 on Microsoft Windows 10 Enterprise 10.0.19045.4780 Build 19045.4780
E0904 19:01:27.191428   22196 start.go:812] api.Load failed for minikube: filestore "minikube": open C:\Users\skmohan\.minikube\machines\minikube\config.json: The system cannot find the file specified.

! Exiting due to GUEST_DRIVER_MISMATCH: The existing "minikube" cluster was created using the "docker" driver, which is incompatible with requested "hyperkit" driver.
* Suggestion: Delete the existing 'minikube' cluster using: 'minikube delete', or start the existing 'minikube' cluster using: 'minikube start --driver=docker'

PS C:\WINDOWS\system32>

PS C:\WINDOWS\system32> minikube start --memory=4096 --driver=hyperkit
* minikube v1.35.0 on Microsoft Windows 11 Home Single Language 10.0.26100.3775 Build 26100.3775

X Exiting due to DRV_UNSUPPORTED_OS: The driver 'hyperkit' is not supported on windows/amd64

PS C:\WINDOWS\system32>



PS C:\WINDOWS\system32> minikube start
* minikube v1.33.1 on Microsoft Windows 10 Enterprise 10.0.19045.4780 Build 19045.4780
* Automatically selected the docker driver. Other choices: hyperv, ssh
* Using Docker Desktop driver with root privileges
* Starting "minikube" primary control-plane node in "minikube" cluster
* Pulling base image v0.0.44 ...
* Creating docker container (CPUs=2, Memory=8100MB) ...
* Preparing Kubernetes v1.30.0 on Docker 26.1.1 ...
  - Generating certificates and keys ...
  - Booting up control plane ...
  - Configuring RBAC rules ...
* Configuring bridge CNI (Container Networking Interface) ...
* Verifying Kubernetes components...
  - Using image gcr.io/k8s-minikube/storage-provisioner:v5
* Enabled addons: storage-provisioner, default-storageclass
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
PS C:\WINDOWS\system32>



PS C:\WINDOWS\system32> kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   40s   v1.30.0
PS C:\WINDOWS\system32>

PS C:\WINDOWS\system32> minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured

PS C:\WINDOWS\system32>

Now Minikube is installed and created a single node k8s cluster successfuly.

**********************************************************************************************************************************
## INSTALL K8S CONTROLLER USING OPERATOR HUB

Link: https://operatorhub.io/operator/argocd-operator --> Install

![image](https://github.com/user-attachments/assets/fe2e212a-cb90-4d13-a4a3-dc820aeec799)

## Execute the below command in your laptop powershell:

## Install on Kubernetes:

## Install Operator Lifecycle Manager (OLM), a tool to help manage the Operators running on your cluster. >>> To install this, need to open Gitbash on windows and execute the command

$ curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.31.0/install.sh | bash -s v0.31.0

## Install the operator by running the following command:What happens when I execute this command?

$ kubectl create -f https://operatorhub.io/install/argocd-operator.yaml

This Operator will be installed in the "operators" namespace and will be usable from all namespaces in the cluster.

After install, watch your operator come up using next command.

$ kubectl get csv -n operators

To use it, checkout the custom resource definitions (CRDs) introduced by this operator to start using it.

## VERIFY THE CI PART:
--------------------
After all the above steps are completed
- Goto Jenkins Dashboard --> ultimate-demo --> Build now --> Check for status and check for the Jenkins logs
- Goto Sonarqube UI --> Projects --> spring-boot-demo --> Check for the code smells.
- Goto dockerhub and verify if the docker image is created, also check in the local server where Jenkins is running (docker images)

**********************************************************************************************************************************
## TO USE ARGOCD TO DEPLOY ON K8S:

- Create an ArgoCD controller
  Goto https://argocd-operator.readthedocs.io/en/latest/usage/basics/ --> create a new Argo CD cluster with the default configuration

apiVersion: argoproj.io/v1alpha1
kind: ArgoCD
metadata:
  name: example-argocd
  labels:
    example: basic
spec: {}


Create a file from PowerShell:

"apiVersion: argoproj.io/v1alpha1
kind: ArgoCD
metadata:
  name: example-argocd
  labels:
    example: basic
spec: {}" | Out-File argocd-basics.yaml

# kubectl apply -f argocd-basics.yaml

PS C:\WINDOWS\system32> kubectl get pods
NAME                                          READY   STATUS    RESTARTS   AGE
example-argocd-application-controller-0       1/1     Running   0          2m41s
example-argocd-redis-5678d59479-fr4kd         1/1     Running   0          2m42s
example-argocd-repo-server-6ddfc9947f-6hsmq   1/1     Running   0          2m42s
example-argocd-server-5b9d85449b-xk6g7        1/1     Running   0          2m42s

PS C:\WINDOWS\system32> kubectl get svc
NAME                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
example-argocd-metrics          ClusterIP   10.97.126.39    <none>        8082/TCP            2m50s
example-argocd-redis            ClusterIP   10.110.17.73    <none>        6379/TCP            2m50s
example-argocd-repo-server      ClusterIP   10.99.228.202   <none>        8081/TCP,8084/TCP   2m50s
example-argocd-server           ClusterIP   10.98.31.93     <none>        80/TCP,443/TCP      2m50s
example-argocd-server-metrics   ClusterIP   10.111.146.47   <none>        8083/TCP            2m50s
kubernetes                      ClusterIP   10.96.0.1       <none>        443/TCP             4h44m
PS C:\WINDOWS\system32> kubectl edit svc example-argocd-server
--> Change the Cluster type from ClusterIP to NodePort

# minikube service argocd-server

**********************************************************************************************************************************

**Prerequisites:**

   -  Java application code hosted on a Git repository
   -  Jenkins server
   -  Install Maven
   -  Kubernetes cluster
   -  Helm package manager
   -  Argo CD

Steps:

    1. Install the necessary Jenkins plugins:
       1.1 Git plugin
       1.2 Maven Integration plugin
       1.3 Pipeline plugin
       1.4 Kubernetes Continuous Deploy plugin

    2. Create a new Jenkins pipeline:
       2.1 In Jenkins, create a new pipeline job and configure it with the Git repository URL for the Java application.
       2.2 Add a Jenkinsfile to the Git repository to define the pipeline stages.

    3. Define the pipeline stages:
        Stage 1: Checkout the source code from Git.
        Stage 2: Build the Java application using Maven.
        Stage 3: Run unit tests using JUnit and Mockito.
        Stage 4: Run SonarQube analysis to check the code quality.
        Stage 5: Package the application into a JAR file.
        Stage 6: Deploy the application to a test environment using Helm.
        Stage 7: Run user acceptance tests on the deployed application.
        Stage 8: Promote the application to a production environment using Argo CD.

    4. Configure Jenkins pipeline stages:
        Stage 1: Use the Git plugin to check out the source code from the Git repository.
        Stage 2: Use the Maven Integration plugin to build the Java application.
        Stage 3: Use the JUnit and Mockito plugins to run unit tests.
        Stage 4: Use the SonarQube plugin to analyze the code quality of the Java application.
        Stage 5: Use the Maven Integration plugin to package the application into a JAR file.
        Stage 6: Use the Kubernetes Continuous Deploy plugin to deploy the application to a test environment using Helm.
        Stage 7: Use a testing framework like Selenium to run user acceptance tests on the deployed application.
        Stage 8: Use Argo CD to promote the application to a production environment.

    5. Set up Argo CD:
        Install Argo CD on the Kubernetes cluster.
        Set up a Git repository for Argo CD to track the changes in the Helm charts and Kubernetes manifests.
        Create a Helm chart for the Java application that includes the Kubernetes manifests and Helm values.
        Add the Helm chart to the Git repository that Argo CD is tracking.

    6. Configure Jenkins pipeline to integrate with Argo CD:
       6.1 Add the Argo CD API token to Jenkins credentials.
       6.2 Update the Jenkins pipeline to include the Argo CD deployment stage.

    7. Run the Jenkins pipeline:
       7.1 Trigger the Jenkins pipeline to start the CI/CD process for the Java application.
       7.2 Monitor the pipeline stages and fix any issues that arise.

This end-to-end Jenkins pipeline will automate the entire CI/CD process for a Java application, from code checkout to production deployment, using popular tools like SonarQube, Argo CD, Helm, and Kubernetes.
