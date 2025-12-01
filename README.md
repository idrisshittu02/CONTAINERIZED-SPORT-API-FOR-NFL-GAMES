# Sports API Management System

## **Project Overview**
This project demonstrates building a containerized API management system for querying sports data. It leverages **Amazon ECS (Fargate)** for running containers, **Amazon API Gateway** for exposing REST endpoints, and an external **Sports API** for real-time sports data. The project showcases advanced cloud computing practices, including API management, container orchestration, and secure AWS integrations.

---

## **Features**
- Exposes a REST API for querying real-time sports data
- Runs a containerized backend using Amazon ECS with Fargate
- Scalable and serverless architecture
- API management and routing using Amazon API Gateway
 
---

## **Prerequisites**
- **Sports API Key**: Sign up for a free account and subscription & obtain your API Key at serpapi.com
- **AWS Account**: Create an AWS Account & have basic understanding of ECS, API Gateway, Docker & Python
- **AWS CLI Installed and Configured**: Install & configure AWS CLI to programatically interact with AWS
- **Serpapi Library**: Install Serpapi library in local environment "pip install google-search-results"
- **Docker CLI and Desktop Installed**: To build & push container images

---

## **Technical Architecture**
![Brown Minimalist Lifestyle Daily Vlog YouTube Thumbnail (2)](https://github.com/user-attachments/assets/32e49fe6-df16-40cb-b262-af1478cf01d5)

---

## **Technologies**
- **Cloud Provider**: AWS
- **Core Services**: Amazon ECS (Fargate), API Gateway, CloudWatch
- **Programming Language**: Python 3.x
- **Containerization**: Docker
- **IAM Security**: Custom least privilege policies for ECS task execution and API Gateway

---

## **Project Structure**

```bash
sports-api-management/
├── app.py # Flask application for querying sports data
├── Dockerfile # Dockerfile to containerize the Flask app
├── requirements.txt # Python dependencies
├── .gitignore
└── README.md # Project documentation
```

---

## **Setup Instructions**

### **Clone the Repository**
```bash
git clone https://github.com/Stormz99/CONTAINERIZED_SPORT_API_FOR_NFL_GAMES.git
cd containerized-sports-api
```
### **Create ECR Repo**
```bash
aws ecr create-repository --repository-name sports-api --region us-east-1
```

### **Authenticate Build and Push the Docker Image**
```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com

docker build --platform linux/amd64 -t sports-api .
docker tag sports-api:latest <AWS_ACCOUNT_ID>.dkr.ecr.us-east-l1.amazonaws.com/sports-api:sports-api-latest
docker push <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/sports-api:sports-api-latest
```
![docker_build](./images/image-on%20building.png)

![docker_build_2](./images/image-on-pushing%20to%20latest.png)

![confirm_checks_on_docker](./images/docker-images.png)

![confirming_on_ecr](./images/confirming-on-ecr.png)

![checking_latest_on_aws_console](./images/checking-the-latest-image.png)

### **Set Up ECS Cluster with Fargate**
1. Create an ECS Cluster:
- Go to the ECS Console → Clusters → Create Cluster
- Name your Cluster (sports-api-cluster)
- For Infrastructure, select Fargate, then create Cluster

![creating_a_cluster](./images/creating-a-cluster.png)

2. Create a Task Definition:
- Go to Task Definitions → Create New Task Definition
- Name your task definition (sports-api-task)
- For Infrastructure, select Fargate
- Add the container:
  - Name your container (sports-api-container)
  - Image URI: <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/sports-api:sports-api-latest
  - Container Port: 8080
  - Protocol: TCP
  - Port Name: Leave Blank
  - App Protocol: HTTP
- Define Environment Eariables:
  - Key: SPORTS_API_KEY
  - Value: <YOUR_SPORTSDATA.IO_API_KEY>
  - Create task definition

![creating_task_definition](./images/creating-task-definition.png)

![creating_a_container](./images/creating-a-container.png)

![creating_env](./images/creating-env.png)

![task_definition_created](./images/task-defintion-created.png)

3. Run the Service with an ALB
- Go to Clusters → Select Cluster → Service → Create.
- Capacity provider: Fargate
- Select Deployment configuration family (sports-api-task)
- Name your service (sports-api-service)
- Desired tasks: 2
- Networking: Create new security group
- Networking Configuration:
  - Type: All TCP
  - Source: Anywhere
- Load Balancing: Select Application Load Balancer (ALB).
- ALB Configuration:
 - Create a new ALB:
 - Name: sports-api-alb
 - Target Group health check path: "/sports"
 - Create service

![creating_a_cluster_1](./images/creating-an-ecs-cluster.png) 

![creating_an_ecs_cluster](./images/ecs-cluster-2.png)

![setting_lb_in_ecs_cluster](./images/setting-lb-in-ecs-cluster.png)

![ecs_cluster](./images/ecs-cluser.png)

![overview](./images/overview.png)

4. Test the ALB:
- After deploying the ECS service, note the DNS name of the ALB (e.g., sports-api-alb-<AWS_ACCOUNT_ID>.us-east-1.elb.amazonaws.com)
- Confirm the API is accessible by visiting the ALB DNS name in your browser and adding /sports at end (e.g, http://sports-api-alb-<AWS_ACCOUNT_ID>.us-east-1.elb.amazonaws.com/sports)

![load_balancer](./images/lb.png)

![alb_testing](./images/alb-uri.png)

### **Configure API Gateway**
1. Create a New REST API:
- Go to API Gateway Console → Create API → REST API
- Name the API (e.g., Sports API Gateway)

![creating_API_GW](./images/creating-API-Gateway.png)

![creating_REST_API](./images/creating-REST-API.png)

![overview_API_GW](./images/overview-API-Resources.png)

2. Set Up Integration:
- Create a resource /sports
- Create a GET method
- Choose HTTP Proxy as the integration type
- Create an Invoke Policy
- Enter the DNS name of the ALB that includes "/sports" (e.g. http://sports-api-alb-<AWS_ACCOUNT_ID>.us-east-1.elb.amazonaws.com/sports

![creating_a_method](./images/creating-a-method.png)

![alb_passthrough](./images/api-gateway-alb-passthrough.png)

![creating_an_invoke_policy](./images/creating-an-invoke-policy.png)

3. Deploy the API:
- Deploy the API to a stage (e.g., prod)
- Note the endpoint URL

![creating_a_stage](./images/creating-a-stage.png)

### **Test the System**
- Use curl or a browser to test:
```bash
curl https://<api-gateway-id>.execute-api.us-east-1.amazonaws.com/prod/sports
```

![testing](./images/api-gateway-consumption.png)

### ***Deleting the Resources***
After the API Gateway is successfully working, all provisioned resources, should be deleted. This is to prevent incurring bills.

![cleaning_resource_1](./images/cleaning-resource-1.png)

![cleaning-resouces-2](./images/cleaning-resource-2.png)

![cleaning_resoruces_3](./images/cleaning-resource-3.png)

![cleaning_resources_4](./images/cleaning-resource-4.png)

![cleaning_resource_5](./images/cleaning-resouce-5.png)

### **What I Learned**
Setting up a scalable, containerized application with ECS
Creating public APIs using API Gateway.


