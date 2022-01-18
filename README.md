# library-infrastructure
This repository along with FluxCD provides with infrastructure as code for a [library application](https://github.com/FarahKa/library) and allows for continuous deployment.

## Tools 
- [FluxCD](https://fluxcd.io/) : GitOps tool
- [Helm](https://helm.sh/) : for packaging our kubernetes application
- [Ingress-Nginx](https://kubernetes.github.io/ingress-nginx/deploy/) : for routing requestes to appropriate deployments
- [Istio](https://istio.io/) : for securing communication inside cluster (MTLS)
- [Kiali](https://kiali.io/) : management console for Istio
- [Prometheus](https://prometheus.io/) : for collecting metrics
- [Prometheus Node Exporter](https://prometheus.io/docs/guides/node-exporter/) : for exposing node metrics
- [Grafana](https://grafana.com/) : for better visualization of prometheus metrics

## Environments : 
There are currently 2 working environments : 
- Development
- Production

The idea here was to have the development environment always pull the latest image of the application while the production would only pull a stable version. This way the decision to deploy to production will require a manual decision.

When choosing which environment to deploy, we must bootstrap flux to the appropriate directory in the repository. 
#### For production : 
```
$ flux bootstrap github --owner=SirineAchour --repository=library-infrastructure --branch=main --personal=true --private=false --path=/flux-prod/ --token-auth 
```

#### For development : 
```
$ flux bootstrap github --owner=SirineAchour --repository=library-infrastructure --branch=main --personal=true --private=false --path=/flux-dev/ --token-auth 
``` 
<br><br>
*NOTE:* for easier installation of FluxCD use the official docker image.

```
# docker run -it --entrypoint=sh -v <path to .kube/config in host>:/root/.kube/config --network host ghcr.io/fluxcd/flux-cli:v0.24.1
```

## Step-By-Step Deployment
1. Connect to kubernetes cluster 
2. Create database-credentials secret and apply it to cluster.
```
apiVersion: v1
kind: Secret
metadata:
  name: database-credentials
type: kubernetes.io/basic-auth
stringData:
  DB_USERNAME: ***********
  DB_PASSWORD: ***********
```
3. Install fluxCD <br>
`docker run -it --entrypoint=sh -v <path to .kube/config in host>:/root/.kube/config --network host ghcr.io/fluxcd/flux-cli:v0.24.1`
4. Bootstrap FluxCD to repository <br>
`flux bootstrap github --owner=SirineAchour --repository=library-infrastructure --branch=main --personal=true --private=false --path=/flux-prod/ --token-auth`
5. Wait for the magic to happen 
   Or force it with `flux reconcile source git flux-system`

## Project Structure

```
── apps
│   ├── common ---> contains commonalities between development environments and production environments
│   │   ├── ingress-nginx 
|   |   |   contains ingress-nginx manifests and 'ingress-nginx' namespace definition
│   │   ├── istio-system
|   |   |   contains helm releases (istio/base, istiod and kiali) and 'istio-system' namespace definition
│   │   ├── library
|   |   |   contains helm release for library and 'library' namespace definition
│   │   └── monitoring 
|   |       contains helm releases (prometheus and grafana) and 'monitoring' namespace definition
|   |   
│   ├── dev ---> contains files specific to development environments
│   │   ├── ingress-nginx
|   |   |   contains kustomization pointing to /apps/common/ingress-nginx
│   │   ├── istio-system
|   |   |   contains kustomization pointing to /apps/common/istio-system
│   │   ├── library
|   |   |   contains kustomization pointing to /apps/common/library
│   │   └── monitoring
|   |       contains kustomization pointing to /apps/common/monitoring
|   |   
│   └── prod ---> contains files specific to production environments
│       ├── ingress-nginx
|       |   contains kustomization pointing to /apps/common/ingress-nginx
│       ├── istio-system 
|       |   contains kustomization pointing to /apps/common/istio-system
│       ├── library
|       |   contains kustomization pointing to /apps/common/library
│       └── monitoring
|           contains kustomization pointing to /apps/common/monitoring
|
├── flux-<dev|prod>
│   ├── flux-system
|   |   contains flux manifests (uneditable!)
│   └── kustomizations 
|
── helm-charts/library ---> contains helm chart definition of application

```


