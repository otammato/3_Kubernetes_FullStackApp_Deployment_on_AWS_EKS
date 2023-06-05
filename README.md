# Kubernetes, a FullStackApp (Node.js / MySQL) Deployment

## Pre-requisites

<details markdown=1><summary markdown="span">Launch a Kubernetes cluster. I am using AWS EKS. Please be aware it's a paid service.</summary>

### Launch a Kubernetes cluster on AWS EKS (use a Terraform template)

1. Clone this official hashicorp repository with terraform templates:


    ```
    git clone https://github.com/hashicorp/learn-terraform-provision-eks-cluster
    ```

> This example repository contains configuration to provision a VPC, security groups, and an EKS cluster with the following architecture:
> <img width="711" alt="Screenshot 2023-05-28 at 17 53 54" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/9ac1f30e-7b2c-4b7c-bcca-95de95b03e04">

<br>
<br>

2. Launch the terraform templates to create infrastructure and a Kubernetes cluster on AWS EKS.

    ```
    terraform init
    terraform validate
    terraform apply
    ```

    <img width="711" alt="Screenshot 2023-05-28 at 17 53 54" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/05c90361-fa22-4b15-8f28-0071bc700691">

<br>
<br>

> If you use AWS Cloud9 as an IDE you also have to disallow AWS Managed Temporary Credentials.
> Go to Cloud9 > Preferences > AWS Settings > AWS Managed Temporary Credentials and turn it off.
> Store your permanent AWS access credentials in the environment. Use ```aws configure``` command.
> 
> <img width="279" alt="Screenshot 2023-05-28 at 14 14 28" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/34f3028e-3fed-4baf-b2af-b1180ea5e5b5">
> <br>
> <img width="584" alt="Screenshot 2023-05-28 at 18 30 32" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/9e0c2e9f-1fba-4496-b28f-798ea60070f2">

<br>
<br>

### Launch and configure a master node to manage the Kubernetes cluster 

1. Launch a new EC2 instance or use the current Cloud9 instance. Here is the example of AWS CLI command to launch a t3.small EC2 instance:

    ```
    aws ec2 run-instances --image-id ami-xxxxxxxx --count 1 --instance-type t3.small --key-name MyKeyPair --security-group-ids sg-903004f8 --subnet-id subnet-6e7f829e
    ```

3. Configure it as a master node to work with a cluster named ```education-eks-hCIH6McB```:

    ```
    aws eks update-kubeconfig --name education-eks-hCIH6McB
    ```

    <img width="711" alt="Screenshot 2023-05-28 at 19 42 24" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/1b444954-1063-4f23-90f4-dff928cdc12a">

<br>
<br>

3. Install ```kubectl```

    ```
    curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.1/2023-04-19/bin/linux/amd64/kubectl
    chmod +x ./kubectl
    mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH
    echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
    kubectl version --short --client
    ```
    <img width="711" alt="Screenshot 2023-05-28 at 19 47 36" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/526e31d2-0d75-4a9a-b6d6-aad35e75c82e">

4. Make sure the master node can access the cluster ```kubectl get svc```

    <img width="491" alt="Screenshot 2023-05-28 at 19 54 05" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/28a52c37-a26f-42fd-8a9c-f33793a482d4">

<br>
<br>

</details>

## Create a deployment

<br><br>
<p align="center" >
  <img width="700" alt="Screenshot 2023-01-31 at 19 23 50" src="https://github.com/otammato/Kubernetes_FullStackApp_Deployment_on_AWS_EKS/assets/104728608/7f4086b3-e800-4316-8009-e4f143727f6c">
</p>

<br><br>

For the deployment I will be pulling images from DockerHub created in the preceding project located here: https://github.com/otammato/FullStack_NodeJS_MySql_Docker.git


<br><br>
<p align="center" >
  <img width="700" alt="Screenshot 2023-01-31 at 19 23 50" src="https://github.com/otammato/Kubernetes_FullStackApp_Deployment_on_AWS_EKS/assets/104728608/f52cd4f8-6030-46bc-af5b-c4c5d0cca544">
</p>

<br><br>

1. Create a Kubernetes manifest and save it as `]`deployment.yaml```

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: node-app
          image: montcarotte/fullstack_nodejs_mysql_demo:node_app
          ports:
            - containerPort: 3000
          env:
            - name: APP_DB_HOST
              value: mysql-service
            - name: APP_DB_USER
              value: "admin"
        - name: mysql-server
          image: montcarotte/fullstack_nodejs_mysql_demo:mysql_server_new
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: "12345678" # the standard security practice for Kubernetes is to manage passwords through a separate secrets.yaml file. For the simplicity and demo purposes I am not doing it here.
---
apiVersion: v1
kind: Service
metadata:
  name: node-app-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: LoadBalancer
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
```

> This is a Kubernetes manifest file written in YAML. It defines a Deployment, two Services, and their associated configurations.
> 
> The Deployment named "my-deployment" specifies the desired state for running two replicas of a containerized application. It uses a selector to match the pods with the label "app: my-app." The template section defines the pod template for the Deployment. Inside the template, there are two containers defined. The first container is named "node-app" and uses the image "montcarotte/fullstack_nodejs_mysql_demo:node_app." It exposes port 3000 and sets environment variables for the application's database host and user. The second container is named "mysql-server" and uses the image "montcarotte/fullstack_nodejs_mysql_demo:mysql_server_new." It exposes port 3306 and sets the root password for the MySQL server.
>
> Please note that this manifest file provides a simplified example, and in a real-world scenario, it is recommended to use secrets or other secure methods to manage sensitive information like passwords.
> 
> Following the Deployment, there are two Service definitions. The first one is named "node-app-service" and exposes port 80. It selects the pods with the label "app: my-app" and forwards traffic to port 3000 of those pods. The type of this Service is LoadBalancer, indicating that it will be externally accessible through a load balancer.
> 
> The second Service is named "mysql-service" and exposes port 3306. It also selects the pods with the label "app: my-app" and forwards traffic to port 3306 of those pods. This Service allows communication with the MySQL server container.
>
> The manifest involves environment variables as the Node.js script suggests using ether the default variables or the provided ones for connecting to the database.
> <p align="center" >
>   <img width="700" alt="Screenshot 2023-01-31 at 19 23 50" 
> src="https://github.com/otammato/Kubernetes_FullStackApp_Deployment_on_AWS_EKS/assets/104728608/7b5fb33a-2fe0-4e99-a64a-12007c7d8d10">
</p>

<br>
<br>

2. Apply the deployment

```
kubectl apply -f deployment.yaml
```

<img width="700" alt="Screenshot 2023-06-05 at 20 18 35" src="https://github.com/otammato/Kubernetes_FullStackApp_Deployment_on_AWS_EKS/assets/104728608/d3d7cb97-95b0-45af-85e4-73fb8a9abefd">
