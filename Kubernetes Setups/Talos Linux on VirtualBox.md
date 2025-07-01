# 📦 Talos Linux v1.10.4 on VirtualBox — Setup Guide

Talos is a container-optimized Linux distribution for Kubernetes. It is designed to be minimal, immutable, and secure by default. Learn more at [Sidero Labs official documentation](https://www.talos.dev/v1.10/introduction/what-is-talos/). This setup provides a development environment for the Talos API. For long-term production, bare-metal setups or type-1 hypervisors like Hyper-V or Proxmox are recommended. Official Talos VirtualBox guide: [here](https://www.talos.dev/v1.10/talos-guides/install/local-platforms/virtualbox/). This guide adds practical tips missing from the official guide.

---

## 💻 Binaries and Packages for Linux Manager

*This machine acts as the Talos cluster management workstation. Recommended distributions: Ubuntu, Debian.*

Install via [Homebrew](https://brew.sh/):

* [Talos Linux CLI (talosctl)](https://www.talos.dev/v1.10/talos-guides/install/talosctl/)
* [Kubernetes CLI (kubectl)](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
* [k9s terminal UI](https://k9scli.io/topics/install/) (optional)

---

## 💻 Talos VM Configuration (VirtualBox)

* Download: [Bare-metal Talos Linux ISO](https://factory.talos.dev/)
* Control Plane: 2048MB RAM, 20GB Disk, 2 vCPUs
* Worker Node: 2048MB RAM, 20GB Disk, 1 vCPU
* Network Adapter: **Bridged Adapter**

  * Promiscuous Mode: **Allow All**
  * Cable Connected: **Enabled**

---

## 🌐 Set a Static IP on Talos VMs (During Boot)

1. Note the **IP Address** and **Gateway** shown at top right of the Talos Summary page.
2. Press `F3` to open menu.
3. Tab to **Interface** > `Enter`
4. Select your interface (e.g., `enp0s3`) > `Enter`
5. Tab to **Mode** > Change `DHCP` to `Static` > `Enter`
6. Enter the desired static IP details (use same address assigned earlier).
7. Tab to **Save** > `Enter`
8. Repeat for other worker/control plane nodes.

---

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

## 💻 Deploy the Talos Control Plane

```bash
export CONTROL_PLANE_IP=<control-plane-IP>

talosctl gen config talos-vbox-cluster https://$CONTROL_PLANE_IP:6443 --output-dir _out

talosctl apply-config --insecure --nodes $CONTROL_PLANE_IP --file _out/controlplane.yaml
```

---

## 💻 Deploy Worker Nodes

```bash
export WORKER_IP=<worker-node-IP>

talosctl apply-config --insecure --nodes $WORKER_IP --file _out/worker.yaml
```

---

## 📅 Bootstrap the Cluster + Configure Context

```bash
export TALOSCONFIG=_out/talosconfig

talosctl --talosconfig $TALOSCONFIG config endpoint $CONTROL_PLANE_IP
talosctl --talosconfig $TALOSCONFIG config node $CONTROL_PLANE_IP

talosctl config get-contexts

talosctl --talosconfig $TALOSCONFIG bootstrap
```

---

## 📆 Fetch Kubeconfig & Test Cluster

```bash
talosctl --talosconfig $TALOSCONFIG kubeconfig ./
kubectl --kubeconfig=./kubeconfig get nodes
kubectl --kubeconfig=./kubeconfig get pods -n kube-system
```

---

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