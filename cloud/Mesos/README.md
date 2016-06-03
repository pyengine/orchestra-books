# Build a Mesos cluster for production

# High level architecture

<img src="http://mesos.apache.org/assets/img/documentation/architecture3.jpg">

# Deployment Architecture
We are going to deploy four EC2 instances.

<img src="https://raw.githubusercontent.com/pyengine/orchestra-books/master/cloud/Mesos/deployment.png">

# Workflow Process

The deployment of Mesos cluster has following process.

<img src="https://raw.githubusercontent.com/pyengine/orchestra-books/master/cloud/Mesos/workflow.png">

Task Name | Task URI
----        | ----
Create Instances| https://github.com/pyengine/orchestra-books/blob/master/cloud/Mesos/provisioning.md
Install Prerequisites  | sudo apt-get update;sudo apt-get install -y python-pip python-dev expect gcc; sudo pip install jeju --upgrade
Configure Master01 | https://github.com/pyengine/orchestra-books/blob/master/cloud/Mesos/mesos-master1.md
Configure Master02 | https://github.com/pyengine/orchestra-books/blob/master/cloud/Mesos/mesos-master2.md
Configure Master03 | https://github.com/pyengine/orchestra-books/blob/master/cloud/Mesos/mesos-master3.md
Configure Slaves    | https://github.com/pyengine/orchestra-books/blob/master/cloud/Mesos/mesos-slave.md

 
# Default Environment

Keyword | Value | Description
----    | ----  | ----
AMI_ID   | ami-9abea4fb | Ubuntu Linux AMI 14.04 (HVM), SSD Volume Type (us-west-2)
SLAVE_NODES   | 2       | Number of Docker swarm nodes
KEY_NAME   | aws_son    | Keypair name
MGMT_INSTANCE_TYPE | t2.small   | Instance type of Management node
SWARM_INSTANCE_TYPE | t2.medium  | Instance type of Swarm node
ZONE_ID | xxx-xxxx-xxx  | Zone ID for Provisioning

AMI_ID must be exactly specified based on Region.

# Reference
http://mesos.apache.org/documentation/latest/architecture/
