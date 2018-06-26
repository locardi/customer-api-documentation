# Locardi Customer API Documentation

This is the documentation for interacting with our APIs.

## SDKs

At this stage we provide our own SDK for the following languages:

- PHP https://github.com/locardi/php-sdk

The SDKs take care of all of the authentication logic, but, if you prefer, you can write your own following the documentation below. 

## Endpoint

All the relative API URLs listed in the following pages must point to the following endpoint:

https://collector.locardiapp.com

## JWT API

The Locardi API implements a JSON Web Token authentication system.

In order to send requests you first need to retrieve a token.

This API is for retrieving a token.

### Request

```
POST /jwt/token
Body:
{
    "_username": "theusername",
    "_password": "thepassword"
}
```

### Response

If the login is successful:

```
Status Code: 200
Body:
{
    "success": true,
    "token":"the-actual-token"
}
```

Otherwise:

```
Status Code: 400 if Bad request (the JSON body you have sent is wrong or 500 if Internal error (please contact us for support)
Body:
{
    "success": false,
    "error": "The error message"
}
```

The token will be valid for 1 hour (60 minutes), after that you will need to apply for a new one with the same process.

You will receive a 401 HTTP status code from an API if you use an expired or invalid token.
 
We recommend to store the token somewhere (in a file or in the session or in a cache) and apply for a new one only in when getting a 401.

### Sending the token when calling an API

The token has to be provided as a request header attribute as follows:

```
Headers:
X-Access-Token: the-token
Content-Type: application/json
```

If the token is not provided or incorrect, you will get a 401 response with this JSON content:

```json
{
    "message": "A Token was not found in the TokenStorage."
}
```

If the token was correct you will get the JSON response for the specific API you called.

## Request API

This API is for sending a new request to the locardi database.

### Request

```
POST /collector/request
```

Headers:

```
X-Access-Token: the-token
Content-Type: application/json
```

Body:

```json
{
  "organization_user_request": {
    "request_type": "html_page",
    "ip_address":"8.8.8.8",
    "http_method":"GET",
    "timestamp":"2017-10-05T16:24:49+02:00",
    "user_agent":"Mozilla\/5.0 (X11; Linux x86_64) AppleWebKit\/537.36 (KHTML, like Gecko) Chrome\/57.0.2987.110 Safari\/537.36",
    "uri":"https:\/\/www.getlocardi.com\/?param1=value1&param2=value2asd123",
    "organization": {
      "id":"123",
      "name":"Organization2"
    },
    "user": {
      "id":"123",
      "name":"User1",
      "type":"internal",
      "timezone":"Europe\/London"
    }
  }
}
```

| Field              | Description                                                                                                                                          | Allowed values                                       | Format           | Example                                                                                                   |
|--------------------|------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------|------------------|-----------------------------------------------------------------------------------------------------------|
| request_type       | The type of the link visited by the user                                                                                                             | html_page, login, login_failed, download             | string           |                                                                                                           |
| ip_address         | The IP address used by the user (be careful if you have a Load Balancer as you want to collect the actual IP address and not the on on the LB.       |                                                      | Any IPv4 or IPv6 | 8.8.8.8, 2001:0db8:0000:0042:0000:8a2e:0370:7334                                                          |
| http_method        | The HTTP method used by the user                                                                                                                     | GET, HEAD, POST, PUT, DELETE, CONNECT, OPTION, TRACE | enum             |                                                                                                           |
| timestamp          | The timestamp when the user request was generated                                                                                                    |                                                      | RFC3339          | 2017-10-05T16:24:49+02:00                                                                                 |
| user_agent         |                                                                                                                                                      |                                                      | string           | Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.110 Safari/537.36 |
| uri                | The full address requested by the user (including query string, anchor, etc)                                                                         |                                                      | string           | https://www.getlocardi.com/?param1=value1&param2=value2asd123                                             |
| organization[id]   | The organization ID in your database. Please note that this is a string, remember to wrap it with quotes in the JSON)                                |                                                      | string(50)       |                                                                                                           |
| organization[name] |                                                                                                                                                      |                                                      | string(255)      |                                                                                                           |
| user[id]           | The user ID in your database. Please note that this is a string, remember to weap it with quotes in the JSON)                                        |                                                      | string(50)       |                                                                                                           |
| user[name]         | This can be the name of the user, the email, the email address or any other field that would allow you to identify the user in the Locardi dashboard |                                                      | string(255)      |                                                                                                           |
| user[type]         | Whether the user is internal (like an admin/editor) or external (like a client)                                                                      | internal, external                                   | enum             |                                                                                                           |
| user[timezone]     | Where the user is based.                                                                                                                             |                                                      | enum             | Europe/London                                                                                             |

If you will ever provide a new name for a user or an organization using the same ID this will result in overriding the user or organization name and NOT in creating a new user/organization.

### Response

Status Codes:

```
201 if successful
400 if Bad request
500 if Internal error
```

The response body is just an empty JSON `[]`

## Failed Login for Unknown Username API

You may want to trace requests where somebody is trying to login with a username/email that you don’t have in the database. In some situations this is the scenario when somebody is trying a brute force attack.

### Request

```
POST /collector/request
```

Headers:

```
X-Access-Token: the-token
Content-Type: application/json
```

Body:

```json
{
  "client_login_failed_unknown_username_request": {
    "ip_address":"8.8.8.8",
    "http_method":"GET",
    "timestamp":"2017-10-05T16:24:49+02:00",
    "user_agent":"Mozilla\/5.0 (X11; Linux x86_64) AppleWebKit\/537.36 (KHTML, like Gecko) Chrome\/57.0.2987.110 Safari\/537.36",
    "uri":"https:\/\/www.getlocardi.com\/?param1=value1&param2=value2asd123",
    "user": {
      "name":"User1"
    }
  }
}
```

| Field              | Description                                                                                                                                          | Allowed values                                       | Format           | Example                                                                                                   |
|--------------------|------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------|------------------|-----------------------------------------------------------------------------------------------------------|
| ip_address         | The IP address used by the user (be careful if you have a Load Balancer as you want to collect the actual IP address and not the on on the LB.       |                                                      | Any IPv4 or IPv6 | 8.8.8.8, 2001:0db8:0000:0042:0000:8a2e:0370:7334                                                          |
| http_method        | The HTTP method used by the user                                                                                                                     | GET, HEAD, POST, PUT, DELETE, CONNECT, OPTION, TRACE | enum             |                                                                                                           |
| timestamp          | The timestamp when the user request was generated                                                                                                    |                                                      | RFC3339          | 2017-10-05T16:24:49+02:00                                                                                 |
| user_agent         |                                                                                                                                                      |                                                      | string           | Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.110 Safari/537.36 |
| uri                | The full address requested by the user (including query string, anchor, etc)                                                                         |                                                      | string           | https://www.getlocardi.com/?param1=value1&param2=value2asd123                                             |
| user[name]         | This can be the name of the user, the email, the email address or any other field that would allow you to identify the user in the Locardi dashboard |                                                      | string(255)      |                                                                                                           |

### Response

Status Codes:

```
201 if successful
400 if Bad request
500 if Internal error
```

The response body is just an empty JSON `[]`
