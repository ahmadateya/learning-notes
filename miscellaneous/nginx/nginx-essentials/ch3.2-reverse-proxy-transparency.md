# Chapter 3: Proxying and Caching (continued)

## Adding transparency
Once forwarded to an upstream server, a request loses certain properties of the original request. 

For example, the virtual host in a forwarded request is replaced by the host/port combination of the destination URL. 

The forwarded request is received from an IP address of the Nginx proxy, and the upstream server's functionality based on the client's IP address might not function properly.

The forwarded request needs to be adjusted so that the upstream server can obtain the missing information of the original request. This can be easily done with the `proxy_set_header` directive:

```nginx
proxy_set_header <header> <value>;
```
The `proxy_set_header` directive takes two arguments, the first of which is the name of the header that you want to set in the proxied request, and the second is the value for this header. Again, both arguments can contain variables.

Here is how you can pass the virtual host name from the original request:

```nginx
location @proxy {
    proxy_pass http://192.168.0.1;
    proxy_set_header Host $host;
}
```

The variable `$host` has a smart functionality. 

It does not simply pass the virtual host name from the original request, but uses the name of the server the request is processed by if the host header of the original request is empty or missing. 

If you insist on using the bare virtual host name from the original request, you can use the `$http_host` variable instead of `$host`.

Now that you know how to manipulate the proxied request, we can let the upstream server know the IP address of the original client. 

This can be done by setting `X-Real-IP` and/or the `X-Forwarded-For` headers:
```nginx
location @proxy {
    proxy_pass http://192.168.0.1;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```
This will make the upstream server aware of the original client's IP address via `X-Real-IP` or the `X-Forwarded-For` header. 

Most application servers support this header and take appropriate actions to properly reflect the original IP address in their API.

---
### Handling redirects
The next challenge is rewriting redirects. When the upstream server issues a temporary or permanent redirect (HTTP status codes 301 or 302), the absolute URI in the location or refresh headers needs to be rewritten so that it contains a proper host name (the host name of the server the original request came to).

This can be done using the `proxy_redirect` directive:
```nginx
location @proxy {
    proxy_pass http://localhost:8080;
    proxy_redirect http://localhost:8080/app http://www.example.com;
}
```
Consider a web application that is running at `http://localhost:8080/app`, while the original server has the address `http://www.example.com`. 

Assume the web application issues a temporary redirect (HTTP 302) to `http://localhost:8080/app/login`. With the preceding configuration, Nginx will rewrite the URI in the location header to `http://www.example.com/login`.

If the redirect URI was not rewritten, the client would be redirected to `http://localhost:8080/app/login`, which is valid only within a local domain, so the web application would not be able to work properly. 

With the `proxy_redirect` directive, the redirect URI will be properly rewritten by Nginx, and the web application will be able to perform the redirect properly.

The host name in the second argument of the `proxy_redirect` directive can be omitted:

```nginx
location @proxy {
    proxy_pass http://localhost:8080;
    proxy_redirect http://localhost:8080/app/;
}
```

The preceding code can be further reduced to the following configuration using variables:

```nginx
location @proxy {
    proxy_pass http://localhost:8080;
    proxy_redirect http://$proxy_host/app/;
}
```
The same transparency option can be applied to cookies. In the preceding example, consider cookies are set to the domain localhost:8080, since the application server replies at http://localhost:8080. The cookies will not be returned by the browser, because the cookie domain does not match the request domain.

---
### Handling cookies
To make cookies work properly, the domain name in cookies needs to be rewritten by the Nginx proxy. To do this, you can use the `proxy_cookie_domain` directive as shown here:
```nginx
location @proxy {
 proxy_pass http://localhost:8080;
 proxy_cookie_domain localhost:8080 www.example.com;
}
```
In the preceding example, Nginx replaces the cookie domain localhost:8080 in the upstream response with www.example.com. The cookies set by the upstream server will refer to the domain www.example.com and the browser will return cookies in subsequent requests.

If cookie path needs to be rewritten as well due to application server being rooted at a different path, you can use the `proxy_cookie_path` directive as shown in the following code:
```nginx
location @proxy {
    proxy_pass http://localhost:8080;
    proxy_cookie_path /my_webapp/ /;
}
```
In this example, whenever Nginx detects a cookie with a prefix specified in the first argument of the `proxy_cookie_path` directive (/my_webapp/), it replaces this prefix with the value in the second argument of the `proxy_cookie_path` directive (/).

Putting everything together for the www.example.com domain and the web application running at localhost:8080, we get the following configuration:
```nginx
location @proxy {
    proxy_pass http://localhost:8080;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_redirect http://$proxy_host/app /;
    proxy_cookie_domain $proxy_host www.example.com;
    proxy_cookie_path /my_webapp/ /;
}
```
The preceding configuration ensures transparency for a web application server so that it doesn't even need to know which virtual host it is running on.