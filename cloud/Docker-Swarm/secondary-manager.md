# Create Swarm Cluster

## Environment

Keyword     | Value     | Description
----        | ----      | ----
MGMT01      | 192.168.1.1   | IP address for consol01

## Install

After creating the discovery backend, you can create the Swarm managers. In this step, you are going to create two Swarm managers in a high-availability configuration. The first manager you run becomes the Swarm’s primary manager. Some documentation still refers to a primary manager as a “master”, but that term has been superseded. The second manager you run serves as a replica. If the primary manager becomes unavailable, the cluster elects the replica as the primary manager.

~~~bash
docker run -d -p 4000:4000 swarm manage -H :4000 --replication --advertise ${IP}:4000 consul://${MGMT01}:8500
~~~

## Testing

Checkout swarm container

~~~bash
docker ps
~~~
