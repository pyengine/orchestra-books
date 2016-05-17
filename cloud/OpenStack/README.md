# Build a OpenStack for production

# High level architecture

![Conceptual architecture](http://docs.openstack.org/kilo/install-guide/install/apt/content/figures/1/a/common/figures/openstack_kilo_conceptual_arch.png)

# Deployment Architecture
We are going to deploy OpenStack at Baremetal Zone.

<img src="https://raw.githubusercontent.com/pyengine/orchestra-books/master/cloud/OpenStack/deployment_architecture.png">

# Workflow Process

The deployment of docker swarm cluster has following process.

<img src="https://raw.githubusercontent.com/pyengine/orchestra-books/master/cloud/OpenStack/workflow.png">


The detailed information for tasks are:

Task name           | Description
----                | ----
Config Controller Node  | https://github.com/pyengine/orchestra-books/blob/master/cloud/OpenStack/openstack-controller.md
Config Network Node     | https://github.com/pyengine/orchestra-books/blob/master/cloud/OpenStack/openstack-network-linuxbridge.md
Config Compute Nodes    | https://github.com/pyengine/orchestra-books/blob/master/cloud/OpenStack/openstack-compute-linuxbridge.md
Add testing account     | https://github.com/pyengine/orchestra-books/blob/master/cloud/OpenStack/openstack-admin.md

# Default Environment

This value can be overrided at Stack environment

Keyword     | Value             | Description
-----       | -----             | ----
CONTROLLER  | 10.1.0.1          | Controller node IP address
ADMIN_PASS  | admin_pass        | user(admin) password
DEMO_PASS   | demo_pass         | user(demo) password
NOVA_PASS   | nova_pass         | nova password
CINDER_PASS | cinder_pass       | cinder password
GLANCE_PASS | glance_pass       | glance password
NEUTRON_PASS | neutron_pass     | neutron password
DASH_DBPASS | dash_dbpass       | dashboard db password
RABBIT_PASS | rabbit_pass       | rabbit password
NOVA_DBPASS | nova_dbpass       | nova db password
GLANCE_DBPASS | glance_dbpass   | glance db password
CINDER_DBPASS | cinder_dbpass   | cinder db password
KEYSTONE_DBPASS | keystone_dbpass   | keystone db password
NEUTRON_DBPASS | neutron_dbpass     | neutron db password
ADMIN_TOKEN | admin_token           | Admin token
REGION      | RegionOne             | Region name
EXTERNAL_INTERFACE | br-ex  | External interface bridge name
PHYSNET1    | bond0.2       | Public Interface of tenant router
PHYSNET2    | bond0         | Private Interface of tenant network
REGION      | RegionOne     | Region name
VLAN_RANGE  | 100:999       | VLAN range for tenant private network

Before deploy OpenStack stack, install jeju on every host.
This can be done by **pip install jeju --upgrade**

# Reference
http://docs.openstack.org/kilo/install-guide/install/apt/content/
