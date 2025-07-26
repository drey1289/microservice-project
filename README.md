# Video-to-Audio Converter Microservices
## Project Overview
This project is a Python-based microservices application that allows users to:
- Upload video files (.mp4)
- Automatically convert them to audio (.mp3)
- Download the converted file
- Uses RabbitMQ, MongoDB, PostgreSQL, and runs on Kubernetes (EKS) with infrastructure provisioned via Terraform.

- ## Architecture Overview
- ### Microservices
- Auth Service: Handles user login and JWT token generation
- Converter Service: Converts .mp4 videos to .mp3
- Notification Service: Sends email notification with file ID
- Gateway Service: Acts as API Gateway to route requests

- ## Infrastructure Provisioning (via Terraform)
- ### Clone the repository
 ```
git clone <your-repo-url>
cd microservices-python-app
```
## Create EKS Cluster using Terraform
Terraform config creates:
- EKS Cluster
- Managed Node Group
- IAM roles and security groups
- Filters out unsupported AZs (e.g. us-east-1e)
```
terraform init
terraform validate
terraform plan
terraform apply
```
## Update kubeconfig to use new cluster
```
aws eks update-kubeconfig --name <cluster-name> --region <region>
```
## Deploy Dependencies
### MongoDB
- Deploy via manifest
- Stores file metadata
### PostgreSQL
- Stores user credentials
### RabbitMQ
- For communication between microservices
- Access it via browser at:
```
http://<nodeIP>:30004
```
Login: guest / guest
Create two queues:
- mp3
- video
## Dockerize Microservices
Build and Push Images:
```
# Build and push the auth-service image
docker build -t <docker-username>/auth ./auth-service
docker push <docker-username>/auth

# Build and push the converter-service image
docker build -t <docker-username>/converter ./converter-service
docker push <docker-username>/converter

# Build and push the gateway-service image
docker build -t <docker-username>/gateway ./gateway-service
docker push <docker-username>/gateway

# Build and push the notification-service image
docker build -t <docker-username>/notification ./notification-service
docker push <docker-username>/notification
```
## Deploy each microservice to Kubernetes:
```
kubectl apply -f src/<service-name>/manifest/
```
## Check pods:
After deploying the microservices, verify the status of all components by running:
```
kubectl get pods
```
## Application Testing Steps
### Login Endpoint (POST)
- URl:
```
curl -X POST http://nodeIP:30002/login -u <email>:<password>
```
- Replace with your actual email and password
- Returns: JWT Token
### upload Endpoint
```
 curl -X POST -F 'file=@./video.mp4' -H 'Authorization: Bearer <JWT Token>' http://nodeIP:30002/upload
```
check yoyur email for fid(file id)
### Download Endpoint
```
 curl --output video.mp3 -X GET -H 'Authorization: Bearer <JWT Token>' "http://nodeIP:30002/download?fid=<Generated fid>"
```
### Can I Test This in a UI Instead?
- goto https://www.postman.com/downloads/ to download and install Postman
- create a new account if you dont have one or login to an existing acct
- set Login Endpoint (POST) to get a token
- upload endpoint(post), to get a fid (file id) that will b used to downlaod the file.
- use postman to downlaod the mp3 version.
- 
## API Definitions
### Login
- Purpose: Authenticates the user using email and password.
- Outcome: Returns a JWT token that authorizes access to protected endpoints.
- Why It Matters: The token is required for securely uploading and downloading files.
### Upload
- Purpose: Uploads a .mp4 video file.
- Process: The uploaded video is sent to the backend, processed, and converted to .mp3 format.
- Outcome: A File ID (fid) is generated and sent to the user’s email.
### Downlaod
- Purpose: Downloads the converted .mp3 file.
- Requirement: Requires the fid (received via email) and the JWT token.
- Outcome: Returns the .mp3 file for download.
## Destroy the Infrastructure
- To destroy, use:
```
terraform destroy
```
