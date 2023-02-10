# Chapter 4: Rewrite Engine and Access Control (continued)

## Access control
Nginx includes a group of modules that let you allow or deny access depending on various conditions.

Nginx denies access to a resource by returning a `403` (Forbidden HTTP) status or `401` (Unauthorized) if accessing the resource requires authentication. This `403` (Forbidden) status code can be intercepted and customized using the `error_page` directive.

### Restricting access by IP address
Nginx allows you to permit or deny access to a virtual host or a location by IP address.

For that, you can use the directives `allow` and `deny`. They have the following format:

```nginx
allow <IP address> | <IP address>/<prefix size> | all;
deny <IP address> | <IP address>/<prefix size> | all;
```

Specifying an IP address allows or denies access to a single IP address within a location, while specifying an IP address with a prefix size (for example 192.168.0.0/24 or 200.1:980::/32) allows or denies access to a range of IP addresses.

- The `allow` and deny directives are processed in order of appearance within a location.
  - The remote IP address of a requesting client is matched against the argument of each directive. 
  - Once an allow directive with a matching address is found, access is immediately allowed. 

- Once a `deny` directive with a matching address is found, access is immediately denied. 

- Once Nginx reaches the allow or deny directive with the `all` argument, access is immediately allowed or denied, regardless of client's IP address.

This, obviously, allows some variation. Here are some simple examples:
```nginx
server {
    deny 192.168.1.0/24;
    allow all;
    [...]
}
```

The preceding configuration makes Nginx deny access to IP addresses 192.168.1.0 to 192.168.1.255, while allowing access to everyone else. This happens because the deny directive is processed first and if matched, is immediately applied. The entire server will be forbidden for specified IP addresses.

```nginx
server {
    […]
    location /admin {
        allow 10.132.3.0/24;
        deny all;
    }
}
```

The preceding configuration makes Nginx allow access to location `/admin` only to IP addresses in the range `10.132.3.0` to `10.132.3.255`. 

Assuming this range of IP addresses corresponds to some privileged group of users, this configuration makes perfect sense, as only they can access the administrative area of this web application

Now, we can improve on that and make the configuration more complicated. Assume that more networks need access to this web application's administrative interface, while the IP address `10.132.3.55` needs to be denied access due to technical or administrative reasons. 

Then, we can extend the preceding configuration as follows:
```nginx
server {
    […]
    location /admin {
        allow 10.129.1.0/24;
        allow 10.144.25.0/24;
        deny 10.132.3.55;
        allow 10.132.3.0/24;
        deny all;
    }
}
```

- As you can see, the directives `allow` and `deny` are quite intuitive to use. Use them as long as the list of IP addresses to match is not too long. 
- Nginx processes these directives in **sequential order**, so the time taken to check the client's IP address against the list is on average proportional to the length of the list no matter which directive the address is matched against.

#### If you need to match client's IP address against a larger list of addresses, consider using the `geo` directive.

### Using the geo directive to restrict access by IP address
With the `geo` directive, you can transform an IP address into a literal or numerical value that can later be used to trigger some actions while processing a request.

The `geo` directive has the following format:
```nginx
geo [$<source variable>] $<target variable> { <address mapping> }
```

If the source variable is omitted, the `$remote_addr` variable is used instead. 

The address mapping is a list of key/value pairs, separated by whitespace. A key is usually an IP address or an IP address with a prefix size specifying a subnet. A value is an arbitrary string of character or a number. 

Let's look at the following code:
```nginx
geo $admin_access {
    default deny;
    10.129.1.0/24 allow;
    10.144.25.0/24 allow;
    10.132.3.0/24 allow;
}
```

The value of the source variable is used as a key to look up an entry in the address mapping. Once found, the target variable is assigned to the looked-up value.

Otherwise, the default value is used.

With the preceding configuration, the variable `$admin_access` will be assigned the value allow if the remote client's IP address originates from the subnet 10.129.1.0/24, 10.144.25.0/24 or 10.132.3.0/24, and deny otherwise.

#### Good Note
The `geo` directive builds an efficient succinct data structure to look up the values by IP address in memory. 
It can handle hundreds of thousands of IP addresses and subnets. To accelerate the startup time, specify IP addresses to the geo directive in ascending order, for example, 1.x.x.x to 10.x.x.x, 1.10.x.x to 1.30.x.x.

The address mapping section can contain directives that affect the behavior of `geo` address mapping. 

The following table lists those directives along with their functions:

| Directive       | Function                                                                                                                                                                                                                                                                                                                                        |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| default         | Specifies a value that is returned when no match is found in the IP address mapping.                                                                                                                                                                                                                                                            |  |
| proxy           | Specifies the address of a proxy server. If a request originates from an address specified by one of the proxy directives, geo will use the last address from the "X-Forwarded-For" header and not from the source variable.                                                                                                                    |  |
| proxy_recursive | If a request originates from an address specified by one of proxy directives, geo will process addresses in the `"X-Forwarded-For"` header from right-to-left in search of an address outside of  the list specified by the proxy directive. In other words, this directive makes geo make a better effort in the search for a real IP address. |  |
| ranges          | Enables IP address ranges in the mapping list.                                                                                                                                                                                                                                                                                                  |  |
| delete          | Removes the specified sub network from the mapping.                                                                                                                                                                                                                                                                                             |  |

### Examples
Consider Nginx receives HTTP traffic from an application-level load balancer or an inbound proxy located at IP 10.200.0.1. 

Since all requests will originate from this IP, we need to examine the "X-Forwarded-For" header in order to obtain the real IP address of the client. We then need to change the preceding configuration as follows:
```nginx
geo $example {
    default deny;
    proxy 10.200.0.1;
    10.129.1.0/24 allow;
    10.144.25.0/24 allow;
    10.132.3.0/24 allow;
}
```
If the server is behind a chain of proxies, the real IP address can be obtained by specifying the `proxy_recursive` directive and listing all proxies in the chain:
```nginx
geo $example {
    default deny;
    proxy 10.200.0.1;
    proxy 10.200.1.1;
    proxy 10.200.2.1;
    proxy_recursive;
    10.129.1.0/24 allow;
    10.144.25.0/24 allow;
    10.132.3.0/24 allow;
}
```

In the preceding example, proxies have IP addresses 10.200.0.1, 10.200.1.1 and 10.200.2.1. The order the addresses are listed in is not important, as Nginx simply iterates over the addresses specified in the "X-Forwarded-For" header from right-to-left and checks their presence in the geo block. The first address outside of the proxy list becomes the real IP address of the client.

If IP addresses need to be specified as ranges instead or in addition to subnets, you can enable this by specifying the `ranges` directive:
```nginx
geo $example {
    default deny;
    ranges;
    10.129.1.0-10.129.1.255 allow;
    10.144.25.0-10.144.25.255 allow;
    10.132.3.0/24 allow;
}
```
Finally, with the help of the `delete` directive, we can define the IP address mapping that allows us to implement an access control procedure analogous to allow and deny directives on a larger scale:
```nginx
geo $admin_access {
    default deny;
    10.129.1.0/24 allow;
    10.144.25.0/24 allow;
    10.132.3.0/24 allow;
    delete 10.132.3.55;
}
```
To put this configuration in action, we need to use the if section to forbid request those client's IP address do not fall in the allow range of the geo directive:

```nginx
server {
    […]
    geo $admin_access {
        default deny;
        10.129.1.0/24 allow;
        10.144.25.0/24 allow;
        10.132.3.0/24 allow;
        delete 10.132.3.55;
    }
    location /admin {
        if($admin_access != allow) {
            return 403;
        }
        [...]
    }
}
```