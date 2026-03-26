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