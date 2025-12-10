# DevSecOps Pipeline Implementation for Tic Tac Toe Game

![Screenshot 2025-03-04 at 7 16 48 PM](https://github.com/user-attachments/assets/7ed79f9c-9144-4870-accd-500085a15592)

This repository contains a classic Tic Tac Toe game built with React, TypeScript, and Vite. The primary focus of this project is to demonstrate a comprehensive DevSecOps CI/CD pipeline using GitHub Actions, Docker, Trivy for security scanning, and automated deployment to a Kubernetes cluster.

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
-   **CI/CD**: GitHub Actions
-   **Containerization**: Docker
-   **Vulnerability Scanning**: Trivy
-   **Deployment**: Kubernetes (AKS)

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
    -   The Kubernetes manifests (`deployment.yaml`, `service.yaml`, etc.) are applied to the cluster, triggering a rolling update of the application.

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
    The application will be available at `http://localhost:5173`.

### Running Tests

To run the unit tests for the game logic:
```bash
npm test
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

## Kubernetes Deployment

The `/kubernetes` directory contains the necessary manifests to deploy the application to a Kubernetes cluster. While the CI/CD pipeline automates this, you can also deploy it manually.

### Prerequisites

-   A running Kubernetes cluster.
-   `kubectl` configured to connect to your cluster.
-   An Ingress controller (like NGINX) installed in your cluster.

### Manual Deployment Steps

1.  **Create the namespace:**
    ```bash
    kubectl apply -f kubernetes/namespace.yaml
    ```

2.  **Create an image pull secret for GHCR:**
    Replace the placeholders with your GitHub credentials.
    ```bash
    kubectl create secret docker-registry ghcr-secret \
      --namespace=tictaktoe \
      --docker-server=ghcr.io \
      --docker-username=YOUR_GITHUB_USERNAME \
      --docker-password=YOUR_GITHUB_TOKEN
    ```

3.  **Apply the manifests:**
    Ensure the `image` field in `kubernetes/deployment.yaml` points to a valid image tag in GHCR.
    ```bash
    kubectl apply -f kubernetes/
    ```

4.  **Access the application:**
    Once deployed, the application will be exposed via the `tictaktoe-service` and, if configured, through the Ingress.