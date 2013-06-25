# Authentication Standards (DRAFT)

The intent of this document is to provide guidance on the implementation of authentication for web services.
It does so by enumerating a distributed set of rules, expectations, and best practices about how to build and maintain authentication systems for web services.

The scope of this web services standards recommendation is to:  

* Specify requirements for all authentication implementations.
* Specify recommended authentication methods.
* Specify implementation requirements for those methods.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY",
and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt).

## Overview

* [Secure Transfer](#secure-transfer)
* [Rate Limiting](#rate-limiting-authentication-endpoints)
* [Recommended Authentication Methods](#recommended-authentication-methods)
	* [OAuth 2.0](#oauth-20)
	* [OAuth 1.0a](#oauth-10a)
	* [Static Keys](#static-keys)

## Secure Transfer

For the entire duration of the authorization process, the authorization server MUST require the use of TLS/SSL. Requests made not using TLS/SSL MAY be redirected using a 302 status to use TLS/SSL properly. If not redirecting, the server MUST refuse the request with an appropriate error. While it is RECOMMENDED that the use of TLS/SSL is maintained for subsequent requests after authorization is completed, it is not required. 

## Rate Limiting Authentication Endpoints

It is REQUIRED that all authentication endpoints implement a static, time-based rate limiting system. This requirement is to mitigate any brute force attacks against the authentication endpoint. Additionally, it is RECOMMENDED that the endpoint implement a Fail2Ban-style system. The implementation of both of these systems is outside the scope of this document.

## Recommended Authentication Methods

### OAuth 2.0

OAuth 2.0 is the RECOMMENDED framework for all authentication systems in which a client will be acting on behalf of an end-user, or needs to access protected resources owned by the end-user that it does not already have explicit access to. This is referred to as 3-legged OAuth. For situations in which the client will be interfacing directly with the server with no interaction or authorization required from an end-user (2-legged OAuth) it is RECOMMENDED to use a static key system or OAuth 1.0a instead; however, 2-legged OAuth using OAuth 2.0 MAY be used if desired. All implementations of OAuth 2.0 MUST adhere to the specification defined in [RFC6749](http://tools.ietf.org/html/rfc6749), as well as the following constraints:  

* Access tokens MUST NOT persist longer than 24 hours.
* Access tokens SHOULD NOT be stored in a location accessible to anyone but the authentication server

Note that the OAuth 2.0 specification REQUIRES the use of TLS/SSL for all requests, including the initial authentication as well as all subsequent resource requests if bearer tokens (the RECOMMENDED token type) are used. If TLS/SSL is not available for requests to the resource server, MAC tokens MUST be used. 

### OAuth 1.0a

OAuth 1.0a is the RECOMMENDED protocol for authentication between a client and a server where no end-user is involved, and the client should have explicit access to all requested resources (2-legged OAuth). OAuth 1.0a MAY be used to authenticate for access to resources owned by an end-user, but it is RECOMMENDED that OAuth 2.0 be used in that case. All implementations of OAuth 1.0a MUST adhere to the specification defined in [RFC5849](http://tools.ietf.org/html/rfc5849). 

While it is REQUIRED by the specification that TLS/SSL be used for the initial transfer of tokens, clients MAY choose to omit the TLS/SSL requirement when requesting resources after tokens have been acquired, as the tokens used in OAuth 1.0a are safe to use over a clear connection. It is RECOMMENDED that such requests continue to utilize TLS/SSL if available. 

Since OAuth 1.0a is less trivial to deploy in enterprise applications, it is RECOMMENDED only for use in browser based environments. For enterprise applications wishing to implement a 2-legged authentication system, static keys are RECOMMENDED instead, while OAuth 2.0 is RECOMMENDED for use in 3-legged authentication.

### Static Keys

A static key is defined as any single string known by both the client and the server which grants the client full access to requested resources. A client MAY possess multiple keys. A single key MAY have a scope associated with it that limits access, and the server SHOULD document such limitations for the client. The scope for a key SHOULD NOT be changed after it has been granted. The scope for a key MUST NOT be changed after it has been granted without notifying the client.  

Static keys MUST NOT be used to take action on behalf of a user. All actions taken through authentication by static key MUST be attributed to the client. It is RECOMMENDED that static keys only be used when an end-user is not involved in the interaction, and the client should have explicit access to all requested resources.

Authentication systems that use static keys MUST abide by the following constraints:

* Keys consist of at least 8 alphanumeric characters
* Keys do not contain characters outside the standard visible ASCII range (0x20-0x7E)

It is RECOMMENDED that keys are constructed in such a manner that they are easily validated without a database lookup. This allows the system to drop malformed keys without consuming additional resources. Verification that the key is in use and what the scope is will require a lookup, but the syntactic validation of the key does not. Implementation of such construction is outside the scope of this document.
