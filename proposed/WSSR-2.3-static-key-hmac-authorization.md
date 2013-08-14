# Static Key HMAC Authorization (DRAFT)

This document proposes a standard mechanism for authentication of web services using static keys and HMAC signatures. This implementation was inspired by the existing HMAC methods used by [Amazon Web Services](http://s3.amazonaws.com/doc/s3-developer-guide/RESTAuthentication.html).

## Terms

### Static Keys

The web service is responsible for issuing static key pairs to the clients. Each client should be given a key pair consisting of a KEYID and KEYDATA. 

- KEYID = a short identifying string. Recommended 8 alpha-numeric characters.

- KEYDATA = a longer string of secret data to be used when generating HMAC signatures.  The string should be long and/or random enough to be hard to copy or guess. Recommended 32 or more random, alpha-numeric characters.

### Web Service URLs

The web service is assumed to consist of a number of application paths sharing a common base URL. In other words:

    URL = BASEURL + PATH
    e.g. "https://api.ncsu.edu/pager" + "/groups?dept=oit"

- The BASEURL will always include the server name, and may contain a subdirectory path.  It should not contain a trailing "/" character. The BASEURL is not used in generating the HMAC-Auth signature.

- The PATH portion of the URL always starts with a "/" and contains whatever resource and :id portions are required. If a query string is used in the URL, that should also be included as part of the PATH when generating the HMAC-Auth signature.

## Sending Requests

### HMAC-Auth Header

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

### Additional Request Headers

- The "Date:" header should be sent with every signed request. The web service will use this string to verify the HMAC signature.

- If the request contains content, then the "Content-MD5:" header should be used. The web service will verify this value matches for the content that was received.

### Example Requests

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
