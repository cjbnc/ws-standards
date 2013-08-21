## RESTful Constraints

There are six constraints that characterise a RESTful Web Service. Only one of these constraints is optional ("code on
demand").  If a service violates any required constraint, it cannot be considered RESTful.

### Client-Server

A uniform interface separates clients from servers. This separation of concerns means that, for example, clients
are not concerned with data storage, which remains internal to each server, so that the portability of client code
is improved. Servers are not concerned with the user interface or user state, so that servers can be simpler and more
scalable. Servers and clients may also be replaced and developed independently, as long as the interface between
them is not altered.

### Stateless

The client–server communication is further constrained by no client context being stored on the server between
requests. Each request from any client contains all of the information necessary to service the request, and any
session state is held in the client.

### Cacheable

As on the World Wide Web, clients can cache responses. Responses must therefore, implicitly or explicitly, define
themselves as cacheable, or not, to prevent clients reusing stale or inappropriate data in response to further requests.
Well-managed caching partially or completely eliminates some client–server interactions, further improving scalability
and performance.

### Layered System

A client cannot ordinarily tell whether it is connected directly to the end server, or to an intermediary along
the way. Intermediary servers may improve system scalability by enabling load-balancing and by providing shared
caches. They may also enforce security policies.

### Code on Demand (optional)

Servers can temporarily extend or customize the functionality of a client by the transfer of executable code.
Examples of this may include compiled components such as Java applets and client-side scripts such as JavaScript.

### Uniform Interface

The uniform interface between clients and servers simplifies and decouples the architecture,
which enables each part to evolve independently.
