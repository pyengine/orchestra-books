# Install Docker engine

## Run install script

~~~bash
curl -sSL https://get.docker.com/ | sh
~~~

## Edit configuration file

edit /etc/sysconfig/docker

~~~text
# The max number of open files for the daemon itself, and all
# running containers.  The default value of 1048576 mirrors the value
# used by the systemd service unit.
DAEMON_MAXFILES=1048576

# Additional startup options for the Docker daemon, for example:
# OPTIONS="--ip-forward=true --iptables=true"
# By default we limit the number of open files per container
OPTIONS="--default-ulimit nofile=1024:4096 -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock"
~~~

## Give root privileges

~~~bash
sudo usermod -aG docker ec2-user
sudo service docker restart
~~~

