# First boot configpost.sh script file setup
### Add command in the local.sh file before the exit command to call the configpost.sh script we'll populate with our first boot commands
```shell
sed -i '/^exit*/i /vmfs/volumes/datastore1/configpost.sh' /etc/rc.local.d/local.sh;
```

## WARNING!! DO NOT USE THIS FOR ESXI THAT WILL HAVE A PUBLIC IP ADDRESS
### Allow Password based Authentication to SSH on ESXi, this is a requirement for vCloud Foundation deployments
```shell
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config;
```

### Create configpost.sh file
```shell
touch /vmfs/volumes/datastore1/configpost.sh;

chmod 755 /vmfs/volumes/datastore1/configpost.sh;
```

# Add contents to configpost.sh file

## While ESXi 6 and 7 didn't care about a proper ShaBang header, ESXi 8 most definitely does.
### Documentation about the changes to the **execInstalledOnly** option driving this can be found [here](https://docs.vmware.com/en/VMware-vSphere/8.0/vsphere-security/GUID-DF6A7974-62F9-47DB-A990-963F3B3AEA77.html)
#### [TLDR from William Lam](https://communities.vmware.com/t5/ESXi-Discussions/esxi-8-shell-script-how-to-run/m-p/2954230/highlight/true#M286671) is that scripts which don't start with the shabang are considered binaries and the execInstalledOnly option blocks binaries that didn't originate from a valid VIB
```shell
echo '#!/bin/sh' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;
```

## The following commands echo the desired esxi relevant cli commands into the configpost.sh file

### Set DNS Server and Search Domain
```shell
echo 'esxcli network ip dns server add --server=172.16.11.4' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;

echo 'esxcli network ip dns search add --domain=sfo.rainpole.io' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;
```

### Set Hostname and FQDN
```shell
echo 'esxcfg-advcfg -s sfo01-m01-esx01.sfo.rainpole.io /Misc/hostname' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;

echo 'esxcli system hostname set -H=sfo01-m01-esx01' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;

echo 'esxcli system hostname set -f=sfo01-m01-esx01.sfo.rainpole.io' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;
```

### Set NTP Server IPs or FQDN and set ntpd to start at boot
```shell
echo 'esxcli system ntp set -s=172.16.11.253 -s=172.16.1.1' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;

echo 'esxcli system ntp set -e=yes' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;
```

### Set default 'VM Network' portgroup VLAN
```shell
echo 'esxcfg-vswitch -p \"VM Network\" -v 1611 vSwitch0' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;
```

### Set Management Network VLAN
```shell
echo 'esxcfg-vswitch -p \"Management Network\" -v 1611 vSwitch0' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;
```

### Delete Equinix created 'Private Network' vmkernel port
```shell
echo 'esxcfg-vmknic -d \"Private Network\"' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;
```

### Delete Equinix created 'Private Network' portgroup
```shell
echo 'esxcfg-vswitch -D \"Private Network\" vSwitch0' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;
```

### Set vmk0 IP address, Netmask, and Gateway
```shell
echo 'esxcli network ip interface ipv4 set -i vmk0 -I 172.16.11.101 -N 255.255.255.0 -g 172.16.11.253 -t static' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;
```

### Set default route for vmk0 Management network
```shell
echo 'esxcfg-route 172.16.11.253' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;
```

### Regenerate SSL Certificates using the new hostname/FQDN that was set previously
```shell
echo 'cd /etc/vmware/ssl' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;

echo '/sbin/generate-certificates' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;
```

### Restart the hostd and vpxa services with use above regenerated certificates
```shell
echo '/etc/init.d/hostd restart \u0026\u0026 /etc/init.d/vpxa restart' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;
```

### Remove the previously added line in the local.sh file that called the 
### configpost.sh file so these commands aren't run during subsequent reboots
```shell
echo 'sed -i '/configpost.sh/d' /etc/rc.local.d/local.sh' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;
```