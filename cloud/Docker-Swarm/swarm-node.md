# Create Swarm Cluster

## Environment

Keyword     | Value     | Description
----        | ----      | ----
MGMT01      | 192.168.1.1   | IP address for consol01

## Install

Connect to swarm node and join them to the cluster.

~~~bash
docker run -d swarm join --advertise=${IP}:2375 consul://${MGMT01}:8500
~~~

## Testing

Checkout swarm container

~~~bash
docker ps
~~~
