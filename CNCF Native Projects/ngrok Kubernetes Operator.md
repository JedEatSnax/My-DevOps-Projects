# 🤖 ngrok Kubernetes Operator — Setup Guide
*Placeholder*

### 📋 Prerequisites:
1. [Helm package manager](https://helm.sh/docs/intro/install/)
2. Your Kubernetes machine, virtual machine, or cloud provider.
3. Placeholder

### Add ngrok to your Helm chart
```bash
helm repo add ngrok https://charts.ngrok.com
```

## Install the latest ngrok version
*Set the appropriate values for your environment. [Here](https://github.com/ngrok/ngrok-operator) is the official guide. Remove the backslash or '\' for Windows OS users.*
```bash
export NAMESPACE=[YOUR_K8S_NAMESPACE]
export NGROK_AUTHTOKEN=[AUTHTOKEN]
export NGROK_API_KEY=[API_KEY]

helm install ngrok-operator ngrok/ngrok-operator \
  --namespace $NAMESPACE \
  --create-namespace \
  --set credentials.apiKey=$NGROK_API_KEY \
  --set credentials.authtoken=$NGROK_AUTHTOKEN
```

## Create your yaml file

## 🚧 Work in progress 🚧