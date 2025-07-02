# ☸️ Talos Linux on Docker Desktop — Setup Guide
Talos is a container-optimized Linux distribution for Kubernetes. It is designed to be minimal, immutable, and secure by default. Learn more at [Sidero Labs official documentation](https://www.talos.dev/v1.10/introduction/what-is-talos/). This setup provides a Docker users to be comfortable with the Talos API. For development or production environments, bare-metal setups or Type-1 / Type-2 hypervisors, such as VirtualBox, Hyper-V, and Proxmox are recommended. Official Talos Docker guide: [here](https://www.talos.dev/v1.10/talos-guides/install/local-platforms/docker/). This guide adds practical tips missing from the official guide.

### 📋 Prerequisites:
1. [Windows Subsystem for Linux](https://learn.microsoft.com/en-us/windows/wsl/install) *(for Windows users)*
2. [Docker Desktop](https://www.docker.com/products/docker-desktop/)
3. [Talos Linux CLI (talosctl)](https://www.talos.dev/v1.10/talos-guides/install/talosctl/)
4. [k9s terminal UI](https://k9scli.io/topics/install/) *(Optional)*


## ⚙️ Creating a Cluster
Do `talosctl cluster create` to create a cluster.
```bash
merging kubeconfig into "/root/.kube/config"
PROVISIONER           docker
NAME                  talos-default
NETWORK NAME          talos-default
NETWORK CIDR          10.5.0.0/24
NETWORK GATEWAY       10.5.0.1
NETWORK MTU           1500
KUBERNETES ENDPOINT   https://127.0.0.1:43887

NODES:

NAME                            TYPE           IP         CPU    RAM      DISK
/talos-default-controlplane-1   controlplane   10.5.0.2   2.00   2.1 GB   -
/talos-default-worker-1         worker         10.5.0.3   2.00   2.1 GB   -
```

You can increase the amount of workers by adding the `--workers <Worker Nodes>`
```bash
talosctl cluster create --workers 2
```

## ⚠️ Disclaimer and Limitations
Certain APIs are unavailable since Talos is running on a container. For instance, if you run a simple container, such as `kubectl run nginx --image=nginx --port=80`, this is the output:
```bash
Warning: would violate PodSecurity "restricted:latest": allowPrivilegeEscalation != false (container "nginx" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "nginx" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "nginx" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "nginx" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
pod/nginx created
```

By doing the `kubectl get pods` command, this will be the output:
```bash
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          9m42s
```

There are two ways to expose a pod: by port forwarding and creating a Kubernetes service (NodePort).

### 🚪 Port Forwarding
First, locate the pod you want to expose using `kubectl get pods`, then forwarding its port `kubectl port-forward pod/<pod-name> 8080:80`. Lastly, access it locally via `http://localhost:8080`

### 🌍 Create a Kubernetes Service (NodePort)
First, locate the pod you want to expose using `kubectl get pods`, next, expose the pod `kubectl expose pod <pod-name> --type=NodePort --port=80 --target-port=80`. Do `kubectl get svc` to see the IP address. It would look something like this:
```bash
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1     <none>        443/TCP        11m
nginx        NodePort    10.96.42.54   <none>        80:31918/TCP   10s
```

Lastly, open a browser or `curl` the URL.
```bash
curl http://10.96.42.54:31918
```

Since certain APIs are unavailable, port forwarding locally would result to refusal to connect. In addition, using a NodePort will fail because it will take too long to respond. Here is the output for the `curl` command:
```bash
curl: (28) Failed to connect to 10.96.42.54 port 31918 after 134274 ms: Couldn't connect to server
```

## 📇 Clean-up
Do `talosctl cluster destroy` to destroy the cluster. To confirm that the cluster is destroyed, do `docker ps` to verify.

## 📌 Notes
* There are more commands I did not feature, such as creating multiple clusters and deleting specific clusters. However, it is not necessary to demonstrate it since Talos on Docker lacks all of the APIs necessary for development environments.