# Install Docker engine

## Environment

Keyword     | Value     | Description
----        | ----      | ----
MGMT01      | 192.168.1.1   | IP address for mgmt01 and consol01


## Run install script

~~~bash
curl -sSL https://get.docker.com/ | sh
~~~

## Edit configuration file

Since we are ec2-user, we needs to change permission,

~~~bash
sudo chmod 777 /etc/sysconfig/docker
~~~

edit /etc/sysconfig/docker

~~~text
# The max number of open files for the daemon itself, and all
# running containers.  The default value of 1048576 mirrors the value
# used by the systemd service unit.
DAEMON_MAXFILES=1048576

# Additional startup options for the Docker daemon, for example:
# OPTIONS="--ip-forward=true --iptables=true"
# By default we limit the number of open files per container
OPTIONS="--default-ulimit nofile=1024:4096 -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --cluster-advertise eth0:2375 --cluster-store consul://${MGMT01}:8500"
~~~

Revert permission

~~~bash
sudo chmod 744 /etc/sysconfig/docker
~~~

## Give root privileges

~~~bash
sudo usermod -aG docker ec2-user
sleep 5
~~~

# Reboot Server

~~~bash
sudo reboot
~~~
