# IaC integrated DevOps Mini Project

This project consists of a FastAPI backend and a Next.js frontend that communicates with the backend.

## Architecture
```
Internet → ALB → Frontend Service (Next.js)
              ↘ /api/* → Backend Service (FastAPI)
```
- Frontend: Next.js application serving the web interface
- Backend: FastAPI providing REST APIs
- Infrastructure: AWS ECS Fargate with Application Load Balancer
- Container Registry: AWS ECR for Docker images
- Orchestration: Terraform for infrastructure as code
## Project Structure

```
.../
├── backend/               # FastAPI backend
│   ├── app/
│   │   └── main.py       # Main FastAPI application
│   └── requirements.txt    # Python dependencies
│── frontend/              # Next.js frontend
│    ├── pages/
│    │   └── index.js     # Main page
│    ├── public/            # Static files
│    └── package.json       # Node.js dependencies
└── infra/
      ├──ecr.tf            # IaC for Elastic Container Registry
      ├──ecs.tf            # IaC for Elastic Container Service
      └──terraform.tfstate

```
### Features
- Containerized Applications: Docker containers for both frontend and backend
- Cloud-Native Architecture: AWS ECS Fargate serverless containers
- Load Balancing: Application Load Balancer with path-based routing
- Infrastructure as Code: Complete Terraform automation
- CI/CD Ready: Automated testing and deployment pipeline structure
- Security: Proper VPC, security groups, and IAM configurations
- Monitoring: CloudWatch logging integration
- Health Checks: Application and infrastructure health monitoring
## Prerequisites

- Python 3.8+
- Node.js 16+
- npm or yarn
- AWS CLI configured on local machine

## Setup and Deployment
### Initialising AWS CLI
1. Install AWS CLI on your machine
   - For Windows:
      - Download the MSI installer from:
        https://awscli.amazonaws.com/AWSCLIV2.msi
      - Run the installer.
   - For Linux/Ubuntu:
     ```bash
     curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
     unzip awscliv2.zip
     sudo ./aws/install
     ```
   - For MacOS
     ```bash
     brew install awscli
     # OR
     curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
     sudo installer -pkg AWSCLIV2.pkg -target /
     ```
2. Verify Installation
   ```bash
   aws --version
   ```

3. Configure AWS CLI
   ```bash
   aws configure
   ```


   After creating the IAM user, don't forget to copy the Access Key and the Secret Access Key and then fill out the following fields:
   ```bash
   AWS Access Key ID:     <your access key>
   AWS Secret Access Key: <your secret key>
   Default region name:   ap-south-1 (or your region)
   Default output format: json (Generally put it as json only)
   ```
And you are done setting up your AWS CLI on the local machine.

### Step 1: Configure AWS CLI
```bash
aws sts get-caller-identity
```
### Setting up Terraform on the machine.
1. Installing Terraform on the machine
   - For Windows:
      - Download from: https://developer.hashicorp.com/terraform/downloads
      - Extract the `.zip` and place `terraform.exe` in a folder (e.g., `C:\Terraform`).
      - Add that folder to your System PATH:
         - Search for "Environment Variables" → Edit the PATH variable → Add `C:\Terraform`.
   - For Linux/Ubuntu:
     ```bash
     sudo apt-get update && sudo apt-get install -y gnupg software-properties-common curl
     curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
     echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
     https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
     sudo tee /etc/apt/sources.list.d/hashicorp.list
     sudo apt update
     sudo apt install terraform
     ```
   - For MacOS:
     ```bash
     brew tap hashicorp/tap
     brew install hashicorp/tap/terraform
     ```
2. Verify Installation
   ```bash
   terraform -v
   ```
### Step 2: Create ECR Repositories
```bash
cd infra
terraform init
terraform apply -target=aws_ecr_repository.backend -target=aws_ecr_repository.frontend
```
### Containerising the app
1. Install Docker
   - For Windows:
      - Download Docker Desktop: https://www.docker.com/products/docker-desktop
      - Install the EXE:
         - Run the Installer.
         - Enable WSL 2 and Hyper-V when prompted (if not already enabled).
         - Reboot when asked.
      - Start Docker Desktop from the Start menu.
   - For Linux/Ubuntu:
     ```bash
     sudo apt-get update
     sudo apt-get install \
     ca-certificates \
     curl \
     gnupg \
     lsb-release

     # Add Docker’s GPG key:
     sudo mkdir -m 0755 -p /etc/apt/keyrings
     curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
     sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

     # Add repo:
     echo \
     "deb [arch=$(dpkg --print-architecture) \
     signed-by=/etc/apt/keyrings/docker.gpg] \
     https://download.docker.com/linux/ubuntu \
     $(lsb_release -cs) stable" | \
     sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

     # Install Docker Engine:
     sudo apt-get update
     sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

     # Enable and start Docker
     sudo systemctl enable docker
     sudo systemctl start docker

     # Allow non-root user (optional)
     sudo usermod -aG docker $USER
     newgrp docker
     ```
   - For MacOS:
      - Download Docker Desktop: https://www.docker.com/products/docker-desktop
      - Install it like any other `.dmg` file.
      - Start Docker Desktop from Launchpad.
3. Verify Docker Installation
   ```bash
   docker --version
   ```

### Step 3: Build and Push Docker Images
**Authenticate Docker with ECR:**
```bash
aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin ACCOUNT-ID.dkr.ecr.ap-south-1.amazonaws.com
```
**Build and Push Backend:**
```bash
cd backend
docker build -t ACCOUNT-ID.dkr.ecr.ap-south-1.amazonaws.com/devops-backend:latest .
docker push ACCOUNT-ID.dkr.ecr.ap-south-1.amazonaws.com/devops-backend:latest
```
**Build and Push Frontend:**
```bash
cd frontend
docker build -t ACCOUNT-ID.dkr.ecr.ap-south-1.amazonaws.com/devops-frontend:latest .
docker push ACCOUNT-ID.dkr.ecr.ap-south-1.amazonaws.com/devops-frontend:latest
```
Replace `ACCOUNT-ID` with your actual AWS account ID

### Step 4: Deploy Infrastructure
```bash
cd infra
terraform plan
terraform apply
```
This will create:

- VPC with public subnets
- Application Load Balancer
- ECS Cluster and Services
- Security Groups
- CloudWatch Log Groups
- IAM Roles and Policies
### Step 5: Access the Application
**After deployment completes, get the ALB URL:**
```bash
terraform output alb_dns_name
```
Visit the URL to access your application:

- Frontend: `http://your-alb-dns-name`
- Backend API: `http://your-alb-dns-name/api/health`
## Local Machine Setup

### Backend Setup

1. Navigate to the backend directory:
   ```bash
   cd backend
   ```

2. Create a virtual environment (recommended):
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: .\venv\Scripts\activate
   ```

3. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

4. Run the FastAPI server:
   ```bash
   uvicorn app.main:app --reload --port 8000
   ```

   The backend will be available at `http://localhost:8000`

### Frontend Setup

1. Navigate to the frontend directory:
   ```bash
   cd frontend
   ```

2. Install dependencies:
   ```bash
   npm install
   # or
   yarn
   ```

3. Configure the backend URL (if different from default):
   - Open `.env.local`
   - Update `NEXT_PUBLIC_API_URL` with your backend URL
   - Example: `NEXT_PUBLIC_API_URL=https://your-backend-url.com`

4. Run the development server:
   ```bash
   npm run dev
   # or
   yarn dev
   ```

   The frontend will be available at `http://localhost:3000`


### API Endpoints

- `GET /api/health`: Health check endpoint
  - Returns: `{"status": "healthy", "message": "Backend is running successfully"}`

- `GET /api/message`: Get the integration message
  - Returns: `{"message": "You've successfully integrated the backend!"}`







   After creating the IAM user, don't forget to copy the Access Key and the Secret Access Key and then fill out the following fields:
   ```bash
   AWS Access Key ID:     <your access key>
   AWS Secret Access Key: <your secret key>
   Default region name:   ap-south-1 (or your region)
   Default output format: json (Generally put it as json only)
   ```
And you are done setting up your AWS CLI on the local machine.



## After checking that the app is running successfully, run the following command:
```bash
terraform destroy
```



