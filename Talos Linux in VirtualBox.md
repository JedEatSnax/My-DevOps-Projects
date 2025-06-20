**Talos Linux ver 1.10.4 in VirtualBox**

#Prerequisites
- Must have an existing instance of Linux distribution (Ubuntu, Debian, etcetera) that will act as a manager for the nodes.
- Must have talosctl installed on the said instance of Linux distro manager.
- Must have a metal-amd64 ISO of Talos Linux.
- All virtual machines must be on a bridged adapter network.
- All Talos Linux virtual machines must have a static IP.

# How to set Talos virtual machines with static IP address
- Take note of the IP and GW at the top right corner of the summary page.
- My control plane IP is 192.168.1.156/24 and the GW or gateway is 192.168.1.1
- Press 'F3' and 'Tab' until you reach Interface. Press 'Enter' and enp0s3.
- Next, press 'Tab' again and 'Enter' to change the Mode from DHCP to Static.
- Lastly, press 'Tab' and 'Enter' to save the configuration.
- Do the same steps for the worker nodes or other control planes you want to setup.

# Setting up the control plane
export CONTROL_PLANE_IP=<control-plane-IP>
echo $CONTROL_PLANE_IP
talosctl gen config talos-vbox-cluster https://$CONTROL_PLANE_IP:6443 --output-dir _out
talosctl apply-config --insecure --nodes $CONTROL_PLANE_IP --file _out/controlplane.yaml

# The control plane IP address will change, unless the virtual machine IP is static.
# Change the CONTROL_PLANE_IP and WORKER_IP exports accordingly.

# Setting up a worker node
export WORKER_IP=<worker-node-IP>
echo $WORKER_IP
talosctl apply-config --insecure --nodes $WORKER_IP --file _out/worker.yaml

# After this, the operating system will automatically reboot.
# Wait for the kubelet to be healthy

export TALOSCONFIG=_out/talosconfig
talosctl --talosconfig $TALOSCONFIG config endpoint $CONTROL_PLANE_IP
talosctl --talosconfig $TALOSCONFIG config node $CONTROL_PLANE_IP

talosctl config get-contexts

# If there are no context shown, you might have messed up something.
talosctl --talosconfig $TALOSCONFIG bootstrap

talosctl kubeconfig .

# Once you do the kubeconfig command, it will ask you to rename or overwrite.
# I chose rename. So, I entered 'r'

# Wait for a few minutes until the worker node says:
"[talos] machine is running and ready"

# And when the control plane says:
"entered forwarding state"

# Enter these commands once everything is good
kubectl get nodes --kubeconfig=kubeconfig
kubectl get pods -n kube-system --kubeconfig=kubeconfig

# Clean-up
talosctl shutdown --nodes $CONTROL_PLANE_IP --talosconfig $TALOSCONFIG
talosctl shutdown --nodes $WORKER_IP --talosconfig $TALOSCONFIG

# Once you poweroff the Talos virtual machines, go to VirtualBox settings
- Settings > Storage > Controller: IDE > Right Click metal-amd64 ISO > Remove Attachment > Press OK
# Commands to reset everything
rm -rf ~/.talos/
rm -rf _out/
rm -f controlplane.yaml worker.yaml talosconfig
unset CONTROL_PLANE_IP
unset WORKER_IP
unset TALOSCONFIG
