**STEP 1: Fork & Sync Repository**

**Fork the repo**

Go to:
[https://github.com/UnpredictablePrashant/StreamingApp](https://github.com/UnpredictablePrashant/StreamingApp)

Click **Fork** → create your own copy.

**Clone your fork**

git clone https://github.com/yogeshvshinde/StreamingApp.git

cd StreamingApp

![]()

**Add upstream**

git remote add upstream [https://github.com/UnpredictablePrashant/StreamingApp.git](https://github.com/UnpredictablePrashant/StreamingApp.git)

**Sync updates**

git fetch upstream

git checkout main

git merge upstream/main

git push origin main

![]()

**STEP 2: Containerize MERN App**

**🔹 Structure**

frontend/

backend/admin-service

backend/auth-service

backend/chat-service

backend/streaming-service

**Install dependencies for each service:**

\# auth service

cd backend/authService && npm install

\# streaming service

cd ../streamingService && npm install

\# admin service

cd ../adminService && npm install

\# chat service

cd ../chatService && npm install

\# frontend

cd ../../frontend && npm install

**Run the services (in separate terminals) after starting MongoDB:**

cd backend/authService && npm run dev

cd backend/streamingService && npm run dev

cd backend/adminService && npm run dev

cd backend/chatService && npm run dev

cd frontend && npm start

**Below are the port numbers of the services,**

**Service**

**Port**

**Description**

authService

3001

User authentication, registration, JWT issuance

streamingService

3002

Video catalogue, S3 playback endpoints, public APIs

adminService

3003

Dedicated admin microservice for asset management and uploads

chatService

3004

Websocket + REST chat for live watch parties

frontend

3000

React SPA with revamped UI and integrated chat

mongo

27017

Shared MongoDB instance

**Create docker files as per requirement.**

**🔹 Backend Dockerfile**

Create backend dockerfiles

![]()

![]()

![]()

![]()

**Frontend Dockerfile**

Create frontend/Dockerfile

**Test locally (important before CI)**

cd frontend && docker build -t frontend .

cd authService && docker build -t auth-service .

cd . ./streamingService && docker build -t streaming-service .

cd ../ adminService && docker build -t admin-service .

cd ../chatService && docker build -t chat-service .
docker run -p 3000:80 frontend
docker run -p 5000:5000 backend

![]()

![]()

Test video upload service:

![]()

Test Chat service
![]()

![]()

![]()

**STEP 3: AWS CLI Setup**

Install AWS CLI and configure:

aws configure

![]()

Enter:

*   Access Key
*   Secret Key
*   Region (e.g., ap-south-1)

**STEP 4: Push Images to ECR**

**🔹 Create repos**

aws ecr create-repository --repository-name frontend

aws ecr create-repository --repository-name admin-service

aws ecr create-repository --repository-name chat-service

aws ecr create-repository --repository-name streaming-service

aws ecr create-repository --repository-name auth-service

**🔹 Login to ECR**

aws ecr get-login-password --region ap-south-1 | \\
docker login --username AWS --password-stdin <ACCOUNT\_ID>.dkr.ecr.ap-south-1.amazona

![]()

🔹 Login to ECR

aws ecr get-login-password --region ap-south-1 | \\

docker login --username AWS --password-stdin <ACCOUNT\_ID>.dkr.ecr.ap-south-1.amazonaws.com

![]()

🔹 Tag & Push

![]()

**STEP 5: Jenkins CI Pipeline**

Since we already have Jenkins. We need to check if below plugins are available and installed.

**🔹 Install plugins**

In Jenkins:

*   Git
*   Docker Pipeline
*   Pipeline
*   AWS Credentials

All plugins were already installed.

**🔹 Add credentials**

*   GitHub token
*   AWS credentials

Build docker images locally to test :

Command as below,

cd authService && docker build -t auth-service .

cd ../streamingService && docker build -t streaming-service .

cd ../adminService && docker build -t admin-service .

cd ../chatService && docker build -t chat-service .

![]()

![]()

![]()

**Step 5: Build Docker images**

![]()

![]()

![]()

**Now run the app locally and test it:**

![]()

Tag the images for ECRs

![]()

Push all the images to respective ECRs

![]()

**Step 6: Create EKS Cluster**

Using eksctl:

eksctl create cluster \\
\--name mern-cluster \\
\--region ap-south-1

![]()

**Step 7: Configure kubectl**

aws eks --region ap-south-1 update-kubeconfig --name mern-cluster

![]()

![]()

Create yaml files to deploy images from ECR to kubernetes pods.

Create these files for all services.

![]()

**STEP 8: Apply Deployments**

Run command: Kubectl apply -f .

![]()

![]()

![]()

**STEP 9: Get Public URL**

kubectl get svc

Run the kubectl get svc command to get the External Ip to access the application.

![]()

Access the application using the external ip

[http://ExternalIP](http://ExternalIP)
http://ad55a5859dada47a38f973847e00b9ae-731481676.ap-south-1.elb.amazonaws.com/

![]()

**Update the user role from user to admin by connecting to the mongo db and changing the role to admin for a user to upload the video.**

**Below are the commands,**

Run this against your EKS Mongo pod.

First open Mongo shell:

**kubectl exec -it deployment/mongo -- mongosh authdb**

List users and roles:

**db.users.find({}, { email: 1, role: 1 }).pretty()**

Update your user to admin:

**db.users.updateOne(**

**{ email: "your-email@example.com" },**

**{ $set: { role: "admin" } }**

**)**

Verify:

**db.users.find(**

**{ email: "your-email@example.com" },**

**{ email: 1, role: 1 }**

**).pretty()**

**![]()**

**Video upload was successful.**

**![]()**

**Other user is able to view the videos by browse feature as below,
![]()**

**Now test the chat functionality.
**

**![]()**

**![]()
**

**STEP 10: Automate via Jenkins (CI/CD)**

Now update your Jenkins pipeline to:

**Add stages:**

**![]()**

Add a webhook on Github for Jenkins server:
![]()
**Enable below plugins on Jenkins server:**

aws cli

kubectl

docker

git

**Add below credentials to Jenkins Credentials:
**github-creds

jwt-secret

aws-access-key-id

aws-secret-access-key

s3-bucket-name

![]()

Install AWS CLI and Kubectl on Jenkins server.

**SSH Into Jenkins VM**

**Install Base Tools**

sudo apt update

sudo apt install -y unzip curl git ca-certificates
**Install AWS CLI v2**
cd /tmp

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86\_64.zip" -o "awscliv2.zip"

unzip -o awscliv2.zip

sudo ./aws/install --update

aws –version

**Install kubectl
**cd /tmp

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x kubectl

sudo mv kubectl /usr/local/bin/kubectl

kubectl version –client
**Make Sure Jenkins User Can Run Docker**

**If Jenkins builds Docker images on this VM**:
sudo usermod -aG docker jenkins

sudo systemctl restart Jenkins
**Test AWS Credentials From Jenkins Pipeline**
Since credentials are stored in Jenkins, the best test is a small Jenkins stage/job:
![]()

![]()

**Test Kubectl access by creating a Jenkins Test Pipeline:**

**![]()
![]()**

**Now push a test change from local machine to Github and check if webhook shows recent deliveries:**

**![]()**

**Check on the Jenkins server if a build job is triggered automatically using SCM,
![]()**

**![]()**

**![]()**

**Test application :**

**![]()
AWS ECR :**

**![]()
EKS Cluster screenshot:**

**![]()**

**
![]()**

**![]()**

**Kubernetes cluster pods,
![]()
**