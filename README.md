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
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_10_Kubernetes_Helm_AWS/blob/main/img/1%20login%20to%20amazon%20ecr.png" width=800 />
   
2. Verify that the config.json file exists at ~/.docker/config.json.

   ```bash
   cat ~/.docker/config.json
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_10_Kubernetes_Helm_AWS/blob/main/img/2%20accessing%20docker%20CLI%20config%20filethat%20contains%20auth.png" width=800 />
   
3. <details><summary><strong> If the token is missing due to credential store usage, follow these steps: </strong></summary><ol>
    <li>Obtain the ECR login password.</li>
      <pre><code class="language-bash">aws ecr get-login-password </code></pre>
      <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_10_Kubernetes_Helm_AWS/blob/main/img/3%20to%20obtain%20the%20auth%20key%20to%20use%20in%20ecr.png" width=800 />
    <li>SSH into the Minikube instance.</li>
      <pre><code class="language-bash">minikube ssh </code></pre>
      <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_10_Kubernetes_Helm_AWS/blob/main/img/4%20to%20add%20the%20auth%20to%20minikube.png" width=800 />
          
    <li>Log in to Amazon ECR from Minikube to generate the config.json file inside Minikube.</li>
      <pre><code class="language-bash">docker login --username AWS -p <copy_password_previous_step> </code></pre>
      <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_10_Kubernetes_Helm_AWS/blob/main/img/5%20entering%20minikube%20and%20login%20from%20minikube%20using%20the%20auth%20key%20from%20before.png" width=800 />
    <li>Verify that the .docker directory was created in Minikube.</li>
        <pre><code class="language-bash">ls -la</code></pre>
        <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_10_Kubernetes_Helm_AWS/blob/main/img/7%20login%20ok%20and%20we%20can%20see%20now%20docker%20diretcory%20created.png" width=800 />
        
    <li>Check that the config.json file exists and contains the credentials.</li>
        <pre><code class="language-bash">cat .docker/config.json </code></pre>
        <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_10_Kubernetes_Helm_AWS/blob/main/img/8%20config%20json%20is%20available%20no%20in%20minikube.png" width=800 />
      
    <li>Copy the config.json file from Minikube to your local .docker directory.</li>
       <pre><code class="language-bash">minikube cp minikue:/home/docker/.docker/config.json ~/.docker/config.json </code></pre>
       <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_10_Kubernetes_Helm_AWS/blob/main/img/10%20copying%20the%20file%20from%20minikube%20to%20the%20locak%20docker%20json%20file.png" width=800 />
</ol>    
</details>

4. Encode the config.json file using Base64.
   ```bash
   cat ~/.docker/config.json | base64
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_10_Kubernetes_Helm_AWS/blob/main/img/11%20base%2064%20encoded.png" width=800 />

5. Insert the encoded credentials into the YAML file under the .dockerconfigjson section.

   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_10_Kubernetes_Helm_AWS/blob/main/img/12%20updating%20secrets%20file%20with%20docker%20json%20encoded.png" width=800 />
  
6. Set the secret name to my-registry-key.
  
7. Apply the secret using kubectl apply -f <secret-file>.yaml.
    ```bash
      kubectl apply -f secret.yaml
    ```

### Creating a Generic Secret Component Using CommandLine
1. Log in to Amazon ECR using the AWS CLI to generate the config.json file.
   ```bash
     aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin <your-registry>
   ```
2. Run the following command to create the secret named my-registry-key:
   ```bash
       kubectl create secret generic my-registry-key \  
      --from-file=.dockerconfigjson=~/.docker/config.json \  
      --type=kubernetes.io/dockerconfigjson 
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_10_Kubernetes_Helm_AWS/blob/main/img/13%20creating%20the%20secret%20file%20from%20kubectl%20directly.png" width=800 />
   
3. Verify that the secret was created:
  
  ```bash  
    kubectl get secret
  ```
  <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_10_Kubernetes_Helm_AWS/blob/main/img/14%20get%20secret.png" width=800 />

### Creating Docker Registry Secret Component Using the Command Line
1. Retrieve the ECR login password:
   ```bash
     aws ecr get-login-password
   ```
2. Use the password to create the Docker registry secret:
   ```bash
         kubectl create secret docker-registry my-registry-key-two \  
      --docker-server=<your-registry> \  
      --docker-username=AWS \  
      --docker-password=$(aws ecr get-login-password)
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_10_Kubernetes_Helm_AWS/blob/main/img/15%20creating%20secret%20with%20the%20docker%20registry%20in%201%20step.png" width=800 />

### Configure the Deployment YAML File
1. Open the deployment YAML file.
2. Under spec.template.spec, add the imagePullSecrets section:
   ```bash
       imagePullSecrets:
         - name: my-registry-key 
   ```
3. Specify the container image from your ECR repository

   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_10_Kubernetes_Helm_AWS/blob/main/img/18%20%20deployment%20file.png" width=800 />
   
4. Apply the deployment YAML file:
   ```bash
   kubectl apply -f deployment.yaml
   ```
5. Verify that the pods are running:
   ```bash
   kubectl get pods
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_10_Kubernetes_Helm_AWS/blob/main/img/19%20applying%20deployment.png" width=800 />
   

  

