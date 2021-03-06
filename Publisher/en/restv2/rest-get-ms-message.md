# REST API: GET message (Marketing Suite)

If you want to get some general information from a mail sent with Marketing
Suite you can make a simple a GET request to the following URL:

`https://api.copernica.com/v2/ms/message/$id?access_token=xxxx`

where `$id` is the unique string that identifies a message. 

You can find the call to retrieve a Publisher message [here](./rest-get-publisher-message).

## Return value

A JSON with the general information about the message.

## PHP Example

The following PHP script demonstrates how to use the API method.

```php
// dependencies
require_once('copernica_rest_api.php');
   
// change this into your access token
$api = new CopernicaRestAPI("your-access-token", 2);

// do the call, and print result
print_r($api->get("ms/message/{$messsageID}"));
```

The example above requires the [CopernicaRestApi class](rest-php).

## More information

* [Overview of all REST API calls](./rest-api)
* [GET message body](./rest-get-message-body)
