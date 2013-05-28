# RESTful API Style Guide (DRAFT)

The intent of this guide is to increase interoperability between web services on campus. It does so
by enumerating a shared set of rules and expectations about how to build and maintain RESTful web services.

The style rules are derived from commonalities among the various projects. When various authors
collaborate across multiple projects, it helps to have one set of guidelines to be used among all those projects.
Thus, the benefit of this guide is not in the rules themselves, but in the sharing of those rules.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY",
and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119][].

[RFC 2119]: http://www.ietf.org/rfc/rfc2119.txt
[WSSR-0]: https://github.ncsu.edu/ncsu-interop-group/ws-standards/blob/constraints-and-style-guidelines/proposed/WSSR-0-restful-constraints.md
[SEMVER]: http://semver.org/spec/v2.0.0-rc.2.html
## Overview

* All services MUST adhere to the constraints set in [WSSR-0][].
* [RESTful URLs](#restful-urls)
* [API Versioning](#api-versioning)
* [Content Negotiation](https://github.ncsu.edu/ncsu-interop-group/ws-standards/issues/9)
* [Request & Response Examples](#request--response-examples)

## RESTful URLs

### General Guidelines for RESTful URLs

* URLs MUST identify a resource in the URL and MUST NOT use the query string to specify the resource.
* URLs MUST include nouns, not verbs.
* URLs SHOULD use plural nouns only for consistency (no singular nouns).
* URLs MUST use HTTP verbs (GET, POST, PUT, DELETE) to operate on collections and elements.
* URLs SHOULD NOT go deeper than resource/:id/resource/:id.
* API version SHOULD BE specified in headers as specified below.  However, the version number MAY be placed in the URL base as an alternative.
    * https://api.ncsu.edu/v1/resource/:id
* Requests SHOULD specify expected content-type in headers as specified below.  However, requests MAY specify content-type as an extension.
* URLs MAY specify resource filters in the query string using a comma separated list.

### Good URL examples

* List of Students:
    * GET https://api.example.edu/v1/students.json

* Filtering as a query string:
    * GET https://api.example.edu/v1/students.json?major=mae
    * GET https://api.example.edu/v1/students.json?class=sophomore&major=csc,mae,ece

* A single student in XML format:
    * GET https://api.example.edu/v1/students/cacard2.xml

* All classes a student is currently taking:
    * GET https://api.example.edu/v1/students/cacard2/classes.json
     
* Specify optional fields in a comma separated list:
    * GET https://api.example.edu/v1/students.json?filter=displayName,class,major

* Add a class to student roster:
    * POST https://api.example.edu/v1/students/cacard2/classes

* Drop a class from student roster:
    * DELETE https://api.example.edu/v1/students/cacard2/classes/1234

### Bad URL examples

* Non-plural noun:
    * GET https://api.example.edu/v1/student
    * GET https://api.example.edu/v1/student/1234
    * GET https://api.example.edu/v1/student/cacard2/class

* Verb in URL, Unsafe GET request:
    * GET https://api.example.edu/v1/students/cacard2/create

* Filter outside of query string
    * GET https://api.example.edu/v1/students/sophomore/csc

## API Versioning

* APIs SHOULD process version numbers through the "Accept" header.  
* APIs MAY use the URL to specify version at the base of the URL.
* APIs MUST NOT accept any requests that do not specify a version number.
* Developers MUST NOT release an API without a version number.
* API version numbers MUST adhere to [SEMVER].

### API Version in Headers (Preferred)

@todo
[#8](https://github.ncsu.edu/ncsu-interop-group/ws-standards/issues/8)

### API Versioning in URL

@todo
[#8](https://github.ncsu.edu/ncsu-interop-group/ws-standards/issues/8)

## API Requests

### Use HTTP Methods to Act on Resources

The HTTP verbs satisfy much of the "Uniform Interface" constraint set in [WSSR-0] and provide a set of standardized actions
that can act on noun-based resources.  The most commonly used HTTP verbs are POST, GET, PUT, and DELETE.  These four verbs/methods
form what is known as CRUD (Create, Read, Updated, and Delete) operations.

The table below represents constraints APIs MUST adhere to when implementing service resources that HTTP verbs/methods act on:

| HTTP Verb / Method | Description of Use | Safe | Idempotent |
| ---------- | ---------- | ---------- | ---------- |
| GET | The GET method is used to read a representation of a resource. | Y | Y |
| PUT | The PUT method is used for resource updates.  PUT-ing to a resource with the request body containing the newly-updated representation of the resource should update that resource.  PUT MUST NOT be used to create new resources. | N | Y |
| POST | The POST method is used for creation of new resources, particularly subordinate resources. | N | N |
| DELETE | The DELETE method is used for deletion of a particular resource.  | N | ? |

DELETE verbs/methods are idempotent according to the HTTP spec.  If you DELETE a resource, it's removed and will be removed on the next DELETE call.  In the
realm of web services, however, repeated calls to DELETE a resource may result in a 404 (Not Found) if resources are hard-deleted rather than soft.  This is an
accepted compromise of the HTTP specification because certain applications will warrant hard deletion of resources while others may not.

### Safe and Idempotent

Taken from [www.w3.org](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html)

>9.1 Safe and Idempotent Methods
>
>9.1.1 Safe Methods
>
>Implementors should be aware that the software represents the user in their interactions over the Internet, and should be careful to allow the user to be aware of any actions they might take which may have an unexpected significance to themselves or others.
>In particular, the convention has been established that the GET and HEAD methods SHOULD NOT have the significance of taking an action other than retrieval. These methods ought to be considered "safe". This allows user agents to represent other methods, such as POST, PUT and DELETE, in a special way, so that the user is made aware of the fact that a possibly unsafe action is being requested.
>Naturally, it is not possible to ensure that the server does not generate side-effects as a result of performing a GET request; in fact, some dynamic resources consider that a feature. The important distinction here is that the user did not request the side-effects, so therefore cannot be held accountable for them.
>
>9.1.2 Idempotent Methods
>
>Methods can also have the property of "idempotence" in that (aside from error or expiration issues) the side-effects of N > 0 identical requests is the same as for a single request. The methods GET, HEAD, PUT and DELETE share this property. Also, the methods OPTIONS and TRACE SHOULD NOT have side effects, and so are inherently idempotent.
>However, it is possible that a sequence of several requests is non- idempotent, even if all of the methods executed in that sequence are idempotent. (A sequence is idempotent if a single execution of the entire sequence always yields a result that is not changed by a reexecution of all, or part, of that sequence.) For example, a sequence is non-idempotent if its result depends on a value that is later modified in the same sequence.
>A sequence that never has side effects is idempotent, by definition (provided that no concurrent operations are being executed on the same set of resources).

### Request Body

[Open for Discussion](https://github.ncsu.edu/ncsu-interop-group/ws-standards/issues/6)

#### Specifying Record Limits

* If no limit is specified, an API MUST return results with a default limit.
* To get records 50 through 75 do this:
    * https://api.ncsu.edu/students.json?limit=25&offset=50
    * offset=50 means, ‘begin with record number fifty’
    * limit=25 means, ‘return 25 records’
* Information about record limits SHOULD also be included in the Example response. 
 
Example:

    {
        "metadata": {
            "resultset": {
                "count": 50,
                "offset": 25,
                "limit": 25
            }
        },
        "results": [
            { .. }
        ]
    }

## API Responses

### Relevant HTTP Response / Status Codes

#### Success
200 - OK

201 - Created

204 - No Content

#### Request Errors
400 - Bad Request

401 - Unauthorized

403 - Forbidden

404 - Not Found

405 - HTTP Method Not Allowed

#### Server Errors
500 - Internal Server Error

[Open for Discussion](https://github.ncsu.edu/ncsu-interop-group/ws-standards/issues/7)

### Response Body

* Values MUST not be included inside keys
* Keys SHOULD NOT use internal-specific names (e.g. "node" and "taxonomy term")
* Metadata SHOULD only contain direct properties of the response set and SHOULD NOT include properties of the members of the response set

#### Good examples

No values in keys:

    "tags": [
        {
            "id": "125",
            "name": "Environment"
        },
        {
            "id": "834",
            "name": "Water Quality"
        }
    ]


#### Bad examples

Values in keys:

    "tags": [
        {
            "125": "Environment"
        },
        {
            "834": "Water Quality"
        }
    ],

### Handling Errors

Error responses should include a common HTTP status code, message for the developer, message for the end-user (when appropriate), internal error code (corresponding to some specific internally determined error number), links where developers can find more info. For example:

    {
        "status" : "400",
        "developerMessage" : "Verbose, plain language description of the problem. Provide developers suggestions about how to solve their problems here",
        "userMessage" : "This is a message that can be passed along to end-users, if needed.",
        "errorCode" : "444444",
        "moreInfo" : "Could include links to API documentation for resource request, etc",
    }

## Request & Response Examples

### API Resources

  - [GET /students](#get-magazines)
  - [GET /students/:id](#get-studentsid)
  - [POST /students/:id/classes](#post-studentsidclasses)

### GET /students

Example: https://api.ncsu.edu/v1/students.json

    {
        "metadata": {
            "resultset": {
                "count": 43283,
                "offset": 0,
                "limit": 10
            }
        },
        "results": [
            {
                "id": "000000000",
                "uid": "cacard",
                "displayName": "Carrie A. Card"
            },
            {
                "id": "000000001",
                "uid": "cacard2",
                "displayName": "Carrie A. Card"
            },
            { ... } x 8
        ]
    }

### GET /students/:id

Example: https://api.ncsu.edu/v1/students/cacard.json

    {
        "id": "000000000",
        "uid": "cacard",
        "displayName": "Carrie A. Card"
    }
