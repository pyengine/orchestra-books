# Build a OpenStack for production

# High level architecture

![Conceptual architecture](http://docs.openstack.org/kilo/install-guide/install/apt/content/figures/1/a/common/figures/openstack_kilo_conceptual_arch.png)

# Deployment Architecture
We are going to deploy OpenStack at Baremetal Zone.

<img src="https://raw.githubusercontent.com/pyengine/orchestra-books/master/cloud/OpenStack/deployment_architecture.png">

# Workflow Process

The deployment of docker swarm cluster has following process.

<img src="https://raw.githubusercontent.com/pyengine/orchestra-books/master/cloud/OpenStack/workflow.png">

# Basic environment
This example installs three-node architecture with OpenStack Networking(neutron).

Keyword     | Value
-----       | -----
ADMIN_PASS  | admin_pass
DEMO_PASS   | demo_pass
NOVA_PASS   | nova_pass
CINDER_PASS | cinder_pass
GLANCE_PASS | glance_pass
NEUTRON_PASS | neutron_pass
DASH_DBPASS | dash_dbpass
RABBIT_PASS | rabbit_pass
NOVA_DBPASS | nova_dbpass
GLANCE_DBPASS | glance_dbpass
CINDER_DBPASS | cinder_dbpass
KEYSTONE_DBPASS | keystone_dbpass
NEUTRON_DBPASS | neutron_dbpass
ADMIN_TOKEN | admin_token
REGION      | RegionOne


## Before you begin
## Security
## Networking
## Network Time Protocol(NTP)
You must install NTP to properly synchronize services among nodes. We recommend that you configure the controller node to reference more accurate servers and other nodes to reference the controller node.

To install the NTP service
~~~bash
apt-get -y install ntp
~~~

## OpenStack packages
* Install the Ubuntu cloud archive keyring and repository
~~~bash
apt-get -y install ubuntu-cloud-keyring
echo "deb http://ubuntu-cloud.archive.canonical.com/ubuntu" \
  "trusty-updates/kilo main" > /etc/apt/sources.list.d/cloudarchive-kilo.list
~~~

* Update the package lists
~~~bash
apt-get update
~~~

## SQL database
* To install and configure the database server
~~~bash
export DEBIAN_FRONTEND=noninteractive
apt-get -y install mariadb-server python-mysqldb
~~~

edit /etc/mysql/conf.d/mysql_openstack.cnf

~~~text
[mysqld]
bind-address = 0.0.0.0
default-storage-engine = innodb
innodb_file_per_table
collation-server = utf8_general_ci
init-connect = 'SET NAMES utf8'
character-set-server = utf8
~~~

* To finalize installation
~~~bash
service mysql restart
~~~


## Message queue
* To install the message queue service
~~~bash
apt-get -y install rabbitmq-server
~~~

* To configure the message queue service
* Add the openstack user
~~~bash
rabbitmqctl add_user openstack ${RABBIT_PASS}
~~~
* Permit configuration, write, and read access for the openstack user:
~~~bash
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
~~~

# Add the Identity service

## Create keystone database and update privileges.

~~~bash
mysql -u root -p <<EOF
CREATE DATABASE nova;
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \
IDENTIFIED BY '${NOVA_DBPASS}';
CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \
IDENTIFIED BY '${GLANCE_DBPASS}';
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
IDENTIFIED BY '${KEYSTONE_DBPASS}';
CREATE DATABASE neutron;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' \
IDENTIFIED BY '${NEUTRON_DBPASS}';
CREATE DATABASE cinder;
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' \
IDENTIFIED BY '${CINDER_DBPASS}';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \
IDENTIFIED BY '${NOVA_DBPASS}';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' \
IDENTIFIED BY '${GLANCE_DBPASS}';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
IDENTIFIED BY '${KEYSTONE_DBPASS}';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' \
IDENTIFIED BY '${NEUTRON_DBPASS}';
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' \
IDENTIFIED BY '${CINDER_DBPASS}';
FLUSH PRIVILEGES;
EOF
~~~

Disable the keystone service from starting automatically after installation:

~~~bash
echo "manual" > /etc/init/keystone.override
~~~

Install keystone packages
~~~bash
apt-get -y install keystone python-openstackclient apache2 libapache2-mod-wsgi memcached python-memcache
~~~

edit /etc/keystone/keystone.conf
~~~text
[DEFAULT]

admin_token = ${ADMIN_TOKEN}
log_dir = /var/log/keystone

[assignment]
[auth]
[cache]
[catalog]
[credential]

[database]
connection = mysql://keystone:${KEYSTONE_DBPASS}@${IP}/keystone

[domain_config]
[endpoint_filter]
[endpoint_policy]
[eventlet_server]
[eventlet_server_ssl]
[federation]
[fernet_tokens]
[identity]
[identity_mapping]
[kvs]
[ldap]
[matchmaker_redis]
[matchmaker_ring]

[memcache]
servers = localhost:11211

[oauth1]
[os_inherit]
[oslo_messaging_amqp]
[oslo_messaging_qpid]
[oslo_messaging_rabbit]
[oslo_middleware]
[oslo_policy]
[paste_deploy]
[policy]
[resource]

[revoke]
driver = keystone.contrib.revoke.backends.sql.Revoke

[role]
[saml]
[signing]
[ssl]

[token]
provider = keystone.token.providers.uuid.Provider
driver = keystone.token.persistence.backends.memcache.Token

[trust]

[extra_headers]
Distribution = Ubuntu
~~~

* Populate the Identity service database:
~~~bash
su -s /bin/sh -c "keystone-manage db_sync" keystone
~~~

* To configure the Apache HTTP Server

~~~bash
echo "ServerName ${IP}" >> /etc/apache2/apache2.conf
~~~

edit /etc/apache2/sites-available/wsgi-keystone.conf

~~~text
Listen 5000
Listen 35357

<VirtualHost *:5000>
    WSGIDaemonProcess keystone-public processes=5 threads=1 user=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-public
    WSGIScriptAlias / /var/www/cgi-bin/keystone/main
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    <IfVersion >= 2.4>
      ErrorLogFormat "%{cu}t %M"
    </IfVersion>
    LogLevel info
    ErrorLog /var/log/apache2/keystone-error.log
    CustomLog /var/log/apache2/keystone-access.log combined
</VirtualHost>

<VirtualHost *:35357>
    WSGIDaemonProcess keystone-admin processes=5 threads=1 user=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-admin
    WSGIScriptAlias / /var/www/cgi-bin/keystone/admin
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    <IfVersion >= 2.4>
      ErrorLogFormat "%{cu}t %M"
    </IfVersion>
    LogLevel info
    ErrorLog /var/log/apache2/keystone-error.log
    CustomLog /var/log/apache2/keystone-access.log combined
</VirtualHost>
~~~

* Enable the Identity service virtual hosts:
~~~bash
ln -s /etc/apache2/sites-available/wsgi-keystone.conf /etc/apache2/sites-enabled
~~~

* Create the directory structure for the WSGI components:
~~~bash
mkdir -p /var/www/cgi-bin/keystone
~~~

* Copy the WSGI components from the upstream repository into this directory:
~~~bash
apt-get install -y curl
curl http://10.251.210.54/openstack/keystone.html \
  | tee /var/www/cgi-bin/keystone/main /var/www/cgi-bin/keystone/admin
~~~

* Adjust ownership and permissions on this directory and the files in it:
~~~bash
chown -R keystone:keystone /var/www/cgi-bin/keystone
chmod 755 /var/www/cgi-bin/keystone/*
~~~

* To finalize installtion
~~~bash
service apache2 restart
rm -f /var/lib/keystone/keystone.db
~~~

## Create the service entity and API endpoint

* To configure prerequisites
* To create the service entity and API endpoint

~~~bash
echo "Create the service entity for the Identity service"
openstack service create \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
--name keystone --description "OpenStack Identity" identity

echo "Create the Identity service API endpoint"
openstack endpoint create \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
--publicurl http://${IP}:5000/v2.0 \
--internalurl http://${IP}:5000/v2.0 \
--adminurl http://${IP}:35357/v2.0 \
--region RegionOne \
identity
~~~

## Create projects, users, and roles

The identity service provides authentication services for each OpenStack service.
The authentication service uses a combination of domains, projects (tenants), users, and roles.

* To create tenants, users, and roles

~~~bash
echo "Create the admin project"
openstack project create \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
--description "Admin Project" admin

echo "Create the admin user"
openstack user create \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
--password ${ADMIN_PASS} admin

echo "Create the admin role"
openstack role create \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
admin

echo "Add the admin role to the admin project and user"
openstack role add \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
--project admin --user admin admin
~~~

* This guide uses a service project that contains a unique user for each service that you add to your environment

~~~bash
openstack project create \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
--description "Service Project" service
~~~

* Regular (non-admin)tasks should use an unprivileged project and user.

~~~bash
openstack project create \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
--description "Demo Project" demo

openstack user create \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
--password ${DEMO_PASS} demo

openstack role create \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
user

openstack role add \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
--project demo --user demo user
~~~

## Create OpenStack client environment scripts

edit admin-openrc.sh

~~~text
export OS_PROJECT_DOMAIN_ID=default
export OS_USER_DOMAIN_ID=default
export OS_PROJECT_NAME=admin
export OS_TENANT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=${ADMIN_PASS}
export OS_AUTH_URL=http://${IP}:35357/v3
export OS_IMAGE_API_VERSION=2
~~~


edit demo-openrc.sh

~~~text
export OS_PROJECT_DOMAIN_ID=default
export OS_USER_DOMAIN_ID=default
export OS_PROJECT_NAME=demo
export OS_TENANT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=${DEMO_PASS}
export OS_AUTH_URL=http://${IP}:5000/v3
export OS_IMAGE_API_VERSION=2
~~~


~~~bash
chmod 755 admin-openrc.sh
chmod 755 demo-openrc.sh
~~~

# Add the Image service

## Install and configure

* Create glance user

~~~bash
echo "Create glance user"
openstack user create \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
--password ${GLANCE_PASS} glance

openstack role add \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
--project service --user glance admin

openstack service create \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
--name glance --description "OpenStack Image Service" image
~~~

* Create the Image service API endpoint:

~~~bash
openstack endpoint create \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
--publicurl http://${IP}:9292 \
--internalurl http://${IP}:9292 \
--adminurl http://${IP}:9292 \
--region RegionOne \
image
~~~

* To install and configure the Image service components

~~~bash
apt-get -y install glance python-glanceclient
~~~

edit /etc/glance/glance-api.conf

~~~text
[DEFAULT]
bind_host = 0.0.0.0
bind_port = 9292

log_file = /var/log/glance/api.log
backlog = 4096

registry_host = 0.0.0.0
registry_port = 9191

registry_client_protocol = http
notification_driver = noop

rabbit_host = ${IP}
rabbit_port = 5672
rabbit_use_ssl = false
rabbit_userid = openstack
rabbit_password = ${RABBIT_PASS}
rabbit_virtual_host = /
rabbit_notification_exchange = glance
rabbit_notification_topic = notifications
rabbit_durable_queues = False

qpid_notification_exchange = glance
qpid_notification_topic = notifications
qpid_hostname = localhost
qpid_port = 5672
qpid_username =
qpid_password =
qpid_sasl_mechanisms =
qpid_reconnect_timeout = 0
qpid_reconnect_limit = 0
qpid_reconnect_interval_min = 0
qpid_reconnect_interval_max = 0
qpid_reconnect_interval = 0
qpid_heartbeat = 5
# Set to 'ssl' to enable SSL
qpid_protocol = tcp
qpid_tcp_nodelay = True

delayed_delete = False
scrub_time = 43200
scrubber_datadir = /var/lib/glance/scrubber

image_cache_dir = /var/lib/glance/image-cache/

[oslo_policy]
[database]
backend = sqlalchemy
connection = mysql://glance:${GLANCE_DBPASS}@${IP}/glance

[oslo_concurrency]

[keystone_authtoken]
auth_uri = http://${IP}:5000
auth_url = http://${IP}:35357
auth_plugin = password
project_domain_id = default
user_domain_id = default
project_name = service
username = glance
password = ${GLANCE_PASS}

[paste_deploy]
flavor=keystone

[store_type_location_strategy]
[profiler]
[task]
[taskflow_executor]
[glance_store]
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
~~~


edit /etc/glance/glance-registry.conf

~~~text
[DEFAULT]
bind_host = 0.0.0.0
bind_port = 9191
log_file = /var/log/glance/registry.log
backlog = 4096
api_limit_max = 1000
limit_param_default = 25
notification_driver = noop
rabbit_host = ${IP}
rabbit_port = 5672
rabbit_use_ssl = false
rabbit_userid = openstack
rabbit_password = ${RABBIT_PASS}
rabbit_virtual_host = /
rabbit_notification_exchange = glance
rabbit_notification_topic = notifications
rabbit_durable_queues = False

qpid_notification_exchange = glance
qpid_notification_topic = notifications
qpid_hostname = localhost
qpid_port = 5672
qpid_username =
qpid_password =
qpid_sasl_mechanisms =
qpid_reconnect_timeout = 0
qpid_reconnect_limit = 0
qpid_reconnect_interval_min = 0
qpid_reconnect_interval_max = 0
qpid_reconnect_interval = 0
qpid_heartbeat = 5
qpid_protocol = tcp
qpid_tcp_nodelay = True

[oslo_policy]
[database]
backend = sqlalchemy
connection = mysql://glance:${GLANCE_DBPASS}@${IP}/glance
[keystone_authtoken]
auth_uri = http://${IP}:5000
auth_url = http://${IP}:35357
auth_plugin = password
project_domain_id = default
user_domain_id = default
project_name = service
username = glance
password = ${GLANCE_PASS}

[paste_deploy]
flavor = keystone

[profiler]
~~~

* Populate the Image service database:

~~~bash
su -s /bin/sh -c "glance-manage db_sync" glance
~~~

* To finalize installation

~~~bash
service glance-registry restart
service glance-api restart
~~~

~~~bash
rm -f /var/lib/glance/glance.sqlite
~~~


# Add the Compute service

## Install and configure controller node

* Create nova user

~~~bash
echo "Create nova user"
openstack user create \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
--password ${NOVA_PASS} nova

openstack role add \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
--project service --user nova admin

openstack service create \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
--name nova --description "OpenStack Compute Service" compute
~~~

* Create the Compute service API endpoint:

~~~bash
openstack endpoint create \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
--publicurl http://${IP}:8774/v2/%\(tenant_id\)s \
--internalurl http://${IP}:8774/v2/%\(tenant_id\)s \
--adminurl http://${IP}:8774/v2/%\(tenant_id\)s \
--region RegionOne \
compute
~~~

* To install and configure Compute controller components

~~~bash
apt-get -y install nova-api nova-cert nova-conductor nova-consoleauth \
  nova-novncproxy nova-scheduler python-novaclient
~~~

edit /etc/nova/nova.conf

~~~text
[DEFAULT]

dhcpbridge_flagfile=/etc/nova/nova.conf
dhcpbridge=/usr/bin/nova-dhcpbridge
logdir=/var/log/nova
state_path=/var/lib/nova
lock_path=/var/lock/nova
force_dhcp_release=True
libvirt_use_virtio_for_bridges=True
verbose=True
ec2_private_dns_show_ip=True
api_paste_config=/etc/nova/api-paste.ini
enabled_apis=ec2,osapi_compute,metadata
rpc_backend = rabbit
auth_strategy = keystone
my_ip = ${IP}
vncserver_listen = ${IP}
vncserver_proxyclient_address = ${IP}

network_api_class = nova.network.neutronv2.api.API
security_group_api = neutron
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[database]
connection = mysql://nova:${NOVA_DBPASS}@${IP}/nova

[oslo_messaging_rabbit]
rabbit_host = ${IP}
rabbit_userid = openstack
rabbit_password = ${RABBIT_PASS}

[keystone_authtoken]
auth_uri = http://${IP}:5000
auth_url = http://${IP}:35357
auth_plugin = password
project_domain_id = default
user_domain_id = default
project_name = service
username = nova
password = ${NOVA_PASS}

[glance]
host = ${IP}

[oslo_concurrency]
lock_path = /var/lib/nova/tmp

[neutron]
url = http://${IP}:9696
auth_strategy = keystone
admin_auth_url = http://${IP}:35357/v2.0
admin_tenant_name = service
admin_username = neutron
admin_password = ${NEUTRON_PASS}
service_metadata_proxy = True
metadata_proxy_shared_secret = METADATA_SECRET


~~~

* Populate the Compute database:

~~~bash
su -s /bin/sh -c "nova-manage db sync" nova
~~~


* To finalize installation

~~~bash
service nova-api restart
service nova-cert restart
service nova-consoleauth restart
service nova-scheduler restart
service nova-conductor restart
service nova-novncproxy restart
~~~

By default, the Ubuntu packages create an SQLite database.

~~~bash
rm -f /var/lib/nova/nova.sqlite
~~~

# Add a networking component

## OpenStack Networking (neutron)

### Install and configure controller node

* Create neutron user

~~~bash
echo "Create neutron user"
openstack user create \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
--password ${NEUTRON_PASS} neutron

openstack role add \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
--project service --user neutron admin

openstack service create \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
--name neutron --description "OpenStack Networking Service" network
~~~

* Create the Networking service API endpoint:

~~~bash
openstack endpoint create \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
--publicurl http://${IP}:9696 \
--adminurl http://${IP}:9696 \
--internalurl http://${IP}:9696 \
--region RegionOne \
network
~~~

* To  install the Networking components

~~~bash
apt-get -y install neutron-server neutron-plugin-ml2 python-neutronclient
~~~

* To configure the Networking server component

edit /etc/neutron/neutron.conf

~~~text
[DEFAULT]
core_plugin = ml2
service_plugins = router
auth_strategy = keystone
allow_overlapping_ips = True
notify_nova_on_port_status_changes = True
notify_nova_on_port_data_changes = True
nova_url = http://${IP}:8774/v2
rpc_backend=rabbit

[matchmaker_redis]
[matchmaker_ring]
[quotas]
[agent]
root_helper = sudo /usr/bin/neutron-rootwrap /etc/neutron/rootwrap.conf

[keystone_authtoken]
auth_uri = http://${IP}:5000
auth_url = http://${IP}:35357
auth_plugin = password
project_domain_id = default
user_domain_id = default
project_name = service
username = neutron
password = ${NEUTRON_PASS}

[database]
connection = mysql://neutron:${NEUTRON_DBPASS}@${IP}/neutron

[nova]
auth_url = http://${IP}:35357
auth_plugin = password
project_domain_id = default
user_domain_id = default
region_name = RegionOne
project_name = service
username = nova
password = ${NOVA_PASS}

[oslo_concurrency]
lock_path = $state_path/lock

[oslo_policy]
[oslo_messaging_amqp]
[oslo_messaging_qpid]
[oslo_messaging_rabbit]
rabbit_host = ${IP}
rabbit_userid = openstack
rabbit_password = ${RABBIT_PASS}
~~~

edit /etc/neutron/plugins/ml2/ml2_conf.ini

~~~text
[ml2]
type_drivers = flat,vlan,gre,vxlan
tenant_network_types = vlan
mechanism_drivers = linuxbridge

#tenant_network_types = gre
#mechanism_drivers = openvswitch

[ml2_type_flat]

[ml2_type_vlan]
network_vlan_ranges = physnet2:100:999

[ml2_type_gre]


[ml2_type_vxlan]

[securitygroup]
enable_security_group = True
enable_ipset = True
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
#firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver
~~~

* To finalize the installation

~~~bash
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
  --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
~~~

Restart the Compute service

~~~bash
service nova-api restart
service neutron-server restart
~~~

# Add the dashboard

## Install and configure

~~~bash
apt-get -y install openstack-dashboard
~~~

edit /etc/openstack-dashboard/local_settings.py

~~~text
import os

from django.utils.translation import ugettext_lazy as _

from openstack_dashboard import exceptions

DEBUG = False
TEMPLATE_DEBUG = DEBUG


WEBROOT = '/'
# Default OpenStack Dashboard configuration.
HORIZON_CONFIG = {
    'user_home': 'openstack_dashboard.views.get_user_home',
    'ajax_queue_limit': 10,
    'auto_fade_alerts': {
        'delay': 3000,
        'fade_duration': 1500,
        'types': ['alert-success', 'alert-info']
    },
    'help_url': "http://docs.openstack.org",
    'exceptions': {'recoverable': exceptions.RECOVERABLE,
                   'not_found': exceptions.NOT_FOUND,
                   'unauthorized': exceptions.UNAUTHORIZED},
    'modal_backdrop': 'static',
    'angular_modules': [],
    'js_files': [],
    'js_spec_files': [],
}

LOCAL_PATH = os.path.dirname(os.path.abspath(__file__))

from horizon.utils import secret_key
SECRET_KEY = secret_key.generate_or_read_from_file('/var/lib/openstack-dashboard/secret_key')

CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': '127.0.0.1:11211',
    }
}

# Send email to the console by default
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
OPENSTACK_HOST = "${IP}"
OPENSTACK_KEYSTONE_URL = "http://%s:5000/v2.0" % OPENSTACK_HOST
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
OPENSTACK_KEYSTONE_BACKEND = {
    'name': 'native',
    'can_edit_user': True,
    'can_edit_group': True,
    'can_edit_project': True,
    'can_edit_domain': True,
    'can_edit_role': True,
}

OPENSTACK_HYPERVISOR_FEATURES = {
    'can_set_mount_point': False,
    'can_set_password': False,
}

OPENSTACK_CINDER_FEATURES = {
    'enable_backup': False,
}

OPENSTACK_NEUTRON_NETWORK = {
    'enable_router': True,
    'enable_quotas': True,
    'enable_ipv6': True,
    'enable_distributed_router': False,
    'enable_ha_router': False,
    'enable_lb': True,
    'enable_firewall': True,
    'enable_vpn': True,

    'profile_support': None,
    'supported_provider_types': ['*'],

    'supported_vnic_types': ['*']
}

IMAGE_CUSTOM_PROPERTY_TITLES = {
    "architecture": _("Architecture"),
    "kernel_id": _("Kernel ID"),
    "ramdisk_id": _("Ramdisk ID"),
    "image_state": _("Euca2ools state"),
    "project_id": _("Project ID"),
    "image_type": _("Image Type"),
}

IMAGE_RESERVED_CUSTOM_PROPERTIES = []

API_RESULT_LIMIT = 1000
API_RESULT_PAGE_SIZE = 20

SWIFT_FILE_TRANSFER_CHUNK_SIZE = 512 * 1024

DROPDOWN_MAX_ITEMS = 30

TIME_ZONE = "UTC"

LOGGING = {
    'version': 1,
    # When set to True this will disable all logging except
    # for loggers specified in this configuration dictionary. Note that
    # if nothing is specified here and disable_existing_loggers is True,
    # django.db.backends will still log unless it is disabled explicitly.
    'disable_existing_loggers': False,
    'handlers': {
        'null': {
            'level': 'DEBUG',
            'class': 'django.utils.log.NullHandler',
        },
        'console': {
            # Set the level to "DEBUG" for verbose output logging.
            'level': 'INFO',
            'class': 'logging.StreamHandler',
        },
    },
    'loggers': {
        # Logging from django.db.backends is VERY verbose, send to null
        # by default.
        'django.db.backends': {
            'handlers': ['null'],
            'propagate': False,
        },
        'requests': {
            'handlers': ['null'],
            'propagate': False,
        },
        'horizon': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'openstack_dashboard': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'novaclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'cinderclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'keystoneclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'glanceclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'neutronclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'heatclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'ceilometerclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'troveclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'swiftclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'openstack_auth': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'nose.plugins.manager': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'django': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'iso8601': {
            'handlers': ['null'],
            'propagate': False,
        },
        'scss': {
            'handlers': ['null'],
            'propagate': False,
        },
    }
}
# 'direction' should not be specified for all_tcp/udp/icmp.
# It is specified in the form.
SECURITY_GROUP_RULES = {
    'all_tcp': {
        'name': _('All TCP'),
        'ip_protocol': 'tcp',
        'from_port': '1',
        'to_port': '65535',
    },
    'all_udp': {
        'name': _('All UDP'),
        'ip_protocol': 'udp',
        'from_port': '1',
        'to_port': '65535',
    },
    'all_icmp': {
        'name': _('All ICMP'),
        'ip_protocol': 'icmp',
        'from_port': '-1',
        'to_port': '-1',
    },
    'ssh': {
        'name': 'SSH',
        'ip_protocol': 'tcp',
        'from_port': '22',
        'to_port': '22',
    },
    'smtp': {
        'name': 'SMTP',
        'ip_protocol': 'tcp',
        'from_port': '25',
        'to_port': '25',
    },
    'dns': {
        'name': 'DNS',
        'ip_protocol': 'tcp',
        'from_port': '53',
        'to_port': '53',
    },
    'http': {
        'name': 'HTTP',
        'ip_protocol': 'tcp',
        'from_port': '80',
        'to_port': '80',
    },
    'pop3': {
        'name': 'POP3',
        'ip_protocol': 'tcp',
        'from_port': '110',
        'to_port': '110',
    },
    'imap': {
        'name': 'IMAP',
        'ip_protocol': 'tcp',
        'from_port': '143',
        'to_port': '143',
    },
    'ldap': {
        'name': 'LDAP',
        'ip_protocol': 'tcp',
        'from_port': '389',
        'to_port': '389',
    },
    'https': {
        'name': 'HTTPS',
        'ip_protocol': 'tcp',
        'from_port': '443',
        'to_port': '443',
    },
    'smtps': {
        'name': 'SMTPS',
        'ip_protocol': 'tcp',
        'from_port': '465',
        'to_port': '465',
    },
    'imaps': {
        'name': 'IMAPS',
        'ip_protocol': 'tcp',
        'from_port': '993',
        'to_port': '993',
    },
    'pop3s': {
        'name': 'POP3S',
        'ip_protocol': 'tcp',
        'from_port': '995',
        'to_port': '995',
    },
    'ms_sql': {
        'name': 'MS SQL',
        'ip_protocol': 'tcp',
        'from_port': '1433',
        'to_port': '1433',
    },
    'mysql': {
        'name': 'MYSQL',
        'ip_protocol': 'tcp',
        'from_port': '3306',
        'to_port': '3306',
    },
    'rdp': {
        'name': 'RDP',
        'ip_protocol': 'tcp',
        'from_port': '3389',
        'to_port': '3389',
    },
}

REST_API_REQUIRED_SETTINGS = ['OPENSTACK_HYPERVISOR_FEATURES']

# Enable the Ubuntu theme if it is present.
try:
  from ubuntu_theme import *
except ImportError:
  pass

# Default Ubuntu apache configuration uses /horizon as the application root.
WEBROOT='/horizon/'

REST_API_REQUIRED_SETTINGS = ['OPENSTACK_HYPERVISOR_FEATURES']

# Enable the Ubuntu theme if it is present.
try:
  from ubuntu_theme import *
except ImportError:
  pass

# Default Ubuntu apache configuration uses /horizon as the application root.
WEBROOT='/horizon/'

# By default, validation of the HTTP Host header is disabled.  Production
# installations should have this set accordingly.  For more information
# see https://docs.djangoproject.com/en/dev/ref/settings/.
ALLOWED_HOSTS = '*'

# Compress all assets offline as part of packaging installation
COMPRESS_OFFLINE = True
~~~

* To finalize installation

~~~bash
service apache2 reload
~~~


# Add the Block Storage service

## Create the service credentials

Create a cinder user

~~~bash
echo "Create a cinder user"
openstack user create \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
--password ${CINDER_PASS} cinder

echo "Add the admin role to the cinder user"
openstack role add \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
--project service --user cinder admin

echo "Create the cinder service entities"
openstack service create \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
--name cinder --description "OpenStack Block Storage" volume

openstack service create \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
--name cinderv2 --description "OpenStack Block Storage" volumev2

echo "Create the Block Storage service API endpoints"
openstack endpoint create \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
--publicurl http://${IP}:8776/v2/%\(tenant_id\)s \
--internalurl http://${IP}:8776/v2/%\(tenant_id\)s \
--adminurl http://${IP}:8776/v2/%\(tenant_id\)s \
--region RegionOne \
volume

openstack endpoint create \
--os-token ${ADMIN_TOKEN} --os-url http://${IP}:35357/v2.0 \
--publicurl http://${IP}:8776/v2/%\(tenant_id\)s \
--internalurl http://${IP}:8776/v2/%\(tenant_id\)s \
--adminurl http://${IP}:8776/v2/%\(tenant_id\)s \
--region RegionOne \
volumev2


~~~

## Install the appropriate packages for Block Storage Service:

~~~bash
apt-get install -y cinder-api cinder-scheduler python-cinderclient
~~~

## Configure Block Storage to use your database

edit /etc/cinder/cinder.conf

~~~text
[DEFAULT]
rootwrap_config = /etc/cinder/rootwrap.conf
api_paste_confg = /etc/cinder/api-paste.ini
iscsi_helper = tgtadm
volume_name_template = volume-%s
volume_group = cinder-volumes
verbose = True
auth_strategy = keystone
state_path = /var/lib/cinder
volumes_dir = /var/lib/cinder/volumes
rpc_backend = rabbit
my_ip = ${IP}

[database]
connection = mysql://cinder:${CINDER_DBPASS}@${IP}/cinder

[oslo_messaging_rabbit]
rabbit_host = ${IP}
rabbit_userid = openstack
rabbit_password = ${RABBIT_PASS}

[keystone_authtoken]
auth_uri = http://${IP}:5000
auth_url = http://${IP}:35357
auth_plugin = password
project_domain_id = default
user_domain_id = default
project_name = service
username = cinder
password = ${CINDER_PASS}

[oslo_concurrency]
lock_path = /var/lock/cinder
~~~

## Polulate the Block Storage database:

Remove SQLite database, since the Ubuntu packages create SQLite database.

~~~bash
rm -f /var/lib/cinder/cinder.sqlite
su -s /bin/sh -c "cinder-manage db sync" cinder
~~~

## Restart the Block Storage services:

~~~bash
service cinder-scheduler restart
service cinder-api restart
~~~

## Install Ceph client package

~~~bash
apt-get install -y python-rbd
~~~
