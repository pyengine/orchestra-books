# KVM

Keyword     |   Value           | Description
----        | ----              | ----
MGMT        | br-mgmt10g        | Bridge network 
PUBLIC      | br-public10g      | Bridge network 
VLAN_MGMT   | 3                 | VLAN ID for management network
VLAN_PUBLIC | 11                | VLAN ID for public  network
MGMT_START  | 10.2.0.1          | IP address range of management network
MGMT_END    | 10.2.0.100        | IP address range of management network
MGMT_IP     | 10.2.0.241        | Management bridge IP
MGMT_NETMASK| 255.255.255.0     | Netmask of Management network
MGMT01_NAME | mgmt01-vm         | Management node name
MGMT01_IP   | 10.2.0.11         | Management node IP address
MGMT01_MAC1 | 52:54:00:00:01:01 | Management node MAC address 1
MGMT01_MAC2 | 52:54:00:00:02:01 | Management node MAC address 2
PUBLIC_START  | 10.3.0.1          | IP address range of public network
PUBLIC_END    | 10.3.0.100        | IP address range of public network
PUBLIC_IP     | 10.3.0.241        | Public bridge IP
PUBLIC_NETMASK| 255.255.255.0     | Netmask of Public network
MGMT01_IP2  | 10.3.0.11         | Management node IP address-2


# Install KVM

~~~bash
yum groupinstall -y "X Window System"
yum install -y vconfig bridge-utils
yum install -y qemu libvirt tightvnc seabios
~~~

## Restart libvirtd service

~~~bash
systemctl enable libvirtd
systemctl restart libvirtd
virsh version
~~~

# Create Management Network

## Management Network

edit /etc/libvirt/qemu/networks/mgmt.xml

~~~text
<network>
  <name>mgmt</name>
  <bridge name='${MGMT}' stp='on' deploy='0' />
  <forward dev='VLAN${VLAN_MGMT}' mode='route' />
  <ip address='${MGMT_IP}' netmask='${MGMT_NETMASK}'>
    <tftp root="/tftpboot/centos" />
    <dhcp>
      <range start='${MGMT_START}' end='${MGMT_END}'/>
      <bootp file="pxelinux.0" />
      <host mac='${MGMT01_MAC1}' name='${MGMT01_NAME}' ip='${MGMT01_IP}'/>
    </dhcp>
  </ip>
</network>
~~~

## Public Network

edit /etc/libvirt/qemu/networks/public.xml

~~~text
<network>
  <name>public</name>
  <bridge name='${PUBLIC}' stp='on' deploy='0' />
  <forward dev='VLAN${VLAN_PUBLIC}' mode='route' />
  <ip address='${PUBLIC_IP}' netmask='${PUBLIC_NETMASK}'>
    <dhcp>
      <range start='${PUBLIC_START}' end='${PUBLIC_END}'/>
      <host mac='${MGMT01_MAC2}' name='${MGMT01_NAME}' ip='${MGMT01_IP}'/>
    </dhcp>
  </ip>
</network>
~~~

## Regist network

Using libvirt command, register networks

~~~bash
virsh net-define /etc/libvirt/qemu/networks/mgmt.xml
virsh net-define /etc/libvirt/qemu/networks/public.xml
virsh net-autostart mgmt
virsh net-autostart public
virsh net-start mgmt
virsh net-start public
~~~

## Connect bridge interface

virsh has problem to connect bridge interfaces.
Connect them manually.

~~~bash
brctl addif br-mgmt10g VLAN${VLAN_MGMT}
brctl addif br-public10g VLAN${VLAN_PUBLIC}
~~~

