# Static Key HMAC Authorization (DRAFT)

This document proposes a standard mechanism for authentication of web services using static keys and HMAC signatures. This implementation was inspired by the existing HMAC methods used by [Amazon Web Services](http://s3.amazonaws.com/doc/s3-developer-guide/RESTAuthentication.html).

This mechanism is designed for two-legged authentication. The web service and the trusted client share a secret between them. OAuth 2 should be used for web services that require a user be able to share permissions with the client.

## Terms

### Static keys

The web service is responsible for issuing static key pairs to the clients. Each client should be given a key pair consisting of a KEYID and KEYDATA. 

- KEYID = a short identifying string. Recommended 8 alpha-numeric characters.

- KEYDATA = a longer string of secret data to be used when generating HMAC signatures.  The string should be long and/or random enough to be hard to copy or guess. Recommended 32 or more random, alpha-numeric characters.

### Web service URLs

The web service is assumed to consist of a number of application paths sharing a common base URL. In other words:

    URL = BASEURL + PATH
    e.g. "https://api.ncsu.edu/pager" + "/groups?dept=oit"

- The BASEURL will always include the server name, and may contain a subdirectory path.  It should not contain a trailing "/" character. The BASEURL is not used in generating the HMAC-Auth signature.

- The PATH portion of the URL always starts with a "/" and contains whatever resource and :id portions are required. If a query string is used in the URL, that should also be included as part of the PATH when generating the HMAC-Auth signature.

## Sending requests

### HMAC-Auth header

Every request must include an HMAC-Auth header containing the KEYID and the request signature computed using the KEYDATA. The header and signature should be created like this:


    "HMAC-Auth: " + KEYID + ":" + base_64(hmac-sha1( METHOD + "\n"
                                                   + PATH + "\n"
                                                   + DATE + "\n"
                                                   + CONTENT-MD5 ))

- The METHOD is the HTTP method or verb to be used in this call. It will usually be one of (GET, PUT, POST, DELETE or PATCH).

- The PATH is the application-specific potion of the URL, including any query portion, as described in [Web Services URLs](#web-services-urls) above.

- The DATE must be the same string used to set the "Date:" header in the request. The Date header should be formatted according to [HTTP standards](http://www.w3.org/Protocols/rfc2616/rfc2616-sec3.html#sec3.3.1). 

- The CONTENT-MD5 string must contain the base-64 encoded MD5 checksum of the message content data. If the message contains no content (as in GET requests) the CONTENT-MD5 should be a an empty string for the purpose of calculating the signature. 

The hmac-sha1 calculation should be seeded using the secret KEYDATA. The resulting signature should be base-64 encoded before including it in the HMAC-Auth header.

### Additional request headers

- The "Date:" header should be sent with every signed request. The web service will use this string to verify the HMAC signature.

- If the request contains content, then the "Content-MD5:" header should be used. The web service will verify this value matches for the content that was received.

### Example requests

Example 1 - a simple GET request using these values:
- KEYID   = "test123"
- KEYDATA = "mysecretkeydata"
- BASEURL = "http://mosa.unity.ncsu.edu/pager"
- PATH    = "/oncall/oit-iws"
- METHOD  = "GET"
- DATE    = "Wed, 14 Aug 2013 18:33:25 GMT"
- CONTENT-MD5 = ""

```
    GET http://mosa.unity.ncsu.edu/pager/oncall/oit-iws
    Date: Wed, 14 Aug 2013 18:33:25 GMT
    Accept: application/vendor.api-v1+json
    User-Agent: OIT-RESTclient/0.1.2
    HMAC-Auth: test123:NbTCv3pArrZEkVbV37tBcpjYPSc

    (no content)
```

Example 2 - a POST request with content
- METHOD = "POST"
- DATE = "Wed, 14 Aug 2013 18:35:30 GMT"
- CONTENT = "foo=bar&baz=blu"
- CONTENT-MD5 = "g26hErLKewirhYsLEW7mDg"

```
    POST http://mosa.unity.ncsu.edu/pager/oncall/oit-iws
    Date: Wed, 14 Aug 2013 18:35:30 GMT
    Accept: application/vendor.api-v1+json
    User-Agent: OIT-RESTclient/0.1.2
    Content-Length: 15
    Content-MD5: g26hErLKewirhYsLEW7mDg
    Content-Type: application/x-www-form-urlencoded
    HMAC-Auth: test123:j5m9Z8Wl+BeYdr0f1nMu0OQOueg

    foo=bar&baz=blu
```

## Validating requests

The web service should perform each of the following steps when validating an incoming request. If any of these steps fails, a 403 Forbidden error should be returned.

- Require a valid Date header.
- Parse the Date header and verify that the timestamp is within a reasonable offset of the current time. A very short offset (5-30 seconds) is best to avoid replay attacks. A longer offset (like 5 minutes) can be used for less critical services.
- Require a valid HMAC-Auth header.
- Verify that the KEYID passed in the HMAC_Auth header is known.
- For requests that contain content:
    - Require a valid Content-MD5 header.
    - Take the MD5 checksum of the passed content and verify that it matches the value given by the Content-MD5 header.
- Look up the KEYDATA for the passed KEYID. Use that to calculate the hmac-sha1 signature for the request. Verify that the calculated signature matches the one passed in the HMAC_Auth header.

## Code examples

### Clients

- Perl - the module [OIT::RESTclient](https://github.ncsu.edu/brabec/p5-OIT-RESTclient) was written to implement this mechanism.

### Servers

- Perl - a plugin module for the Dancer framework is being developed. 

## Points for discussion

### Why not use the Authorization header?

The HTTP Authorization header is supposed to be processed by the web server. Apache does not pass it along to scripts because it considers that to be a security risk. Given that Basic Auth passed the username/password pair as encoded clear text, passing that along to a script to read could be a problem. Rather than fight with reconfiguring Apache to break this security measure, it seemed better to use a custom header that would be passed along.

### What about replay attacks?

This mechanism as currently proposed does not prevent the same request from being processed twice. The timestamp can only limit how soon a duplicate requests could be made.

A better mechanism would include some sort of unique request ID in the signed data. It would also require that every request include a unique request ID, and that the web service be able to track seen requests. This would almost certainly require a shared database for a service running on multiple servers. 

I think OAuth 1.0a, or two-legged OAuth, handles this better. 

