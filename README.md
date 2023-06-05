# Kubernetes_FullStackApp_Deployment


## Launch a Kubernetes cluster on AWS EKS (use a Terraform template)

1. Clone this official hashicorp repository with terraform templates:


    ```
    git clone https://github.com/hashicorp/learn-terraform-provision-eks-cluster
    ```

> This example repository contains configuration to provision a VPC, security groups, and an EKS cluster with the following architecture:
> <img width="711" alt="Screenshot 2023-05-28 at 17 53 54" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/9ac1f30e-7b2c-4b7c-bcca-95de95b03e04">

<br>
<br>

## Launch and configure a master node to manage the Kubernetes cluster 

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
