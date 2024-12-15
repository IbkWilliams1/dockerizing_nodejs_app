## Dockerizing a Node.js, Java, and Go Application with Best Practices
This task involves packaging Node.js, Java, and Go applications into Docker containers. It will cover single-stage and multi-stage builds and demonstrate best practices for creating efficient, secure, and optimized Docker images. Finally, the images will be pushed to Docker Hub and AWS ECR.
### General Prerequisites
##### 1. Basic Knowledge

Familiarity with Docker concepts (e.g., images, containers, Dockerfile).
Basic understanding of the programming languages: Node.js, Java, and Go.
##### 2. Environment Setup

- Docker Installed: Ensure Docker is installed on your machine.
Download Docker if not already installed.
- AWS CLI Installed: For pushing images to AWS ECR, install and configure the AWS Command Line Interface.
AWS CLI Installation Guide.
- Programming Language Tools:
  -  **Node.js**: Install Node.js and npm for the Node.js application.
  -  **Java**: Install JDK and Maven for the Java application.
  -  **Go**: Install Go for the Go application.

##### 3. Docker Hub Account

-   Sign up for a free Docker Hub account to push your images.
Docker Hub Signup.
#####   4. AWS Account

-   Create an AWS account to push images to AWS Elastic Container Registry (ECR).
AWS Signup.
### Specific Prerequisites for Each Application

##### 1. Node.js Application
-   Project Setup: A Node.js project with a valid ```package.json``` file.
Run the following command to initialize:
    ```bash
    npm init -y
    ```
-   Dependencies Installed: Install dependencies locally to test the application before containerizing.
Example for Express app:
    ```bash
    npm install express
    ```
##### 2. Java Application
-   Maven or Gradle Installed: Install a build tool like Maven or Gradle for managing dependencies and building .jar files.
-   Java Build Tools Configuration: Ensure your pom.xml (for Maven) or build.gradle (for Gradle) is correctly set up.
##### 3. Go Application
-   Go Modules Enabled: Initialize Go modules in your project directory:
    ```bash
    go mod init your-module-name
    ```
-   Test Locally: Ensure the application compiles and runs locally
    ```bash
    go build
    ./your-app
    ```
### Infrastructure Prerequisites
1. AWS ECR Setup
-   ECR Repository Created: Create a repository in AWS ECR for each application:
    ```bash
    aws ecr create-repository --repository-name <repository-name>
    ```
2. AWS CLI Configuration
Configure the AWS CLI with credentials:
    ```bash
    aws configure
    ```
   ##### You'll need:
-   AWS Access Key ID
-   AWS Secret Access Key
-   Default region (e.g., us-east-1).
**step-by-step guide** to dockerizing a Node.js application and deploying it on an AWS Ubuntu instance:

---

### **1. Prepare the AWS Ubuntu Instance**

#### **Step 1.1: Launch the AWS EC2 Instance**
1. Log in to your AWS Management Console.
2. Navigate to **EC2 Dashboard** and click **Launch Instances**.
3. Configure the instance:
   - **Choose an AMI:** Select an Ubuntu Server (e.g., Ubuntu 22.04 LTS).
   - **Instance Type:** Choose a type, such as `t2.micro` (free tier eligible).
   - **Key Pair:** Create or select an existing key pair for SSH access.
   - **Security Group Rules:**
     - Allow SSH (port 22) from your IP.
     - Allow HTTP (port 80) and any other relevant ports (e.g., 5000 for Node.js).

4. Launch the instance and note its **Public IP** address.

#### **Step 1.2: Connect to the Instance**
Use SSH to connect to your instance:
```bash
ssh -i /path/to/your-key.pem ubuntu@<instance-public-ip>
```

---

### **2. Set Up the Ubuntu Instance**

#### **Step 2.1: Update and Install Dependencies**
Run the following commands to update the system and install necessary tools:
```bash
sudo apt update && sudo apt upgrade -y
```

#### **Step 2.2: Install Docker**
1. Install Docker:
   ```bash
   sudo apt-get install -y docker.io
   ```
2. Start and enable the Docker service:
   ```bash
   sudo systemctl start docker
   sudo systemctl enable docker
   ```
3. Verify Docker installation:
   ```bash
   docker --version
   ```

#### **Step 2.3: Install Docker Compose (Optional)**
If needed, install Docker Compose:
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

#### **Step 2.4: Install Node.js and npm**
Install Node.js and npm to test the application locally before containerizing:
```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs
node -v
npm -v
```

---

### **3. Prepare the Node.js Application**

#### **Step 3.1: Clone or Upload Your Application**
Clone your Node.js application repository from GitHub or upload it to the instance:
```bash
git clone <your-repo-url> node-app
cd node-app
```

#### **Step 3.2: Test the Application**
1. Install dependencies:
   ```bash
   npm install
   ```
2. Start the application locally:
   ```bash
   node app.js
   ```
3. Verify it's working by accessing the instance’s public IP and port in a browser:
   ```
   http://<instance-public-ip>:5000
   ```

---

### **4. Write the Dockerfile**

Create a `Dockerfile` in the Node.js application root directory:
```Dockerfile
# Use an official Node.js runtime as the base image
FROM node:18

# Set the working directory inside the container
WORKDIR /app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application
COPY . .

# Expose the application port
EXPOSE 5000

# Define the command to run the application
CMD ["node", "app.js"]
```

---

### **5. Build and Run the Docker Container**

#### **Step 5.1: Build the Docker Image**
Run the following command in the application directory:
```bash
docker build -t node-app:latest .
```

#### **Step 5.2: Run the Docker Container**
Run the container and map port `5000` of the container to port `5000` on the host:
```bash
docker run -d -p 5000:5000 node-app:latest
```

#### **Step 5.3: Verify the Application**
Open your browser and access:
```
http://<instance-public-ip>:5000
```

---

### **6. Push the Docker Image to Docker Hub**

#### **Step 6.1: Log in to Docker Hub**
Log in using your Docker Hub credentials:
```bash
docker login
```

#### **Step 6.2: Tag the Docker Image**
Tag the image with your Docker Hub username:
```bash
docker tag node-app:latest your-dockerhub-username/node-app:latest
```

#### **Step 6.3: Push the Image**
Push the image to your Docker Hub repository:
```bash
docker push your-dockerhub-username/node-app:latest
```

---

### **7. Push the Docker Image to AWS ECR**

#### **Step 7.1: Authenticate Docker to AWS ECR**
Get the ECR login command and authenticate:
```bash
aws ecr get-login-password --region <your-region> | docker login --username AWS --password-stdin <aws-account-id>.dkr.ecr.<your-region>.amazonaws.com
```

#### **Step 7.2: Tag the Docker Image**
Tag the Docker image for your ECR repository:
```bash
docker tag node-app:latest <aws-account-id>.dkr.ecr.<your-region>.amazonaws.com/node-app:latest
```

#### **Step 7.3: Push the Image to ECR**
Push the image to your ECR repository:
```bash
docker push <aws-account-id>.dkr.ecr.<your-region>.amazonaws.com/node-app:latest
```

---

### **8. Run the Container from ECR**

1. Pull the image from ECR:
   ```bash
   docker pull <aws-account-id>.dkr.ecr.<your-region>.amazonaws.com/node-app:latest
   ```
2. Run the container:
   ```bash
   docker run -d -p 5000:5000 <aws-account-id>.dkr.ecr.<your-region>.amazonaws.com/node-app:latest
   ```

---

### **9. Automate Deployment (Optional)**

- Use **Docker Compose** for multi-container setups.
- Use **AWS ECS** for orchestrating Docker containers.

---

This step-by-step guide covers everything from preparing the AWS Ubuntu instance to deploying the Node.js application in a Docker container. Let me know if you need help automating these steps or setting up CI/CD pipelines!
#### Steps to Dockerize a Node.js Application
- Example file structure of node.js application
```go
node-app/
├── app.js
├── package.json
└── package-lock.json
```