# argocd-demo
Presentation of the tool ArgoCD.

<!-- TOC -->
* [Prerequisites](#prerequisites)
  * [Start-up kubernetes dashboard](#start-up-kubernetes-dashboard)
    * [Installation](#installation)
    * [Access to UI](#access-to-ui)
  * [Start-up ArgoCD](#start-up-argocd)
    * [Installation](#installation-1)
    * [Access to UI](#access-to-ui-1)
* [Presentation flow](#presentation-flow)
* [Useful commands](#useful-commands)
<!-- TOC -->

## Prerequisites

Install kubernetes / openshift and [helm](https://helm.sh/) on the machine and [argo-cd](https://argo-cd.readthedocs.io/en/stable/).

> [!TIP]
> If kubernetes installation, you can install [k9s](https://k9scli.io/) or [kubernetes-dashboard](https://github.com/kubernetes/dashboard).

### Start-up kubernetes dashboard

#### Installation
With kubernetes dashboard, you can have some port issues, then run `helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard --set kong.admin.tls.enabled=false` instead.

#### Access to UI
To enable the access to kubernetes dashboard, run `kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8443:443`.

You also nees to create a user with specific access. To do so, create a file calles `dashboard-adminuser.yaml` as follow and run `kubectl apply -f dashboard-adminuser.yaml`.
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```
Then run `kubectl -n kubernetes-dashboard create token admin-user` in order to create a token used to login in the UI.

Go on https://localhost:8443/ and use the previously generated token.

### Start-up ArgoCD

#### Installation
To install argo cd, simply run the following commands:
```sh
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
On openshift, simply use l'operator store.

#### Access to UI

To enable the access to kubernetes dashboard, run `kubectl port-forward svc/argocd-server -n argocd 8080:443`.

The initial password for the admin account is auto-generated and stored as clear text in the field password in a secret named `argocd-initial-admin-secret` in your Argo CD installation namespace

Go on https://localhost:8080/ and use `admin` with the password previously found.

## Presentation flow

1. Create a new branch on the repo called `demo` and apply `demo-application.yaml` resource on argocd.
2. Deploy a new app: merge branch `demo-step-1` into `demo`.
3. Update the app with a new version: : merge branch `demo-step-2` into `demo`
4. Try to update the app with a container that is not able to start: : merge branch `demo-step-3` into `demo`

## Useful commands

```bash
# Get all application resources from all namespaces
kubectl get Application -A
```
