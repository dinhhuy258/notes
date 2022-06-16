# CORS

Cross-origin resource sharing (CORS) is a browser mechanism which enables controlled access to resources located outside of a given domain.

## Same-origin policy (SOP)

The same-origin policy is a web browser security mechanism that aims to prevent websites from attacking each other.

The same-origin policy restricts scripts on one origin from accessing data from another origin.

An origin consists of a URI scheme, domain and port number.

![](../../assets/images/web/origin.png)

## Relaxation of the same-origin policy

The same-origin policy is very restrictive and consequently various approaches have been devised to circumvent the constraints.

Many websites interact with subdomains or third-party sites in a way that requires full cross-origin access.

A controlled relaxation of the same-origin policy is possible using cross-origin resource sharing (CORS).

The cross-origin resource sharing protocol uses a suite of HTTP headers that define trusted web origins and associated properties such as whether authenticated access is permitted. These are combined in a header exchange between a browser and the cross-origin web site that it is trying to access.

## How does CORS work?

### Simple requests

A simple request is one that meets all the following conditions:

- One of the allowed methods: GET, HEAD, POST
- A [CORS safe-listed](https://fetch.spec.whatwg.org/#cors-safelisted-request-header) header is used
- When using the `Content-Type` header, only the following values are allowed: `application/x-www-form-urlencoded`, `multipart/form-data`, or `text/plain`
- No event listeners are registered on any XMLHttpRequestUpload object
- No ReadableStream object is used in the request

Step 1: Client (browser) request

When the browser is making a cross-origin request, the browser adds an **Origin** header with the current origin (scheme, host, and port).

Step 2: Server response

On the server side, when a server sees this header, and wants to allow access, it needs to add an **Access-Control-Allow-Origin** header to the response specifying the requesting origin (or \* to allow any origin.)

Step 3: Browser receives response

When the browser sees this response with an appropriate **Access-Control-Allow-Origin** header, the browser allows the response data to be shared with the client site.

### Preflighted requests

Unlike simple requests, for "preflighted" requests the browser first sends an HTTP request using the OPTIONS method to the resource on the other origin, in order to determine if the actual request is safe to send. Such cross-origin requests are preflighted since they may have implications for user data.

For example, a client might be asking a server if it would allow a DELETE request, before sending a DELETE request, by using a preflight request:

```
OPTIONS /resource/foo
Access-Control-Request-Method: DELETE
Access-Control-Request-Headers: origin, x-requested-with
Origin: https://foo.bar.org
```

If the server allows it, then it will respond to the preflight request with an **Access-Control-Allow-Methods** response header, which lists **DELETE**:

```
HTTP/1.1 204 No Content
Connection: keep-alive
Access-Control-Allow-Origin: https://foo.bar.org
Access-Control-Allow-Methods: POST, GET, OPTIONS, DELETE
Access-Control-Max-Age: 86400
```

The preflight response can be optionally cached for the requests created in the same URL using **Access-Control-Max-Age** header like in the above example.

## CORS headers

Request headers

| Header Name                    | Example Value         | Description                                                                                                   | Used in preflight requests | Used in CORS requests |
| ------------------------------ | --------------------- | ------------------------------------------------------------------------------------------------------------- | -------------------------- | --------------------- |
| Origin                         | https://www.test.com  | Combination of protocol, domain, and port of the browser tab opened                                           | YES                        | YES                   |
| Access-Control-Request-Method  | POST                  | For the preflight request, specifies what method the original CORS request will use                           | YES                        | NO                    |
| Access-Control-Request-Headers | Authorization, X-PING | For the preflight request, a comma separated list specifying what headers the original CORS request will send | YES                        | NO                    |

Response Headers

| Header Name                      | Example Value          | Description | Used in preflight requests | Used in CORS requests |
| -------------------------------- | ---------------------- | ----------- | -------------------------- | --------------------- |
| Access-Control-Allow-Origin      | https://www.test.com   | Item3.1     | YES                        | YES                   |
| Access-Control-Allow-Credentials | true                   | Item3.2     | YES                        | YES                   |
| Access-Control-Expose-Headers    | Date, X-Device-Id      | Column1     | NO                         | YES                   |
| Access-Control-Max-Age           | 600                    | Item3.1     | YES                        | NO                    |
| Access-Control-Allow-Methods     | GET, POST, PUT, DELETE | Item3.2     | YES                        | NO                    |
| Access-Control-Allow-Headers     | Authorization, X-PING  | Column1     | YES                        | NO                    |

## Common Pitfalls

1. Using \* operator for Access-Control-Allow-Origin.

CORS is a relaxation of same-origin policy while attempting to remain secure. Using \* disables most security rules of CORS. There are use cases where wildcard is OK such as an open API that integrates into many 3rd party websites.

2. Returning multiple domains for Access-Control-Allow-Origin.

Unfortunately, the spec does not allow Access-Control-Allow-Origin: https://mydomain.com, https://www.mydomain.com. The server can only respond with one domain or \*, but you can leverage the Origin request header.

3. Using wildcard selection like \*.mydomain.com.

This is not part of the CORS spec, wildcard can only be used to imply all domains are allowed.

4. Not including protocol or non-standard ports.

`Access-Control-Allow-Origin: mydomain.com` is not valid since the protocol is not included.

In a similar way, you will have trouble with `Access-Control-Allow-Origin: http://localhost` unless the server is actually running on a standard HTTP port like :80.

5. Not including Origin in the Vary response header

Most CORS frameworks do this automatically, you must specify to clients that server responses will differ based on the request origin.

6. Not specifying the Access-Control-Expose-Headers

If a required header is not included, the CORS request will still pass, but response headers not whitelisted will be hidden from the browser tab. The default response headers always exposed for CORS requests are:

- Cache-Control
- Content-Language
- Content-Type
- Expires
- Last-Modified
- Pragma

7. Using wildcard when Access-Control-Allow-Credentials is set to true

This is a tricky case that catches many people. If response has `Access-Control-Allow-Credentials: true`, then the wildcard operator cannot be used on any of the response headers like `Access-Control-Allow-Origin`.
