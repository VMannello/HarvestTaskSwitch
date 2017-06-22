# HarvestTaskSwitch
Simple Python3 script using Requests to update Harvest task associations.

|Requirements| |
|----|----|
|[Requests](http://python-requests.org) | `pip3 install requests`|

Code:
```python
from base64 import b64encode
import requests

domain = 'YOURACCOUNT' # https://YOURACCOUNT.harvestapp.com/
username = '' # This uses Basic Authentication
password = '' #
old_tasks = [7206757, 7206766]
new_task = 6868062


class harvestAuth(requests.auth.AuthBase):
    """This class sets up the authorization and JSON request headers."""
    def __init__(self, username, password):
        self.username = username
        self.password = password

    def __eq__(self, other):
        return all([
            self.username == getattr(other, 'username', None),
            self.password == getattr(other, 'password', None)
        ])

    def __ne__(self, other):
        return not self == other

    def __call__(self, r):
        r.headers['Content-type'] = 'application/json'
        r.headers['Accept'] = 'application/json'
        encode = ':'.join((self.username, self.password))
        r.headers['Authorization'] = 'Basic ' + (b64encode(encode.encode('utf8'))).decode('ascii').strip()
        return r


def caller(page, action='get', payload=None, params=None, domain=domain, username=username, password=password):
    """This function sets up an easy way to GET or POST Harvest API"""
    if (action == 'get' and payload is None):
        r = requests.get('https://' + domain + '.harvestapp.com/' + page, auth=harvestAuth(username=username, password=password))
        if (r.status_code == requests.codes.ok):
            return {'json': r.json(), 'text': r.text, 'status': r.status_code}
        else:
            return {'status': r.status_code, 'text': r.text}
    elif (action == 'post' and payload is not None and params is not None):
        r = requests.post('https://' + domain + '.harvestapp.com/' + page, auth=harvestAuth(username=username, password=password), json=payload, params=params)
        return {'status': r.status_code, 'text': r.text}
    else:
        return False


resp = caller('people')

count = 0

for person in resp['json']:
    mC = 0
    person = person['user']
    tasks = caller('people/'+str(person['id'])+'/entries?from=20000101&to=22221230')
    for task in tasks['json']:
        task = task['day_entry']
        if (task['task_id'] in old_tasks):
            resp = caller(page='daily/update/'+str(task['id']),
                          action='post',
                          payload={"notes": task['notes'], "hours": task['hours'], "project_id": task['project_id'], "task_id": str(new_task), "spent_at": task['spent_at']},
                          params={'of_user': str(person['id'])})
            if (resp['status'] == requests.codes.ok):
                mC = mC + 1
            else:
                print('Error: Could not update User: '+str(person['id'])+' Task: ' + str(task['id']))
    count = count + mC
    print('Finished ' + person['first_name'] + ' with ' + str(mC) + ' updates.')

print('Total updates: ' + str(count))
```
