# Install ArgoCD locally from Helm chart

## Install Argo helm repo
```shell
helm repo add argo https://argoproj.github.io/argo-helm
```

## Build dependencies
```shell
helm dependency build helm/argo-cd
```

## Apply ArgoCD Helm charts
```shell
helm install argo-cd helm/argo-cd --namespace argocd --create-namespace
```

## Enter UI locally:

1. Port-forward argocd-server
```shell
kubectl -n argocd port-forward svc/argo-cd-argocd-server 8080:443
```

2. Navigate to https://localhost:8080

3. Admin credentials:
Username: Admin
Password: `kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d`

4. Change admin password:
![image](./img/change-admin-password.png)

## TODO: Test in EKS

# Install Argo workflow

TODO: I need someone that can provide helm chart to install workflow.
deliverable: README.md. - so i can deploy workflow on my existing cluster.