# Mesos Setup

# Environment

Keyword | Value
----    | -----
MASTER01 | 192.168.0.1
MASTER02 | 192.168.0.2
MASTER03 | 192.168.0.3
MYID	| 1
DEV	| eth0

# Setup Mesos

apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys E56151BF
If you have proxy, use above command instead of using keyserver.ubuntu.com

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

# Edit Zookeeper

## Update myid

myid must be unique on cluster

edit /etc/zookeeper/conf/myid

~~~text
${MYID}
~~~

## Update zookeeper zoo.cfg

~~~bash
echo "server.1=${MASTER01}:2888:3888" >> /etc/zookeeper/conf/zoo.cfg
echo "server.2=${MASTER02}:2888:3888" >> /etc/zookeeper/conf/zoo.cfg
echo "server.3=${MASTER03}:2888:3888" >> /etc/zookeeper/conf/zoo.cfg
~~~

## Restart Zookeeper

~~~bash
service zookeeper restart
~~~

# Edit mesos

Update date zookeeper with proper IP

edit /etc/mesos/zk

~~~text
zk://${MASTER01}:2181,${MASTER02}:2181,${MASTER03}:2181/mesos
~~~

## Update Quorum

edit /etc/mesos-master/quorum

~~~text
2
~~~

## Stop mesos slave

~~~bash
service mesos-slave stop
echo "manual" > /etc/init/mesos-slave.override
update-rc.d -f mesos-slave remove
~~~

edit /etc/mesos-master/ip

~~~bash
/sbin/ifconfig ${DEV} | grep 'inet addr:' | cut -d: -f2 | awk '{ print $1}' > /etc/mesos-master/ip
~~~

## Restart mesos master

~~~bash
service mesos-master restart
~~~

# Reference

https://mesosphere.com/downloads/
