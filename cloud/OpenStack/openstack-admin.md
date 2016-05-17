# Basic environment
This example installs three-node architecture with OpenStack Networking(neutron).

Keyword                 | Value             | Description              
-----                   | -----             | ----
EXTERNAL_NETWORK_CIDR | 192.168.0.0/24      | External network(public) range
FLOATING_IP_START | 192.168.0.100           | Floating IP start
FLOATING_IP_END | 192.168.0.200             | Floating IP end
EXTERNAL_NETWORK_GATEWAY | 192.168.0.1      | Gateway of external network
TENANT_NETWORK_CIDR | 192.168.2.0/24        | Default Tenant network range
TENANT_NETWORK_GATEWAY | 192.168.2.1        | Default Tenant network gateway


# Add a networking component

## Create external network

Before running, make sure you already update environments like 'source admin-openrc.sh'

### To create the external network

Create the network:

~~~bash
source admin-openrc.sh
neutron net-create ext-net --router:external \
  --provider:physical_network external --provider:network_type flat
~~~


### To create a subnet on the external network

Create the subnet

~~~bash
source admin-openrc.sh
neutron subnet-create ext-net ${EXTERNAL_NETWORK_CIDR} --name ext-subnet \
  --allocation-pool start=${FLOATING_IP_START},end=${FLOATING_IP_END} \
  --disable-dhcp --gateway ${EXTERNAL_NETWORK_GATEWAY}
~~~

## Create Tenant network

### Create the network

~~~bash
source admin-openrc.sh
neutron net-create demo-net
~~~

### Create subnet on the tenant network

~~~bash
source admin-openrc.sh
neutron subnet-create demo-net ${TENANT_NETWORK_CIDR} \
  --name demo-subnet --gateway ${TENANT_NETWORK_GATEWAY}
~~~

### Create router

To create a router on the tenant network and attach the external and tenant networks to it

Create the router:

~~~bash
source admin-openrc.sh
neutron router-create demo-router
~~~

Attach the router to the demo tenant subnet:

~~~bash
source admin-openrc.sh
neutron router-interface-add demo-router demo-subnet
~~~

Attach the router to the external network by settting it as the gateway:

~~~bash
source admin-openrc.sh
neutron router-gateway-set demo-router ext-net
~~~
