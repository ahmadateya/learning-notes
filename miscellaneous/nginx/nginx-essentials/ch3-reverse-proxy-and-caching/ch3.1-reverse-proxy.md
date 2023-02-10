# Chapter 3: Proxying and Caching

## Nginx as a reverse proxy

HTTP is a complex protocol that deals with data of different modality and has numerous optimizations that—if implemented properly—can lead to a significant increase in web service performance.

At the same time, web application developers have less time to deal with low-level issues and optimizations. 

The mere idea of decoupling a web application server from a frontend server shifts the focus on managing incoming traffic to the frontend, while shifting the focus on functionality, application logic, and features to the web application server. This is where Nginx comes into play as a decoupling point.

An example of a decoupling point is SSL termination: Nginx receives and processes inbound SSL connections, it forwards the request over plain HTTP to an application server, and wraps the received response back into SSL. The application server no longer needs to take care of storing certificates, SSL sessions, handling encrypted and unencrypted transmission, and so on.

#### Other examples of decoupling are as follows:
- Efficient handling of static files and delegating the dynamic part to the upstream
- Rate, request, and connection limiting
- Compressing responses from the upstream
- Caching responses from the upstream
- Accelerating uploads and downloads

---
### Setting up Nginx as a reverse proxy
Nginx can be easily configured to work as a reverse proxy:
```nginx
location /example {
    proxy_pass http://upstream_server_name:port;
}
```

If a path is specified in the destination URL, it will replace a part of the path from the original request that corresponds to the matching part of the location. 
For example, consider the following configuration:
```nginx
location /download {
    proxy_pass http://192.168.0.1/media;
}
```
If a request for `/download/BigFile.zip` is received, the path in the destination URL is `/media` and it corresponds to the matching `/download` part of the original request URI. This part will be replaced with `/media` before passing to the upstream server, so the passed request path will look like `/media/BigFile.zip`.

If proxy_pass directive is used inside a regex location, the matching part cannot be computed. 

In this case, a destination URI without a path must be used:
```nginx
location ~* (script1|script2|script3)\.php$ {
    proxy_pass http://192.168.0.1;
}
```
The same applies to cases where the request path was changed with the rewrite directive and is used by a `proxy_pass` directive.

---
### Setting the backend the right way
The right way to configure a backend is to avoid passing everything to it. 
Nginx has powerful configuration directives that help you ensure that only specific requests are delegated to the backend.

Consider the following configuration:
```nginx
location ~* \.php$ {
    proxy_pass http://backend;
    [...]
}
```
This passes every request with a URI that ends with `.php` to the PHP interpreter.

This is not only inefficient due to the intensive use of regular expressions, but also a serious security issue on most PHP setups because it may allow arbitrary code execution by an attacker.

#### try_files
Nginx has an elegant solution for this problem in the form of the `try_files` directive. The `try_files` directive takes a list of files and a location as the last argument. 

Nginx tries specified files in consecutive order and if none of them exists, it makes an internal redirect to the specified location. 

Consider the following example:
```nginx
location / {
    try_files $uri $uri/ @proxy;
}
location @proxy {
    proxy_pass http://backend;
}
```

The preceding configuration first looks up a file corresponding to the request URI, looks for a directory corresponding to the request URI in the hope of returning an index of that directory, and finally makes an internal redirect to the named location `@proxy` if none of these files or directories exist.

This configuration makes sure that whenever a request URI points to an object in the filesystem it is handled by Nginx itself using efficient file operations, and only if there is no match in the filesystem for the given request URI is it delegated to the backend.

---
### Using SSL
If the upstream server supports SSL, connections to the upstream server can be secured by simply changing the destination URL scheme to `https`:
```nginx
location @proxy {
 proxy_pass https://192.168.0.1;
}
```
If the authenticity of the upstream server needs to be verified, this can be enabled using the `proxy_ssl_verify` directive:
```nginx
location @proxy {
    proxy_pass https://192.168.0.1;
    proxy_ssl_verify on;
}
```
The certificate of the upstream server will be verified against certificates of well-known certification authorities. 
In Unix-like operating systems, they are usually stored in `/etc/ssl/certs`.

If an upstream uses a trusted certificate that cannot be verified by well-known certification authorities or a self-signed certificate, it can be specified and declared as trusted using the proxy_ssl_trusted_certificate directive. 

This directive specifies the path to the certificate of the upstream server or a certificate chain required to authenticate the upstream server in PEM format. 

Consider the following example:
```nginx
location @proxy {
    proxy_pass https://192.168.0.1;
    proxy_ssl_verify on;
    proxy_ssl_trusted_certificate /etc/nginx/upstream.pem;
}
```
If Nginx needs to authenticate itself to the upstream server, the client certificate and the key can be specified using the `proxy_ssl_certificate` and `proxy_ssl_certificate_key` directives. 

The directive `proxy_ssl_certificate` specifies the path to the client certificate in PEM format, while `proxy_ssl_certificate_key` specifies the path to the private key from the client certificate in PEM format.

Consider the following example:
```nginx
location @proxy {
    proxy_pass https://192.168.0.1;
    proxy_ssl_certificate /etc/nginx/client.pem;
    proxy_ssl_certificate_key /etc/nginx/client.key;
}
```
The specified certificate will be presented while setting up the secure connection to the upstream server, and its authenticity will be verified by specified private key.

---
### Handling errors
If Nginx experiences a problem contacting the upstream server or the upstream server returns an error, there is an option to take certain actions.

The upstream server connectivity errors can be handled using the `error_page` directive:
```nginx
location ~* (script1|script2|script3)\.php$ {
    proxy_pass http://192.168.0.1;
    error_page 500 502 503 504 /50x.html;
}
```
This will make Nginx return the document from the file `50x.html` once an upstream connectivity error has occurred.

This will not change the HTTP status code in the response. To change the HTTP status code to successful, you can use the following syntax:
```nginx
location ~* (script1|script2|script3)\.php$ {
    proxy_pass http://192.168.0.1;
    error_page 500 502 503 504 =200 /50x.html;
}
```
A more sophisticated action can be taken upon failure of an upstream server using an `error_page` directive that points to a named location:
```nginx
location ~* (script1|script2|script3)\.php$ {
    proxy_pass http://upstreamA;
    error_page 500 502 503 504 @retry;
}
location @retry {
    proxy_pass http://upstreamB;
    error_page 500 502 503 504 =200 /50x.html;
}
```

In the preceding configuration, Nginx first tries to fulfill the request by forwarding it to the `upstreamA` server. 

If this results in an error, Nginx switches to a named location `@retry` in an attempt to try with the `upstreamB` server. Request an URI while switching so that the upstreamB server will receive an identical request.

If this doesn't help either, Nginx returns a static file `50x.html` pretending no error occurred.

If an upstream has replied but returned an error, it can be intercepted rather than passed to the client using the `proxy_intercept_errors` directive:
```nginx
location ~* (script1|script2|script3)\.php$ {
    proxy_pass http://upstreamA;
    proxy_intercept_errors on;
    error_page 500 502 503 504 403 404 @retry;
}
location @retry {
    proxy_pass http://upstreamB;
    error_page 500 502 503 504 =200 /50x.html;
}
```

In the preceding configuration, the upstreamB server will be called even when the `upstreamA` server replies but returns erroneous HTTP status code, such as 403 or 404. This gives `upstreamB` an opportunity to fix the soft errors of upstreamA, if necessary.

---
### Choosing an outbound IP address
Sometimes, when your proxy server has multiple network interfaces, it becomes necessary to choose which IP address should be used as outbound address for upstream connections. 

By default, the system will choose the address of the interface that adjoins the network containing the host used as destination in the default route. To choose a particular IP address for outbound connections, you can use the `proxy_bind` directive:
```nginx
location @proxy {
    proxy_pass https://192.168.0.1;
    proxy_bind 192.168.0.2;
}
```
This will make Nginx bind outbound sockets to the IP address `192.168.0.2` before making a connection. The upstream server will then see connections coming from IP address `192.168.0.2`.

---
### Accelerating downloads
Nginx is very efficient at heavy operations, such as handling large uploads and downloads. These operations can be delegated to Nginx using built-in functionality and third-party modules.

To accelerate download, the upstream server must be able to issue the `X-AccelRedirect` header that points to the location of a resource which needs to be returned, instead of the response obtained from the upstream. 

Consider the following configuration:
```nginx
location ~* (script1|script2|script3)\.php$ {
    proxy_pass https://192.168.0.1;
}
location /internal-media/ {
    internal;
    alias /var/www/media/;
}
```
With the preceding configuration, once Nginx detects the `X-Accel-Redirect` header in the upstream response, it performs an internal redirect to the location specified in this header. 

Assume the upstream server instructs Nginx to perform an internal redirect to `/internal-media/BigFile.zip`. This path will be matched against the location `/internal-media`. This location specifies the document root at `/var/www/media`. So if a file `/var/www/media/BigFile.zip` exists, it will be returned to the client using efficient file operations.

For many web application servers, this feature provides an enormous speed-up both because they might not handle large downloads efficiently and because proxying reduces efficiency of large downloads.

