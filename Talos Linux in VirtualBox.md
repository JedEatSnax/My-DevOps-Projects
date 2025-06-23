# 📦 Talos Linux v1.10.4 on VirtualBox — Setup Guide
Talos is a container optimized Linux distribution for Kubernetes. It is designed to be minimal, immutable, and secure by default. Learn more at [Sidero Labs official documentation](https://www.talos.dev/v1.10/introduction/what-is-talos/). This setup is a development environment for the Talos API. It is recommended to use bare-metal setups or type-1 hypervisors, such as Hyper-V or Proxmox for long-term production environments. The official setup guide from Siderolabs is [here](https://www.talos.dev/v1.10/talos-guides/install/local-platforms/virtualbox/). The purpose of this guide is to add information that is missing from the official guide.

## 🖥️ Binaries and Packages for Linux Manager:
*This machine will act as a manager for the Talos API. The recommended Linux distributions are Ubuntu and Debian. I suggest downloading the binaries using Homebrew.*
- [Homebrew Package Manager](https://brew.sh/)
- [Talos Linux CLI (talosctl)](https://www.talos.dev/v1.10/talos-guides/install/talosctl/)
- [Kubernetes CLI (kubectl)](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-using-other-package-management)
- [k9s terminal UI](https://k9scli.io/topics/install/) (Optional)

## 🖥️ Talos Linux Virtual Machine Configurations:
- [Bare-metal Talos Linux ISO](https://factory.talos.dev/)
- 2048 MB of memory, 20 GB of disk space
- 2 CPUs for Control Planes, 1 CPU for Workers
- All virtual machines must be configured to use a **Bridged Adapter**.
- **Promiscuous Mode** must allow VMs.
- **Cable Connected** must be allowed.

---

## 🌐 Setting a Static IP address on Talos VMs using Network Config:
*This step is essential because all Talos VMs will reboot. The IP addresses will change because DHCP is on by default.*
1. Take note of the **IP address** and **Gateway** displayed at the top right of the Talos **Summary**.
![Summary](https://github.com/user-attachments/assets/d0b8b2e0-e359-4209-9de1-373b49310c0c)
2. Press `F3` to open the menu.
3. Navigate to **Interface** using `Tab`, then press `Enter`.
4. Select your Network Interface. Mine was `enp0s3`, then press `Enter`.
5. Use `Tab` to navigate to **Mode**, switch from `DHCP` to `Static`, and press `Enter`.
6. Enter your desired static IP configuration. I used the same IP address initially provided by Talos.
7. Press `Tab` to select **Save** and hit `Enter`.
![NetworkConfig](https://github.com/user-attachments/assets/16cf5d15-592f-410d-b016-240cc31e9c38)
8. Repeat for any worker nodes or additional control planes.

## 🌐 Setting a Static IP address on Linux Manager using Netplan:
*Placeholder*
1. Do `ip -a`
2. Look at your IP address and Network Interface
3. Copy paste format
4. Stuff

---

## 🖥️ Setting up the Control Plane

```bash
export CONTROL_PLANE_IP=<control-plane-IP>

talosctl gen config talos-vbox-cluster https://$CONTROL_PLANE_IP:6443 --output-dir _out

talosctl apply-config --insecure --nodes $CONTROL_PLANE_IP --file _out/controlplane.yaml
```

## 🖥️ Setting up a Worker Node
```bash
export WORKER_IP=<worker-node-IP>

talosctl apply-config --insecure --nodes $WORKER_IP --file _out/worker.yaml
```

## 📝 Bootstrap the Cluster and Set Talos Config
```bash
export TALOSCONFIG=_out/talosconfig

talosctl --talosconfig $TALOSCONFIG config endpoint $CONTROL_PLANE_IP
talosctl --talosconfig $TALOSCONFIG config node $CONTROL_PLANE_IP

talosctl config get-contexts
```
*If you have multiple control planes, change the IP addresses accordingly to bootstrap all control planes.*
```bash
talosctl --talosconfig $TALOSCONFIG config endpoint <second-control-plane-ip>
talosctl --talosconfig $TALOSCONFIG config node <second-control-plane-ip>
```

---

## 🗑️ Delete or Reset Talos Config and Clean Working Directory
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

