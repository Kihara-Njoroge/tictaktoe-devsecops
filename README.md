# DevSecOps Pipeline Implementation for Tic Tac Toe Game


This repository contains a classic Tic Tac Toe game built with React, TypeScript, and Vite. The primary focus of this project is to demonstrate a comprehensive DevSecOps CI/CD pipeline using GitHub Actions, Docker, Trivy for security scanning, and automated deployment to Azure Kubernetes Service (AKS).
## Architecture

```
┌─────────────────────────────────────────────────┐
│              GitHub Repository                   │
│  (Source Code + Kubernetes Manifests)           │
└────────────────┬────────────────────────────────┘
                 │
                 │ Push to master
                 │
┌────────────────▼────────────────────────────────┐
│           GitHub Actions CI/CD                   │
│  • Test & Lint                                  │
│  • Build React App                              │
│  • Build Docker Image                           │
│  • Scan with Trivy                              │
│  • Push to GHCR                                 │
│  • Update K8s Manifests                         │
│  • Deploy to AKS                                │
└────────────────┬────────────────────────────────┘
                 │
                 │ Deploy
                 │
┌────────────────▼────────────────────────────────┐
│       Azure Kubernetes Service (AKS)            │
│  ┌──────────────────────────────────────────┐  │
│  │        Namespace: tictaktoe              │  │
│  │  ┌────────────────────────────────────┐  │  │
│  │  │   Deployment (3 replicas)          │  │  │
│  │  │   • Pod 1: Nginx + React App       │  │  │
│  │  │   • Pod 2: Nginx + React App       │  │  │
│  │  │   • Pod 3: Nginx + React App       │  │  │
│  │  └────────────────────────────────────┘  │  │
│  │                    │                      │  │
│  │  ┌─────────────────▼──────────────────┐  │  │
│  │  │    Service (LoadBalancer)          │  │  │
│  │  │    Port 80 → Pod Port 80           │  │  │
│  │  └─────────────────┬──────────────────┘  │  │
│  └────────────────────┼──────────────────────┘  │
└───────────────────────┼─────────────────────────┘
                        │
                        │ Azure Load Balancer
                        │
┌───────────────────────▼─────────────────────────┐
│            Public Internet                       │
│         http://EXTERNAL-IP                       │
└──────────────────────────────────────────────────┘
```

## Features

-   Classic Tic Tac Toe gameplay.
-   Scoreboard to track wins for 'X', 'O', and draws.
-   Game history log with timestamps.
-   Responsive design for various screen sizes.
-   "New Game" and "Reset Stats" functionality.
-   Visual indication of the winning line.

## Tech Stack

-   **Frontend**: React, TypeScript, Vite, Tailwind CSS
-   **Testing**: Vitest
-   **Linting**: ESLint
-   **CI/CD**: GitHub Actions
-   **Containerization**: Docker
-   **Container Registry**: GitHub Container Registry (GHCR)
-   **Vulnerability Scanning**: Trivy
-   **Deployment**: Azure Kubernetes Service (AKS)

## DevSecOps Pipeline Overview

The CI/CD pipeline is defined in `.github/workflows/ci-cd.yml` and automates the entire process from code commit to deployment.

1.  **Trigger**: The workflow is triggered on every `push` and `pull_request` to the `master` branch.

2.  **Test & Lint**:
    -   `test`: Runs unit tests using Vitest to ensure code correctness.
    -   `lint`: Performs static code analysis with ESLint to enforce code quality standards.

3.  **Build**:
    -   The React application is built for production, creating optimized static assets.

4.  **Dockerize & Scan**:
    -   A multi-stage Dockerfile is used to create a lightweight production image based on Nginx.
    -   The Docker image is scanned for `CRITICAL` and `HIGH` severity vulnerabilities using Trivy. The pipeline will fail if any are found.
    -   The image is tagged with the Git SHA and pushed to GitHub Container Registry (GHCR).

5.  **Update Kubernetes Manifest**:
    -   On a push to `master`, a job automatically updates the `kubernetes/deployment.yaml` file with the new Docker image tag.
    -   This change is committed back to the repository, providing a GitOps-style audit trail.

6.  **Deploy to AKS**:
    -   The workflow connects to an Azure Kubernetes Service (AKS) cluster.
    -   It creates the necessary namespace and image pull secrets.
    -   The Kubernetes manifests (`deployment.yaml`, `service.yaml`) are applied to the cluster, triggering a rolling update of the application.
    -   The deployment is verified and service endpoints are displayed.

## Getting Started

To run this project locally, follow these steps.

### Prerequisites

-   Node.js (v20 or later)
-   npm

### Local Development

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/kihara-njoroge/tictaktoe-devsecops.git
    cd tictaktoe-devsecops
    ```

2.  **Install dependencies:**
    ```bash
    npm ci
    ```

3.  **Run the development server:**
    ```bash
    npm run dev
    ```
    The application will be available at `http://localhost:3000`.

### Running Tests

To run the unit tests for the game logic:
```bash
npm test
```

### Running Linter

To check code quality:
```bash
npm run lint
```

## Containerization with Docker

You can build and run the application using Docker.

1.  **Build the Docker image:**
    ```bash
    docker build -t tictaktoe-app .
    ```

2.  **Run the Docker container:**
    ```bash
    docker run -d -p 8080:80 tictaktoe-app
    ```
    The application will be accessible at `http://localhost:8080`.

## Azure Kubernetes Service (AKS) Setup

### Prerequisites

Before deploying to AKS, ensure you have:

-   An active Azure subscription
-   Azure CLI installed ([Installation Guide](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli))
-   kubectl installed ([Installation Guide](https://kubernetes.io/docs/tasks/tools/))
-   A GitHub account with a Personal Access Token (PAT) with `read:packages` and `write:packages` permissions

### Step 1: Install Azure CLI

**macOS:**
```bash
brew install azure-cli
```

**Linux/WSL:**
```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

**Windows:**
Download and install from [here](https://aka.ms/installazurecliwindows)

### Step 2: Login to Azure

```bash
az login
```

### Step 3: Set Your Subscription

```bash
# List all subscriptions
az account list --output table

# Set active subscription
az account set --subscription "YOUR_SUBSCRIPTION_NAME_OR_ID"

# Verify
az account show --output table
```

### Step 4: Create Resource Group

```bash
az group create \
  --name tictaktoe-rg \
  --location eastus
```

Available locations: `eastus`, `westeurope`, `southeastasia`, `uksouth`, etc.

### Step 5: Create AKS Cluster

**Basic Setup (Recommended for testing):**
```bash
az aks create \
  --resource-group tictaktoe-rg \
  --name tictaktoe-aks \
  --node-count 2 \
  --node-vm-size Standard_B2s \
  --enable-managed-identity \
  --generate-ssh-keys \
  --network-plugin azure \
  --load-balancer-sku standard
```

**Production Setup (with autoscaling):**
```bash
az aks create \
  --resource-group tictaktoe-rg \
  --name tictaktoe-aks \
  --node-count 3 \
  --node-vm-size Standard_D2s_v3 \
  --enable-managed-identity \
  --enable-cluster-autoscaler \
  --min-count 2 \
  --max-count 5 \
  --generate-ssh-keys \
  --network-plugin azure \
  --load-balancer-sku standard \
  --zones 1 2 3
```

⏱️ **Cluster creation takes 5-10 minutes**

### Step 6: Configure kubectl

```bash
az aks get-credentials \
  --resource-group tictaktoe-rg \
  --name tictaktoe-aks \
  --overwrite-existing
```

Verify connection:
```bash
kubectl get nodes
```

### Step 7: Create Service Principal for GitHub Actions

```bash
# Get subscription ID
SUBSCRIPTION_ID=$(az account show --query id --output tsv)

# Create service principal
az ad sp create-for-rbac \
  --name "github-actions-tictaktoe" \
  --role contributor \
  --scopes /subscriptions/$SUBSCRIPTION_ID/resourceGroups/tictaktoe-rg \
  --sdk-auth
```

**⚠️ Save the entire JSON output!** You'll need it for GitHub secrets.

### Step 8: Configure GitHub Secrets

Go to your GitHub repository → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**

Add these secrets:

| Secret Name | Value |
|------------|-------|
| `AZURE_CREDENTIALS` | Entire JSON output from Step 7 |
| `AZURE_RESOURCE_GROUP` | `tictaktoe-rg` |
| `AZURE_CLUSTER_NAME` | `tictaktoe-aks` |
| `TOKEN` | GitHub Personal Access Token with `write:packages` and `read:packages` permissions |

### Step 9: Allow HTTP Traffic (Important!)

Azure Network Security Groups block external traffic by default. Allow port 80:

```bash
# Get NSG name
NSG_NAME=$(az network nsg list \
  --resource-group MC_tictaktoe-rg_tictaktoe-aks_eastus \
  --query "[0].name" -o tsv)

# Create rule to allow HTTP
az network nsg rule create \
  --resource-group MC_tictaktoe-rg_tictaktoe-aks_eastus \
  --nsg-name $NSG_NAME \
  --name AllowHTTP \
  --priority 100 \
  --source-address-prefixes '*' \
  --source-port-ranges '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 80 \
  --access Allow \
  --protocol Tcp \
  --direction Inbound
```

**Note:** Replace `eastus` with your chosen location if different.

## Automated Deployment

Once the AKS cluster is set up and GitHub secrets are configured:

1.  **Push to master branch:**
    ```bash
    git add .
    git commit -m "Deploy to AKS"
    git push origin master
    ```

2.  **Watch the deployment:**
    - Go to your repository on GitHub
    - Click the **Actions** tab
    - Monitor the workflow execution

3.  **Get your application URL:**
    ```bash
    kubectl get service tictaktoe-service -n tictaktoe
    ```
    
    Wait for `EXTERNAL-IP` to be assigned (takes 2-5 minutes), then access your app at:
    ```
    http://EXTERNAL-IP
    ```

## Manual Kubernetes Deployment

If you prefer to deploy manually without CI/CD:

### Prerequisites

-   A running AKS cluster
-   `kubectl` configured to connect to your cluster

### Deployment Steps

1.  **Create the namespace:**
    ```bash
    kubectl create namespace tictaktoe
    ```

2.  **Create image pull secret:**
    ```bash
    kubectl create secret docker-registry ghcr-secret \
      --namespace=tictaktoe \
      --docker-server=ghcr.io \
      --docker-username=YOUR_GITHUB_USERNAME \
      --docker-password=YOUR_GITHUB_TOKEN
    ```

3.  **Apply the manifests:**
    ```bash
    kubectl apply -f kubernetes/deployment.yaml
    ```

4.  **Check deployment status:**
    ```bash
    kubectl get pods -n tictaktoe
    kubectl get services -n tictaktoe
    ```

5.  **Get the external IP:**
    ```bash
    kubectl get service tictaktoe-service -n tictaktoe
    ```
    
    Access your application at `http://EXTERNAL-IP`

## Monitoring and Debugging

### View pod logs:
```bash
kubectl logs -f deployment/tictaktoe-deployment -n tictaktoe
```

### Check pod status:
```bash
kubectl get pods -n tictaktoe
```

### Describe a pod:
```bash
kubectl describe pod POD_NAME -n tictaktoe
```

### View all resources:
```bash
kubectl get all -n tictaktoe
```

### Check events:
```bash
kubectl get events -n tictaktoe --sort-by='.lastTimestamp'
```

### Port forward for local testing:
```bash
kubectl port-forward service/tictaktoe-service 8080:80 -n tictaktoe
```
Then access at `http://localhost:8080`

## Scaling the Application

### Manual scaling:
```bash
kubectl scale deployment tictaktoe-deployment -n tictaktoe --replicas=5
```

### Enable cluster autoscaler:
```bash
az aks update \
  --resource-group tictaktoe-rg \
  --name tictaktoe-aks \
  --enable-cluster-autoscaler \
  --min-count 2 \
  --max-count 5
```

## Cost Management

### Check cluster costs:
```bash
az aks show \
  --resource-group tictaktoe-rg \
  --name tictaktoe-aks \
  --query "agentPoolProfiles[].{Name:name, Count:count, Size:vmSize}"
```

### Stop cluster (to save costs):
```bash
az aks stop --resource-group tictaktoe-rg --name tictaktoe-aks
```

### Start cluster:
```bash
az aks start --resource-group tictaktoe-rg --name tictaktoe-aks
```

## Cleanup

### Delete the application:
```bash
kubectl delete namespace tictaktoe
```

### Delete the AKS cluster:
```bash
az aks delete \
  --resource-group tictaktoe-rg \
  --name tictaktoe-aks \
  --yes --no-wait
```

### Delete everything (resource group):
```bash
az group delete --name tictaktoe-rg --yes --no-wait
```

## Troubleshooting

### Issue: Pods in ImagePullBackOff state
**Solution:** Check if the image pull secret is correct:
```bash
kubectl get secret ghcr-secret -n tictaktoe
```

Recreate if needed:
```bash
kubectl delete secret ghcr-secret -n tictaktoe
kubectl create secret docker-registry ghcr-secret \
  --namespace=tictaktoe \
  --docker-server=ghcr.io \
  --docker-username=YOUR_GITHUB_USERNAME \
  --docker-password=YOUR_GITHUB_TOKEN
```

### Issue: Pods in CrashLoopBackOff state
**Solution:** Check the logs:
```bash
kubectl logs deployment/tictaktoe-deployment -n tictaktoe
```

### Issue: Cannot access application via external IP
**Solution:** 
1. Verify NSG rules allow port 80 (see Step 9)
2. Check if LoadBalancer has assigned an IP:
   ```bash
   kubectl describe service tictaktoe-service -n tictaktoe
   ```

### Issue: Deployment timeout
**Solution:** Check pod status and events:
```bash
kubectl get pods -n tictaktoe
kubectl describe pod POD_NAME -n tictaktoe
kubectl get events -n tictaktoe --sort-by='.lastTimestamp'
```

## Security Features

-   **Vulnerability Scanning**: Trivy scans Docker images for known vulnerabilities
-   **Code Quality**: ESLint enforces code standards
-   **Automated Testing**: Vitest ensures code correctness
-   **Secure Secrets**: All sensitive data stored in GitHub Secrets
-   **Image Pull Secrets**: Kubernetes secret for private container registry access
-   **Network Security**: Azure NSG controls inbound/outbound traffic

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License.

## Acknowledgments

-   Built with React, TypeScript, and Vite
-   Deployed on Azure Kubernetes Service
-   CI/CD powered by GitHub Actions
-   Security scanning by Trivy