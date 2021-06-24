# Sample implement patterns of Ingress on Azure Kubernetes Service

## Table of Contents

- [About](#about)
- [Getting Started](#getting_started)
- [Usage](#usage)
- [Contributing](../CONTRIBUTING.md)

## About <a name = "about"></a>

Deployment codes for 3 sample implement patterns of Ingress on AKS.

1. Application Gateway Ingress Controller
2. NGINX Ingress Controller
3. Combination of 1 and 2

## Getting Started <a name = "getting_started"></a>

Mainly use Azure CLI and flux v2

### Prerequisites / Tested

* [Azure CLI](https://docs.microsoft.com/ja-jp/cli/azure/install-azure-cli): 2.25.0
  * [aks-preview extension](https://docs.microsoft.com/ja-jp/cli/azure/azure-cli-extensions-overview): 0.5.19
* Azure Kubernetes Service: 1.19.11
* [flux](https://fluxcd.io/docs/get-started/): 0.15.3

## Usage <a name = "usage"></a>

Clone or fork this repo. And export environemt variables related to GitHub.

```
export GITHUB_TOKEN=<your-token>
export GITHUB_USER=<your-username>
```

### 1. Azure Application Gateway Ingress Controller

```
az group create -n rg-ingress-appgw -l japaneast
```

```
az aks create \
  -g rg-ingress-appgw \
  -n <your-prefix>-ingress-appgw \
  --network-plugin="azure" \
  --network-policy="calico" \
  --enable-managed-identity \
  -a ingress-appgw \
  --appgw-name agw-ingress \
  --appgw-subnet-cidr "10.2.0.0/16"
```

```
az aks get-credentials -g rg-ingress-appgw -n <your-prefix>-ingress-nginx --admin
```

```
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=aks-ingress-pattern \
  --branch=main \
  --path=./clusters/appgw \
  --personal
```

After use
```
az group delete -n rg-ingress-appgw
```

### 2. NGINX Ingress Controller

```
az group create -n rg-ingress-nginx -l japaneast
```

```
az aks create \
  -g rg-ingress-nginx \
  -n <your-prefix>-ingress-nginx \
  --network-plugin="azure" \
  --network-policy="calico" \
  --enable-managed-identity
```

```
az aks get-credentials -g rg-ingress-nginx -n <your-prefix>-ingress-nginx --admin
```

```
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=aks-ingress-pattern \
  --branch=main \
  --path=./clusters/nginx \
  --personal
```

After use
```
az group delete -n rg-ingress-nginx
```

### 3. Combination of 1 and 2

```
az group create -n rg-ingress-combi -l japaneast
```

```
az aks create \
  -g rg-ingress-combi \
  -n <your-prefix>-ingress-combi \
  --network-plugin="azure" \
  --network-policy="calico" \
  --enable-managed-identity
```

```
az aks get-credentials -g rg-ingress-combi -n <your-prefix>-ingress-combi --admin
```

```
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=aks-ingress-pattern \
  --branch=main \
  --path=./clusters/combi \
  --personal
```

```
az network vnet subnet create \
  -g <your-aks-node-resource-group> \
  -n snet-appgw \
  --vnet-name <your-aks-vnet> \
  --address-prefix 10.2.0.0/16
```

```
az network public-ip create \
  -g <your-aks-node-resource-group> \
  -n pip-appgw \
  --sku Standard
```

```
az network application-gateway create \
  -g <your-aks-node-resource-group> \
  -n agw-ingress \
  --vnet-name <your-aks-vnet> \
  --subnet snet-appgw \
  --capacity 2 \
  --sku Standard_v2 \
  --http-settings-cookie-based-affinity Disabled \
  --frontend-port 80 \
  --http-settings-port 80 \
  --http-settings-protocol Http \
  --public-ip-address pip-appgw \
  --servers 10.240.255.1
```
You can replace NGINX Ingress Service IP address with --servers option. Match the definition in release vaule.

After use
```
az group delete -n rg-ingress-combi
```

