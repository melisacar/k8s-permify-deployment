# Deploy an App Using K8s

This repository demonstrates how to deploy the Permify application using Kubernetes on an AWS EKS (Elastic Kubernetes Service) cluster. The infrastructure is managed using Terraform, and the application is exposed via a LoadBalancer service.

## Prerequisites

1. **AWS Account:** An active AWS account with sufficient permissions.
2. **Terraform:** Installed and configured.
3. **kubectl:** Installed and configured for interacting with the Kubernetes cluster.
4. **AWS CLI:** Installed and configured.
5. **Docker:** Installed for building and running containers.
6. **Permify Docker Image:** We use the `permify/permify:latest` image for the Permify service.

## Steps to Deploy

1. **Setup the Infrastructure with Terraform**

    First, you'll need to deploy the infrastructure using Terraform. This step involves creating an EKS cluster and the necessary resources.

    - Clone the repository:

    ```bash
    git clone https://github.com/melisacar/terraform-k8s.git
    cd terraform-k8s
    ```

    - Initialize Terraform:

    ```bash
    terraform init
    ```

    - Apply the Terraform configuration to provision the EKS cluster:

    ```bash
    terraform apply
    ```

    - Once the cluster is created, update your `kubeconfig` to point to your new EKS cluster:

    ```bash
    aws eks --region <region> update-kubeconfig --name <cluster-name>
    ```

2. **Kubernetes Deployment Configuration (`deployment.yaml`)**

    After the cluster is ready, you need to deploy the **(Permify)**[https://docs.permify.co/setting-up/configuration] application using the `deployment.yaml` file. This defines the application, its container image, and its resource requirements.

    Here is the deployment.yaml configuration:

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: permify-deployment
    labels:
        app: permify
    spec:
    replicas: 2
    selector:
        matchLabels:
        app: permify
    template:
        metadata:
        labels:
            app: permify
        spec:
        containers:
        - name: permify
            image: permify/permify:latest
            ports:
            - containerPort: 3478
            env:
            - name: PERMIFY_ENV
            value: "production"
    ```

    Explanation:

    - The `Deployment` specifies that two replicas of the Permify container should be running.
    - The container uses the `permify/permify:latest` image.
    - Port `3478` is exposed inside the container.
    - The environment variable `PERMIFY_ENV` is set to "production".

3. **Kubernetes Service Configuration (`service.yaml`)**

    Next, you need to expose the Permify application to the outside world via a **LoadBalancer**. This is done using the `service.yaml` file.

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
    name: permify-service
    spec:
    selector:
        app: permify
    ports:
    - protocol: TCP
        port: 80
        targetPort: 3476
    type: LoadBalancer
    ```

    Explanation:

    - The `Service` type is set to `LoadBalancer`, which will provision an AWS ELB (Elastic Load Balancer) to route traffic to the pods.
    - The service listens on port `80` and forwards traffic to `3476`, which is the port exposed by the Permify container.

4. **Apply the Kubernetes Configurations**

    Now, apply both the `deployment.yaml` and `service.yaml` to the EKS cluster.

    ```bash
    kubectl apply -f deployment.yaml
    kubectl apply -f service.yaml
    ```

5. **Access the Application**

    Once the resources are deployed, you can check the external IP provided by the LoadBalancer. To get the external IP, run the following command:

    ```bash
    kubectl get svc permify-service
    ```

    This will display the external IP of the LoadBalancer. It may take a few minutes for the IP to be assigned. Once you have the IP, open your browser and navigate to:

    ```bash
    http://<external-ip>/healthz
    ```

6. **Health Check**

    The `/healthz` endpoint is a simple HTTP endpoint that allows you to verify that the **Permify** application is running and healthy. When the service is up and running, you should see the following response in your browser:

    ```bash
    {"status":"SERVING"}
    ```

    This indicates that the application is up and ready to handle requests. It’s a common practice to include a health check endpoint like `/healthz` to ensure that the service is functioning properly.

7. **Troubleshooting**

    If the service is not accessible:

    - Ensure the security groups attached to your LoadBalancer allow inbound traffic on port `80` and `443`.
    - Verify that the pod is running using `kubectl get pods`.
    - Check the LoadBalancer status in the AWS Console.

8. **License**

    This project is licensed under the **MIT License**.