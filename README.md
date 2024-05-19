# devops
To create an Infrastructure as Code (IaC) setup using Terraform for deploying a "Hello World" Node.js app on AWS ECS/Fargate, along with a Continuous Deployment (CD) pipeline using GitHub Actions, follow these steps:

### Prerequisites
1. AWS Account
2. AWS CLI configured
3. GitHub Account
4. Terraform installed
5. Docker installed

### Step 1: Setup the Node.js Application
1. Create a simple Node.js app:
    ```bash
    mkdir hello-world-nodejs
    cd hello-world-nodejs
    npm init -y
    npm install express
    ```

2. Create `app.js`:
    ```javascript
    const express = require('express');
    const app = express();
    const port = 3000;

    app.get('/', (req, res) => {
      res.send('Hello World!');
    });

    app.listen(port, () => {
      console.log(`App running at http://localhost:${port}`);
    });
    ```

3. Create a `Dockerfile`:
    ```dockerfile
    FROM node:14
    WORKDIR /app
    COPY package*.json ./
    RUN npm install
    COPY . .
    EXPOSE 3000
    CMD ["node", "app.js"]
    ```

### Step 2: Dockerize the Application
1. Build the Docker image:
    ```bash
    docker build -t hello-world-nodejs .
    ```

2. Run the Docker container locally to test:
    ```bash
    docker run -p 3000:3000 hello-world-nodejs
    ```

### Step 3: Create Terraform Configuration
1. Create a new directory for Terraform files:
    ```bash
    mkdir terraform
    cd terraform
    ```

2. Create `main.tf` for the ECS/Fargate setup:
    ```hcl
    provider "aws" {
      region = "us-east-1"
    }

    resource "aws_ecs_cluster" "hello-world-cluster" {
      name = "hello-world-cluster"
    }

    resource "aws_iam_role" "ecsTaskExecutionRole" {
      name = "ecsTaskExecutionRole"
      assume_role_policy = jsonencode({
        Version = "2012-10-17",
        Statement = [
          {
            Effect = "Allow",
            Principal = {
              Service = "ecs-tasks.amazonaws.com"
            },
            Action = "sts:AssumeRole"
          },
        ],
      })
    }

    resource "aws_iam_role_policy_attachment" "ecsTaskExecutionRole_policy" {
      role       = aws_iam_role.ecsTaskExecutionRole.name
      policy_arn = "arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy"
    }

    resource "aws_ecs_task_definition" "hello-world" {
      family                   = "hello-world"
      network_mode             = "awsvpc"
      requires_compatibilities = ["FARGATE"]
      cpu                      = 256
      memory                   = 512
      execution_role_arn       = aws_iam_role.ecsTaskExecutionRole.arn

      container_definitions = jsonencode([{
        name  = "hello-world"
        image = "YOUR_DOCKERHUB_USERNAME/hello-world-nodejs:latest"
        essential = true
        portMappings = [{
          containerPort = 3000
          hostPort      = 3000
        }]
      }])
    }

    resource "aws_ecs_service" "hello-world-service" {
      name            = "hello-world-service"
      cluster         = aws_ecs_cluster.hello-world-cluster.id
      task_definition = aws_ecs_task_definition.hello-world.arn
      desired_count   = 1

      network_configuration {
        subnets          = ["subnet-12345", "subnet-67890"]
        security_groups  = [aws_security_group.ecs_security_group.id]
        assign_public_ip = true
      }
    }

    resource "aws_security_group" "ecs_security_group" {
      name        = "ecs-security-group"
      description = "Allow traffic to ECS tasks"
      vpc_id      = "vpc-12345"

      ingress {
        from_port   = 3000
        to_port     = 3000
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
      }

      egress {
        from_port   = 0
        to_port     = 0
        protocol    = "-1"
        cidr_blocks = ["0.0.0.0/0"]
      }
    }
    ```

### Step 4: Create GitHub Actions Workflow
1. Create a `.github/workflows/deploy.yml` file:
    ```yaml
    name: Deploy to AWS ECS

    on:
      push:
        branches:
          - main

    jobs:
      deploy:
        runs-on: ubuntu-latest

        steps:
        - name: Checkout code
          uses: actions/checkout@v2

        - name: Set up Docker Buildx
          uses: docker/setup-buildx-action@v1

        - name: Login to DockerHub
          uses: docker/login-action@v1
          with:
            username: ${{ secrets.DOCKER_USERNAME }}
            password: ${{ secrets.DOCKER_PASSWORD }}

        - name: Build and push Docker image
          run: |
            docker build -t YOUR_DOCKERHUB_USERNAME/hello-world-nodejs .
            docker push YOUR_DOCKERHUB_USERNAME/hello-world-nodejs

        - name: Configure AWS credentials
          uses: aws-actions/configure-aws-credentials@v1
          with:
            aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
            aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
            aws-region: us-east-1

        - name: Install Terraform
          uses: hashicorp/setup-terraform@v1

        - name: Initialize Terraform
          run: terraform init
          working-directory: terraform

        - name: Apply Terraform
          run: terraform apply -auto-approve
          working-directory: terraform
    ```

### Step 5: Push Code to GitHub
1. Create a new GitHub repository and push your code:
    ```bash
    git init
    git remote add origin https://github.com/YOUR_GITHUB_USERNAME/hello-world-nodejs.git
    git add .
    git commit -m "Initial commit"
    git push -u origin main
    ```

### Step 6: Configure GitHub Secrets
In your GitHub repository, add the following secrets:
- `DOCKER_USERNAME`: Your Docker Hub username
- `DOCKER_PASSWORD`: Your Docker Hub password
- `AWS_ACCESS_KEY_ID`: Your AWS access key ID
- `AWS_SECRET_ACCESS_KEY`: Your AWS secret access key

### Step 7: Create a Screencast
1. Use a tool like OBS Studio to record your screen.
2. Demonstrate the following:
    - The GitHub Actions workflow running and deploying the application.
    - Accessing the "Hello World" application via its public URL.

### Step 8: Share GitHub Repository and Screencast
1. Make your GitHub repository public.
2. Upload the screencast to a video sharing platform or provide a downloadable link.

### Expected Completion
This entire setup and demonstration should be completed within 2 days. 

Feel free to reach out if you need further assistance or modifications.
