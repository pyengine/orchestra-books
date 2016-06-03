
# Environment

Keyword | Value | Description
----    | ----  | ----
URL     | http://127.0.0.1/api/v1 | URL for request
TOKEN   | my_token              | Token for API (must be overrided)
METADATA      | http://127.0.0.1/api/v1/catalog/{stack_id}/env | Environment URL for stack
ZONE_ID | 14b14664-705e-42e9-8106-240b83f9df79  | Zone ID (must be overrided)
AMI_ID   | ami-9abea4fb | Ubuntu 14.04 (HVM), SSD Volume Type (us-west-2)
SLAVE_NODES   | 2       | Number of Docker swarm nodes
KEY_NAME   | aws_son    | Keypair name
MGMT_INSTANCE_TYPE | t2.small   | Instance type of Management node
SWARM_INSTANCE_TYPE | t2.medium  | Instance type of Swarm node

# Create Server

~~~python
import requests
import json
import time

hdr = {'Content-Type':'application/json','X-Auth-Token':'${TOKEN}'}
def createServer(req):
    url = '${URL}/provisioning/servers'
    try:
        r =requests.post(url, headers=hdr, data=json.dumps(req))
        if (r.status_code == 200):
            result = json.loads(r.text)
            return result
    except requests.exception.ConnectionError:
        print "Failed to connect"

def getServerDetail(server_id, key):
    url = '${URL}/provisioning/servers/%s/detail' % server_id
    req = {'get':key}
    try:
        r = requests.post(url, headers=hdr, data=json.dumps(req))
        if (r.status_code == 200):
            result = json.loads(r.text)
            return result
    except requests.exception.ConnectionError:
        print "Failed to connect"

def addEnv(url, body):
    r = requests.post(url, headers=hdr, data=json.dumps(body))
    if r.status_code == 200:
        result = json.loads(r.text)

# Node name
master01 = []
master02 = []
master03 = []
slave_nodes = []

# Create master01
print "Create mmaster01"
boto_request = {'ImageId':'${AMI_ID}',
        'KeyName':'${KEY_NAME}',
        'MinCount':1,
        'MaxCount':1,
        'InstanceType':'${MGMT_INSTANCE_TYPE}'
        }

req = {'zone_id': '${ZONE_ID}',
        'name': 'master01',
        'key_name': '${KEY_NAME}',
        'login_id': 'ubuntu',
        'floatingIP':True,
        'request':boto_request}

server = createServer(req)
master01.append(server['server_id'])

# Add environment of master01
body = {'add':{'master01':master01}}
addEnv('${METADATA}', body)

time.sleep(30)
# Get IP of master01
addr = getServerDetail(server['server_id'],'private_ip_address')
body = {'add':{'jeju':{'MASTER01':addr['private_ip_address']}}}
addEnv('${METADATA}', body)

# Create master02
print "Create mmaster02"
boto_request = {'ImageId':'${AMI_ID}',
        'KeyName':'${KEY_NAME}',
        'MinCount':1,
        'MaxCount':1,
        'InstanceType':'${MGMT_INSTANCE_TYPE}'
        }

req = {'zone_id': '${ZONE_ID}',
        'name': 'master02',
        'key_name': '${KEY_NAME}',
        'login_id': 'ubuntu',
        'floatingIP':True,
        'request':boto_request}

server = createServer(req)
master02.append(server['server_id'])

# Add environment of master02
body = {'add':{'master02':master02}}
addEnv('${METADATA}', body)

time.sleep(30)
# Get IP of master02
addr = getServerDetail(server['server_id'],'private_ip_address')
body = {'add':{'jeju':{'MASTER02':addr['private_ip_address']}}}
addEnv('${METADATA}', body)

# Create master03
print "Create mmaster03"
boto_request = {'ImageId':'${AMI_ID}',
        'KeyName':'${KEY_NAME}',
        'MinCount':1,
        'MaxCount':1,
        'InstanceType':'${MGMT_INSTANCE_TYPE}'
        }

req = {'zone_id': '${ZONE_ID}',
        'name': 'master03',
        'key_name': '${KEY_NAME}',
        'login_id': 'ubuntu',
        'floatingIP':True,
        'request':boto_request}

server = createServer(req)
master03.append(server['server_id'])

# Add environment of master03
body = {'add':{'master03':master03}}
addEnv('${METADATA}', body)

time.sleep(30)
# Get IP of master03
addr = getServerDetail(server['server_id'],'private_ip_address')
body = {'add':{'jeju':{'MASTER03':addr['private_ip_address']}}}
addEnv('${METADATA}', body)



# Create Slave nodes
print "Create Slave nodes"
boto_request = {'ImageId':'${AMI_ID}',
        'KeyName':'${KEY_NAME}',
        'MinCount':1,
        'MaxCount':1,
        'InstanceType':'${SWARM_INSTANCE_TYPE}'
        }


for i in range(${SLAVE_NODES}):
    node_name = "slave-%.2d" % (i+1)
    req = {'zone_id': '${ZONE_ID}', 
            'name':node_name,
            'key_name':'${KEY_NAME}',
            'login_id':'ubuntu',
            'floatingIP':True,
            'request': boto_request
        }
    server = createServer(req)
    print server
    slave_nodes.append(server['server_id'])

body = {'add':{'slave_nodes':slave_nodes}}
addEnv('${METADATA}', body)

~~~
