# What is a server?

Anything that provides something is a server. In case of computers and backends,
a server is a computer that serves a particular type of resource. One that is
requested to by the other to provide something is a server. A server doesn't
have the ability to request for anything, it can only respond to incoming
requests.

## Elements of a Server

- Common medium and code for exchanging requests and responses which both
  parties can understand.
- Process requests accordingly if capable of doing it using the available
  resources and respond with the results.
  - Both internal and external resources can be used by the server to achieve
    the results that have been requested.

## What's a Protocol?

The common medium or code that the client and server uses to communicate with
each other; the contract between them to send messages that both of them can
understand is called a protocol.

- HTTP
- FTP
- Socket

## Types of Servers

Servers are used by clients through protocols or SDKs that are provided.

- Web Servers
- File Servers
- Email Servers
- Database Servers
- Game Servers
- Proxy Servers
- DNS Servers

## Resources

Every resource available to a server should have a unique identifier.

- URL: Uniform Resource Locator
- URI: Universal Resource Identifier

## Response

HTTP responses from the server contain a status code, a set of headers and optionally a content body among other things.

- Status Code
  - `200`
- Headers
  - `Content-Type application/json`
- Body (Optional)

## Request

HTTP requests from clients contain the same things as a server response except
for the status code; which is not sent by the client and instead, a request
method is sent.

- Method
  - `GET`
- Headers
  - `X-API-KEY 31709277-8077-4387-8A45-178F5B6D090F`
- Body (Optional)

## Request-Response Lifecycle

A request is sent from the client to the server with an HTTP method, headers and
optionally a body. The server receives the request and processes it accordingly.
The server then sends a response back to the client with a status code, headers
and optionally a body.

The server can take time to process the data so HTTP requests are asynchronous
in nature. The client can send multiple requests to the server and the server
will process them in the order they were received.

## HTTP Server in Node.js

```javascript
const http = require("http");

const server = http.createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "application/json" });
  res.write(JSON.stringify({ message: "Heyo! How's it going?" }));
  res.end();
});

server.listen(3000, () => {
  console.log("Server is listening on port 3000");
});
```

## References

- [Lecture Notes](https://hmnayem.notion.site/Lecture-2-04-05-2023-464dbf5851684b9fa07f8e8b0484e853)
- [Node.js HTTP Library](https://nodejs.org/api/http.html)
- [HTTP Response Status Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
- [HTTP Headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)
- [HTTP Messages](https://developer.mozilla.org/en-US/docs/Web/HTTP/Messages)
