# Web Mechanics, Architecture & Network Fundamentals  
## Part 0 — Introduction to the Series

### Demystifying the “Black Box” Behind Modern Web Applications

When you open a website, tap a button, submit a form, or watch a video, the experience often feels instantaneous and self-contained.

You see:

- A page appear in your browser
- Images load
- Menus respond to clicks
- Search results update
- A login form accepts your credentials
- A shopping cart remembers your items
- Messages travel between users
- Data appears to come from “somewhere”

But behind every visible interaction is a chain of systems working together:

1. Your device runs client-side software.
2. The browser interprets HTML, CSS, and JavaScript.
3. A network connection carries messages between machines.
4. DNS translates human-readable names into addresses.
5. HTTP defines how requests and responses are structured.
6. Servers execute business logic.
7. Databases store and retrieve information.
8. Authentication systems verify identity.
9. APIs allow separate components to communicate.
10. Data centers, routers, CDNs, and physical cables move information across the world.

This series explains those systems from the ground up.

The goal is not merely to memorize technical vocabulary. The goal is to develop a reliable mental model of what happens when software communicates across a network.

---

# 1. What This Series Is About

This series is an introduction to the mechanics and architecture of web applications.

It focuses on questions such as:

- What happens when I type a URL into a browser?
- Where does a website actually run?
- What is the difference between frontend and backend code?
- How does a browser find a server?
- What is DNS?
- What is an IP address?
- How does HTTP carry information?
- What is the difference between a request and a response?
- Why are some URLs secure and others not?
- How does a login request work?
- How do web applications exchange data?
- What makes an API different from a regular webpage?
- How can I inspect network traffic?
- What do status codes such as `404`, `401`, and `500` actually mean?
- Why does a website sometimes load quickly in one location and slowly in another?
- What happens when a web application uses a database?
- How do modern applications combine frontend and backend code?

The series is intentionally designed to come before deep framework-specific programming.

You do not need to begin with React, Vue, Angular, Node.js, Django, Laravel, Rails, or another framework to understand the web.

Frameworks are useful tools, but the underlying principles remain relatively stable:

- A client sends messages.
- A server receives and processes them.
- Data travels through networks.
- Protocols define communication rules.
- The application maintains boundaries between trusted and untrusted systems.

Once these ideas are clear, learning frameworks becomes much easier.

---

# 2. The Central Mental Model

A web application can be understood as a group of cooperating layers.

A simplified model looks like this:

```text
User
  ↓
Browser or mobile application
  ↓
Network connection
  ↓
DNS and IP addressing
  ↓
HTTP or another application protocol
  ↓
Server-side application
  ↓
Database, file storage, and external services
```

For example, suppose you visit:

```text
https://example.com/products
```

A simplified sequence might be:

```text
1. The browser interprets the URL.
2. The browser determines the server's IP address.
3. A secure connection is established.
4. The browser sends an HTTP request.
5. The server receives the request.
6. The server identifies the requested resource.
7. The application may query a database.
8. The server generates a response.
9. The response travels back to the browser.
10. The browser renders the result.
```

That is the basic web request cycle.

Real applications may add:

- Browser caches
- DNS caches
- CDNs
- Load balancers
- Reverse proxies
- Authentication systems
- Application servers
- Databases
- Message queues
- Object storage
- Monitoring systems
- Third-party APIs
- Security filters
- Multiple regions and data centers

The series gradually introduces these pieces instead of treating them as one enormous system.

---

# 3. Why Web Fundamentals Matter

Modern development often begins with tools.

A beginner may immediately encounter:

- A package manager
- A JavaScript framework
- A build tool
- A component system
- A routing library
- An API client
- A database abstraction layer
- A cloud deployment platform
- Environment variables
- Authentication providers

These tools solve real problems, but they can hide important concepts.

For example, a framework may make it easy to fetch data:

```javascript
fetch("/api/products")
```

But this line raises many questions:

- What is `/api/products`?
- Is it a URL or a file path?
- What protocol is being used?
- Which server receives the request?
- What HTTP method is used?
- What data is sent?
- What does the server return?
- How does the browser know whether the request succeeded?
- Where does the server get the product data?
- What happens if the user is not authenticated?
- How can the request be inspected?
- What happens between the browser and the server?

If you understand the underlying mechanics, the framework abstraction becomes useful rather than mysterious.

Without the fundamentals, developers often respond to problems by trying random fixes:

- Changing frontend code when the server is failing
- Debugging the database when DNS is broken
- Treating a `401` as a server crash
- Treating a `404` as a network outage
- Exposing secrets in client-side code
- Assuming HTTPS means the entire application is automatically secure
- Assuming a successful HTTP response means the operation itself succeeded

This series is meant to replace guesswork with structured reasoning.

---

# 4. What You Will Learn

By the end of the series, you should be able to explain:

## Architecture

- What frontend code does
- What backend code does
- Why browsers are considered untrusted clients
- Where authentication and authorization belong
- Why databases should not normally be exposed directly to browsers
- How static sites, SPAs, SSR applications, and full-stack frameworks differ

## Networking

- The difference between the Internet and the Web
- What IP addresses are
- What DNS does
- How a domain name becomes an IP address
- What routers, data centers, and CDNs do
- Why distance and network topology affect performance

## HTTP and HTTPS

- How requests and responses are structured
- What HTTP methods mean
- How headers and bodies are used
- What status codes communicate
- How URLs are organized
- What TLS protects
- The difference between encryption, authentication, and integrity

## APIs

- What an API is
- What REST means
- How resources and HTTP verbs work together
- How GraphQL differs from REST
- How RPC systems differ from resource-oriented APIs
- How JSON, XML, and multipart form data represent information

## Diagnostics

- How to inspect browser traffic
- How to identify failed requests
- How to read request and response headers
- How to understand timing information
- How to test APIs using cURL, Postman, or Bruno
- How to separate frontend, backend, network, and infrastructure problems

---

# 5. What You Do Not Need Before Starting

This series is beginner-friendly.

You do not need to already know:

- Advanced programming
- A frontend framework
- A backend framework
- Database administration
- Cloud infrastructure
- Networking certification material
- Operating system internals
- Cryptographic mathematics
- Linux administration
- Professional DevOps practices

Some programming familiarity will help, but most topics can be understood conceptually before writing code.

You should be comfortable with basic ideas such as:

- Files and folders
- Web browsers
- URLs
- Copying commands into a terminal
- Simple data structures such as strings, numbers, lists, and objects

The series will introduce technical terms as they become relevant.

---

# 6. How to Use This Series

Each part is designed to build on the previous parts.

A recommended learning process is:

## Step 1: Read for the big picture

Do not try to memorize everything on the first pass.

Ask:

> What problem is this technology solving?

For example:

- DNS solves the problem of locating services by name.
- HTTP solves the problem of structured web communication.
- TLS solves the problem of protecting communication.
- APIs solve the problem of structured interaction between software systems.
- CDNs solve the problem of delivering content efficiently around the world.

## Step 2: Build a vocabulary

Technical concepts become easier when their names are familiar.

Important terms will be defined carefully, including:

- Client
- Server
- Host
- Domain
- IP address
- Port
- Protocol
- Request
- Response
- Header
- Payload
- Resource
- Endpoint
- Route
- Cache
- Session
- Token
- Database
- CDN

## Step 3: Trace complete examples

Instead of learning isolated definitions, follow an interaction from beginning to end.

For example:

```text
User clicks “Log in”
  ↓
Browser collects form values
  ↓
Browser sends an HTTPS request
  ↓
Server validates the request
  ↓
Server checks stored user data
  ↓
Server creates an authenticated session
  ↓
Server sends a response
  ↓
Browser updates the interface
```

This kind of trace is more useful than memorizing the definition of “HTTP request” separately from “authentication.”

## Step 4: Inspect real traffic

The browser already gives you tools for observing web communication.

You will learn to inspect:

- Request URLs
- HTTP methods
- Status codes
- Headers
- Request bodies
- Response bodies
- Timing phases
- Cookies
- Redirects
- Failed requests

The goal is to turn the browser from a passive viewing tool into an investigation tool.

## Step 5: Explain concepts in your own words

A useful test is whether you can explain a concept without repeating a textbook definition.

For example:

> DNS is like a distributed directory that helps software find the network address associated with a human-readable domain name.

That is more useful than memorizing:

> DNS is a hierarchical decentralized naming system.

The formal definition is valuable, but understanding comes first.

---

# 7. A Running Example: Opening an Online Store

To make the series concrete, we will repeatedly refer to an imaginary online store.

Suppose a user visits:

```text
https://shop.example/products
```

They then:

1. View the product list
2. Open a product
3. Add it to a cart
4. Log in
5. Submit an order

Behind the scenes, different actions may involve different systems.

## Viewing the product list

The browser may request:

```text
GET /products
```

The server may:

- Locate the requested route
- Query a product database
- Render HTML
- Return CSS and JavaScript
- Return product images from a CDN

## Opening a product

The browser may request:

```text
GET /api/products/123
```

The server may:

- Read the product identifier `123`
- Query the database
- Confirm the product is public
- Return JSON data

Example response:

```json
{
  "id": 123,
  "name": "Example Keyboard",
  "price": 79.99,
  "available": true
}
```

## Adding an item to a cart

The browser may send:

```text
POST /api/cart/items
```

With a request body such as:

```json
{
  "productId": 123,
  "quantity": 1
}
```

The server may:

- Verify the user’s session
- Check whether the product exists
- Check inventory
- Update cart data
- Return the new cart state

## Placing an order

The server may additionally communicate with:

- A payment provider
- An inventory system
- An email service
- A tax calculation service
- A shipping provider

The browser does not need to know every internal step. It communicates with the application through a defined boundary.

That boundary is one of the most important ideas in web architecture.

---

# 8. The Boundary Between Client and Server

A web application is usually divided into at least two major environments.

## The client

The client is the software interacting with the user.

Common clients include:

- Web browsers
- Mobile applications
- Desktop applications
- Command-line programs
- Other servers

In a browser-based application, the client usually handles:

- Rendering visible interfaces
- Responding to user actions
- Managing temporary interface state
- Performing basic input validation
- Sending requests
- Displaying responses
- Storing limited local information

## The server

The server is a computer or software process that receives requests and performs operations on behalf of clients.

It commonly handles:

- Business rules
- Authentication
- Authorization
- Database access
- Payment processing
- File handling
- Sensitive calculations
- Access to private services
- Coordination with external systems

## The important security rule

Anything running on the client should be treated as observable and modifiable by the user.

A user can often:

- Inspect client-side code
- Modify JavaScript
- Change form fields
- Replay requests
- Send requests without using the interface
- Disable browser checks
- Alter local storage
- Manipulate visible interface state

Therefore:

> Client-side validation improves user experience, but server-side validation enforces security and correctness.

For example, a browser may prevent a user from entering a negative quantity. A malicious user can still manually send:

```json
{
  "productId": 123,
  "quantity": -100
}
```

The server must validate the request independently.

---

# 9. The Web as a Conversation

A useful way to understand web communication is to imagine a structured conversation.

The client says:

> I want this resource.

The server responds:

> Here is the result.

A more detailed conversation could be:

```text
Client:
  Method: GET
  Path: /products/123
  Accept: application/json

Server:
  Status: 200 OK
  Content-Type: application/json
  Body: product data
```

Or:

```text
Client:
  Method: POST
  Path: /orders
  Content-Type: application/json
  Body: order information

Server:
  Status: 201 Created
  Content-Type: application/json
  Body: newly created order
```

Or:

```text
Client:
  Method: GET
  Path: /account

Server:
  Status: 401 Unauthorized
  Body: authentication required
```

HTTP gives both sides a common language.

The client does not need to know exactly how the server is implemented internally. The server does not need to know how the client visually renders the response.

They coordinate through an agreed contract.

---

# 10. The Layers of a Web Interaction

When something goes wrong, it helps to classify the problem by layer.

Consider the following simplified stack:

```text
Layer 1: User interaction
Layer 2: Browser or application code
Layer 3: HTTP request and response
Layer 4: TLS encryption
Layer 5: TCP or QUIC transport
Layer 6: IP routing
Layer 7: DNS resolution
Layer 8: Physical network infrastructure
Layer 9: Server application
Layer 10: Database and external services
```

A problem at one layer may look similar to a problem at another layer.

For example, a page that does not load could be caused by:

- A broken browser script
- An invalid URL
- A DNS failure
- A TLS certificate problem
- A server returning `500`
- A database timeout
- A blocked firewall port
- A failed third-party service
- A local network outage

Good debugging begins by identifying which layer is failing.

---

# 11. The Difference Between the Internet and the Web

The Internet and the Web are related but not identical.

## The Internet

The Internet is the global network infrastructure that connects devices and networks.

It includes:

- Fiber-optic cables
- Wireless links
- Routers
- Network providers
- Data centers
- IP addressing
- Routing systems
- Transport protocols

The Internet is the underlying communication infrastructure.

## The Web

The World Wide Web is an application system built on top of the Internet.

It includes:

- Web browsers
- Websites
- Web servers
- URLs
- HTTP and HTTPS
- HTML
- CSS
- JavaScript
- Hyperlinks
- Web APIs

A useful analogy:

```text
Internet = roads, vehicles, intersections, and traffic systems
Web = one category of service using those roads
```

Other systems also use the Internet, including:

- Email
- Video calls
- Online games
- File transfer
- Remote administration
- Streaming systems
- Voice communication

The Web is therefore one important application of the Internet, not a synonym for the entire Internet.

---

# 12. Why Protocols Matter

A protocol is an agreed set of rules for communication.

People can communicate because they share conventions such as language, grammar, and turn-taking.

Computers need formal rules.

A protocol may define:

- How a message begins
- How the destination is identified
- How data is represented
- How errors are reported
- How the receiver responds
- How security is established
- How communication ends

Examples include:

- DNS for name resolution
- IP for addressing and routing
- TCP for reliable transport
- QUIC for modern transport over UDP
- TLS for secure communication
- HTTP for web requests and responses

A protocol does not necessarily describe the entire application. It defines a particular part of the communication process.

---

# 13. Application State and the Stateless Web

A common source of confusion is the idea of state.

## State means remembered information

Examples include:

- Whether a user is logged in
- Items in a shopping cart
- The current page number
- A selected theme
- A partially completed form
- A server-side order status

HTTP itself is generally described as stateless.

That means each request should contain enough information for the server to understand it without depending entirely on a previous request.

For example:

```text
Request 1: GET /products
Request 2: GET /products/123
Request 3: POST /cart/items
```

The protocol does not automatically assume that request 3 is connected to request 1.

Applications build continuity using mechanisms such as:

- Cookies
- Sessions
- Access tokens
- Refresh tokens
- Database records
- Client-side state
- Server-side caches

This distinction is important:

```text
HTTP communication can be stateless
while the application still manages state.
```

---

# 14. Trust, Identity, and Permissions

Web architecture involves multiple security questions.

## Authentication

Authentication asks:

> Who are you?

Examples:

- Username and password
- Passkeys
- One-time codes
- Security keys
- Social login

## Authorization

Authorization asks:

> What are you allowed to do?

A user may be authenticated but still not authorized to:

- View another user’s private records
- Delete an organization
- Access administrative tools
- Refund a payment
- Change billing settings

## Encryption

Encryption asks:

> Can unauthorized parties read the communication?

HTTPS helps protect data while it travels between the client and server.

## Integrity

Integrity asks:

> Was the data changed during transmission?

Secure transport helps detect unauthorized modification.

These are different concerns:

```text
Authentication = identity
Authorization = permission
Encryption = confidentiality
Integrity = protection against undetected modification
```

A system can succeed at one and fail at another.

For example:

- HTTPS may encrypt a request.
- The application may still incorrectly authorize the user.
- A valid login may still grant too many permissions.
- A client-side interface may hide an admin button while the server still fails to enforce the restriction.

---

# 15. The Evolution of Web Application Architecture

Web architecture has changed over time.

## Static sites

A static site serves files that already exist.

Typical assets include:

- HTML
- CSS
- JavaScript
- Images
- Fonts
- Videos

The server often returns these files as-is.

Advantages:

- Simple deployment
- High reliability
- Good caching
- Strong performance
- Few moving parts

Limitations:

- Dynamic behavior may require JavaScript
- Personalized content is more difficult
- Data-driven features require external services or build steps

## Server-rendered websites

The server generates HTML for each request.

For example:

```text
Browser requests /products
  ↓
Server queries database
  ↓
Server generates HTML
  ↓
Browser displays the page
```

Advantages:

- Useful initial page content
- Often good for search engines
- Less client-side JavaScript required
- Server controls more of the rendering process

## Single-page applications

A single-page application often loads a substantial JavaScript application once and then communicates with APIs.

Typical flow:

```text
Browser loads application shell
  ↓
JavaScript starts
  ↓
JavaScript requests data from APIs
  ↓
The interface updates without full page reloads
```

Advantages:

- Rich interactive experiences
- Smooth navigation
- Client-side responsiveness
- Clear separation between UI and APIs

Tradeoffs:

- More client-side complexity
- More JavaScript to download and execute
- Initial loading may be slower
- Security boundaries can be misunderstood

## Server-side rendering and hybrid rendering

Modern frameworks may combine several approaches.

A single route may:

- Render HTML on the server
- Add client-side interactivity
- Fetch some data on the server
- Fetch other data in the browser
- Cache content at the edge
- Revalidate content periodically

This is why modern architecture is often hybrid rather than purely “frontend” or “backend.”

---

# 16. The Full-Stack Contract

The frontend and backend need a clear agreement.

This agreement may define:

- Available URLs
- HTTP methods
- Required headers
- Authentication requirements
- Request body formats
- Response formats
- Error behavior
- Pagination rules
- Rate limits
- Versioning
- Data validation rules

For example:

```text
Endpoint: POST /api/orders
Authentication: required
Content-Type: application/json
Request body:
{
  "items": [
    {
      "productId": 123,
      "quantity": 2
    }
  ]
}
```

Possible responses:

```text
201 Created
```

```json
{
  "orderId": "ord_456",
  "status": "pending"
}
```

Or:

```text
400 Bad Request
```

```json
{
  "error": "Quantity must be greater than zero"
}
```

The frontend should understand how to use this contract.

The backend must still validate every request because the frontend cannot be trusted as an enforcement mechanism.

---

# 17. Common Beginner Misconceptions

## Misconception 1: A website is a single file

A modern website may involve:

- Multiple HTML documents
- JavaScript bundles
- CSS files
- Image servers
- API servers
- Databases
- Authentication providers
- Payment services
- CDNs
- Monitoring systems

The visible page is only one part of the system.

## Misconception 2: The browser is the application

The browser is a client environment.

It may execute a large amount of application code, but it is not automatically authoritative.

The server remains responsible for protecting sensitive operations.

## Misconception 3: Hiding a button provides security

Removing a button from the interface does not prevent a user from manually sending the corresponding request.

Security must be enforced at the server boundary.

## Misconception 4: A `404` means the Internet is broken

A `404 Not Found` generally means the server was reachable but could not find the requested resource.

That is different from:

- DNS failure
- Timeout
- Connection refusal
- TLS failure
- Server crash

## Misconception 5: A successful response means the business operation succeeded

An HTTP response may be technically successful while containing an application-level error.

For example:

```text
HTTP 200 OK
```

could contain:

```json
{
  "success": false,
  "message": "Payment declined"
}
```

This is why both transport-level and application-level results matter.

## Misconception 6: APIs are databases

An API is an interface for communication.

It may read from a database, but it can also:

- Calculate results
- Call another API
- Trigger an email
- Start a background task
- Validate permissions
- Combine information from multiple sources

## Misconception 7: HTTPS makes the application completely secure

HTTPS protects communication in transit, but it does not automatically prevent:

- Weak passwords
- Broken authorization
- SQL injection
- Cross-site scripting
- Insecure file handling
- Exposed secrets
- Vulnerable dependencies
- Business logic errors

HTTPS is essential, but it is only one part of security.

---

# 18. A Practical Debugging Mindset

When a web feature fails, avoid asking only:

> What code should I change?

Instead ask:

1. Did the user action occur?
2. Did the browser create a request?
3. Was the URL correct?
4. Did DNS resolve the domain?
5. Was a secure connection established?
6. Did the request reach the server?
7. What status code came back?
8. What did the response body say?
9. Did the server access the database?
10. Did an external service fail?
11. Did the browser correctly interpret the response?
12. Is the problem reproducible outside the interface?

This process narrows the problem systematically.

For example:

```text
No request appears in DevTools
→ likely UI or client-side code problem

Request appears, but DNS fails
→ name-resolution or network problem

Request reaches server and returns 401
→ authentication problem

Request returns 403
→ permission or authorization problem

Request returns 500
→ server-side failure

Request returns 200 but UI is wrong
→ response interpretation or rendering problem
```

The exact diagnosis depends on context, but this layered approach is powerful.

---

# 19. A Preview of the Series

## Part 1 — Deconstructing Software Architecture

You will learn:

- What frontend and backend systems do
- How browser execution works
- Why client-side code is untrusted
- How servers manage sensitive operations
- How static sites, SPAs, SSR, and full-stack frameworks differ
- How the frontend and backend coordinate through contracts

## Part 2 — How the Internet and the Web Work

You will learn:

- The difference between the Internet and the Web
- How packets travel through networks
- What IP addresses represent
- How DNS resolution works
- What routers, CDNs, and data centers do
- Why physical distance influences latency

## Part 3 — HTTP, HTTPS, and the Request-Response Cycle

You will learn:

- How URLs are structured
- How HTTP requests are formed
- What methods such as `GET`, `POST`, `PUT`, `PATCH`, and `DELETE` mean
- How headers and bodies work
- How to interpret status codes
- How TLS protects communication

## Part 4 — RESTful Services and API Paradigms

You will learn:

- What REST means
- How to design resource-oriented endpoints
- How GraphQL and RPC differ from REST
- How data is serialized
- When to use JSON, XML, or multipart form data

## Part 5 — Network Inspection and Diagnostic Workflows

You will learn:

- How to use the browser Network tab
- How to inspect request and response details
- How to read timing information
- How to test APIs with cURL, Postman, and Bruno
- How to debug network problems methodically

---

# 20. The Core Vocabulary to Remember

Before continuing, become familiar with these basic terms.

## Client

A program or device that initiates a request.

Examples:

- Browser
- Mobile app
- Desktop application
- Command-line tool

## Server

A program or computer that receives requests and provides services or data.

## Network

A system that allows devices to communicate.

## Protocol

A set of rules for communication.

## URL

A structured address identifying a resource or service.

Example:

```text
https://example.com/products?page=2
```

## Domain name

The human-readable name portion of an address.

Example:

```text
example.com
```

## IP address

A numerical network address used to identify a device or service.

## Request

A message sent by a client to a server.

## Response

A message sent by a server back to a client.

## Header

Metadata attached to a request or response.

## Body

The main data carried by a request or response.

## API

A defined interface through which software systems communicate.

## Database

A system for storing and retrieving structured information.

## Cache

A stored copy of data kept for faster reuse.

## Authentication

The process of verifying identity.

## Authorization

The process of determining permissions.

## Endpoint

A network-accessible location where a service accepts requests.

Example:

```text
/api/products
```

## Resource

A thing represented or managed by a system.

Examples:

- User
- Product
- Order
- Image
- Article

---

# 21. A Simple Exercise Before Part 1

Choose a website you use regularly and answer these questions conceptually:

1. What is the site’s domain name?
2. What visible parts are likely frontend code?
3. What operations probably require a backend?
4. What information might be stored in a database?
5. What actions require authentication?
6. What external services might be involved?
7. Which features could be delivered as static files?
8. Which features probably require API requests?
9. What could happen if the database became unavailable?
10. What could happen if the client tried to modify a request?

For an online store, your answers might look like:

```text
Frontend:
- Product cards
- Search box
- Shopping cart display
- Checkout form

Backend:
- Product availability
- Price calculation
- Order creation
- Payment verification
- User permissions

Database:
- Products
- Users
- Orders
- Inventory

External services:
- Payments
- Email
- Shipping
- Tax calculation
```

There may not be one perfect answer. The purpose is to begin seeing a website as a collection of responsibilities rather than a single mysterious object.

---

# 22. The Most Important Idea in This Series

The web is not magic.

It is a collection of systems communicating through agreed rules.

When a web application works, several boundaries align:

```text
The browser knows how to send a request.
The network knows how to deliver it.
DNS helps locate the destination.
TLS protects the connection.
HTTP describes the message.
The server knows how to process it.
The database provides stored information.
The response tells the client what happened.
The browser renders the result.
```

When something fails, one or more of those boundaries may be broken.

Learning web fundamentals means learning how to identify:

- Which component is responsible
- Which protocol is involved
- Which direction the data is traveling
- Which system owns the state
- Which side can be trusted
- What evidence can confirm the problem

That mental framework will remain useful regardless of which language, framework, hosting platform, or database you use.

---

# Part 0 Summary

In this introduction, we established that:

- Web applications are distributed systems made of many cooperating parts.
- The browser is a client, not the entire application.
- Servers handle protected logic, data, and integrations.
- Clients and servers communicate through protocols and contracts.
- The Internet provides infrastructure; the Web is an application layer built on it.
- HTTP organizes web communication into requests and responses.
- HTTPS adds secure transport through TLS.
- DNS helps convert human-readable domain names into network addresses.
- APIs provide structured communication between software systems.
- Web architectures have evolved from static sites to SPAs, SSR, and hybrid full-stack systems.
- The client must be treated as untrusted.
- Effective debugging requires identifying the failing layer.
- Understanding the fundamentals makes frameworks easier to learn.

In **Part 1**, we will examine software architecture more closely by separating frontend responsibilities from backend responsibilities and tracing how modern applications divide work between the browser and the server.
