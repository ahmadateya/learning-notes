# Chapter 5: Managing Inbound and Outbound Traffic (continued)

## Managing outbound traffic

Nginx has a variety of options for outbound traffic management:
- Distributing outbound connections among multiple servers
- Configuring backup servers
- Enabling persistent connections with backend servers
- Limiting transfer rate while reading from backend servers
To enable most of these functions, the first thing you need is to declare your upstream servers explicitly.

### Declaring upstream servers
Nginx allows you to declare upstream servers explicitly. You can then refer to them multiple times as a single entity from any part of the http configuration. 

If the location of your server or servers changes, there is no need to go over the entire configuration and adjust it. If new servers join a group, or existing servers leave a group, it's only necessary to adjust the declaration and not the usage.

An upstream server is declared in the upstream section:
```nginx
http {
    upstream backend {
        server server1.example.com;
        server server2.example.com;
        server server3.example.com;
    }
    [...]
}
```
The `upstream` section can only be specified inside the http section. 

The preceding configuration declares a logical upstream named backend with three physical
servers. Each server is specified using the `server` directive. The server directive has
the following syntax:
```
server <address> [<parameters>];
```
The `<address>` parameter specifies an IP address or a domain name of a physical server. 
- If a domain name is specified, it is resolved at the startup time and the resolved IP address is used as the address of a physical server. 
- If the domain name resolves into multiple IP addresses, a separate entry is created for each of the resolved IP addresses. This is equivalent to specifying a server directive for each of these addresses.

The address can contain optional port specification, for example, server1.example.com:8080. 
If this specification is omitted, port 80 is used. Let's look at an example of upstream declaration:
```nginx
upstream numeric-and-symbolic {
    server server.example.com:8080;
    server 127.0.0.1;
}
```

- The preceding configuration declares an `upstream` named numeric-and-symbolic.
- The first server in the server list has a symbolic name and its port changed to 8080.
- The second server has the numerical address 127.0.0.1 that corresponds to the local host and the port is 80.

Let's look at another example:
```nginx
upstream numeric-only {
    server 192.168.1.1;
    server 192.168.1.2;
    server 192.168.1.3;
}
```
The preceding configuration declares an upstream named numeric-only, which consists of three servers with three different numerical IP addresses listening on the default port.

#### More Information can be found in the book pages 99, 100

---
### Using upstream servers
Once an upstream server is declared, it can be used in the `proxy_pass` directive:
```nginx
http {
    upstream my-cluster {
        server server1.example.com;
        server server2.example.com;
        server server3.example.com;
    }
    […]
    server {
        […]
        location @proxy {
            proxy_pass http://my-cluster;
        }
    }
}
```
The upstream can be referred multiple times from the configuration. With the preceding configuration, once the location @proxy is requested, Nginx will pass the request to one of the servers in the server list of the upstream.

#### The algorithm of resolving the final address of an upstream server
![Resolving final address Algorithm](/assets/images/nginx/resolving-final-address-in-upstream-server.png)

Because a destination URL can contain variables, it is evaluated at `runtime` and parsed as HTTP URL. The server name is extracted from the evaluated destination URL. 

Nginx looks up an upstream section that matches the server name and if such exists, forwards the request to one of the servers from the upstream server list according to a request distribution strategy.

If an upstream section that matches the server name exists, Nginx checks if the server name is an IP address. 
  - If so, Nginx uses the IP address as the final address of the upstream server. 
  - If the server name is symbolic, Nginx resolves the server name in DNS into an IP address. If successful, the resolved IP address is used as the final address of the upstream server.

The address of the DNS server or servers can be configured using the `resolver` directive:
```
resolver 127.0.0.1;
```
The preceding directive takes a list of IP addresses of the DNS servers as its arguments. If a server name cannot be successfully resolved using the configured resolver, Nginx returns HTTP status 502 (Bad Gateway).

When an upstream contains more than one server in the server list, Nginx distributes requests among these servers in an attempt to split the load among the available servers. This is also called `clustering`, as multiple servers act as one—altogether they are called a cluster.

---

### Choosing a request distribution strategy
I wrote a blogpost about this [here](https://ahmadateya.github.io/blog/request-distribtion-strategies-in-nginx/).