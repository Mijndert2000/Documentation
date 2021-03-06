# Rest API v1

## Authentication

Authentication to the API can be done using tokens that can be inserted into
the database directly, or created in the web interface. An `Authorization` header should be added
to each request. For example, `Authorization: Bearer <token>` would be the correct way to authenticate
to the API.



## Pauses

MailerQ offers very fine-grained control over delivery flow, and allows you to pause almost any combination
of pool/mta and domain/ip.

### GET

A GET request returns all currently active pauses. It returns a JSON array of pauses, see the example below.

```
GET /v1/pauses HTTP/1.0
Authorization: Bearer ...
```

Might result in
```
HTTP/1.0 200 Ok
Content-Type: application/json
...

[
    {
        "mta": "1.2.3.4",
        "ip": "123.2.3.4",
        "cluster": false
    }, 
    {
        "mta": "1.2.3.4",
        "domain": "gmail.com",
        "cluster": false
    },
    {
        "domain": "hotmail.com",
        "cluster: true
    },
    {
        "pool": "shared-pool-1",
        "domain": "yahoo.com",
        "cluster": false
    }
]
```
In this example, traffic from `1.2.3.4` to `123.2.3.4` and `gmail.com` is paused on the instance, all traffic from `shared-pool-1` to `yahoo.com` is paused, and traffic to `hotmail.com` is paused for the entire cluster. 

### POST

A POST request allows you to create a new pause, or to update an existing one. For the request format, see
the table below. All fields are optional.

| Field | Required  | Type | Description
|---|---|---|---|
| pool  | no  | string | Pool that the pause applies to. 
| mta  | no | string | MTA IP that the pause applies to. 
| domain | no | string | Domain that the pause applies to. 
| ip | no | string | Remote IP that the pause applies to.
| tag  | no  | string | The tag that the pause applies to.
| cluster | no | bool | Whether or not this pause is for the entire cluster or only this instance.

Do note that not all fields may be passed at the same time. `pool` and `mta` are mutually exclusive, and so are `domain`
and `ip`. That means that they may not be supplied together, or they will result in a `400` response. 

For example, the request below will pause sending to `hotmail.com` for a single campaign.
```
POST /v1/pauses HTTP/1.0
Content-Type: application/json
Authorization: Bearer ...

{
    "domain": "hotmail.com"
    "tag": "Campaign1"
}
```
and equivalently, with urlencoded body
```
POST /v1/pauses HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Authorization: Bearer ...

domain=hotmail.com&tag=Campaign1
```

### DELETE

A DELETE request allows you to remove a pause. For example, the request below removes the pause for the entire cluster
to gmail.com from the list of pauses.

```
DELETE /v1/pauses HTTP/1.0
Authorization: Bearer ...
Content-Type: application/x-www-form-urlencoded

cluster=true&domain=gmail.com
```

The request format is exactly the same as a POST request, but with the DELETE method. Delete also supports both JSON and
urlencoded requests.



## Errors

MailerQ offers very fine-grained control over delivery flow, and allows you to intercept it and forcing an error
on any combination of pool/mta and domain.

### GET

A GET request returns all currently active errors. It returns a JSON array of errors, see the example below.

```
GET /v1/errors HTTP/1.0
Authorization: Bearer ...
```

Might result in
```
HTTP/1.0 200 Ok
Content-Type: application/json
...

[
    {
        "mta": "1.2.3.4",
        "ip": "123.2.3.4",
        "cluster": false,
        "code": 500
    }, 
    {
        "mta": "1.2.3.4",
        "domain": "gmail.com",
        "cluster": false,
        "code": 421,
        "description": "forced deferring"
    },
    {
        "domain": "hotmail.com",
        "cluster: true,
        "code": 521,
        "extended": "5.2.1"
    },
    {
        "pool": "shared-pool-1",
        "domain": "yaho0.com",
        "cluster": false,
        "code": "553",
        "description": "invalid mailbox"
    }
]
```
In this example, traffic from `1.2.3.4` to `123.2.3.4` is failed with an error code 500, anc traffic to `gmail.com` is deferred with
an error code of 421 to be tried again later. In the third value, all emails to `hotmail.com` get an `521` error on the entire cluster.
Lastly, all mail from `shared-pool-1` to `yaho0.com` on this instance are failed because the mailbox is invalid.

### POST

A POST request allows you to create a new error, or to update an existing one. For the request format, see
the table below. All fields are optional.

| Field | Required  | Type | Description
|---|---|---|---|
| code | yes | int | Numeric error code between 200 and 599 (smtp error codes). 
| extended | false | string | Extended SMTP error code, e.g. 5.7.1.
| description | false | string | Description to put in the message result.
| pool  | no  | string | Pool that the error applies to. 
| mta  | no | string | MTA IP that the error applies to. 
| domain | no | string | Domain that the error applies to.
| tag  | no  | string | The tag that the error applies to.
| cluster | no | bool | Whether or not this error is for the entire cluster or only this instance.

Do note that not all fields may be passed at the same time. `pool` and `mta` are mutually exclusive. 
That means that they may not be supplied together, or they will result in a `400` response. 

For example, the request below will install a forced error with code `421` to `hotmail.com` for a single campaign.
```
POST /v1/errors HTTP/1.0
Content-Type: application/json
Authorization: Bearer ...

{
    "domain": "hotmail.com",
    "code": 421,
    "tag": "Campaign1"
}
```
and equivalently, with urlencoded body
```
POST /v1/errors HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Authorization: Bearer ...

domain=hotmail.com&code=421&tag=Campaign1
```

### DELETE

A DELETE request allows you to remove an error. For example, the request below removes the error for the entire cluster
to gmail.com from the list of errors.

```
DELETE /v1/errors HTTP/1.0
Authorization: Bearer ...
Content-Type: application/x-www-form-urlencoded

cluster=true&code=421&domain=gmail.com
```

The request format is exactly the same as a POST request, but with the DELETE method. Delete also supports both JSON and
urlencoded requests.



## Injection

MailerQ offers an HTTP injection API. Check out the [message format](json-messages) for the required structure of injected
messages. 

### POST
```
POST /v1/inject HTTP/1.0
Authorization: Bearer ...
Content-Type: application/json

{
    "envelope": "my-sender-address@my-domain.com",
    "recipient": "info@example.org",
    "mime": "From: my-sender-address@my-domain.com\r\nTo: info@example.org\r\nSubject: ..."
}
``` 

Injected messages simply get published to the [inbox queue](rabbitmq-config#rabbitmq-queues) specified in the config file. The injection endpoint does not support urlencoded post, only JSON directly. 

Injected messages will get an extra `http` property, for its properties see [incoming messages](json-incoming#rest-api).
