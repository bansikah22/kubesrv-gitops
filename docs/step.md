# What is GitOps?

GitOps is a way of implementing Continuous Deployment for cloud native applications. It works by using Git as a single source of truth for declarative infrastructure and applications. With GitOps, the desired state of your system is stored in a Git repository. A software agent, like Flux, running in your cluster, continuously compares the desired state in the Git repository with the actual state of your cluster and automaticallyupdates the cluster to match the desired state.

# CI/CD with GitHub Actions and Flux

Yes, you can absolutely have a CI/CD pipeline with GitHub Actions that works with minikube. Here's how it would work:

1.  **Application Repository (`kubesrv`):**
    *   You would set up a GitHub Actions workflow in your `kubesrv` repository.
    *   This workflow would be triggered on every push to the `main` branch.
    *   The workflow would:
        1.  Build the Docker image.
        2.  Push the Docker image to a container registry (like Docker Hub).
        3.  Trigger the workflow in the `kubesrv-gitops` repository.

2.  **GitOps Repository (`kubesrv-gitops`):**
    *   This repository contains a GitHub Actions workflow that updates the `deployment.yaml` file with the new image tag.
    *   **This workflow is only triggered when the CI workflow in the `kubesrv` repository completes successfully.**
    *   Flux is monitoring this repository.
    *   When you push a change to this repository directly (like you did when you changed the number of replicas), Flux will detect the change and update the cluster. **This is the GitOps workflow, and it does not involve GitHub Actions.**
    *   When the GitHub Actions workflow updates the `deployment.yaml` file, Flux will also detect that change and update the cluster. **This is the CD part of your CI/CD pipeline.**

This setup creates a complete CI/CD pipeline. The "CI" part is handled by GitHub Actions in the `kubesrv` repository (building and pushing the image), and the "CD" part is handled by GitHub Actions in the `kubesrv-gitops` repository and Flux (deploying the new version).

**Can this work with minikube?**

Yes, this will work perfectly with minikube. The only requirement is that your GitHub Actions runner has network access to your minikube cluster. If you are running minikube on your local machine, you would need to use a self-hosted GitHub Actions runner that is also running on your local machine.

# Simulating the CI/CD Pipeline

To simulate the CI/CD pipeline, you can manually trigger the workflow in this repository (`kubesrv-gitops`). Here's how:

1.  Go to the "Actions" tab of the `kubesrv-gitops` repository on GitHub.
2.  In the left sidebar, click on the "CD" workflow.
3.  Click on the "Run workflow" dropdown.
4.  Enter a new image tag in the "Image tag to deploy" field. For example, you can use one of the tags you provided earlier, like `1.2.0`.
5.  Click on the "Run workflow" button.

This will trigger the workflow, which will update the `deployment.yaml` file with the new image tag. Flux will then detect the change and deploy the new version of your application.

# Using the GitHub Actions Workflow

I have created a GitHub Actions workflow file at `.github/workflows/ci.yaml`. This workflow is responsible for updating the `deployment.yaml` file with the new image tag. To use this workflow, you will need to do the following:

1.  **Create secrets:** In this repository (`kubesrv-gitops`), you will need to create the following secret:
    *   `GITOPS_TOKEN`: A GitHub personal access token with `repo` scope for this repository. This is the same token you used to bootstrap Flux.
2.  **Triggering the workflow:** This workflow is designed to be triggered by another workflow, for example, the CI workflow in your `kubesrv` application repository. After your CI workflow builds and pushes the new image, it should trigger this workflow and pass the new image tag as a parameter.

# Summary of Commands

Here is a summary of the commands that were used in this tutorial:

*   **Install Flux CLI:**
    ```bash
    curl -s https://fluxcd.io/install.sh | sudo bash
    ```
*   **Export GitHub Token:**
    ```bash
    export GITHUB_TOKEN=<your_token>
    ```
*   **Bootstrap Flux:**
    ```bash
    flux bootstrap github \
      --owner=bansikah22 \
      --repository=kubesrv-gitops \
      --branch=main \
      --path=./clusters/minikube \
      --personal
    ```
*   **Verify Deployment:**
    ```bash
    flux get all && kubectl get pods && kubectl get svc
    ```
*   **Force Reconciliation:**
    ```bash
    flux reconcile kustomization flux-system --with-source
    ```
*   **Commit and Push Changes:**
    ```bash
    git add . && git commit -m "Your commit message" && git push
    ```

# GitOps Repository Setup Steps

1. **Install Flux CLI**
   ```bash
   curl -s https://fluxcd.io/install.sh | sudo bash
   ```
2. **Create a GitHub Personal Access Token**
    *   **For a personal account:**
        1. Go to your GitHub settings
        2. Click on "Developer settings"
        3. Click on "Personal access tokens"
        4. Click on "Generate new token"
        5. Give your token a name
        6. Select the following scopes:
            *   `repo`
            *   `workflow`
            *   `write:packages`
            *   `read:packages`
            *   `delete:packages`
            *   `admin:org`
            *   `admin:repo_hook`
            *   `admin:org_hook`
            *   `gist`
            *   `notifications`
            *   `user`
            *   `delete_repo`
            *   `write:discussion`
            *   `read:discussion`
            *   `write:public_key`
            *   `read:public_key`
            *   `write:gpg_key`
            *   `read:gpg_key`
        7. Click on "Generate token"
        8. Copy the token and save it in a safe place
    *   **For an organization:**
        1. Go to your organization's page.
        2. Click on "Settings"
        3. In the left sidebar, click on "Developer settings".
        4. In the left sidebar, click on "Personal access tokens".
        5. Click on "Generate new token".
        6. Give your token a name.
        7. Select the same scopes as for a personal account.
        8. Click on "Generate token".
        9. Copy the token and save it in a safe place.
3.  **Export the GitHub Token**
    ```bash
    export GITHUB_TOKEN=<your_token>
    ```
4.  **Run Flux Bootstrap**
    ```bash
    flux bootstrap github \
      --owner=bansikah22 \
      --repository=kubesrv-gitops \
      --branch=main \
      --path=./clusters/minikube \
      --personal
    ```
5.  **Created the directory structure:**
    *   `clusters/minikube/`
    *   `apps/kubesrv/`

6.  **Created the Kubernetes manifests:**
    *   `clusters/minikube/kustomization.yaml`
    *   `apps/kubesrv/deployment.yaml`
    *   `apps/kubesrv/service.yaml`
    *   `apps/kubesrv/kustomization.yaml`

7.  **Updated the cluster configuration:**
    *   Modified `clusters/minikube/kustomization.yaml` to include the `kubesrv` application.