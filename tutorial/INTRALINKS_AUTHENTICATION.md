# How to authenticate on Intralinks with the Developer Portal

## Log in on the Developer Portal

![Authentication - Step 1](/images/Authentication1.png)

## Init the API

![Authentication - Step 2](/images/Authentication2.png)
![Authentication - Step 3](/images/Authentication3.png)

## Log in on Intralinks

![Authentication - Step 4](/images/Authentication4.png)

## Retrieve the Access Token

![Authentication - Step 5](/images/Authentication5.png)
![Authentication - Step 6](/images/Authentication6.png)

# Old tutorial

## Login

[Developer Documentation](https://developers.intralinks.com/content/oauth_token)

```python
import requests

base_url = 'https://test-api.intralinks.com'

client_id = 'your_consumer_key'
client_secret = 'your_consumer_secret'
email =  'your_email'
password = 'your_password'

response = requests.post(base_url + '/v2/oauth/token', data={
    'grant_type':'client_credentials',
    'client_id':client_id,
    'client_secret':client_secret,
    'endOtherSessions':'True',
    'email':email,
    'password':password
})

if response.status_code != 200:
    raise Exception(response.status_code, response.text)

json_data = response.json()

if 'access_token' not in json_data:
    raise Exception(response.text)

access_token = json_data['access_token']
```

Calling ```print(response.text)``` would show something like:

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

