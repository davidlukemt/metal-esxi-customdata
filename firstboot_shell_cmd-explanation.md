# First boot configpost.sh script file setup
### Add command in the local.sh file before the exit command to call the configpost.sh script we'll populate with our first boot commands
sed -i '/^exit*/i /vmfs/volumes/datastore1/configpost.sh' /etc/rc.local.d/local.sh;

### Create configpost.sh file
touch /vmfs/volumes/datastore1/configpost.sh;

chmod 755 /vmfs/volumes/datastore1/configpost.sh;

# Add contents to configpost.sh file

## WARNING!! DO NOT USE THIS FOR ESXI THAT WILL HAVE A PUBLIC IP ADDRESS
### Allow Password based Authentication to SSH on ESXi 
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config;

## The following commands echo the desired esxi relevant cli commands into the configpost.sh file

### Set DNS Server and Search Domain
echo 'esxcli network ip dns server add --server=172.16.11.4' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;

echo 'esxcli network ip dns search add --domain=sfo.rainpole.io' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;

### Set Hostname and FQDN
echo 'esxcfg-advcfg -s sfo01-m01-esx01.sfo.rainpole.io /Misc/hostname' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;

echo 'esxcli system hostname set -H=sfo01-m01-esx01' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;

echo 'esxcli system hostname set -f=sfo01-m01-esx01.sfo.rainpole.io' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;

### Set NTP Server and set ntpd to start at boot
echo 'echo \"server 172.16.11.253\" \u003e\u003e /etc/ntp.conf' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;

echo 'chkconfig ntpd on' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;

### Rename default 'VM Network' portgroup
echo 'esxcfg-vswitch -p \"VM Network\" -v 1611 vSwitch0' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;

### Set Management Network VLAN
echo 'esxcfg-vswitch -p \"Management Network\" -v 1611 vSwitch0' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;

### Delete Equinix created 'Private Network' vmkernel port
echo 'esxcfg-vmknic -d \"Private Network\"' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;

### Delete Equinix created 'Private Network' portgroup
echo 'esxcfg-vswitch -D \"Private Network\" vSwitch0' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;

### Set vmk0 IP address, Netmask, and Gateway
echo 'esxcli network ip interface ipv4 set -i vmk0 -I 172.16.11.101 -N 255.255.255.0 -g 172.16.11.253 -t static' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;

### Set default route for vmk0 Management network
echo 'esxcfg-route 172.16.11.253' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;

### Regenerate SSL Certificates using the new hostname/FQDN that was set previously
echo 'cd /etc/vmware/ssl' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;

echo '/sbin/generate-certificates' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;

### Restart the hostd and vpxa services with use above regenerated certificates
echo '/etc/init.d/hostd restart \u0026\u0026 /etc/init.d/vpxa restart' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;

### Remove the previously added line in the local.sh file that called the 
### configpost.sh file so these commands aren't run during subsequent reboots
echo 'sed -i '/configpost.sh/d' /etc/rc.local.d/local.sh' \u003e\u003e /vmfs/volumes/datastore1/configpost.sh;