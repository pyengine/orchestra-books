
# Environment

Keyword | Value | Description
----    | ----  | ----
URL     | http://127.0.0.1/api/v1 | URL for request
TOKEN   | my_token              | Token for API (must be overrided)
METADATA      | http://127.0.0.1/api/v1/catalog/{stack_id}/env | Environment URL for stack
ZONE_ID | 14b14664-705e-42e9-8106-240b83f9df79  | Zone ID (must be overrided)
AMI_ID   | ami-ac85fbcc | Amazon Linux AMI 2016.03.1 (HVM), SSD Volume Type (us-west-2)
SWARM_NODES   | 2       | Number of Docker swarm nodes
KEY_NAME   | aws_son    | Keypair name
MGMT_INSTANCE_TYPE | t2.micro   | Instance type of Management node
SWARM_INSTANCE_TYPE | t2.micro   | Instance type of Swarm node

# Create Server

~~~python
import requests
import json

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
mgmt01 = []
mgmt02 = []
swarm_nodes = []
all = []

# Create mgmt01
boto_request = {'ImageId':'${AMI_OD}',
        'MinCount':1,
        'MaxCount':1,
        'InstanceType':'${MGMT_INSTANCE_TYPE}'
        }

req = {'zone_id': '${ZONE_ID}',
        'name': 'mgmt01',
        'key_name': '${KEY_NAME}',
        'floatingIP':True,
        'request':boto_request}

server = createServer(req)
mgmt01.append(server['server_id'])
all.append(server['server_id'])

# Add environment of mgmt01
body = {'add':{'mgmt01':mgmt01}}
addEnv('${METADATA}', body)

time.sleep(30)
# Get IP of mgmt01
addr = getServerDetail(server['server_id'],'private_ip_address')
body = {'add':{'jeju':{'MGMT01':addr}}}
addEnv('${METADATA}', body)

# Create Mgmt02
req = {'zone_id': '${ZONE_ID}',
        'name': 'mgmt02',
        'key_name': '${KEY_NAME}',
        'floatingIP':True,
        'request':boto_request}

server = createServer(req)
mgmt02.append(server['server_id'])
all.append(server['server_id'])

# Add environment of mgmt02
body = {'add':{'mgmt02':mgmt02}}
addEnv('${METADATA}', body)

# Create Swarm nodes
boto_request = {'ImageId':'${AMI_OD}',
        'MinCount':1,
        'MaxCount':1,
        'InstanceType':'${SWARM_INSTANCE_TYPE}'
        }


for i in range(${SWARM_NODES}):
    node_name = "swarm-%.2d" % (i+1)
    req = {'zone_id': '${ZONE_ID}', 
            'name':node_name,
            'floatingIP':True,
            'request': boto_request
        }
    server = createServer(req)
    print server
    swarm_nodes.append(server['server_id'])
    all.append(server['server_id'])

body = {'add':{'swarm_nodes':swarm_nodes}}
addEnv('${METADATA}', body)

body = {'add':{'all':all}}
addEnv('${METADATA}', body)

~~~
