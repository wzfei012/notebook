On Fedora
Install libvirt and qemu-kvm on your system:

$ sudo dnf install libvirt qemu-kvm
Add yourself to the libvirt group:

$ sudo usermod -a -G libvirt $(whoami)
Update your current session to apply the group change:

$ newgrp libvirt
As root, install the KVM driver binary and make it executable as follows:

# curl -L https://github.com/dhiltgen/docker-machine-kvm/releases/download/v0.10.0/docker-machine-driver-kvm-centos7 -o /usr/local/bin/docker-machine-driver-kvm
# chmod +x /usr/local/bin/docker-machine-driver-kvm


Start libvirtd service
Check the status of libvirtd:

$ systemctl is-active libvirtd
If libvirtd is not active, start the libvirtd service:

$ sudo systemctl start libvirtd
Configure libvirt networking
Some distributions set up the default libvirt network for you, while on others this might have to be done manually.

Check your network status:

$ sudo virsh net-list --all
Name                 State      Autostart     Persistent
---------------------------------------------------------
default              active     yes           yes
If your output looks like the above then you’re done. However, if State is not active or Autostart is not yes you’ll need to follow the steps below.

Start the default libvirt network:

$ sudo virsh net-start default
Now mark the default network as autostart:

$ sudo virsh net-autostart default

go get -u github.com/openshift/imagebuilder/cmd/imagebuilder