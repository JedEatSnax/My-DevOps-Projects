# ☸️ Talos Linux v1.10.4 on VirtualBox or bare-metal — Setup Guide

Talos is a container-optimized Linux distribution for Kubernetes. It is designed to be minimal, immutable, and secure by default. Learn more at [Sidero Labs official documentation](https://www.talos.dev/v1.10/introduction/what-is-talos/). This setup provides a development environment for the Talos API. For long-term production, bare-metal setups or Type-1 hypervisors, such as Hyper-V, Proxmox, and KVM/QEMU are recommended. Official Talos setup guide: [here](https://www.talos.dev/v1.10/talos-guides). This guide adds practical tips missing from the official guide.

---

## 💻 Binaries and Packages for Linux Manager

*This machine acts as the Talos cluster management workstation. Recommended distributions are Ubuntu and Debian.*

Install via [Homebrew](https://brew.sh/):

* [Talos Linux CLI (talosctl)](https://www.talos.dev/v1.10/talos-guides/install/talosctl/)
* [Kubernetes CLI (kubectl)](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
* [k9s terminal UI](https://k9scli.io/topics/install/) *(Optional)*

## 💻 Talos VM Configuration (VirtualBox)

* Download: [Bare-metal Talos Linux ISO](https://factory.talos.dev/)
* Control Plane: 2048MB RAM, 20GB Disk, 2 vCPUs
* Worker Node: 2048MB RAM, 20GB Disk, 1 vCPU
* Network Adapter: **Bridged Adapter**
  * Promiscuous Mode: **Allow All**
  * Cable Connected: **Enabled**

---

## 🌐 Set a Static IP on Talos Instances (During Boot)

1. Note the **IP Address** and **Gateway** shown at top right of the Talos Summary page.
2. Press `F3` to open menu.
3. Tab to **Interface** > `Enter`
4. Select your interface (e.g., `enp0s3`) > `Enter`
5. Tab to **Mode** > Change `DHCP` to `Static` > `Enter`
6. Enter the desired static IP details (use same address assigned earlier).
7. Tab to **Save** > `Enter`
8. Repeat for other worker/control plane nodes.

## 🌐 Set a Static IP on Linux Manager (Netplan)

1. Run `ip a` to identify your interface name and current address.

```bash
ip a
```

2. Run `ip route | grep default` to identify your gateway.

```bash
ip route | grep default
```

3. List netplan configs:

```bash
ls /etc/netplan
```

4. Edit your appropriate netplan file (usually `50-cloud-init.yaml`):

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

5. Replace or edit its content like this:

**For Wi-Fi (wlp3s0)**

```yaml
network:
  version: 2
  wifis:
    wlp3s0:
      addresses:
        - <Your IP>/24
      routes:
        - to: default
          via: <Your Gateway>
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
      access-points:
        "<SSID>":
          password: "<WiFi Password>"
      dhcp4: no
```

**For Ethernet (enp0s3)**

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      addresses:
        - <Your IP>/24
      routes:
        - to: default
          via: <Your Gateway>
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
      dhcp4: no
```

6. Apply config:

```bash
sudo netplan try
sudo netplan apply
```

**Note:** `netplan try` will wait for confirmation; auto-reverts if you don't confirm.

---

## ⚙️ Deploy the Talos Control Plane

```bash
export CONTROL_PLANE_IP=<control-plane-IP>

talosctl gen config talos-vbox-cluster https://$CONTROL_PLANE_IP:6443 --output-dir _out

talosctl apply-config --insecure --nodes $CONTROL_PLANE_IP --file _out/controlplane.yaml
```

## ⚙️ Deploy Worker Nodes

```bash
export WORKER_IP=<worker-node-IP>

talosctl apply-config --insecure --nodes $WORKER_IP --file _out/worker.yaml
```

## 🔍 Accessing the Dashboard
Do the `talosctl dashboard -n <node-ip>` to see the Talos dashboard of a specific node.

## 📅 Bootstrap the Cluster + Configure Context

```bash
export TALOSCONFIG=_out/talosconfig

talosctl --talosconfig $TALOSCONFIG config endpoint $CONTROL_PLANE_IP
talosctl --talosconfig $TALOSCONFIG config node $CONTROL_PLANE_IP

talosctl config get-contexts

talosctl --talosconfig $TALOSCONFIG bootstrap
```

## 📆 Fetch Kubeconfig & Test Cluster

```bash
talosctl --talosconfig $TALOSCONFIG kubeconfig ./
kubectl --kubeconfig=./kubeconfig get nodes
kubectl --kubeconfig=./kubeconfig get pods -n kube-system
```

## 📇 Clean-up + Reset Configs

```bash
rm -rf ~/.talos/
rm -rf _out/
rm -f controlplane.yaml worker.yaml talosconfig

unset CONTROL_PLANE_IP
unset WORKER_IP
unset TALOSCONFIG

brew uninstall talosctl
brew uninstall kubectl
brew uninstall k9s
```

---

## 🔧 Notes

* Avoid having multiple `network:` keys in a single netplan config file.
* Talos defaults to `ext4` filesystem unless customized.
* For long-term use, prefer Type-1 hypervisors or bare-metal.
* Lens IDE and `k9s` are great tools to monitor your cluster.

## ☸️ Talos Linux on Docker Desktop — Setup Guide
This setup provides a Docker users to be comfortable with the Talos API. For development or production environments, bare-metal setups or Type-1 / Type-2 hypervisors, such as VirtualBox, Hyper-V, Proxmox, and KVM/QEMU are recommended. Official Talos Docker guide: [here](https://www.talos.dev/v1.10/talos-guides/install/local-platforms/docker/). This guide adds practical tips missing from the official guide.

### 📋 Prerequisites:
1. [Windows Subsystem for Linux](https://learn.microsoft.com/en-us/windows/wsl/install) *(for Windows users)*
2. [Docker Desktop](https://www.docker.com/products/docker-desktop/)
3. [Talos Linux CLI (talosctl)](https://www.talos.dev/v1.10/talos-guides/install/talosctl/)
4. [k9s terminal UI](https://k9scli.io/topics/install/) *(Optional)*


## ⚙️ Creating a Cluster
Do `talosctl cluster create` to create a cluster. Here is the output:
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

## 🔍 Accessing the Dashboard
Do the `talosctl dashboard -n <node-ip>` to see the Talos dashboard of a specific node.

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
```
curl: (28) Failed to connect to 10.96.42.54 port 31918 after 134274 ms: Couldn't connect to server
```

## 📇 Clean-up
Do `talosctl cluster destroy` to destroy the cluster. To confirm that the cluster is destroyed, do `docker ps` to verify.

## 📌 Notes
* There are more commands I did not feature, such as creating multiple clusters and deleting specific clusters. However, it is not necessary to demonstrate it since Talos on Docker lacks all of the APIs necessary for development environments.
