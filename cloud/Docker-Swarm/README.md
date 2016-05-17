# Build a Swarm cluster for production

# High level architecture

<img src="https://raw.githubusercontent.com/pyengine/orchestra-books/master/cloud/Docker-Swarm/docker-swarm-architecture.png">

# Deployment Architecture
We are going to deploy four EC2 instances.

<img src="https://raw.githubusercontent.com/pyengine/orchestra-books/master/cloud/Docker-Swarm/deployment_architecture.png">

# Workflow Process

The deployment of docker swarm cluster has following process.

<img src="https://raw.githubusercontent.com/pyengine/orchestra-books/master/cloud/Docker-Swarm/workflow.png">

# Default Environment

Keyword | Value | Description
----    | ----  | ----
AMI_ID   | ami-d0f506b0 | Amazon Linux AMI 2016.03.1 (HVM), SSD Volume Type (us-west-2)
SWARM_NODES   | 2       | Number of Docker swarm nodes
KEY_NAME   | aws_son    | Keypair name
MGMT_INSTANCE_TYPE | t2.small   | Instance type of Management node
SWARM_INSTANCE_TYPE | t2.medium  | Instance type of Swarm node

AMI_ID must be exactly specified based on Region.

# Reference
https://docs.docker.com/swarm/install-manual/
