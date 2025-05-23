# ☸️ Kubernetes
## 📦 Demo 4: 
This demo project is part of **Kubernetes Module** from Nana **TWN DevOps Bootcamp**. It focuses on deploying a Dockerized application to a **Kubernetes cluster** using **Helm**, with the container image hosted in **Amazon Elastic Container Registry (ECR)**.

## 🚀 Technologies Used
- **Kubernetes (minikube)**: Container orchestration.
- **AWS ECR**: Private container registry.
- **Docker**: build and package an application.
- **Helm**: Kubernetes package manager.
  
## 📋 Prerequisites
- Ensure minikube is running.
- Ensure Helm is installed on your machine.
- Ensure the application from module 7 is still in ECR.

## 🎯 Features

- Create a Secret for credentials for the private Docker registry.
- Configure the Docker registry secret in the application deployment component.
- Deploy web application image from our private Docker registry in the K8s cluster. 

## 🏗 Project Architecture

<img src=""/>

## ⚙️ Project Configuration
### Clone Repository
1. Obtain the deployment and secret file from the repository.

### Creating Secret Component Using YAML File.
1. Log in to Amazon ECR using the AWS CLI. This generates a config.json file containing the authentication token.

   ```bash
     aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin <your-registry>
   ```
   <img src="" width=800 />
2. Verify that the config.json file exists at ~/.docker/config.json.
3. <details>
  <summary><strong> If the token is missing due to credential store usage, follow these steps: </strong></summary>
  <br>
  1. Obtain the ECR login password.
  2. SSH into the Minikube instance.
  3. Log in to Amazon ECR from Minikube to generate the config.json file inside Minikube.
  4. Verify that the .docker directory was created in Minikube.
  5. Check that the config.json file exists and contains the credentials.
  6. Copy the config.json file from Minikube to your local .docker directory.
       
</details>
4. Encode the config.json file using Base64.
   ```bash
   cat ~/.docker/config.json | Base64
   ```
5. Insert the encoded credentials into the YAML file under the .dockerconfigjson section.
  <img src="" width=800 />
6. Set the secret name to my-registry-key.
  <img src="" width=800 />
7. Apply the secret using kubectl apply -f <secret-file>.yaml.
  ```bash
    kubectl apply -f secret.yaml
  ```

### Creating Generic Secret Component Using CommandLine
1. Log in to Amazon ECR using the AWS CLI to generate the config.json file.
   ```bash
     aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin <your-registry>
   ```
3. Run the following command to create the secret named my-registry-key:
   ```bash
       kubectl create secret generic my-registry-key \  
      --from-file=.dockerconfigjson=~/.docker/config.json \  
      --type=kubernetes.io/dockerconfigjson 
   ```
4. Verify that the secret was created:
  ```bash  
    kubectl get secret
  ```

### Creating Docker Registry Secret Component Using the Command Line
1. Retrieve the ECR login password:
   ```bash
   aws ecr get-login-password 
2. Use the password to create the Docker registry secret:
   ```bash
         kubectl create secret docker-registry my-registry-key-two \  
      --docker-server=<your-registry> \  
      --docker-username=AWS \  
      --docker-password=$(aws ecr get-login-password)
   ```

### Configure the Deployment YAML File
1. Open the deployment YAML file.
2. Under spec.template.spec, add the imagePullSecrets section:
   ```bash
       imagePullSecrets:
         - name: my-registry-key 
   ```
3. Specify the container image from your ECR repository
4. Apply the deployment YAML file:
   ```bash
   kubectl apply -f deployment.yaml
   ```
5. Verify that the pods are running:
   ```bash
   kubectl get pods
   ```
   

  

