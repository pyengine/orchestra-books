# Influx DB

## Environment

Keyword     | Value                 | Description
----        | ----                  | ----
REPO        | https://repos.influxdata.com  | Ubuntu Repository for Influx DB

# Installation

Debian and Ubuntu users can install the latest stable version of InfluxDB using the apt-get package manager.

~~~bash
curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add -
source /etc/lsb-release
echo "deb ${REPO}/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
~~~

And then to install and start the InfluxDB service

~~~bash
apt-get update
apt-get install -y --force-yes influxdb
~~~

* Reference
https://docs.influxdata.com/
