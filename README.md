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
.
├── .github/workflows/
│   └──ci-cd.yml           # GitHub Actions CI/CD pipeline  
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
### Step 1: Configure AWS CLI
```
aws configure
# output something like this:
AWS Access Key ID [****************CZUR]: <your-access-key>
AWS Secret Access Key [****************T391]: <your-secret-access-key>
Default region name [ap-south-1]: ap-south-1
Default output format [json]: json
aws sts get-caller-identity
```
### Step 2: Create ECR Repositories
```
cd infra
terraform init
terraform apply -target=aws_ecr_repository.backend -target=aws_ecr_repository.frontend
```
### Step 3: Build and Push Docker Images
*Authenticate Docker with ECR:*
```
aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin ACCOUNT-ID.dkr.ecr.ap-south-1.amazonaws.com
```
*Build and Push Backend:*
```
cd backend
docker build -t ACCOUNT-ID.dkr.ecr.ap-south-1.amazonaws.com/devops-backend:latest .
docker push ACCOUNT-ID.dkr.ecr.ap-south-1.amazonaws.com/devops-backend:latest
```
*Build and Push Frontend:*
```
cd frontend
docker build -t ACCOUNT-ID.dkr.ecr.ap-south-1.amazonaws.com/devops-frontend:latest .
docker push ACCOUNT-ID.dkr.ecr.ap-south-1.amazonaws.com/devops-frontend:latest
```
Replace `ACCOUNT-ID` with your actual AWS account ID

### Step 4: Deploy Infrastructure
```
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
After deployment completes, get the ALB URL:
```
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

### Changing the Backend URL

To change the backend URL that the frontend connects to:

1. Open the `.env.local` file in the frontend directory
2. Update the `NEXT_PUBLIC_API_URL` variable with your new backend URL
3. Save the file
4. Restart the Next.js development server for changes to take effect

Example:
```
NEXT_PUBLIC_API_URL=https://your-new-backend-url.com
```

### For deployment:
   ```bash
   npm run build
   # or
   yarn build
   ```

   AND

   ```bash
   npm run start
   # or
   yarn start
   ```

   The frontend will be available at `http://localhost:3000`

### Testing the Integration

1. Ensure both backend and frontend servers are running
2. Open the frontend in your browser (default: http://localhost:3000)
3. If everything is working correctly, you should see:
   - A status message indicating the backend is connected
   - The message from the backend: "You've successfully integrated the backend!"
   - The current backend URL being used
4. Alternatively: We can integrate Keploy API testing using [Keploy's Browser Extension](https://github.com/keploy/extension)
5. We have also implemented `cypress` for Frontend unit testing. This is to be strictly implemented during the CI/CD so that the code can be passed for containerising.
6. 

### API Endpoints

- `GET /api/health`: Health check endpoint
  - Returns: `{"status": "healthy", "message": "Backend is running successfully"}`

- `GET /api/message`: Get the integration message
  - Returns: `{"message": "You've successfully integrated the backend!"}`

### Dockerising the app
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

4. Build the docker image
   - In this project, there are 2 directories; `frontend` and `backend`, so we will create 2 dockerfiles in the 2 directories and build 2 separate docker images of the frontend-app and the backend-app.
   - So we will run the following commands from the root directory:
   ```bash
   docker build --platform=linux/amd64 -t devops-backend ./backend
   docker build --platform=linux/amd64 -t devops-frontend ./frontend
   ```

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
   Upon executing the above command, you'll be prompted to enter a few fields. But for that, you'll have to create an IAM user in your AWS account to grant access to the CLI in your machine.

   While creating your IAM user, make sure to assign the following roles:
   - `AmazonEC2ContainerRegistryFullAccess`
   - `AmazonECS_FullAccess`
   - `IAMFullAccess`

   After creating the IAM user, don't forget to copy the Access Key and the Secret Access Key and then fill out the following fields:
   ```bash
   AWS Access Key ID:     <your access key>
   AWS Secret Access Key: <your secret key>
   Default region name:   ap-south-1 (or your region)
   Default output format: json (Generally put it as json only)
   ```
And you are done setting up your AWS CLI on the local machine.

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
3. As we are building the infrastructure for this project, we will create a separate folder for it and build the `.tf` files in this folder.
   ```bash
   cd .\DevOps-Assignment\infra
   ```
   - After that we will simply start building the infrastructure.
   - So in the `ecr.tf` we have set up the Container Registry for storing the Docker Images that have been built locally.
   - In the `ecs.tf` we have set up the ECS Fargate service to host the application images that are present in the Container Registry.
   After everything is done and dusted, run the following commands:
   ```bash
   terraform init
   terraform plan
   terraform apply
   ```

## What the CI/CD pipeline does
### CI: Continuous Integration (Triggered on Push to dev or main)
#### 1. Checkout Code: Clones the repo to the GitHub Actions runner.

#### 2. Set up Python: Installs Python 3.11 for backend testing.

#### 4. Install Backend Dependencies: Creates a virtual environment and installs requirements from `backend/requirements.txt`.

#### 5. Run Backend Unit Tests: Executes `pytest` to validate backend functionality.

#### 6. Set up Node.js: Installs Node.js 20 for the frontend.

#### 7. Install Frontend Dependencies: Runs `npm ci` inside the `frontend/` directory to install dependencies.

#### 8. Build Docker Images
   - Builds backend image tagged with Git SHA.
   - Builds frontend image tagged with Git SHA.

#### 9. Authenticate with AWS: Configures AWS CLI using secrets.

#### 10. Push Docker Images to Amazon ECR: Pushes both backend and frontend images to your private Elastic Container Registry.

### CD: Continuous Deployment (Triggered on Push to `main`)
#### 1. Checkout Code: Clones the repo again in a clean environment.

#### 2. Authenticate with AWS: Same AWS setup as in CI.

#### 3. Generate ECS Task Definitions: Dynamically generates `backend-task-def.json` and `frontend-task-def.json` using current Docker image SHA.

#### 4. Deploy to AWS ECS

   - Uses `amazon-ecs-deploy-task-definition` to update ECS services with the new task definitions.

   - Waits until the service becomes stable.

### Required GitHub Secrets
| Name | Description |
|------|-------------|
| `AWS_ACCESS_KEY_ID` | AWS IAM user access key |
| `AWS_SECRET_ACCESS_KEY` | AWS IAM user secret |
| `AWS_ACCOUNT_ID` | Your AWS account ID |
| `ECR_BACKEND_REPO` | ECR repo URI for the backend |
| `ECR_FRONTEND_REPO` | ECR repo URI for the frontend |
| `ECS_BACKEND_SERVICE` | Name of the ECS service for the backend |
| `ECS_CLUSTER_NAME` | ECS Cluster name where services are deployed |


