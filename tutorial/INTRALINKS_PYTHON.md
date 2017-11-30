# Introduction to the Intralinks API

* Login
* Exchanges
  * Listing Exchanges
  * Entering an Exchange
* Documents & Folders
  * Listing Folders
  * Listing Documents
  * Downloading a document
* Users & Groups
  * Listing Users
  * Listing Groups
* Listing Permissions
* Listing Audits
* Logout

Additional topics
* Create Folder
* Create Document
* Create User
* Create Group
* Set Permission
* Send Alerts

## Login

[Developer Documentation](https://developers.intralinks.com/content/oauth_token)

```python
import requests

base_url = 'https://test-api.intralinks.com'

client_id = 'your_consumer_key'
client_secret = 'your_consumer_secret'
email =  'your_email'
password = 'your_password'

request = requests.post(base_url + '/v2/oauth/token', data={
    'grant_type':'client_credentials',
    'client_id':client_id,
    'client_secret':client_secret,
    'endOtherSessions':'True',
    'email':email,
    'password':password
})

if request.status_code != 200:
    raise Exception(request.status_code, request.text)
    
print(request.text)

json_data = request.json()
access_token = json_data['access_token']
```

Status code 200

```json
{
    "access_token":"XXXX",
    "token_type":"BearerToken",
    "expires_in":3599,
    "flags":[{"name":"NEW"}],
    "email":"foo@bar.com"
}
```

Status code 400

```json
{
    "ErrorCode":"invalid_request",
    "Error":"invalid or null consumer key: XXXX",
    "Hint":"please ensure that you are passing the consumer key like so: 'client_id={consumer_key}'"
}
```

Status code 403

```json
{
    "error":{
        "code":403,
        "message":"Credential provided is invalid. or there was problem with your account.",
        "subcode":"017"
    }
}
```

Status code 403

```json
{
    "error":{
        "code":403,
        "message":"Your session has been logged out as the same user is logged in elsewhere.",
        "subcode":"018"
    }
}
```

## Listing exchanges

[Developer Documentation](https://developers.intralinks.com/swagger/api-ui.html#!/Workspaces/get_workspaces)

```python
import requests

base_url = 'https://test-api.intralinks.com'

access_token = 'your_access_token'

request = requests.get(base_url + '/v2/workspaces', headers={
    'Authorization': 'Bearer {}'.format(access_token)
})

if request.status_code != 200:
    raise Exception(request.status_code, request.text)
    
print(request.text)

json_data = request.json()
exchanges = json_data['workspace']

for exchange in exchanges:
    print('{name:50.50} | {id:>9} | {host:20.20} | {phase:20}'.format(**exchange))
```

See https://docs.python.org/3.4/library/string.html#format-examples about ```format()```

Status code 200

```
{
    "status":{
        "code":200,
        "message":"Request completed fine, no errors"
    },
    "workspace":[
        {
            "workspaceName":"Exchange A",
            "parentTemplateId":12345,
            "host":"Host A",
            "securityLevel":1,
            "type":"ARC",
            "statistics":{
                "newTasks":0,
                "newParticipantRequests":0
            },
            "pvpEnabled":"F",
            "version":0,
            "htmlViewEnabled":"F",
            "phase":"OPEN",
            "name":"Exchange A",
            "id":11111155
        },{
            "workspaceName":"Exchange B",
            "parentTemplateId":23456,
            "host":"Host B",
            "securityLevel":1,
            "type":"ARC",
            "statistics":{
                "newTasks":0,
                "newParticipantRequests":0
            },
            "pvpEnabled":"F",
            "version":2,
            "htmlViewEnabled":"F",
            "phase":"OPEN",
            "name":"Exchange B",
            "id":22222255
        }
    ]
}
```

## (Variant) Listing exchanges associated to a given branding

[Developer Documentation](https://developers.intralinks.com/swagger/api-ui.html#!/Workspaces/get_workspaces)

```python
brand_id = 1234

request = requests.get(base_url + '/v2/workspaces', params={'brandId': brand_id}, headers={
    'Authorization': 'Bearer {}'.format(access_token)
})
```

## Entering an exchange

### Getting the splash screen

[Developer Documentation](https://developers.intralinks.com/swagger/api-ui.html#!/Workspaces/get_splash)

```python
import requests

base_url = 'https://test-api.intralinks.com'

access_token = 'your_access_token'
exchange_id = 1234

request = requests.get(base_url + '/v2/workspaces/{}/splash'.format(exchange_id), headers={
    'Authorization': 'Bearer {}'.format(access_token)
})

if request.status_code != 200:
    raise Exception(request.status_code, request.text)
    
print(request.text)
```

```json
{
    "status":{
        "code":200,
        "message":"Request completed fine, no errors"
    },
    "branding":{
        "global":true
    },
    "splash":{
        "acceptText":"agree",
        "displayType":"DISPLAY_EVERYTIME",
        "hasImage":true,
        "splashRequired":true,
        "splashText":"HTML CODE OF THE DISCLAIMER",
        "splashType":"PUBLIC",
        "splashUrl":"",
        "workspaceName":"Exchange A"
    }
}
```

### Accepting the splash screen

[Developer Documentation](https://developers.intralinks.com/swagger/api-ui.html#!/Workspaces/post_workspaces_workspace_id_splash)

```python
import requests
import json

base_url = 'https://test-api.intralinks.com'

access_token = 'your_access_token'
exchange_id = 1234

request = requests.post(base_url + '/v2/workspaces/{}/splash'.format(exchange_id), data=json.dumps({
        "acceptSplash": True
    }), headers={
        'Authorization': 'Bearer {}'.format(access_token),
        'Content-Type': 'application/json'
})

if request.status_code != 201:
    raise Exception(request.status_code, request.text)
    
print(request.text)

json_data = request.json()
accept_splash_state = json_data['state']
```

```json
{
    "status":{
        "code":201,
        "message":"The entities have been created"
    },
    "state":"ALLOW"
}
```

## Listing folders

[Developer Documentation](https://developers.intralinks.com/swagger/api-ui.html#!/Folders/get_folders)

```python
import requests

base_url = 'https://test-api.intralinks.com'

access_token = 'your_access_token'
exchange_id = 1234

request = requests.get(base_url + '/v2/workspaces/{}/folders'.format(exchange_id), headers={
    'Authorization': 'Bearer {}'.format(access_token)
})

if request.status_code != 200:
    raise Exception(request.status_code, request.text)
    
print(request.text)

json_data = request.json()
folders = json_data['folder']

for folder in folders:
    print('{name:50.50} | {id:>9} | {indexNumber:20.20}'.format(**folder))
```

```json
{
    "status":{
        "code":200,
        "message":"Request completed fine, no errors"
    },"folder":[
        {
            "id":4714545918,
            "name":"Folder B",
            "indexNumber":"1.1",
            "orderNumber":1,
            "version":"356a192b7913b04c54574d18c28d46e6395428ab",
            "hasChildFolders":true,
            "indexingDisabled":false,
            "parentId":4714065398,
            "createdOn":{
                "milliseconds":1364560076000
            },
            "createdBy":{
                "firstName":"John",
                "lastName":"Doe",
                "firstNameSort":"John",
                "lastNameSort":"Doe",
                "organization":"TheCompany",
                "organizationSort":"TheCompany"
            },
            "lastModifiedOn":{
                "milliseconds":1364560076000
            },
            "lastModifiedBy":{
                "firstName":"John",
                "lastName":"Doe",
                "firstNameSort":"John",
                "lastNameSort":"Doe"
            },
            "isFavorite":false,
            "versionNumber":0,
            "isEmailin":false
        },{
            "id":4714065398,
            "name":"Folder A",
            "indexNumber":"1",
            "orderNumber":1,
            "version":"b6589fc6ab0dc82cf12099d1c2d40ab994e8410c",
            "hasChildFolders":true,
            "indexingDisabled":false,
            "createdOn":{
                "milliseconds":1364560075000
            },
            "createdBy":{
                "firstName":"John",
                "lastName":"Doe",
                "firstNameSort":"John",
                "lastNameSort":"Doe",
                "organization":"TheCompany",
                "organizationSort":"TheCompany"
            },
            "lastModifiedOn":{
                "milliseconds":1364560075000
            },
            "lastModifiedBy":{
                "firstName":"John",
                "lastName":"Doe",
                "firstNameSort":"John",
                "lastNameSort":"Doe"
            },
            "isFavorite":false,
            "versionNumber":0,
            "isEmailin":false
        }
    ]
}
```

## Listing documents

[Developer Documentation](https://developers.intralinks.com/swagger/api-ui.html#!/Documents/get_documents)

```python
import requests

base_url = 'https://test-api.intralinks.com'

access_token = 'your_access_token'
exchange_id = 1234

request = requests.get(base_url + '/v2/workspaces/{}/documents'.format(exchange_id), headers={
    'Authorization': 'Bearer {}'.format(access_token)
})

if request.status_code != 200:
    raise Exception(request.status_code, request.text)
    
print(request.text)

json_data = request.json()
documents = json_data['document']

for document in documents:
    print('{name:50.50} | {id:>9} | {indexNumber:20.20}'.format(**document))
```

## Dowloading a file

[Developer Documentation](https://developers.intralinks.com/swagger/api-ui.html#!/Documents/get_document_file)

```python
import requests

base_url = 'https://test-api.intralinks.com'

access_token = 'your_access_token'
exchange_id = 1234
document_id = 5678
file_path = r'C:\Temp\MyFile.pdf'

request = requests.get(base_url + '/v2/workspaces/{}/documents/{}/file'.format(exchange_id, document_id), headers={
    'Authorization': 'Bearer {}'.format(access_token)
}, stream=True)

if request.status_code != 200:
    raise Exception(request.status_code, request.text)

with open(file_path, 'wb') as file:
    for chunk in request.iter_content(chunk_size=1024): 
        if chunk: 
            file.write(chunk)
```

https://stackoverflow.com/questions/16694907/how-to-download-large-file-in-python-with-requests-py

## Listing users

[Developer Documentation](https://developers.intralinks.com/swagger/api-ui.html#!/Workspace_Users/get_workspaces_workspace_id_users)

```python
import requests

base_url = 'https://test-api.intralinks.com'

access_token = 'your_access_token'
exchange_id = 1234

request = requests.get(base_url + '/v2/workspaces/{}/users'.format(exchange_id), headers={
    'Authorization': 'Bearer {}'.format(access_token)
})

if request.status_code != 200:
    raise Exception(request.status_code, request.text)
    
print(request.text)

json_data = request.json()
users = json_data['users']

for user in users:
    print('{lastName:50.50} | {firstName:50.50} | {organization:50.50} | {email:50.50} | {role:20.20}'.format(**user))
```

## Listing groups

[Developer Documentation](https://developers.intralinks.com/swagger/api-ui.html#!/Workspace_Groups/get_workspaces_workspace_id_groups)

```python
import requests

base_url = 'https://test-api.intralinks.com'

access_token = 'your_access_token'
exchange_id = 1234

request = requests.get(base_url + '/v2/workspaces/{}/groups'.format(exchange_id), headers={
    'Authorization': 'Bearer {}'.format(access_token)
})

if request.status_code != 200:
    raise Exception(request.status_code, request.text)
    
print(request.text)

json_data = request.json()
groups = json_data['groups']

for group in groups:
    print('{name:50.50} | {id:>9} | {type:20.20}'.format(**group))
```

## Logout

[Developer Documentation](https://developers.intralinks.com/content/oauth_revoke)

```python
import requests

base_url = 'https://test-api.intralinks.com'

access_token = 'your_access_token'
client_id = 'your_consumer_key'
client_secret = 'your_consumer_secret'

request = requests.put(base_url + '/v2/oauth/revoke', params={
        'token': access_token,
        'client_id': client_id,
        'client_secret': client_secret
    }, headers={
        'Authorization': 'Bearer {}'.format(access_token)
})

if request.status_code != 200:
    raise Exception(request.status_code, request.text)
    
print(request.text)
```

Status code 200

```json
{
    "message":"the passed token and associated refresh/access token have been revoked",
    "token":"XXXX",
    "cascade":"true"
}
```

## Typical Errors

### Wrong Access Token

Status Code 401

```json
{
    "status":"Unauthorized",
    "error":{
        "code":401,
        "description":"invalid access token: Bearer XXXX",
        "subcode":"the passed access token is either not valid or is formatted incorrectly in the request"
    }
}
```

### Wrong ID

You are trying a resource that does not exists. Or you are trying to access a resource you do not have access to.

Status Code 400

```json
{
    "status":{
        "code":400,
        "subcode":"3-1",
        "message":"Cannot find entity in repository",
        "errorId":"XXXX"
    },
    "errors":[{
        "code":400,
        "subCode":"3-1",
        "message":"Cannot find entity in repository"
    }]
}
```

### Splash not accepted

You are trying to access the content of an Exchange. But you have not accepted the Splash.

Status code 303

```json
{
    "status":{
        "code":303,
        "subcode":"1",
        "message":"Workspace entrance requirements not met.",
        "errorId":"XXXX"
    },
    "errors":[{
        "code":303,
        "subCode":"1",
        "message":"Workspace entrance requirements not met."
    }]
}
```
