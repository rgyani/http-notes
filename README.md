## What happens when you type a URL into your browser?
 
1. You type a URL in your browser and press Enter
2. Browser looks up the IP address for the domain using a DNS lookup.   
Because DNS is complex and has to be blazingly fast, DNS data is cached at different layers between your browser and at various places across the Internet.   
Your browser checks its own cache, the operating system cache, a local network cache at your router, and a DNS server cache on your corporate network or at your internet service provider (ISP).  
If the browser cannot find the IP address at any of those cache layers, the DNS server on your corporate network or at your ISP does a recursive DNS lookup.  
A recursive DNS lookup asks multiple DNS servers around the Internet, which in turn ask more DNS servers for the DNS record until it is found.
3. Browser initiates TCP connection with the server
4. Browser sends the HTTP request to the server
5. Server processes request and sends back a response
6. Browser renders the content

### HTTP versions
The first response that a client receives on an HTTP GET request is often not the fully rendered page.  
Instead, it contains links to additional resources needed by the requested page.  
The client discovers that the full rendering of the page requires these additional resources from the server only after it downloads the page.  
Because of this, the client will have to make additional requests to retrieve these resources. 

* In HTTP/1.0, the client had to break and remake the TCP connection with every new request, a costly affair in terms of both time and resources.  
* In HTTP/1.1, the client uses multiple TCP connections to lessen the effect of HOL blocking.   
* In HTTP/2, the client establishes a single connection object between the two machines, called Binary Framing Layer.   
Within this connection there are multiple streams of data. Each stream consists of multiple messages in the familiar request/response format. Finally, each of these messages split into smaller units called frames
* HTTP/3 uses QUIC, a multiplexed transport protocol built on UDP instead of TCP

#### HTTP v2.0

From a technical point of view, one of the most significant features that distinguishes HTTP/1.1 and HTTP/2 is the binary framing layer, which can be thought of as a part of the application layer in the internet protocol stack.   
As opposed to HTTP/1.1, which keeps all requests and responses in plain text format, HTTP/2 uses the binary framing layer to encapsulate all messages in binary format, while still maintaining HTTP semantics, such as verbs, methods, and headers.   
An application level API would still create messages in the conventional HTTP formats, but the underlying layer would then convert these messages into binary. This ensures that web applications created before HTTP/2 can continue functioning as normal when interacting with the new protocol.

#### HTTP v3.0
HTTP/3 uses similar semantics compared to earlier revisions of the protocol, including the same request methods, status codes, and message fields, but encodes them and maintains session state differently.   

However, partially due to the protocol's adoption of QUIC, HTTP/3 has lower latency and loads more quickly in real-world usage when compared with previous versions: in some cases over 3× faster than with HTTP/1.1 (which remains the only HTTP version deployed by many websites).

![img](imgs/2022-cloudfront-quic-1.jpg)

On 15 Aug, 20222, AWS Cloudfront announced support for Http/3 

### [A much better explaination](https://www.linkedin.com/pulse/http3-over-quic-its-revolutionary-saurabh-pandey/)

### Cross-Origin Resource Sharing (CORS)
Cross-Origin Resource Sharing (CORS) is an HTTP-header based mechanism that allows a server to indicate any origins (domain, scheme, or port) other than its own from which a browser should permit loading resources.   
CORS also relies on a mechanism by which browsers make a "preflight" request to the server hosting the cross-origin resource, in order to check that the server will permit the actual request. In that preflight, the browser sends headers that indicate the HTTP method and headers that will be used in the actual request.

An example of a cross-origin request: the front-end JavaScript code served from https://domain-a.com uses XMLHttpRequest to make a request for https://domain-b.com/data.json.

For security reasons, browsers restrict cross-origin HTTP requests initiated from scripts. For example, XMLHttpRequest and the Fetch API follow the same-origin policy. This means that a web application using those APIs can only request resources from the same origin the application was loaded from unless the response from other origins includes the right CORS headers.

#### What requests use CORS?
This cross-origin sharing standard can enable cross-origin HTTP requests for:

1. Invocations of the XMLHttpRequest or Fetch APIs, as discussed above.
2. Web Fonts (for cross-domain font usage in @font-face within CSS), so that servers can deploy TrueType fonts that can only be loaded cross-origin and used by web sites that are permitted to do so.
3. WebGL textures.
4. Images/video frames drawn to a canvas using drawImage().
5. CSS Shapes from images.

#### Preflighted requests
Unlike simple requests, for "preflighted" requests the browser first sends an HTTP request using the OPTIONS method to the resource on the other origin, in order to determine if the actual request is safe to send. Such cross-origin requests are preflighted since they may have implications for user data.

The following is an example of a request that will be preflighted:
```text
const xhr = new XMLHttpRequest();
xhr.open('POST', 'https://bar.other/resources/post-here/');
xhr.setRequestHeader('X-PINGOTHER', 'pingpong');
xhr.setRequestHeader('Content-Type', 'application/xml');
xhr.onreadystatechange = handler;
xhr.send('<person><name>Arun</name></person>');
```

![img](imgs/preflight_correct.png)



### How does HTTPS work?

Hypertext Transfer Protocol Secure (HTTPS) is an extension of the Hypertext Transfer Protocol (HTTP.)   
HTTPS transmits encrypted data using Transport Layer Security (TLS.) If the data is hijacked online, all the hijacker gets is binary code. 

#### How is the data encrypted and decrypted?

![img](imgs/1651679057259.jpg)

1. The client (browser) and the server establish a TCP connection.
2. The client sends a “client hello” to the server. The message contains a set of necessary encryption algorithms (cipher suites) and the latest TLS version it can support. The server responds with a “server hello” so the browser knows whether it can support the algorithms and TLS version.
3. The server then sends the SSL certificate to the client. The certificate contains the public key, host name, expiry dates, etc. The client validates the certificate. 
4. After validating the SSL certificate, the client generates a session key and encrypts it using the public key. The server receives the encrypted session key and decrypts it with the private key. 
5. Now that both the client and the server hold the same session key (symmetric encryption), the encrypted data is transmitted in a secure bi-directional channel.

#### Why does HTTPS switch to symmetric encryption during data transmission? There are two main reasons:
1. Security: The asymmetric encryption goes only one way. This means that if the server tries to send the encrypted data back to the client, anyone can decrypt the data using the public key.
2. Server resources: The asymmetric encryption adds quite a lot of mathematical overhead. It is not suitable for data transmissions in long sessions.

### Client Certificate vs Server Certificate

Server certificates perform encryption on data-in-transit to assure data confidentiality. eg. SSL certificates
Client certificate does not encrypt any data, it only serves as a more secure authentication mechanism than passwords. eg. Email certificates
