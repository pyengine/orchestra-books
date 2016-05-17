# Build a OpenStack for production

# High level architecture

![Conceptual architecture](http://docs.openstack.org/kilo/install-guide/install/apt/content/figures/1/a/common/figures/openstack_kilo_conceptual_arch.png)

# Deployment Architecture
We are going to deploy OpenStack at Baremetal Zone.

<img src="https://raw.githubusercontent.com/pyengine/orchestra-books/master/cloud/OpenStack/deployment_architecture.png">

# Workflow Process

The deployment of docker swarm cluster has following process.

<img src="https://raw.githubusercontent.com/pyengine/orchestra-books/master/cloud/OpenStack/workflow.png">

# Basic environment
This example installs three-node architecture with OpenStack Networking(neutron).

Keyword     | Value         | Description
-----       | -----         | ----
CONTROLLER  | 10.1.0.1      | Controller node
NOVA_PASS   | nova_pass     | nova password
NEUTRON_PASS | neutron_pass | neutron password
RABBIT_PASS | rabbit_pass   | rabbit password
EXTERNAL_INTERFACE | br-ex  | External interface bridge name
PHYSNET1    | bond0.2       | Public Interface of tenant router
PHYSNET2    | bond0         | Private Interface of tenant network
REGION      | RegionOne     | Region name
VLAN_RANGE  | 100:999       | VLAN range for tenant private network

## Before you begin
## Security
## Networking
## Network Time Protocol(NTP)
You must install NTP to properly synchronize services among nodes. We recommend that you configure the controller node to reference more accurate servers and other nodes to reference the controller node.

To install the NTP service

~~~bash
apt-get -y install ntp
~~~

## OpenStack packages
* Install the Ubuntu cloud archive keyring and repository

~~~bash
apt-get -y install ubuntu-cloud-keyring
echo "deb http://ubuntu-cloud.archive.canonical.com/ubuntu" \
  "trusty-updates/kilo main" > /etc/apt/sources.list.d/cloudarchive-kilo.list
~~~

* Update the package lists

~~~bash
apt-get update
~~~

# Add a networking component

## Install and configure compute node

edit /etc/sysctl.conf

~~~text
net.ipv4.ip_forward=1
net.ipv4.conf.all.rp_filter=0
net.ipv4.conf.default.rp_filter=0
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
~~~

Implement the changes:

~~~bash
sysctl -p
~~~

* To install the Networking components

~~~bash
apt-get -y install neutron-plugin-ml2 neutron-plugin-linuxbridge neutron-plugin-linuxbridge-agent \
  neutron-l3-agent neutron-dhcp-agent neutron-metadata-agent
~~~

* To configure the Networking common components

edit /etc/neutron/neutron.conf

~~~text
[DEFAULT]
state_path = /var/lib/neutron

rpc_backend = rabbit
rabbit_host = ${CONTROLLER}
rabbit_userid = openstack
rabbit_password = ${RABBIT_PASS}


core_plugin = ml2
service_plugins = router
allow_overlapping_ips = True

auth_strategy = keystone

notification_driver = neutron.openstack.common.notifier.rpc_notifier
[quotas]
[agent]
root_helper = sudo /usr/bin/neutron-rootwrap /etc/neutron/rootwrap.conf
[keystone_authtoken]
auth_uri = http://${CONTROLLER}:5000
auth_url = http://${CONTROLLER}:35357
auth_plugin = password
project_domain_id = default
user_domain_id = default
project_name = service
username = neutron
password = ${NEUTRON_PASS}

[database]
[service_providers]

[oslo_messaging_rabbit]
[oslo_concurrency]
lock_path = /var/lib/neutron/tmp

~~~

* To configure the Modular Layer 2 (ML2) plug-in

edit /etc/neutron/plugins/ml2/ml2_conf.ini

~~~text
[ml2]
type_drivers = flat,vlan
tenant_network_types = vlan
mechanism_drivers = linuxbridge

[ml2_type_vlan]
network_vlan_ranges = physnet2:${VLAN_RANGE}

[ml2_type_gre]

[ml2_type_vxlan]
enable_vxlan = False

[securitygroup]

[linux_bridge]
physical_interface_mappings = physnet1:${PHYSNET1}, physnet2:${PHYSNET2}

~~~

edit /etc/neutron/l3_agent.ini

~~~text
[DEFAULT]
interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
use_namespaces = True
~~~

edit /etc/neutron/dhcp_agent.ini

~~~text
[DEFAULT]
interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
use_namespaces = True
dhcp_delete_namespaces = True
dnsmasq_config_file = /etc/neutron/dnsmasq-neutron.conf
~~~

edit /etc/neutron/dnsmasq-neutron.conf

~~~text
dhcp-option-force=26,1454
~~~

Kill any existing dnsmasq processes:

~~~bash
pkill dnsmasq
~~~

edit /etc/neutron/metadata_agent.ini

~~~text
[DEFAULT]
auth_uri = http://${CONTROLLER}:5000
auth_url = http://${CONTROLLER}:35357
auth_region = ${REGION}
auth_plugin = password
project_domain_id = default
user_domain_id = default
project_name = service
username = neutron
password = ${NEUTRON_PASS}

nova_metadata_ip = ${CONTROLLER}
metadata_proxy_shared_secret = METADATA_SECRET
~~~


Restart the Neutron services:

~~~bash

service neutron-plugin-linuxbridge-agent restart
service neutron-l3-agent restart
service neutron-dhcp-agent restart
service neutron-metadata-agent restart

~~~

