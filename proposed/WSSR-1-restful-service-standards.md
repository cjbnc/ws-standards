# Basic RESTful Service Standards (DRAFT)

The intent of this document is to provide guidance on the implementation of RESTful web services according to the constraints set forth in [WSSR-0].
It does so by enumerating a distributed set of rules, expectations, and best practices about how to build and maintain RESTful web services.

The scope of this web services standards recommendation is to:
* Specify basic requirements to fulfill RESTful constraints
* Specify requirements for service endpoint URL construction, resource naming, and methods for operating on resources.
* Specify requirements for requests made to services.
* Specify requirements for responses given to clients of services.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY",
and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119][].

[RFC 2119]: http://www.ietf.org/rfc/rfc2119.txt
[WSSR-0]: https://github.ncsu.edu/ncsu-interop-group/ws-standards/blob/master/proposed/WSSR-0-restful-constraints.md
[SEMVER]: http://semver.org/spec/v2.0.0-rc.2.html

## Overview

* All services MUST adhere to the constraints set in [WSSR-0][].
* [RESTful URLs and Actions](#restful-urls-and-actions)
    * [Resource Naming](#resource-naming)
    * [HTTP Methods](#http-methods--verbs)
        * [Safe and Idempotent Methods](#safe-and-idempotent-http-methods)
        * [Use HTTP Methods to Act on Resources](#use-http-methods-to-act-on-resources)
        * [Mapping Relationships Between Resources](#relationships-between-resources)
        * [Custom Actions Outside of CRUD Operations](#custom-actions-outside-of-crud-operations)
* [API Requests](#api-requests)
    * [Filtering, Sorting, Searching Results, and Limiting Fields Returned](#filtering-sorting-searching-results-and-limiting-fields-returned)
    * [Specifying Record Limits](#specifying-record-limits)
    * [Specifying API Version](#specifying-api-version)
    * [Specifying Media Type](#specifying-media-type)
    * [Request Body](#request-body)
* [API Responses](#api-responses)
    * [Use HTTP Response / Status Codes](#use-http-response--status-codes)
    * [Response Body](#response-body)

## RESTful URLs and Actions

The constraints of [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) describe the segregation of logical resources within a web service.  These resources are
acted upon using HTTP requests using the standard HTTP methods / verbs: GET, POST, PUT, PATCH, and DELETE.

### Resource Naming

* Resource names MUST be represented as nouns.
* Resource names MUST use their plural form.
* Resource fields MUST use ```snake_case``` in all RESTful interactions. **Note: This excludes libraries that consume the service.  Libraries are subject to the coding standards for the language they're written in.**
* There is no requirement for resources to map one-to-one with underlying models.  The idea is to abstract away technical detail from the API consumer.

### HTTP Methods / Verbs

* HTTP Methods MUST be used to operate on collections and elements in the API.
* API MUST NOT include verbs as part of the URL with a few exceptions detailed [below](#custom-actions-outside-of-crud-operations).

#### Safe and Idempotent HTTP Methods

This section is included for completion as a definition of Safe and Idempotent methods, as defined by W3.  These terms complement intended behavior of certain HTTP methods.

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

#### Use HTTP Methods to Act on Resources

The HTTP verbs satisfy much of the "Uniform Interface" constraint set in [WSSR-0] and provide a set of standardized actions
that can act on noun-based resources.  The most commonly used HTTP verbs are POST, GET, PUT, and DELETE.  These four verbs/methods
form what is known as CRUD (Create, Read, Update, and Delete) operations.  PATCH is an additional HTTP verb used for partial updates and
is defined below.

The table below represents constraints APIs MUST adhere to when implementing service resources that HTTP verbs/methods act on:

| HTTP Verb / Method | Description of Use | Safe | Idempotent |
| ---------- | ---------- | ---------- | ---------- |
| GET | The GET method is used to read a representation of a resource. | Y | Y |
| POST | The POST method is used for creation of new resources, particularly subordinate resources. | N | N |
| PUT | The PUT method is used for resource updates.  PUT-ing to a resource with the request body containing the newly-updated representation of the resource should update that resource.  PUT SHOULD NOT be used to create new resources. | N | Y |
| DELETE | The DELETE method is used for deletion of a particular resource.  | N | ? |

##### Example

The examples below demonstrate using HTTP verbs to operate on web service resources to perform basic CRUD operations.

* ```GET /students``` - Retrieves a list of student elements
* ```GET /students/cacard``` - Retrieves a single student element
* ```POST /students``` - Creates a new student
* ```PUT /students/cacard``` - Updates student "cacard"
* ```DELETE /students/cacard``` - Deletes student "cacard"

#### Relationships Between Resources

There will be times when you need to be able to easily provide a list of "sub-resources" or "resources owned by another resource".  Relationships
can be mapped using the URL with certain constraints:

* URL depth MUST NOT exceed /resource/:id/resource/:id

##### Example

* ```GET /students/cacard/classes``` - Retrieves a list of classes for student "cacard"
* ```GET /students/cacard/classes/csc316``` - Retrieves a specific class student "cacard" is enrolled in
* ```POST /students/cacard/classes``` - Enrolls student "cacard" in a class
* ```DELETE /students/cacard/classes/csc316``` - Removes student "cacard" from a specific class

#### Custom Actions Outside of CRUD Operations

There will be use cases for actions that need to be performed on resources outside of typical CRUD operations.  These usually
include toggling attributes of a resource.  In those cases, use best judgement and document your decision in conjunction with
considering the following constraints:

* It is RECOMMENDED that custom actions be implemented as a property of a resource and modified through a PUT or PATCH HTTP method (e.g.
activating a resource MAY be implemented by setting a property named "activated" on the resource model)
* However, custom actions MAY be implemented as a "sub-resource" sparingly and with appropriate documentation (e.g. GitHub's API
allows a gist to be starred with ```PUT /gists/:id/star``` and unstarred with ```DELETE /gists/:id/star```.

## API Requests

### Filtering, Sorting, Searching Results, and Limiting Fields Returned

#### Filtering

* APIs MUST use a unique query parameter for every resource field that implements filtering.
* Filters MUST be mapped to the query string and MUST NOT be included in the resource mapping.
* Multiple filters on one resource field MUST be separated by comma.

##### Examples

* ```GET students?major=mae``` - Retrieves a list of MAE students.
* ```GET students?major=mae,csc``` - Retrieves a list of MAE and CSC students.

#### Sorting

* APIs MUST use a generic query parameter, ```sort```, to describe sorting rules.
* APIs MUST allow for a comma separated list of resource fields to sort, each with a unary negative, per field, to specify descending order.
* APIs MUST process sorted fields from left to right.

##### Examples

* ```GET students?sort=last_name,first_name``` - Get list of students sorted by last name, ascending, and first name, ascending.
* ```GET errors?sort=-created_date``` - Get list of most recent errors.

#### Searching Results

* Full text searches, where implemented, MUST be implemented through a generic query parameter, ```q```.

##### Example

* ```GET errors?q=php``` - Get list of errors mentioning the keyword "php".

#### Limiting Fields Returned

* APIs that allow for limiting fields returned for a resource MUST implement that functionality as a generic query parameter, ```fields```.
* Multiple fields MUST be specified as a comma-separated list.

##### Example

* ```GET students?fields=unity,first_name,last_name``` - Get list of students, but only include Unity ID, first and last name.

### Specifying Record Limits


### Specifying API Version

* APIs MUST be versioned.
* API version numbers MUST adhere to [SEMVER].
* APIs SHOULD include version numbers in request headers as specified [below](#using-the-accept-header).
* APIs MAY use the URL to specify version at the base of the URL as specified [below](#using-the-url).
* APIs MUST NOT accept any requests that do not specify a version number.

#### Using the Accept Header (Preferred)

The following constraints are placed on APIs who implement "Specifying API Version Using the Accept Header":

* API version MUST be included as part of the Accept header as specified below.
* API version MUST be included as part of the Content-Type header in all responses as specified below.

In the example below, notice that request response content type is specified as JSON in the Accept header as well.  This is documented at [Specifying Media Type](#specifying-media-type).

```
===>
GET /students/cacard HTTP/1.1
Accept: application/vendor.api-v1+json

<===
HTTP/1.1 200 OK
Content-Type: application/vendor.api-v1+json

...
```

#### Using the URL

While it is STRONGLY advised to use request headers for specifying version, developers are allowed to use the URL to specify version.

* If version is included in the URL, it MUST be placed at the base.
* URL Aliasing SHOULD be done if possible (e.g. v1.0 could alias to v1).  

##### Examples

* https://api.ncsu.edu/v1/resource/:id
* https://api.ncsu.edu/v1.1/resource/:id
* https://api.ncsu.edu/v2/resource/:id

### Specifying Media Type

* APIs MUST document supported media types for both requests and responses.
* API requests SHOULD specify media type as part of the URL through an appropriate "file extension".
* API requests MAY specify media type in addition or solely in the Accept header for the request.
* API responses MUST include requested media type in the Content-Type header.

#### Using the URL

* APIs that specify media type in the URL MUST do so as an extension appended to the URL before query string arguments.
* Extensions in the URL MUST only specify requested response media type.

##### Examples

* https://api.ncsu.edu/v1/resource.json
* https://api.ncsu.edu/v1/resource/:id.xml

#### Using the Header

* APIs that specify media type in the header MUST do so in the Accept header.

In the example below, both the requested API version and media type are included in the Accept header and the same information is included in the response.

```
===>
GET /students/cacard HTTP/1.1
Accept: application/vendor.api-v1+json

<===
HTTP/1.1 200 OK
Content-Type: application/vendor.api-v1+json

...
```

#### Conflicting URL / Header Content-types

* Requests that specify conflicting content types in the URL and the Accept header MUST reject the request with a series 4xx Bad Request response (e.g. 406 Not Acceptable)

### Request Body

## API Responses

### Use HTTP Response / Status Codes

* 200 OK - Response to a successful GET, PUT, PATCH or DELETE.  Can also be used for a POST that doesn't result in a creation.
* 201 Created - Response to a POST that results in a creation.
* 202 Accepted - The request has been accepted for processing, but the processing has not been completed.
* 204 No Content - Response to a successful request that won't be returning a response body.
* 304 Not Modified - Used for HTTP caching.
* 400 Bad Request - Request body isn't parse-able.
* 401 Unauthorized - When no or invalid authentication details are given.
* 402 Payment Required - Highly Specialized, Reserved
* 403 Forbidden - Authentication succeeded but user is not authorized to use requested resource.
* 404 Not Found - Response to a request for a resource that doesn't exist.
* 405 Method Not Allowed - Used when an HTTP method is not allowed for an authenticated user (can GET a resource, but not POST/PUT)
* 410 Gone - Indicates the resource is no longer available (i.e. removed after deprecation)
* 415 Unsupported Media Type - If a content type is requested that is not supported by the service (e.g. requests XML from a JSON-only service)
* 422 Unprocessable Entity - Used for validation errors
* 429 Too Many Requests - Rate Limit reached
* 500 Internal Server Error
* 501 Not Implemented - The service does not support the functionality required to fulfill the request.
* 502 Bad Gateway - This could be used to highlight unexpected errors as requests pass through abstraction layers (e.g. authentication => authorization => rate=limiters => resource endpoint)
* 503 Service Unavailable - Oh snap the world is ending.

### Response Body

#### Handling Errors

Error responses SHOULD include a common HTTP status code, message for the developer, and a message for the end-user (when appropriate). For example:

```
{
    "success": false,
    "status": 404,
    "user_message": "Plugin not found.",
    "developer_message": "Plugin not found."
}
```
