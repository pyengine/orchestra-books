# Mesos Setup

# Environment

Keyword | Value
----    | -----
MASTER01 | 192.168.0.1
MASTER02 | 192.168.0.2
MASTER03 | 192.168.0.3
DEV	| eth0

# Setup Mesos

~~~bash
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv E56151BF
DISTRO=$(lsb_release -is | tr '[:upper:]' '[:lower:]')
CODENAME=$(lsb_release -cs)

echo "deb http://repos.mesosphere.io/${DISTRO} ${CODENAME} main" | \
  sudo tee /etc/apt/sources.list.d/mesosphere.list
sudo apt-get -y update
~~~

# Installation

~~~bash
apt-get -y install mesos
~~~


# Remove Zookeeper

~~~bash
service zookeeper stop
echo "manual" > /etc/init/zookeeper.override
update-rc.d -f zookeeper remove
~~~

# Edit mesos

Update date zookeeper with proper IP

edit /etc/mesos/zk

~~~text
zk://${MASTER01}:2181,${MASTER02}:2181,${MASTER03}:2181/mesos
~~~


## Stop mesos master

~~~bash
service mesos-master stop
echo "manual" > /etc/init/mesos-master.override
update-rc.d -f mesos-master remove
~~~

## Restart mesos slave

update /etc/mesos-slave/ip

~~~bash
/sbin/ifconfig ${DEV} | grep 'inet addr:' | cut -d: -f2 | awk '{ print $1}' > /etc/mesos-slave/ip
~~~

## Update containerizers

edit /etc/mesos-slave/containerizers

~~~text
docker,mesos
~~~

We support docker

## Update registration timeout

timeout update

create /etc/mesos-slave/executor_registration_timeout

~~~bash
echo "5mins" > /etc/mesos-slave/executor_registration_timeout
~~~


~~~bash
service mesos-slave restart
~~~

# Reference

https://mesosphere.com/downloads/
