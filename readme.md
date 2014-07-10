Maytech APIs
=============



Looking to integrate with a Maytech product? We've got everything you need in this guide. Code samples, detailed API documentation, notes and tips that will help you to start in no time.

----------------------



Authentication
=============

To perform non-anonymous call on Quatrix API, client application should obtain session token. Additionally every call should be signed with user's password pbkdf2 to form 'authorization' header.

Login call accepts user login (email), referer (to differentiate accounts) and authorization token. Authorization token is calculated based on HTTP request method, route URI, login, timestamp and user's password pbkdf2.

If login was successful - API returns 200 OK status with X-Auth-Token header containing newly created session token. This session id is than used as variable to calculate authorization token for next API calls that require authorization.

Failed login returns 401 HTTP error.

Login
=============
_Route: /session/login_

**HTTP Request format**
```
GET /session/login
X-Auth-Login: <user_login>
X-Auth-Timestamp: <unix_timestamp>
Authorization: <authorization_token>
Referer: <referer_url>
```

Authorization token calculation

1. Generate pbkdf2 of user password (salt=''):
```
password_pbkdf2 = PBKDF2(password, salt, iterations=4096, length=32)
```
2. Form SIGNATURE_SUBJECT string using following template:
```
// Signature subject
"[METHOD] [URI]\n
x-auth-login: [user_login]\n
x-auth-timestamp: [unix_timestamp]\n"
```
Signature subject example
```
"GET /session/login\n
x-auth-login: user@example.com\n
x-auth-timestamp: 1320930744\n"
```
3. HMAC SIGNATURE_SUBJECT using generated password_pbkdf2 hex string representation:
```
authorization_token = hmac_sha1(SIGNATURE_SUBJECT, hex(password_pbkdf2))
```
----------------------

Example request/response
=============
**Request**
```
GET /session/login
X-Auth-Login: user@example.com
X-Auth-Timestamp: 1320930744
Authorization: 7695ceb53fca87677ba4b21b418d419d68da604e
Referer: http://ibm.maytech.net/
```

**Response**
```
HTTP/1.1 200 OK
HTTP/1.1 401 Unauthorized
X-Auth-Token: token
```
----------------------


Request signature
=============
Every HTTP request that is not anonymous should contain following headers:
 - X-Auth-Token - user session token
 - Authorization - request metadata signed with user's password pbkdf2
 - X-Auth-Timestamp - request timestamp

These values also can be sent as query string parameters.

Request metadata to be signed is formed using following template:
```
[METHOD] [URI]
X-Auth-Timestamp: [unix_timestamp]
X-Auth-Token: [session_token]
```
----------------------


Notifications
=============

API notifications are used to notify of any changes occurred in real-time. Notifications are sent via comet connection (using long polling).

Each notification is sent to user affected by change being notified (for example file deletion is notified to all users seeing this file).

Users must subscribe to notifications using their "channel" property which is available through '/profile/get' API call.

To start listening for notifications on specified channel '/mq/req?id=<channel>' GET request should be used.

**Notifications have the following general format:**
```
{
  "status": {
    "errmsg": <error_message>,
    "errcode": <error_code>
  },
  "job_id": <job_id>,
  "data": [<modified_data>]
}
```

Help us make it better
----------------------

Please tell us how we can make the APIs better. If you have a specific feature request or if you found a bug, please contact us: support@maytech.net