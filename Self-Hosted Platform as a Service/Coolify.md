# 🇨 Coolify v4.0.0-beta.420.6 — Setup Guide
This is a self-hosting guide for Coolify. The official Coolify self-hosted guide is [here](https://coolify.io/docs/get-started/installation). This guide mainly features my security configurations for Coolify on virtual machines or bare-metal instances. I had trouble configuring Coolify on Type-1 and Type-2 hypervisors, such as KVM/QEMU and Virtual Box.

### Prerequisites
1. A registered domain name on Cloudflare
2. A virtual private server or bare-metal instance with at least 2 CPU cores, 2 GB memory, and 20 GB disk space
3. Knowledge of configuring SSH


### Featured Security Measures:
1. Tailscale mesh VPN
2. Uncomplicated Firewall (ufw)
3. Coolify Two-Factor Authentication and HTTPS Dashboard Encryption
4. Cloudflare Origin Certificate
5. Fail2Ban Intrusion Prevention Software Framework

## Update Server Binaries and Dependencies while Enabling Automatic Security Updates
```
sudo apt-get update --fix-missing -y; sudo apt-get full-upgrade -y; sudo apt autoremove -y; sudo apt autoclean -y; \
sudo apt install unattended-upgrades; sudo dpkg-reconfigure --priority=low unattended-upgrades -y
```

## Install and Setup Tailscale SSH
**Warning: Installing Tailscale will give you an accessible public IP address. Others can use this IP regardless if Uncomplicated Firewall is closed due to Docker container networking being NAT-based.**
*Run this shell script to your client and host instances.*
```
curl -fsSL https://tailscale.com/install.sh | sh
```
*After that, run `sudo tailscale up` to register your instances. Clicking the link will provide access to the Tailscale admin dashboard to manage your registered devices.*

## Setting up Uncomplicated Firewall (ufw) with Tailscale SSH
*Install uncomplicated firewall to configure your server ports.*
```
sudo apt install ufw; sudo ufw enable; sudo ufw default deny incoming; sudo ufw default allow outgoing
```
*Allow your client to the server.*
```
#Use your client IP address given by Tailscale
sudo ufw allow in on tailscale0 from <ip-address>

#To allow Coolify onboarding. We can close this later after configuring HTTPS
sudo ufw allow 22
```

## Install Fail2Ban
```
sudo apt install fail2ban -y
```

## Install Coolify on your Server
```
curl -fsSL https://cdn.coollabs.io/coolify/install.sh |  
```

## Install Cloudflare Tunnels on your Server
Go to your **Account Dashboard**, **Choose your domain name**,

---

# 🚧 Work in Progress 🚧
