# Deploying Odoo to Google Cloud

This guide will walk you through the process of deploying this Odoo application to Google Cloud using Docker and Google Kubernetes Engine (GKE).

## Prerequisites

- A Google Cloud Platform (GCP) account with billing enabled.
- The `gcloud` command-line tool installed and configured.
- The `docker` command-line tool installed.
- The `kubectl` command-line tool installed.

## 1. Build and Push the Docker Image

1.  **Enable the required APIs:**

    ```bash
    gcloud services enable container.googleapis.com containerregistry.googleapis.com
    ```

2.  **Configure Docker to use `gcloud` as a credential helper:**

    ```bash
    gcloud auth configure-docker
    ```

3.  **Set your GCP Project ID:**

    ```bash
    export PROJECT_ID=$(gcloud config get-value project)
    ```

4.  **Build the Docker image:**

    ```bash
    docker build -t gcr.io/${PROJECT_ID}/odoo:latest .
    ```

5.  **Push the Docker image to Google Container Registry (GCR):**

    ```bash
    docker push gcr.io/${PROJECT_ID}/odoo:latest
    ```

6.  **Update `odoo-deployment.yaml` with your Project ID:**

    Open `odoo-deployment.yaml` and replace `your-project-id` with your actual GCP Project ID in the `image` field.

    ```yaml
    # odoo-deployment.yaml
    ...
      containers:
        - name: odoo
          image: gcr.io/your-project-id/odoo:latest # <-- UPDATE THIS LINE
    ...
    ```


## 2. Create a GKE Cluster

1.  **Create a GKE cluster:**

    ```bash
    gcloud container clusters create odoo-cluster --num-nodes=1 --zone=us-central1-a
    ```

    *This will create a new GKE cluster named `odoo-cluster` with one node in the `us-central1-a` zone. You can adjust the number of nodes and the zone as needed.*

2.  **Get the credentials for your new cluster:**

    ```bash
    gcloud container clusters get-credentials odoo-cluster --zone=us-central1-a
    ```

## 3. Deploy the Application

1.  **Apply the Kubernetes manifests:**

    ```bash
    kubectl apply -f postgres-secret.yaml
    kubectl apply -f postgres-pvc.yaml
    kubectl apply -f odoo-pvc.yaml
    kubectl apply -f postgres-deployment.yaml
    kubectl apply -f postgres-service.yaml
    kubectl apply -f odoo-deployment.yaml
    kubectl apply -f odoo-service.yaml
    ```

2.  **Check the status of the deployments:**

    ```bash
    kubectl get deployments
    ```

    *Wait until the `odoo` and `postgres` deployments both show `1/1` in the `READY` column.*

3.  **Get the external IP address of the Odoo service:**

    ```bash
    kubectl get service odoo
    ```

    *It may take a few minutes for an external IP address to be assigned. Once an IP address is assigned, you can access your Odoo application by navigating to that IP address in your web browser.*

## 4. Cleaning Up

To avoid incurring charges to your GCP account, you can delete the resources you created:

```bash
gcloud container clusters delete odoo-cluster --zone=us-central1-a
gcloud container images delete gcr.io/${PROJECT_ID}/odoo:latest --force-delete-tags
```
