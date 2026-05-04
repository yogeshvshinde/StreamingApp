**STEP 1: Fork & Sync Repository**

**Fork the repo**

Go to:  
<https://github.com/UnpredictablePrashant/StreamingApp>

Click **Fork** → create your own copy.

--------------------------------------------------------------

**Clone your fork**

git clone https://github.com/yogeshvshinde/StreamingApp.git

cd StreamingApp

<img width="940" height="211" alt="image" src="https://github.com/user-attachments/assets/a2d4eb51-59a4-4290-a73e-92e0fe60f2ba" />




**Add upstream**

git remote add upstream <https://github.com/UnpredictablePrashant/StreamingApp.git>

**Sync updates**

git fetch upstream

git checkout main

git merge upstream/main

git push origin main

![](media/image2.png){width="6.268055555555556in" height="2.2555555555555555in"}

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

| **Service**      | **Port** | **Description**                                               |
|------------------|----------|---------------------------------------------------------------|
| authService      | 3001     | User authentication, registration, JWT issuance               |
| streamingService | 3002     | Video catalogue, S3 playback endpoints, public APIs           |
| adminService     | 3003     | Dedicated admin microservice for asset management and uploads |
| chatService      | 3004     | Websocket + REST chat for live watch parties                  |
| frontend         | 3000     | React SPA with revamped UI and integrated chat                |
| mongo            | 27017    | Shared MongoDB instance                                       |

**Create docker files as per requirement.**

**🔹 Backend Dockerfile**

Create backend dockerfiles

![](media/image3.png){width="6.268055555555556in" height="2.6034722222222224in"}

![](media/image4.png){width="6.268055555555556in" height="2.803472222222222in"}

![](media/image5.png){width="6.268055555555556in" height="2.9145833333333333in"}

![](media/image6.png){width="6.268055555555556in" height="3.797222222222222in"}

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

![](media/image7.png){width="6.268055555555556in" height="3.801388888888889in"}

![](media/image8.png){width="6.268055555555556in" height="4.313194444444444in"}

Test video upload service:

![](media/image9.jpeg){width="6.268055555555556in" height="3.3222222222222224in"}

Test Chat service  
  
  
![](media/image10.png){width="6.268055555555556in" height="4.646527777777778in"}

![](media/image11.png){width="6.268055555555556in" height="3.0965277777777778in"}

![](media/image12.png){width="6.268055555555556in" height="1.6715277777777777in"}

**STEP 3: AWS CLI Setup**

Install AWS CLI and configure:

aws configure

![](media/image13.png){width="6.268055555555556in" height="1.6090277777777777in"}

Enter:

- Access Key

- Secret Key

- Region (e.g., ap-south-1)

**STEP 4: Push Images to ECR**

**🔹 Create repos**

aws ecr create-repository \--repository-name frontend

aws ecr create-repository \--repository-name admin-service

aws ecr create-repository \--repository-name chat-service

aws ecr create-repository \--repository-name streaming-service

aws ecr create-repository \--repository-name auth-service

**🔹 Login to ECR**

aws ecr get-login-password \--region ap-south-1 \| \\  
docker login \--username AWS \--password-stdin \<ACCOUNT_ID\>.dkr.ecr.ap-south-1.amazona

![](media/image14.png){width="6.268055555555556in" height="4.298611111111111in"}

🔹 Login to ECR

aws ecr get-login-password \--region ap-south-1 \| \\

docker login \--username AWS \--password-stdin \<ACCOUNT_ID\>.dkr.ecr.ap-south-1.amazonaws.com

![](media/image15.png){width="6.268055555555556in" height="1.1in"}

🔹 Tag & Push

![](media/image16.png){width="6.268055555555556in" height="3.566666666666667in"}

**STEP 5: Jenkins CI Pipeline**

Since we already have Jenkins. We need to check if below plugins are available and installed.

**🔹 Install plugins**

In Jenkins:

- Git

- Docker Pipeline

- Pipeline

- AWS Credentials

All plugins were already installed.

**🔹 Add credentials**

- GitHub token

- AWS credentials

Build docker images locally to test :

Command as below,

cd authService && docker build -t auth-service .

cd ../streamingService && docker build -t streaming-service .

cd ../adminService && docker build -t admin-service .

cd ../chatService && docker build -t chat-service .

![](media/image17.png){width="6.268055555555556in" height="2.8243055555555556in"}

![](media/image18.png){width="6.268055555555556in" height="3.3513888888888888in"}

![](media/image19.png){width="6.268055555555556in" height="3.5166666666666666in"}

**Step 5: Build Docker images**

![](media/image20.png){width="6.268055555555556in" height="0.9805555555555555in"}

![](media/image21.png){width="6.268055555555556in" height="3.303472222222222in"}

![](media/image22.png){width="6.268055555555556in" height="4.532638888888889in"}

**Now run the app locally and test it:**

![](media/image23.png){width="5.8217290026246715in" height="4.348558617672791in"}

Tag the images for ECRs

![](media/image24.png){width="6.268055555555556in" height="0.5673611111111111in"}

Push all the images to respective ECRs

![](media/image25.png){width="6.268055555555556in" height="1.9013888888888888in"}

**Step 6: Create EKS Cluster**

Using eksctl:

eksctl create cluster \\  
\--name mern-cluster \\  
\--region ap-south-1

![](media/image26.png){width="6.268055555555556in" height="3.3194444444444446in"}

**Step 7: Configure kubectl**

aws eks \--region ap-south-1 update-kubeconfig \--name mern-cluster

![](media/image27.png){width="6.268055555555556in" height="2.813888888888889in"}

![](media/image28.png){width="6.268055555555556in" height="2.970833333333333in"}

Create yaml files to deploy images from ECR to kubernetes pods.

Create these files for all services.

![](media/image29.png){width="6.268055555555556in" height="3.4034722222222222in"}

**STEP 8: Apply Deployments**

Run command: Kubectl apply -f .

![](media/image30.png){width="6.268055555555556in" height="3.5069444444444446in"}

![](media/image30.png){width="6.268055555555556in" height="3.5069444444444446in"}

![](media/image31.png){width="6.268055555555556in" height="3.0319444444444446in"}

**STEP 9: Get Public URL**

kubectl get svc

Run the kubectl get svc command to get the External Ip to access the application.

![](media/image32.png){width="6.268055555555556in" height="2.2868055555555555in"}

Access the application using the external ip

<http://ExternalIP>  
http://ad55a5859dada47a38f973847e00b9ae-731481676.ap-south-1.elb.amazonaws.com/

![](media/image33.png){width="6.268055555555556in" height="5.192361111111111in"}

**Update the user role from user to admin by connecting to the mongo db and changing the role to admin for a user to upload the video.**

**Below are the commands,**

Run this against your EKS Mongo pod.

First open Mongo shell:

**kubectl exec -it deployment/mongo \-- mongosh authdb**

List users and roles:

**db.users.find({}, { email: 1, role: 1 }).pretty()**

Update your user to admin:

**db.users.updateOne(**

**{ email: \"your-email@example.com\" },**

**{ \$set: { role: \"admin\" } }**

**)**

Verify:

**db.users.find(**

**{ email: \"your-email@example.com\" },**

**{ email: 1, role: 1 }**

**).pretty()**

![](media/image34.png){width="6.268055555555556in" height="4.847222222222222in"}

**Video upload was successful.**

![](media/image35.png){width="6.268055555555556in" height="3.5375in"}

**Other user is able to view the videos by browse feature as below,  
  
**![](media/image36.png){width="6.268055555555556in" height="3.4256944444444444in"}

**Now test the chat functionality.  
**

![](media/image37.png){width="6.268055555555556in" height="3.3305555555555557in"}

![](media/image38.png){width="6.268055555555556in" height="4.834027777777778in"}**  
**

**STEP 10: Automate via Jenkins (CI/CD)**

Now update your Jenkins pipeline to:

**Add stages:**

![](media/image39.png){width="6.268055555555556in" height="3.61875in"}

Add a webhook on Github for Jenkins server:  
  
![](media/image40.png){width="6.268055555555556in" height="5.102777777777778in"}  
  
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

![](media/image41.png){width="6.268055555555556in" height="4.91875in"}

Install AWS CLI and Kubectl on Jenkins server.

**SSH Into Jenkins VM**

**Install Base Tools**

sudo apt update

sudo apt install -y unzip curl git ca-certificates  
  
**Install AWS CLI v2**  
  
cd /tmp

curl \"https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip\" -o \"awscliv2.zip\"

unzip -o awscliv2.zip

sudo ./aws/install \--update

aws --version

**Install kubectl  
**cd /tmp

curl -LO \"https://dl.k8s.io/release/\$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl\"

chmod +x kubectl

sudo mv kubectl /usr/local/bin/kubectl

kubectl version --client  
  
**Make Sure Jenkins User Can Run Docker**

**If Jenkins builds Docker images on this VM**:  
  
sudo usermod -aG docker jenkins

sudo systemctl restart Jenkins  
  
**Test AWS Credentials From Jenkins Pipeline**  
Since credentials are stored in Jenkins, the best test is a small Jenkins stage/job:  
  
![](media/image42.png){width="6.268055555555556in" height="4.586805555555555in"}

![](media/image43.png){width="5.736406386701662in" height="4.9655325896762905in"}

**Test Kubectl access by creating a Jenkins Test Pipeline:**

![](media/image44.png){width="6.268055555555556in" height="4.871527777777778in"}**  
  
**![](media/image45.png){width="6.268055555555556in" height="5.165277777777778in"}

**Now push a test change from local machine to Github and check if webhook shows recent deliveries:**

![](media/image46.png){width="6.268055555555556in" height="2.717361111111111in"}

**Check on the Jenkins server if a build job is triggered automatically using SCM,  
  
**![](media/image47.png){width="6.268055555555556in" height="5.095833333333333in"}

![](media/image48.png){width="6.268055555555556in" height="4.947916666666667in"}

![](media/image49.png){width="6.268055555555556in" height="4.91875in"}

**Test application :**

![](media/image50.png){width="6.268055555555556in" height="4.334027777777778in"}**  
  
AWS ECR :**

![](media/image51.png){width="6.268055555555556in" height="3.4402777777777778in"}**  
  
EKS Cluster screenshot:**

![](media/image52.png){width="6.268055555555556in" height="3.904861111111111in"}

**  
  
**![](media/image53.png){width="6.268055555555556in" height="4.3902777777777775in"}

![](media/image54.png){width="6.268055555555556in" height="5.813194444444444in"}

**Kubernetes cluster pods,  
  
**![](media/image55.png){width="6.268055555555556in" height="1.0340277777777778in"} **  
  
**
