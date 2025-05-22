# Overview:
In this guide, I will show you how to deploy your Kubernetes container globally using [ngrok](https://ngrok.com/) Kubernetes operator. I'll also demonstrate how to install Ingress NGINX and Istio, along with their integrations, using the Helm package manager.

## Prerequisites
- [Azure for Students Subscription](https://education.github.com/pack)
- [Azure Command-Line Interface](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-windows?view=azure-cli-latest&pivots=msi)
- [kubectl on Windows OS](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/)
- [Lens Kubernetes IDE](https://k8slens.dev/)

## Disclaimer and Best Practices
- This guide is based on my experience and memory after deactivating my Azure for Students subscription.
- All commands are based on Windows PowerShell.
- Always document every command you run to keep track of your configuration changes.
- When editing YAML files, document or include comments explaining the reasons behind each change for future reference.
- Don't forget to turn off your Kubernetes cluster after each session to avoid wasting credits.
- It's okay to create another Kubernetes cluster if you want to start over from scratch.
- This guide consolidates insights from various tutorials and resources. I've included links to all the guides I followed.

### Creating a Kubernetes Cluster in Azure
Once your GitHub Student Developer Pack is active:
- Head over to [Microsoft Azure Portal](portal.azure.com)
- Click **Use another account** -> **Sign in with GitHub** to claim your free subscription from GitHub
- Next, I suggest you watch a YouTube guide for a step-by-step demo. Here is the one I followed: [How to Create a Kubernetes Cluster in Azure?](https://youtu.be/YlR9AkDJMMA?si=DZ6g193hB_mphYBK)

### My AKS Cluster Settings
- Dev/Test Configuration
- No availability zones
- Standard A2_v2 Virtual Machine
- Minimum node count: **2**
- Maximum node count: **5**
