# Stack

```bash
brew install --cask docker
brew install kubernetes-cli kubectx helm argo argocd k9s step
```

Enable kubernetes on docker desktop, we will use it instead of minikube.

## Repositories

| Name | URL |
| --- | --- |
| `linkerd` | `https://helm.linkerd.io/stable` |
| `kubernetes-dashboard` | `https://kubernetes.github.io/dashboard` |
| `ingress-nginx` | `https://kubernetes.github.io/ingress-nginx` |
| `argo` | `https://argoproj.github.io/argo-helm` |
| `grafana` | `https://grafana.github.io/helm-charts` |
| `bitnami` | `https://charts.bitnami.com/bitnami` |

## Releases

```bash
step certificate create root.linkerd.cluster.local ca.crt ca.key \
  --profile root-ca --no-password --insecure
step certificate create identity.linkerd.cluster.local issuer.crt issuer.key \
  --profile intermediate-ca --not-after 8760h --no-password --insecure \
  --ca ca.crt --ca-key ca.key

RELEASE() {
  helm upgrade --install --create-namespace "$@"
}

# linkerd control plane
RELEASE -n linkerd linkerd-crds linkerd/linkerd-crds
RELEASE -n linkerd linkerd-control-plane linkerd/linkerd-control-plane \
  --set-file identityTrustAnchorsPEM=ca.crt \
  --set-file identity.issuer.tls.crtPEM=issuer.crt \
  --set-file identity.issuer.tls.keyPEM=issuer.key \
  -f values/linkerd-control-plane.yaml
 
# wait until linkerd control plane is deployed

RELEASE -n linkerd-viz linkerd-viz linkerd/linkerd-viz \
  -f values/linkerd-viz.yaml
  
# use `linkerd viz dashboard` to view dashboard

RELEASE -n ingress-nginx ingress-nginx ingress-nginx/ingress-nginx \
  -f values/ingress-nginx.yaml
  
# add hosts file content to /etc/hosts

# argo projects
RELEASE -n argo argo-workflows argo/argo-workflows \
  -f values/argo-workflows.yaml
RELEASE -n argo argo-cd argo/argo-cd \
  -f values/argo-cd.yaml
RELEASE -n argo argo-rollouts argo/argo-rollouts \
  -f values/argo-rollouts.yaml

# for tcp/udp ingress controller passthrough testing
RELEASE -n csgo csgo ./charts/csgo \
  -f values/csgo.yaml \
  -f csgo.secrets.yaml \
  --set mountPath=$(pwd)/csgo-data
RELEASE -n valheim valheim ./charts/valheim \
  -f values/valheim.yaml \
  -f valheim.secrets.yaml \
  --set mountPath=$(pwd)/valheim-data
```


# Notes

Add `linkerd.io/inject: enabled` as annotation to a namespace if you want to inject proxies to its pods (also restart all its pods after editing)
