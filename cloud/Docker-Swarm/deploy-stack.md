# Deploy Docker Swarm cluster stack

## Environment

Keyword | Value
----    | ----
URL     | http://127.0.0.1/api/v1
USER_ID     | sunshout
PASSWORD     | 123456

## Portfolio

~~~python
import requests
import json
import pprint,sys,re

class writer:
    def write(self, text):
        text=re.sub(r'u\'([^\']*)\'', r'\1',text)
        sys.stdout.write(text)

wrt=writer()

pp = pprint.PrettyPrinter(indent=2, stream=wrt)

def show(dic):
    print pp.pprint(dic)

def display(title):
    print "\n"
    print "################# " + '{:20s}'.format(title) + "##############"

header = {'Content-Type':'application/json'}

def makePost(url, header, body):
    r = requests.post(url, headers=header, data=json.dumps(body))
    if r.status_code == 200:
        return json.loads(r.text)
    print r.text
    raise NameError(url)

def makePut(url, header, body):
    r = requests.put(url, headers=header, data=json.dumps(body))
    if r.status_code == 200:
        return json.loads(r.text)
    print r.text
    raise NameError(url)


def makeGet(url, header):
    r = requests.get(url, headers=header)
    if r.status_code == 200:
        return json.loads(r.text)
    print r.text
    raise NameError(url)

def makeDelete(url, header):
    r = requests.delete(url, headers=header)
    if r.status_code == 200:
        return json.loads(r.text)
    print r.text
    raise NameError(url)

######################################
# Deploy Stack
######################################
display('Change to User')
display('Auth')
url = '${URL}/token/get'
user_id='${USER_ID}'
password='${PASSWORD}'
header2 = {'Content-Type':'application/json'}
body = {'user_id':user_id, 'password':password}
token = makePost(url, header2, body)
token_id = token['token']
header2.update({'X-Auth-Token':token_id})

display('List Portfolio')
url = '${URL}/catalog/portfolios'
portfolios = makeGet(url, header2)
show(portfolios)
portfolio_id = raw_input('portfolio_id=>')

display('List Products')
url = '${URL}/catalog/products?portfolio_id=%s' % portfolio_id
products = makeGet(url, header2)
show(products)
product_id = raw_input('product_id=>')

display('List Packages')
url ='${URL}/catalog/packages?product_id=%s' % product_id
packages = makeGet(url, header2)
show(packages)
package_id = raw_input('package_id=>')

display('Select Zone for deploy')
zone_url = '${URL}/provisioning/zones'
zones = makeGet(zone_url, header2)
show(zones)
zone_id = raw_input('zone_id=>')

display('Swarm Cluster Size')
num_size = raw_input('cluster size(#n)=>')

print "=== AMI list ==="
print "us-west-1: ami-6e84fa0e"
print "us-west-2: ami-d0f506b0"
ami_id = raw_input('AMI ID (ex. ami-d0f506b0):')

stack_url = '${URL}/catalog/stacks'
display('Deploy Stack')
body = {'package_id':package_id, 'env': {'jeju':{'SWARM_NODES':num_size,'ZONE_ID':zone_id, 'AMI_ID':ami_id}}}
stack = makePost(stack_url, header2, body)
stack_id = stack['stack_id']
show(stack)
stack_id = stack['stack_id']

import time
display('Workflow State')
stack_url2 = '${URL}/catalog/stacks/%s/env' % stack_id
while 1:
    body = {'get':'workflow_state'}
    kv = makePost(stack_url2, header2, body)
    show(kv)
    time.sleep(10)
~~~
