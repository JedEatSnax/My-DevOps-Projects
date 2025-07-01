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
- **Promiscuous Mode** must allow all.
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
1. Do `ip a` to look for your IP address and network interface.
```git bash
ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp2s0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN group default qlen 1000
    link/ether b4:a9:fc:6b:62:c8 brd ff:ff:ff:ff:ff:ff
3: wlp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 0c:7a:15:56:55:68 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.223/24 brd 192.168.1.255 scope global dynamic noprefixroute wlp3s0
       valid_lft 86175sec preferred_lft 86175sec
    inet6 2001:4451:664:2b00:cdad:51fb:1459:2635/64 scope global temporary dynamic 
       valid_lft 258975sec preferred_lft 85706sec
    inet6 2001:4451:664:2b00:e70:cbaa:4da0:9faf/64 scope global dynamic mngtmpaddr noprefixroute 
       valid_lft 258975sec preferred_lft 172575sec
    inet6 fe80::bf98:7c34:ab8a:b70b/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```
2. Do `ip route | grep default` to look for your gateway.
```git bash
default via **192.168.1.1** dev wlp3s0 proto dhcp src 192.168.1.233 metric 600
```
3. Do `ls /etc/netplan` and look at the contents. If you're using a bare-metal Linux device as a manager, you should do `nano 01-network-manager-all.yaml`. If your Linux manager is on VirtualBox, most likely `50-cloud-init.yaml` would only show up. Therefore, you should `nano 50-cloud-init.yaml`.
4. Copy and paste this format in the yaml file. Add the IP address, gateway, and network interface on its respective place.
```git bash
# For wlp3s0 or other Wi-Fi network interfaces:
network:
  version: 2
  wifis:
    wlp3s0:
      addresses:
        - <Your IP address>/24
      routes:
        - to: default
          via: <Your gateway>
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
      access-points:
        "Your_WiFi_Name_or_SSID":
          password: "Your_WiFi_Password"
      dhcp4: no

---

# For enp0s3 or other ethernet network interfaces:
network:
  version: 2
  ethernets:
    enp0s3:
      addresses:
        - <Your IP address>/24
      routes:
        - to: default
          via: <Your gateway>
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
      dhcp4: no
```
5. Do `netplan try` to see if it is compatible. If there are no issues, do `netplan apply`.

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

