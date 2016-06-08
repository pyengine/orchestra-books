# Installation

This document is targeted for CentOS 7 and CloudStack 4.8~

## Environment

Keyword         | Value             | Description
----            | ----              | ----
VER             | 4.8               | CloudStack Version
PRIMARY         | /primary          | Primary storage directory path
SECONDARY       | /secondary        | Secondary storage directory path
DOMAIN          | cloud.priv        | Domain for management
PASSWORD        | password          | MySQL password for cloud account

## SELinux

SELinux must be set to permissive. We want to both configure this for future boots and modify in the current running system.

To configure SELinux to be permissive in the running system we need to run the following command:

~~~bash
setenforce 0
~~~

To ensure that it remains in that state we need to configure the file /etc/selinux/config to reflect the permissive state.

edit /etc/selinux/config

~~~text
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
# enforcing - SELinux security policy is enforced.
# permissive - SELinux prints warnings instead of enforcing.
# disabled - No SELinux policy is loaded.
SELINUX=permissive
# SELINUXTYPE= can take one of these two values:
# targeted - Targeted processes are protected,
# mls - Multi Level Security protection.
SELINUXTYPE=targeted
~~~

## NTP

NTP configuration is a necessity for keeping all of the clocks in your cloud servers in sync.

~~~bash
yum -y install ntp
systemctl enable ntpd.service
systemctl start ntpd.service
~~~

## Configure the CloudStack Package Repository

We need to configure the machine to use a CloudStack package repository.

edit /etc/yum.repos.d/cloudstack.repo

~~~text
[cloudstack]
name=cloudstack
baseurl=http://cloudstack.apt-get.eu/centos/7/${VER}/
enabled=1
gpgcheck=0
~~~

# Installation

Installation of the KVM agent is trivial with just a single command, but afterewards we'll need to configure a few things.


~~~bash
yum -y install cloudstack-agent
~~~

## KVM Configuration

* QEMU Configuration

edit /etc/libvirt/qemu.conf

~~~text
vnc_listen = "0.0.0.0"
~~~

* Libvirt Configuration

edit /etc/libvirt/libvirtd.conf

~~~text
listen_tls = 0
listen_tcp = 1
tcp_port = "16059"
auth_tcp = "none"
mdns_adv = 0
~~~

Turning on "listen_tcp" in libvirtd.conf is not enough, we have to change the parameters as well

edit /etc/sysconfig/libvirtd

~~~text
LIBVIRTD_ARGS="--listen"
~~~

Restart libvirt


Reference

http://docs.cloudstack.apache.org/projects/cloudstack-installation/en/4.8/qig.html
