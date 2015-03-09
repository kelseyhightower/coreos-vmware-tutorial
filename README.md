The following tutorial will walk you through downloading the official CoreOS VMware images
and configuring them using a cloud config drive. Once configured, the VMware images will
be launched under VMware Fusion, and we will utilize the vmrun cli tool to interact with it.

## Download VMware Images

```
wget http://alpha.release.core-os.net/amd64-usr/current/coreos_production_vmware.vmx
wget http://alpha.release.core-os.net/amd64-usr/current/coreos_production_vmware_image.vmdk.bz2
```

Decompress the disk image:

```
bzip2 -d coreos_production_vmware_image.vmdk.bz2
```

## Generate cloud configs

Many of the vmrun guest OS commands require a valid username and password in order to work, some
command require root access.

### Generate the password hash for the core user

```
openssl passwd -1
Password:
Verifying - Password:
$1$83p0.W1P$rx1Gw1llFuNKfwWtcceA7/
```

### Generate the password hash for the root user

```
openssl passwd -1
Password:
Verifying - Password:
$1$AwvTHbE9$RUB6nEX8dW5BJFbVDb207/
```

### Create a cloud config file

A cloud config file can be used to bootstrap a CoreOS host. In our example we will create two user, core and root
using the password generated above

edit cloud-config.yaml

```
#cloud-config

users:
  - name: core
    passwd: $1$83p0.W1P$rx1Gw1llFuNKfwWtcceA7/
    groups:
      - sudo
      - docker
  - name: root
    passwd: $1$AwvTHbE9$RUB6nEX8dW5BJFbVDb207/
```

## Create a config drive

### Generate a cloud config drive ISO

```
mkdir -p /tmp/new-drive/openstack/latest
cp cloud-config.yaml /tmp/new-drive/openstack/latest/user_data
hdiutil makehybrid -iso -joliet -joliet-volume-name "config-2" -o ~/cloudconfig.iso /tmp/new-drive
rm -r /tmp/new-drive
```

### Add the cloud config drive to the VM

Append the following lines to `coreos_production_vmware.vmx`

```
ide0:0.present = "TRUE"
ide0:0.autodetect = "TRUE"
ide0:0.deviceType = "cdrom-image"
ide0:0.fileName = "/Users/kelseyhightower/cloudconfig.iso"
```

### Running commands

```
vmrun checkToolsState /Users/kelseyhightower/vmware-demo/coreos_production_vmware.vmx
```

```
vmrun getGuestIPAddress /Users/kelseyhightower/vmware-demo/coreos_production_vmware.vmx
```

```
vmrun listProcessesInGuest /Users/kelseyhightower/vmware-demo/coreos_production_vmware.vmx
```
