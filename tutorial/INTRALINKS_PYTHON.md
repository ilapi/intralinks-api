# Introduction to the Intralinks API with Python

Open Questions

* End other sessions
* What if no Splash
* How to create multiple groups
* purpose of submissionId
* create folder tree
* Versions
* Dates

Content

* [Pre requisites](#pre-requisites)
* Exchanges
  * [Listing Exchanges](#listing_exchanges)
    * [Variant - Listing Exchanges associated to a brand](#variant-Listing-exchanges-associated-to-a-given-branding)
  * [Entering an Exchange](#entering-an-exchange)
* Documents & Folders
  * [Listing Folders](#listing-folders)
  * [Listing Documents](#listing-documents)
  * [Downloading a file](#dowloading-a-file)
  * [Calculating a filehash](#calculating-locally-a-filehash)
* Users & Groups
  * [Listing Users](#listing-users)
  * [Listing Groups](#listing-groups)
    * [Variant - Listing groups with users](#listing-groups-with-users)
* [Listing Permissions](#)
* [Listing Audits](#)
* [Logout](#)

Additional topics
* Create Folder
* Create Document
* Create User
* Create Group
* Set Permission
* Send Alerts
* Custom Fields

## Pre requisites

* You need to set up a developer account and an application [Open](INTRALINKS_DEVELOPER.md)
* You need to start a session on Intralinks [Open](INTRALINKS_AUTHENTICATION.md)

## Listing exchanges

[Developer Documentation](https://developers.intralinks.com/swagger/api-ui.html#!/Workspaces/get_workspaces)

```python
import requests

base_url = 'https://test-api.intralinks.com'

access_token = 'your_access_token'

response = requests.get(base_url + '/v2/workspaces', headers={
    'Authorization': 'Bearer {}'.format(access_token)
})

if response.status_code != 200:
    raise Exception(response.status_code, response.text)

json_data = response.json()

if 'workspace' not in json_data:
    raise Exception(response.text)

exchanges = json_data['workspace']

for exchange in exchanges:
    print('{id:>9} | {name:50.50} | {host:20.20} | {phase:20}'.format(**exchange))
```

See https://docs.python.org/3.4/library/string.html#format-examples about ```format()```

Calling ```print(response.text)``` would show something like:

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

## [Variant] Listing exchanges associated to a given branding

*Not documented*

```python
brand_id = 1234

response = requests.get(base_url + '/v2/workspaces', params={'brandId': brand_id}, headers={
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

response = requests.get(base_url + '/v2/workspaces/{}/splash'.format(exchange_id), headers={
    'Authorization': 'Bearer {}'.format(access_token)
})

if response.status_code != 200:
    raise Exception(response.status_code, response.text)

json_data = response.json()

if 'splash' not in json_data:
    raise Exception(response.text)

splash = json_data['splash']
```

Calling ```print(response.text)``` would show something like:

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

### Getting the splash screen image

*Not documented*

```python
import requests

base_url = 'https://test-api.intralinks.com'

access_token = 'your_access_token'
exchange_id = 1234
file_path_without_extension = r'C:\temp\img.jpg'

response = requests.get(base_url + '/services/workspaces/splashImage', params={'workspaceId': exchange_id}, headers={
    'Authorization': 'Bearer {}'.format(access_token)
}, stream=True)

if response.status_code != 200:
    raise Exception(response.status_code, response.text)

extensions = {
    'image/jpeg':'.jpg',
    'image/gif':'.gif',
}

extension = extensions[response.headers['Content-Type']]

with open(file_path_without_extension + extension, 'wb') as file:
    for chunk in response.iter_content(chunk_size=1024): 
        if chunk: 
            file.write(chunk)
```

### Accepting the splash screen

[Developer Documentation](https://developers.intralinks.com/swagger/api-ui.html#!/Workspaces/post_workspaces_workspace_id_splash)

```python
import requests
import json

base_url = 'https://test-api.intralinks.com'

access_token = 'your_access_token'
exchange_id = 1234

response = requests.post(base_url + '/v2/workspaces/{}/splash'.format(exchange_id), data=json.dumps({
        "acceptSplash": True
    }), headers={
        'Authorization': 'Bearer {}'.format(access_token),
        'Content-Type': 'application/json'
})

if response.status_code != 201:
    raise Exception(response.status_code, response.text)

json_data = response.json()

if 'state' not in json_data:
    raise Exception(response.text)

accept_splash_state = json_data['state']

if accept_splash_state != 'ALLOW':
    raise Exception(response.text)
```

Calling ```print(response.text)``` would show something like:

```json
{
    "status":{
        "code":201,
        "message":"The entities have been created"
    },
    "state":"ALLOW"
}
```

## Documents & Folders

### Listing folders

[Developer Documentation](https://developers.intralinks.com/swagger/api-ui.html#!/Folders/get_folders)

```python
import requests

base_url = 'https://test-api.intralinks.com'

access_token = 'your_access_token'
exchange_id = 1234

response = requests.get(base_url + '/v2/workspaces/{}/folders'.format(exchange_id), headers={
    'Authorization': 'Bearer {}'.format(access_token)
})

if response.status_code != 200:
    raise Exception(response.status_code, response.text)

json_data = response.json()

if 'folder' not in json_data:
    raise Exception(response.text)

folders = json_data['folder']

for folder in folders:
    if 'parentId' not in folder:
        folder['parentId'] = -1

    print('{id:>9} | {indexNumber:20.20} | {name:50.50} | {parentId:>9}'.format(**folder))
```

Calling ```print(response.text)``` would show something like:

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

### Listing documents

[Developer Documentation](https://developers.intralinks.com/swagger/api-ui.html#!/Documents/get_documents)

```python
import requests

base_url = 'https://test-api.intralinks.com'

access_token = 'your_access_token'
exchange_id = 1234

response = requests.get(base_url + '/v2/workspaces/{}/documents'.format(exchange_id), headers={
    'Authorization': 'Bearer {}'.format(access_token)
})

if response.status_code != 200:
    raise Exception(response.status_code, response.text)

json_data = response.json()

if 'document' not in json_data:
    raise Exception(response.text)

documents = json_data['document']

for document in documents:
    print('{id:>9} | {indexNumber:20.20} | {name:50.50} | {extension:5.5} | {parentId:>9}'.format(**document))
```

Calling ```print(response.text)``` would show something like:

### Dowloading a file

[Developer Documentation](https://developers.intralinks.com/swagger/api-ui.html#!/Documents/get_document_file)

```python
import requests

base_url = 'https://test-api.intralinks.com'

access_token = 'your_access_token'
exchange_id = 1234
document_id = 5678
file_path = r'C:\Temp\MyFile.pdf'

response = requests.get(base_url + '/v2/workspaces/{}/documents/{}/file'.format(exchange_id, document_id), headers={
    'Authorization': 'Bearer {}'.format(access_token)
}, stream=True)

if response.status_code != 200:
    raise Exception(response.status_code, response.text)

with open(file_path, 'wb') as file:
    for chunk in response.iter_content(chunk_size=1024): 
        if chunk: 
            file.write(chunk)
```

https://stackoverflow.com/questions/16694907/how-to-download-large-file-in-python-with-requests-py

### Calculating locally a filehash

[How to calculate SHA1 for a file](https://stackoverflow.com/questions/22058048/hashing-a-file-in-python)

[How to encode the SHA1 hash using Base64](https://gist.github.com/formido/821003)

```python
import hashlib
import base64

file_path = r'C:\temp\report.pdf'

hasher = hashlib.sha1()

with open(file_name, 'rb') as f:
    while True:
        data = f.read(64*1024)
        if not data:
            break
        hasher.update(data)

file_hash = base64.b64encode(hasher.digest())
```

## Users & Groups

### Listing users

[Developer Documentation](https://developers.intralinks.com/swagger/api-ui.html#!/Workspace_Users/get_workspaces_workspace_id_users)

```python
import requests

base_url = 'https://test-api.intralinks.com'

access_token = 'your_access_token'
exchange_id = 1234

response = requests.get(base_url + '/v2/workspaces/{}/users'.format(exchange_id), headers={
    'Authorization': 'Bearer {}'.format(access_token)
})

if response.status_code != 200:
    raise Exception(response.status_code, response.text)

json_data = response.json()

if 'users' not in json_data:
    raise Exception(response.text)

users = json_data['users']

for user in users:
    print('{lastName:20.20} | {firstName:20.20} | {organization:20.20} | {emailId:50.50} | {roleType:20.20}'.format(**user))
```

Calling ```print(response.text)``` would show something like:

```json
    {"alertPreference": "IMMEDIATE",
    "country": "FRANCE",
    "createdBy": {"firstName": "John",
                "firstNameSort": "John",
                "lastName": "Doe",
                "lastNameSort": "Doe"},
    "createdOn": {"milliseconds": 1295622214000},
    "doNotSendAlert": False,
    "eitherASubmitterOrCoordinator": False,
    "emailId": "john.doe@bigcorporate.com",
    "firstAccessed": {"milliseconds": 1295622221000},
    "firstName": "John",
    "firstNameSort": "John",
    "functionalArea": "UNAVAILABLE",
    "id": 395175832,
    "industry": "PHARMACEUTICALS_BIOTECHNOLOGY",
    "isPlaceholderUser": False,
    "isWelcomeAlertSent": True,
    "keyContact": True,
    "lastAccessedOn": {"milliseconds": 1512163625000},
    "lastAlertSentDate": {"milliseconds": 1508501696000},
    "lastModifiedBy": {"firstName": "John",
                        "firstNameSort": "John",
                        "lastName": "Doe",
                        "lastNameSort": "Doe"},
    "lastModifiedOn": {"milliseconds": 1295622214000},
    "lastName": "Doe",
    "lastNameSort": "Doe",
    "officePhone": "33610801953",
    "organization": "BigCorporate Inc.",
    "organizationSort": "BigCorporate Inc.",
    "roleType": "MANAGER_PLUS",
    "status": "ACTIVE",
    "title": "UNAVAILABLE",
    "userId": 9620005,
    "version": "44fef8aafc39d926a3cdcb891a93dc2b6a63380d"}
```

About IDs: a user in an exchange has 3 different IDs

Property | Independent of the exchange | Meaning
---------|-----------------------------|--------
emailId | Yes | identify the user, independently of an exchange, not stable over time (a user can change email addresses)
userId | Yes | identify the user, independently of an exchange, stable over time
id | No | identify user in this exchange


### Listing groups

[Developer Documentation](https://developers.intralinks.com/swagger/api-ui.html#!/Workspace_Groups/get_workspaces_workspace_id_groups)

```python
import requests

base_url = 'https://test-api.intralinks.com'

access_token = 'your_access_token'
exchange_id = 1234

response = requests.get(base_url + '/v2/workspaces/{}/groups'.format(exchange_id), headers={
    'Authorization': 'Bearer {}'.format(access_token)
})

if response.status_code != 200:
    raise Exception(response.status_code, response.text)

json_data = response.json()

if 'groups' not in json_data:
    raise Exception(response.text)

groups = json_data['groups']

for group in groups:
    print('{groupName:50.50} | {id:>9} | {groupType:20.20}'.format(**group))
```

Calling ```print(response.text)``` would show something like:

```json
{"groups": [{"version": "2c93844f99e2f239b73a66d2234df2eaaaf46d5b", "groupName": "Internal", "groupType": "WORKSPACE", "ftsEnabled": True, "note": " ", "createdBy": {"firstName": "John", "lastName": "Doe", "firstNameSort": "John", "lastNameSort": "Doe"}, "createdOn": {"milliseconds": 1295622323000}, "lastModifiedBy": {"firstName": "John", "lastName": "Doe", "firstNameSort": "John", "lastNameSort": "Doe"}, "lastModifiedOn": {"milliseconds": 1506611922000}, "groupMemberCount": 3, "buyerGroupDetails": {}, "id": 10093765}, {"version": "c0e67ee04016ecdf316f372ce5d9dffe1266c048", "groupName": "1. Interest", "groupType": "WORKSPACE", "ftsEnabled": False, "note": " ", "createdBy": {"firstName": "John", "lastName": "Doe", "firstNameSort": "John", "lastNameSort": "Doe"}, "createdOn": {"milliseconds": 1422281439000}, "lastModifiedBy": {"firstName": "John", "lastName": "Doe", "firstNameSort": "John", "lastNameSort": "Doe"}, "lastModifiedOn": {"milliseconds": 1511431059000}, "groupMemberCount": 46, "buyerGroupDetails": {}, "id": 18180962}, {"version": "404af031f11be2b5ac7ca1cae378599b54feb179", "groupName": "2. Commit", "groupType": "WORKSPACE", "ftsEnabled": False, "note": " ", "createdBy": {"firstName": "John", "lastName": "Doe", "firstNameSort": "John", "lastNameSort": "Doe"}, "createdOn": {"milliseconds": 1422281459000}, "lastModifiedBy": {"firstName": "John", "lastName": "Doe", "firstNameSort": "John", "lastNameSort": "Doe"}, "lastModifiedOn": {"milliseconds": 1511431061000}, "groupMemberCount": 77, "buyerGroupDetails": {}, "id": 18180982}], "workspaceGroupMembers": []}
```

### [Variant] Listing groups with users

[Developer Documentation](https://developers.intralinks.com/swagger/api-ui.html#!/Workspace_Groups/get_workspaces_workspace_id_groups)

```python
response = requests.get(base_url + '/v2/workspaces/{}/groups'.format(exchange_id), params={'includeWorkspaceGroupMembers':'true'}, headers={
    'Authorization': 'Bearer {}'.format(access_token)
})

if response.status_code != 200:
    raise Exception(response.status_code, response.text)

json_data = response.json()

if 'groups' not in json_data or 'workspaceGroupMembers' not in json_data:
    raise Exception(response.text)

groups = json_data['groups']
group_members = json_data['workspaceGroupMembers']
```

```json
{
    "groups":[{
        "version":"c0e67ee04016ecdf316f372ce5d9dffe1266c048",
        "groupName":"1. Interest",
        "groupType":"WORKSPACE",
        "ftsEnabled":false,
        "note":" ",
        "createdBy":{
            "firstName":"John",
            "lastName":"Doe",
            "firstNameSort":"John",
            "lastNameSort":"Doe"
        },
        "createdOn":{
            "milliseconds":1422281439000
        },
        "lastModifiedBy":{
            "firstName":"John",
            "lastName":"Doe",
            "firstNameSort":"John",
            "lastNameSort":"Doe"
        },
        "lastModifiedOn":{
            "milliseconds":1511431059000
        },
        "groupMemberCount":46,
        "buyerGroupDetails":{},
        "id":18180962
    },{
        ...
    }],
    "workspaceGroupMembers":[
        {"workspaceGroupId":10093765,"workspaceUserId":395175832,"userId":9620005},
        {"workspaceGroupId":10093765,"workspaceUserId":3254281522,"userId":27605305}
    ]
}
```

## Logout

[Developer Documentation](https://developers.intralinks.com/content/oauth_revoke)

```python
import requests

base_url = 'https://test-api.intralinks.com'

access_token = 'your_access_token'
client_id = 'your_consumer_key'
client_secret = 'your_consumer_secret'

response = requests.put(base_url + '/v2/oauth/revoke', params={
        'token': access_token,
        'client_id': client_id,
        'client_secret': client_secret
    }, headers={
        'Authorization': 'Bearer {}'.format(access_token)
})

if response.status_code != 200:
    raise Exception(response.status_code, response.text)
    
print(response.text)
```

Status code 200

```json
{
    "message":"the passed token and associated refresh/access token have been revoked",
    "token":"XXXX",
    "cascade":"true"
}
```

## Listing Permissions

## Listing Audits

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

### Wrong URL

You are trying to access a wrong API URL

Status code 404

```html
<html><head>\r\n<meta http-equiv="content-type" content="text/html; charset=UTF-8"><title>Error</title>\r\n\r\n<script src="support_files/showmaintenance.js" type="text/javascript">\r\n</script>\r\n\r\n<style>\r\nbody{\r\n\tmargin:0px;\r\n\tpadding:0px;\r\n\theight: 100%;\r\n\twidth: 100%;\r\n\tfont-family: Arial Unicode MS,Arial,sans-serif;\r\n}\r\n\r\n.main_header{\r\n\tbackground-color:#FFFFFF;\r\n\theight:56px;\r\n\tpadding-left: 20px;\r\n\tpadding-top: 20px;\r\n}\r\n\r\n.content{\r\n\tbackground-color:#FFFFFF;\r\n\ttext-align:center;\r\n\tpadding:100px 0px 250px 0px;\r\n\tfont-weight:bold;\r\n\tfont-size:12px;\r\n}\r\n.footer{\r\n    color: #666666;\r\n\tfont-family: Arial,Helvetica,sans-serif;\r\n\tfont-size: 10px;\r\n\tpadding-top: 5px;\r\n\tpadding-left: 5px;\r\n\ttext-align: center;\r\n}\r\n.footer a {\r\n
 color: #006699;\r\n    textDecoration: underline;\r\n}\r\n\r\n</style>\r\n</head>\r\n<body>\r\n\r\n<div class="main_header">\r\n\t<img src="/images/mastLogo_new.png">\r\n</div>\r\n<div class="content">\r\n\r\nYour request cannot be completed, please retry. <br><br>\r\nIf the problem persists please contact <a href="mailto:support@intralinks.com">IntraLinks Support</a> for more information.</div>\r\n<div class="footer">\r\n<script type="text/javascript">document.write(\'ï¿½ 2000-\'+new Date().getFullYear()+\' Intralinks, Inc.\');</script><a href="/legalNotices.html">legal notices</a>\r\n</div>\r\n\r\n</body></html>
```
