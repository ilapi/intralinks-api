# Introduction to the Intralinks API

## Authentication

```python
import requests

base_url = 'https://test-api.intralinks.com'

client_id = 'your_consumer_key'
client_secret = 'your_consumer_secret'
email =  'your_email'
password = 'your_password'

token_request = requests.post(base_url + '/v2/oauth/token', data={
    'grant_type':'client_credentials',
    'client_id':client_id,
    'client_secret':client_secret,
    'endOtherSessions':True,
    'email':email,
    'password':password
})

token_data = token_request.json()
access_token = token_data['access_token']
```

## Listing exchanges

```python
import requests

base_url = 'https://test-api.intralinks.com'
access_token = 'your_access_token'

exchange_request = requests.get(base_url + '/v2/workspaces', headers={
    'Authorization': 'Bearer ' + access_token
})

exchanges_data = exchange_request.json()

for exchange in exchanges_data['workspace']:
    print('{name:50.50} | {id:>9} | {host:20.20} | {phase:20}'.format(exchange))
```

See https://docs.python.org/3.4/library/string.html#format-examples about ```format()```

## Entering an exchange

## Listing folders

## Listing documents

## Listing groups

## Listing users

## Logout
