# Install ArgoCD locally from Helm chart

## Install Argo helm repo
```shell
helm repo add argo https://argoproj.github.io/argo-helm
```

## Apply ArgoCD Helm charts
```shell
helm install argocd argo/argo-cd --namespace argocd --create-namespace --version 5.13.5
```

## Enter UI locally:
1. Port-forward argocd-server
```shell
kubectl -n argocd port-forward svc/argocd-server 8080:443
```

2. Navigate to https://localhost:8080

3. Admin credentials:
Username: Admin
Password: `kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d`

4. Change admin password:
![image](./img/change-admin-password.png)

5. You should delete the initial secret afterwards as suggested by the Getting Started Guide: https://argo-cd.readthedocs.io/en/stable/getting_started/#4-login-using-the-cli
```shell
kubectl -n argo delete secret argocd-initial-admin-secret
```

# Install Argo workflow

## If it's not installed already, Install Argo helm repo
```shell
helm repo add argo https://argoproj.github.io/argo-helm
```

## Apply Argo Workflow Helm charts
```shell
helm install argowf -f helm/argo-wf/values.yaml argo/argo-workflows --namespace argo --create-namespace --version 0.20.6
```

## Enter UI locally:
1. Port-forward argo-workflows-server
```shell
kubectl -n argo port-forward deployment/argowf-argo-workflows-server 2746:2746
```

2. Navigate to https://localhost:2746

## Install Argo CLI to interact with Workflows
https://github.com/argoproj/argo-workflows/releases/tag/v3.4.3

For Linux:
```shell
# Download the binary
curl -sLO https://github.com/argoproj/argo-workflows/releases/download/v3.4.3/argo-linux-amd64.gz

# Unzip
gunzip argo-linux-amd64.gz

# Make binary executable
chmod +x argo-linux-amd64

# Move binary to path
mv ./argo-linux-amd64 /usr/local/bin/argo

# Test installation
argo version
```

## Service Account used by Argo Workflows
This is shipped from helm chart
- **SA:** argo-workflow
- **Role:** argowf-argo-workflows-workflow
- **RoleBinding:** argowf-argo-workflows-workflow

## Create AWS S3 secrets
This is used by the requested workflow and the Artifact Repository below:

1. Export AWS secrets as env vars
```shell
export AWS_ACCESS_KEY_ID="replace-with-access-key"
export AWS_SECRET_ACCESS_KEY="replace-with-secret-key"
export AWS_DEFAULT_REGION="replace-with-default-region"
```

2. Create secret in argo namespace
```shell
kubectl -n argo create secret generic aws-s3-creds \
    --from-literal=accessKey=$AWS_ACCESS_KEY_ID \
    --from-literal=secretKey=$AWS_SECRET_ACCESS_KEY \
    --from-literal=region=$AWS_DEFAULT_REGION
```

## Create volume
This is to be passed around containers as workspace
```
kubectl create -f wf/local-volume.yaml
```

## Run Argo Workflow
[Workflow details](./wf/requested-workflow.yaml)

For this case:
- Local volume to pass around containers
- S3 input bucket: s3://inputzzz
- S3 output bucket: s3://outputzzz
```shell
argo -n argo submit wf/requested-workflow.yaml --serviceaccount argo-workflow --watch
```

## Argo Workflow Useful commands
```shell
# List workflows
argo -n argo list

# See logs
argo -nargo logs process-bak-files6hsm7

# Delete workflow
argo -n argo delete process-bak-files6hsm7
```