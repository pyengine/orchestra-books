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

# Reference
http://docs.openstack.org/kilo/install-guide/install/apt/content/
