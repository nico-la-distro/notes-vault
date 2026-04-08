## **Anatomy of a URL**
![[URL's anatomy.png]]

## **http req & rep**
![[HTTP req & rep.png]]

**Start Line**
The start line is like the introduction of the message. It tells you what kind of message is being sent—whether it's a request from the user or a response from the server. This line also gives important details about how the message should be handled.

**Headers**
Headers are made up of key-value pairs that provide extra information about the HTTP message. They give instructions to both the client and the server handling the request or response. These headers cover all sorts of things, like security, content types, and more, making sure everything goes smoothly in the communication.

**Empty Line**
The empty line is a little divider that separates the header from the body. It’s essential because it shows where the headers stop and where the actual content of the message begins. Without this empty line, the message might get messed up, and the client or server could misinterpret it, causing errors.

**Body**
The body is where the actual data is stored. In a request, the body might include data the user wants to send to the server (like form data). In a response, it’s where the server puts the content that the user requested (like a webpage or API data).

**HTTP versions**

| **HTTP Version** | **Year** | **Key Features**                                           |
| ---------------- | -------- | ---------------------------------------------------------- |
| **HTTP/0.9**     | 1991     | Basic version, supports only GET requests                  |
| **HTTP/1.0**     | 1996     | Introduced headers and support for multiple content types  |
| **HTTP/1.1**     | 1997     | Persistent connections, chunked transfer, improved caching |
| **HTTP/2**       | 2015     | Multiplexing, header compression, request prioritisation   |
| **HTTP/3**       | 2022     | Uses QUIC protocol for faster and more secure connections  |

## **http request**

![[HTTP req.png]]
**HTTP Methods**

- **GET**: Retrieves data without modifying it. Sensitive information should never be exposed in URLs.

- **POST**: Sends data to create or update a resource. Input must be validated and sanitized to prevent attacks such as SQL injection or XSS.

- **PUT**: Replaces or updates an existing resource. Proper authorization is required.

- **DELETE**: Removes a resource. Only authorized users should be allowed to perform this action.

- **PATCH**: Partially updates a resource. Data must be validated to avoid inconsistencies.

- **HEAD**: Similar to GET but returns only headers, not the response body.

- **OPTIONS**: Indicates which HTTP methods are allowed for a resource.

- **TRACE**: Used for debugging and often disabled for security reasons.

- **CONNECT**: Establishes a secure connection, commonly used for HTTPS.


**Key point**: All HTTP methods must enforce authorization and strict input validation to ensure security.

**HTTP request headers**

| **Request Header** | **Example**                                                                      | **Description**                                                                          |
| ------------------ | -------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Host               | `Host: tryhackme.com`                                                            | Specifies the name of the web server the request is for.                                 |
| User-Agent         | `User-Agent: Mozilla/5.0`                                                        | Shares information about the web browser the request is coming from.                     |
| Referer            | `Referer: https://www.google.com/`                                               | Indicates the URL from which the request came from.                                      |
| Cookie             | `Cookie: user_type=student; room=introtowebapplication; room_status=in_progress` | Information the web server previously asked the web browser to store is held in cookies. |
| Content-Type       | `Content-Type: application/json`                                                 | Describes what type or format of data is in the request.                                 |

**HTTP request body

|**Format**|**Content-Type**|**Description**|**Typical Use**|
|---|---|---|---|
|**URL Encoded**|`application/x-www-form-urlencoded`|Data sent as `key=value` pairs separated by `&`, special characters are encoded|Simple form submissions|
|**Form Data**|`multipart/form-data`|Data split into parts using boundaries; supports binary data|File uploads, images|
|**JSON**|`application/json`|Data structured as name–value pairs in JSON format|APIs, modern web apps|
|**XML**|`application/xml`|Data wrapped in nested tags|Legacy systems, structured data exchange|
**URL Encoded**
![[URL Encoded.png]]

**Form Data**
![[Form data (http req).png]]

**JSON**
![[JSON (http req).png]]

**XML**
![[XML (http req).png]]

## **http response**

![[HTTP rep.png]]

**Status Line**  
Every HTTP response starts with a **Status Line**, which contains:
- **HTTP Version**: Protocol version in use
- **Status Code**: Three-digit number showing the result of the request
- **Reason Phrase**: Short, human-readable explanation of the status

**Status Code Categories**

|**Range**|**Category**|**Meaning**|
|---|---|---|
|**100–199**|Informational|Request received, continue sending data|
|**200–299**|Success|Request processed successfully|
|**300–399**|Redirection|Resource moved to a different location|
|**400–499**|Client Error|Problem with the request (e.g. bad URL, missing auth)|
|**500–599**|Server Error|Server failed to process the request|

**Common Status Codes**

|**Code**|**Reason Phrase**|**Description**|
|---|---|---|
|**100**|Continue|Server ready for the rest of the request|
|**200**|OK|Request succeeded|
|**301**|Moved Permanently|Resource has a new permanent URL|
|**404**|Not Found|Resource does not exist|
|**500**|Internal Server Error|Server-side failure|

**HTTP response headers**  
HTTP response headers are **key–value pairs** sent by the server to describe the response and instruct the client on how to handle it.

**Important Response Headers**

|**Header**|**Example**|**Description**|
|---|---|---|
|**Date**|`Date: Fri, 23 Aug 2024 10:43:21 GMT`|Time and date when the response was generated|
|**Content-Type**|`Content-Type: text/html; charset=utf-8`|Defines the response content type and character encoding|
|**Server**|`Server: nginx`|Identifies the server software (often hidden for security)|

**Other Common Response Headers**

| **Header**        | **Example**                           | **Description / Security Notes**                               |
| ----------------- | ------------------------------------- | -------------------------------------------------------------- |
| **Set-Cookie**    | `Set-Cookie: sessionId=38af1337es7a8` | Sends cookies to the client; use `HttpOnly` and `Secure` flags |
| **Cache-Control** | `Cache-Control: max-age=600`          | Controls how responses are cached                              |
| **Location**      | `Location: /index.html`               | Used in redirects (3xx); validate to prevent open redirects    |

## **Security headers**

### **Content-Security-Policy (CSP)**

Helps protect web applications from attacks such as **Cross-Site Scripting (XSS)** by restricting where content can be loaded from.

A CSP allows administrators to define **trusted sources** for different types of content (scripts, styles, images, etc.). Any content loaded from non-approved sources is blocked by the browser.

**Common CSP Directives**

|**Directive**|**Purpose**|
|---|---|
|**default-src**|Sets the default allowed source for all content types|
|**script-src**|Defines where JavaScript can be loaded from|
|**style-src**|Defines where CSS stylesheets can be loaded from|
|**'self'**|Allows content only from the same domain as the website|

**Exemple CSP Header**

Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.tryhackme.com; style-src 'self'

- **default-src 'self'**: Only content from the current website is allowed by default
- **script-src 'self' [https://cdn.tryhackme.com](https://cdn.tryhackme.com)**: Scripts can load from the site itself and the specified CDN
- **style-src 'self'**: Stylesheets can only be loaded from the current site


### **Strict-Transport-Security (HSTS)**

**Strict-Transport-Security (HSTS)** is a security response header that forces browsers to **always use HTTPS** when connecting to a website, preventing protocol downgrade and man-in-the-middle attacks.

**Exemple de HSTS herder**

Strict-Transport-Security: max-age=63072000; includeSubDomains; preload

**HSTS Directives**

|**Directive**|**Description**|
|---|---|
|**max-age**|Duration (in seconds) that the browser must enforce HTTPS|
|**includeSubDomains**|Applies HSTS to all subdomains|
|**preload**|Allows the site to be included in browser HSTS preload lists|

**Why HSTS Matters**
- Prevents users from accessing the site over HTTP
- Protects against SSL stripping attacks
- Ensures encrypted communication by default

**Key point**: Once HSTS is enabled, browsers will refuse any HTTP connection until the `max-age` expires.

### **X-Content-Type-Options**

Prevents the browser from guessing (sniffing) the MIME type of a resource.

**Exemple header**

X-Content-Type-Options: nosniff

|**Directive**|**Description**|
|---|---|
|**nosniff**|Forces the browser to respect the `Content-Type` header only|
Prevents attacks where a file is interpreted as executable code (e.g. JS) instead of its declared type.


### **Referrer-Policy**

Controls how much referrer information is sent when navigating to another page or site.

**Exemple header**

Referrer-Policy: no-referrer
Referrer-Policy: same-origin
Referrer-Policy: strict-origin
Referrer-Policy: strict-origin-when-cross-origin

|**Policy**|**Behavior**|
|---|---|
|**no-referrer**|No referrer information is sent|
|**same-origin**|Referrer sent only to same-origin requests|
|**strict-origin**|Sends origin only, and only if protocol stays the same (HTTPS → HTTPS)|
|**strict-origin-when-cross-origin**|Full URL for same-origin, origin only for cross-origin|

**Key point**: These headers reduce information leakage and help prevent certain privacy and security issues.
