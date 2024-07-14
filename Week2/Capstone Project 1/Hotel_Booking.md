
# Microservices Architecture for Hotel Booking Application

The microservices architecture for a hotel booking application like Stay Booker Pro, we can break down the functionalities into distinct microservices. Each microservice will handle a specific aspect of the application, ensuring scalability, maintainability, and independent deployment.

[GitHub Repository](https://github.com/balogsun/hotel-booking.git)

## Microservices Overview

- **User Service**: Manages user registration, authentication, and profiles.
- **Booking Service**: Handles booking processes, availability checks, and reservations.
- **Payment Service**: Manages payment processing and transactions.
- **Notification Service**: Sends booking confirmations, reminders, and notifications.
- **Review Service**: Manages user reviews and ratings for hotels.
- **Hotel Management Service**: Handles hotel listings, room details, and amenities.

### Technology Stack

- **Programming Language**: JavaScript/TypeScript
- **Framework**: Node.js with Express.js for the backend
- **Frontend**: React with Tailwind CSS

However, in our scenario, the services have been bundled to run together as a single bulk microservice due to deployment complexity and resource limitations.

### Next, we will write a Dockerfile, test the code build, and upon successful build, insert the Dockerfile path into our proposed GitHub workflow

## Dockerfile

```dockerfile
# Use Node.js LTS version as the base image
#FROM node:lts-alpine
FROM node:14

# Set the working directory in the container
WORKDIR /app

# Copy package.json and package-lock.json 
COPY package*.json ./
#COPY package.json package-lock.json /app/

# Install project dependencies
RUN npm install

# Copy the entire project files to the container
COPY . /app/
#COPY . .

# Build the Next.js application for production
RUN npm run build

# Expose the port used by your Next.js app (if needed)
EXPOSE 3000

# Define the default command to start the Next.js app
CMD ["npm", "start"]
#CMD ["node", "server.js"]
```

### Dockerfile Explanation

- **Base Image**: Using Node.js LTS version as the base image ensures stability and compatibility.
- **Working Directory**: Sets the working directory in the container to `/app`.
- **Copying Dependencies**: Copies `package.json` and `package-lock.json` to the container to install dependencies.
- **Installing Dependencies**: Runs `npm install` to install project dependencies.
- **Copying Project Files**: Copies the entire project files to the container.
- **Building the Application**: Runs `npm run build` to build the Next.js application for production.
- **Exposing Port**: Exposes port 3000, which the Next.js app listens on.
- **Starting the Application**: Defines the default command to start the Next.js app using `npm start`.

## Kubernetes Cluster Setup

I set up a Kubernetes cluster using the following Terraform modules in my repo: [Terraform EKS Module](https://github.com/balogsun/terraform-eks-with-basic-app.git) and steps summarized below:

- **Install AWS CLI**
  - Follow AWS documentation: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

- **Install Terraform**
  - Follow instructions: https://developer.hashicorp.com/terraform/install

- **Create Terraform Files and Configure AWS Provider**
  - **provider.tf**: Configure AWS provider for `ca-central-1` region, retrieve current region and availability zones.
  ```hcl
  provider "aws" {
    region  = "ca-central-1"
  }

  data "aws_region" "current" {}

  data "aws_availability_zones" "available" {}
  ```
  - **worker-node.tf**: Configure IAM role, EKS node group with instance types, subnets, scaling parameters, and SSH access.

  - **variables.tf**: Define input variables for EKS cluster deployment:
    - `cluster-name`: "pjct-cluster"
    - `eks_version`: "1.30"
    - `key_pair_name`: "project-key"
    - `eks_node_instance_type`: "t2.medium"
  ```hcl
  variable "cluster-name" {
    default = "pjct-cluster"
    type    = string
  }
  variable "eks_version" {
    default = "1.30"
    type    = string
  }
  variable "key_pair_name" {
    default = "project-key"
  }
  variable "eks_node_instance_type" {
    default = "t2.medium"
  }
  ```
  - **cluster.tf**: Set up AWS EKS cluster with necessary IAM roles, security groups, and the cluster itself.

  - **vpc.tf**: Create VPC with public and private subnets across three availability zones, configure internet and NAT gateways, and set up route tables.

- **Step 2: Configure AWS CLI**
  - Run `aws configure` and enter AWS credentials, default region (`ca-central-1`), and default output format.
  ```bash
  aws configure

  Access Key ID: *********************
  Secret key: ******************************************
  Default region Name [none]: ca-central-1
  Default output format [none]:
  ```

- **Step 3: Initialize Terraform Modules**
  - Clone the repository, navigate to `terraform-eks-seun`, and run `terraform init`.

- **Step 4: Create Clusters and Resources**
  - Run `terraform plan` to create an execution plan.
  - Run `terraform apply -auto-approve` to apply the changes.

- **Step 5: Update Kubeconfig**
  - Run `aws eks update-kubeconfig --region ca-central-1 --name pjct-cluster` to update the kubeconfig file.

- **Step 6: Confirm Cluster and Nodes**
  - Run `aws eks list-clusters` to list EKS clusters.
  - Run `kubectl get nodes -o wide` to get node details.

## Define Kubernetes Manifests

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hotel-deployment
spec:
  replicas: 2  
  selector:
    matchLabels:
      app: hotel
  template:
    metadata:
      labels:
        app: hotel
    spec:
      containers:
      - name: hotel
        image: balogsen/hotel:latest
        ports:
        - containerPort: 3000  # Port your application listens on
---
apiVersion: v1
kind: Service
metadata:
  name: hotel-service
spec:
  selector:
    app: hotel
  ports:
    - protocol: TCP
      port: 80  # Port exposed by the service externally (outside the cluster)
      targetPort: 3000  # Port your application listens on inside the pods
  type: LoadBalancer
```

### Kubernetes YAML Explanation

- **Deployment**: Defines a Deployment for the hotel service with 2 replicas for high availability.
  - **Selectors and Labels**: Matches labels to identify the pods managed by the deployment.
  - **Containers**: Specifies the container image (`balogsen/hotel:latest`) and the port it listens on (3000).
- **Service**: Defines a Service to expose the hotel application.
  - **Selector**: Matches the app label to route traffic to the appropriate pods.
  - **Ports**: Maps external port 80 to internal port 3000.
  - **Type**: Uses a LoadBalancer to expose the service externally.

### Set up monitoring and logging using Prometheus, Grafana

# Helm Chart Installations:

```bash
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

## Add Helm Repositories

### Add the Helm Stable Charts for your local client:
```bash
helm repo add stable https://charts.helm.sh/stable
```

### Add Prometheus Helm repo:
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

```bash
helm repo ls
<img width="530" alt="image" src="https://github.com/user-attachments/assets/286962f9-e0ad-40fb-b41a-88a2bd4c9ef4">

helm repo update
```

## Create Prometheus Namespace
```bash
kubectl create namespace prometheus
```

## Install Prometheus/kube-prometheus-stack
```bash
helm install stable prometheus-community/kube-prometheus-stack -n prometheus
```
<img width="611" alt="image" src="https://github.com/user-attachments/assets/0a831ae1-f389-41a3-9cee-7a873303ee89">

## Check Installation Status
```bash
kubectl get pods -n prometheus

<img width="611" alt="image" src="https://github.com/user-attachments/assets/6d986a88-2f09-42be-992c-89a629a56550">

kubectl get svc -n prometheus # you should see both prometheus and grafana services here
```
<img width="614" alt="image" src="https://github.com/user-attachments/assets/0d4719aa-f210-4b5e-958c-bcf4ced56bf0">

## Expose Prometheus for External Access
```bash
kubectl edit svc stable-kube-prometheus-sta-prometheus -n prometheus
```
Change the service type from `ClusterIP` to `LoadBalancer`. Save the file.

### Configuration example:
```yaml
selector:
  app.kubernetes.io/name: prometheus
  operator.prometheus.io/name: stable-kube-prometheus-sta-prometheus
sessionAffinity: None
type: ClusterIP
```

Access the GUI with the load balancer URL.

## Provision Grafana
Edit/change the `stable-Grafana` service from `ClusterIP` to `LoadBalancer`
```bash
kubectl edit svc stable-grafana -n prometheus
```

### Configuration example:
```yaml
selector:
  app.kubernetes.io/instance: stable
  app.kubernetes.io/name: grafana
sessionAffinity: None
type: ClusterIP
```

```bash
kubectl get svc -n prometheus
```
Observe the output change, adding the load balancer URL.
<img width="959" alt="image" src="https://github.com/user-attachments/assets/88743afb-b856-410b-a8f4-13498022a7f8">


## Access Grafana GUI
Access the Grafana GUI with the load balancer URL and login to Grafana:

### Credentials:
- username: admin
- password: prom-operator

## Visualize Metrics
To visualize metrics, add a data source:

1. Click **Add data source** and select Prometheus.
2. OR open the 3-line menu icon, select **Connections**, then **Data sources**.

The data source "prometheus" is already added by default.

Open the dashboard panel to see the default available dashboards and click on the one that suits the visualization of the cluster environment.

This will show a monitoring dashboard for all cluster nodes.
<img width="683" alt="image" src="https://github.com/user-attachments/assets/995665f9-6e60-4d2c-9f0f-1bb2b019b4f6">

<img width="734" alt="image" src="https://github.com/user-attachments/assets/13265223-f111-48ea-b23c-7bf33b20e5d6">

<img width="728" alt="image" src="https://github.com/user-attachments/assets/faa0f2e1-270d-4d5c-84d1-82d116f3524f">

<img width="724" alt="image" src="https://github.com/user-attachments/assets/b4e2ef24-ace6-43d6-bf85-f59d8f0d3d20">


## Implement CI/CD Pipelines for Automated Builds, Testing, and Deployments

Below is the Github Workflow to automate the deployment.

```yaml
name: Hotel Pipeline

on:
  push:
    branches:
      - main

jobs:
  codeql-analysis:
    name: Build-CodeQL-and-deploy hotel
    runs-on: ${{ matrix.language == 'swift' && 'macos-latest' || 'ubuntu-latest' }}
    timeout-minutes: ${{ matrix.language == 'swift' && 120 || 360 }}
    permissions:
      security-events: write
      packages: read
      actions: read
      contents: read

    strategy:
      fail-fast: false
      matrix:
        language:
          - javascript-typescript
        build-mode:
          - none

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}
          build-mode: ${{ matrix.build-mode }}

      - if: matrix.build-mode == 'manual'
        name: Build Project
        run: |
          echo 'If manual build mode is set, replace this with your build commands.'
          exit 1

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v3
        with:
          category: "/language:${{ matrix.language }}"

      - name: Install Trivy
        run: |
         wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
         echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -cs) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
         sudo apt-get update && sudo apt-get install trivy

      - name: Docker login
        run: echo ${{ secrets.DOCKERHUB_PASSWORD }} | docker login -u ${{ secrets.DOCKERHUB_USERNAME }} --password-stdin     

      - name: Build hotel booking image
        run: |
          docker build -t balogsen/hotel:latest .

      - name: Scan Docker image with Trivy
        run: |
          trivy image -f json -o trivy_report.json balogsen/hotel:latest
 
      - name: Convert Trivy JSON to human-readable format
        run: |
          trivy image -f table balogsen/hotel:latest > trivy_report.txt
 
      - name: Save Trivy scan results as artifact
        uses: actions/upload-artifact@v4
        with:
          name: trivy-scan-results
          path: trivy_report.txt

      - name: Push hotel booking image
        run: |
         docker push balogsen/hotel:latest
         docker rmi balogsen/hotel:latest

      - name: Set up kubectl
        uses: azure/setup-kubectl@v3
        with:
         version: 'latest'

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v3
        with:
         aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
         aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
         aws-region: ca-central-1

      - name: Update kubeconfig for EKS
        run: |
         aws eks update-kubeconfig --region ca-central-1 --name pjct-cluster
 
      - name: Deploy to EKS
        run: |
         kubectl apply -f K8S/deployment.yml
```

### GitHub Workflow Explanation

- **Trigger**: The workflow triggers on a push to the `main` branch.
- **Jobs**: Defines a job named `codeql-analysis` that builds, scans, and deploys the hotel application.
  - **Strategy**: Uses a matrix strategy for different languages

 and build modes.

- **Steps**:
  - **Checkout**: Checks out the repository code.
  - **Initialize CodeQL**: Initializes CodeQL for security analysis.
  - **Perform CodeQL Analysis**: Analyzes the code using CodeQL.
  - **Install Trivy**: Installs Trivy for security scanning.
  - **Docker Login**: Logs in to DockerHub.
  - **Build Docker Image**: Builds the Docker image for the hotel application.
  - **Scan Docker Image**: Scans the Docker image with Trivy.
  - **Convert Trivy JSON**: Converts Trivy scan results to a human-readable format.
  - **Save Trivy Scan Results**: Saves the scan results as an artifact.
  - **Push Docker Image**: Pushes the Docker image to DockerHub.
  - **Set up kubectl**: Sets up kubectl for interacting with Kubernetes.
  - **Configure AWS Credentials**: Configures AWS credentials for EKS access.
  - **Update kubeconfig**: Updates the kubeconfig for EKS.
  - **Deploy to EKS**: Deploys the application to EKS using Kubernetes manifests.

### Check the deployment by accessing the loadbalancer URL that has been created from the `kubectl get svc` command.

![Screenshot 2024-06-20 185746](https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/3c0631e4-fc05-434c-b2ea-8d710846d330)

![Screenshot 2024-06-20 185853](https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/8bd17c5e-d135-4dee-ba0b-c4adb39685d1)

![Screenshot 2024-06-20 190305](https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/bf2638e3-6a6b-46b9-8f9f-cee476aef5d7)


### Below I have attached screenshots to show failed and successful workflow runs.
![Screenshot 2024-06-20 184929](https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/a4f4b43b-c5e8-49d8-a5d3-4048556992c3)


### Also, i made a change to the `AboutUs.jsx` file [updating the contact information to my own email address details] and commited the codes, upon pushing the commit, the pipeline was triggered and the change was effected immediately.

<img width="749" alt="Screenshot 2024-06-20 191107" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/88b84062-507a-4b21-80cf-05268a34808e">

<img width="917" alt="Screenshot 2024-06-20 191159" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/59c88e8a-2527-40c5-afb5-7b10e15c5ed1">

<img width="782" alt="Screenshot 2024-06-20 192041" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/b922e957-a6f3-4c28-9008-ce0261022a0b">


### The Artifact for the Trivy scan results was generated in the repository download URL at: [Scan Results](<https://github.com/balogsun/hotel-booking/actions/runs/9589669410/artifacts/1618838601>)

Screenshot also attached.
<img width="867" alt="image" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/bf2c3f37-f789-4e78-8867-6ced99b95b97">


### The CodeQL scanned 20 out of 20 JavaScript files in this invocation. Check the status page for overall coverage information: [CodeQL Scan result](<https://github.com/balogsun/hotel-booking/security/code-scanning/tools/CodeQL/status/>)

Screenshot also attached.
<img width="521" alt="image" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/91a44d8e-256f-4ce7-b8d0-dbedd83b1ad9">

- **Monitoring and Logging**: Set up monitoring and logging using Prometheus, Grafana, and the ELK stack.
Link to my repo using prometheus to monitor a kunbernetes environment.  [Prometheus Monitoring Repo](https://github.com/Makinates/SeunB-tasks-docs/tree/288703b081afc29d2e7e85f4785bc41575ef621b/Week1/Task2/Monitoring%20and%20Logging)

- **Security Best Practices**: I applied secrets management by setting up secrets and credentials in the GitHub repo settings, as well as place them as strings in the GitHub workflow.

## Conclusion

The microservices architecture for the Hotel Service application ensures modularity, scalability, and maintainability. By bundling services for deployment simplicity, utilizing Docker for containerization, and leveraging Kubernetes for orchestration, I have created a robust and efficient deployment pipeline. Implementing CI/CD with GitHub Actions, along with comprehensive monitoring and logging, guarantees continuous delivery and operational visibility. This architecture lays a solid foundation for the application's future growth and scalability.
