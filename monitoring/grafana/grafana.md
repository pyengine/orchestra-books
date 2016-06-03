# Install Grafana

## Environment

Keyword     | Value             | Description
----        | ----              | ----
VER         | grafana_3.0.4-1464167696.deb  | Stable .deb for Debian-based Linux

Last update 2016-06-03


## Install Stable

~~~bash
wget https://grafanarel.s3.amazonaws.com/builds/${VER}
apt-get install -y adduser libfontconfig
dpkg -i ${VER}
~~~

## Start the server

Start Grafana

~~~bash
service grafana-server start
~~~
