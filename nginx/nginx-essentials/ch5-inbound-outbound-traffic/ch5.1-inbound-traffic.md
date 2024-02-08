# Chapter 5: Managing Inbound and Outbound Traffic

## Managing inbound traffic

Nginx has various options for managing inbound traffic. This includes the following:
- Limiting the request rate.
- Limiting the number of simultaneous connections.
- Limiting the transfer rate of a connection.

### Limiting the request rate
Nginx has a built-in module for limiting the request rate. 
Before you can enable it, you need to configure a shared memory segment (also known as a `zone`) in the `http` section using the `limit_req_zone` directive. 

This directive has the following format:
```nginx
limit_req_zone <key> zone=<name>:<size> rate=<rate>;
```
- The `<key>` argument specifies a single variable or a script (since version 1.7.6) to which the rate limiting state is bound. 

    - In simple terms, by specifying the `<key>` argument, you are creating a number of small pipes for each value of the `<key>` argument evaluated at runtime, each of them with its request rate limited with `<rate>`. 

    - Each request for a location where this zone is used will be submitted to the corresponding pipe and if the rate limit is reached, the request will be delayed so that the rate limit within the pipe is satisfied.

- The `<name>` argument defines the name of the zone.
- The `<size>` argument defines the size of the zone. 
  
Consider the following example:
```nginx
http {
    limit_req_zone $remote_addr zone=rate_limit1:12m rate=30r/m;
    [...]
}
```
In the preceding code, we define a zone named `primary` that is 12 MB in size and has a rate limit of 30 requests per minute (0.5 request per second). 
We use the `$remote_addr` variable as a key. This variable evaluates into a symbolic value of the IP address the request came from, which can take up to 15 bytes per IPv4 address and even more per IPv6 address.

To conserve space occupied by the key, we can use the variable `$binary_remote_addr` that evaluates into a binary value of the remote IP address:
```nginx
http {
    limit_req_zone $binary_remote_addr zone=rate_limit1:12m rate=30r/m;
    [...]
}
```

To enable request rate limiting in a location, use the `limit_req` directive:
```nginx
location / {
    limit_req zone=rate_limit1;
}
```
Once a request is routed to location /, a rate-limiting state will be retrieved from the specified shared memory segment and Nginx will apply the ***Leaky Bucket*** algorithm to manage the request rate.

### The Leaky Bucket Algorithm
![Leaky Bucket Algorithm](/assets/images/nginx/leaky-bucket-algorithm.png)

In Leaky Bucket Algorithm: 
- incoming requests can arrive at an arbitrary rate, but the outbound request rate will never be higher than the specified one. 
- Incoming requests "fill the bucket" and if the "bucket" overflows, excessive requests will get the HTTP status 503 (Service Temporarily Unavailable) response.

---

### Limiting the number of simultaneous connections

Request rate limiting cannot help mitigate abuses in case of long-running requests, such as long uploads and downloads.

In this situation, limiting the number of simultaneous connections comes in handy.
In particular, it is advantageous to limit the number of simultaneous connections from a single IP address.

Enabling the simultaneous connections limit starts from configuring a shared memory segment (a `zone`) for storing state information, just like when limiting the request rate. 

This is done in the http section using the `limit_conn_zone` directive. This directive is similar to the `limit_req_zone` directive and has the following format:

```nginx
limit_conn_zone <key> zone=<name>:<size>;
```
- The `<key>` argument specifies a single variable or a script (since version 1.7.6) to which the connection limiting state is bound. 
- The `<name>` argument defines the name of the zone and
- The `<size>` argument defines the size of the zone. 
 
Consider the following example:
```nginx
http {
    limit_conn_zone $remote_addr zone=conn_limit1:12m;
    [...]
}
```

To conserve the space occupied by the key, we can again use the variable `$binary_remote_addr`. 

It evaluates into a binary value of the remote IP address:
```nginx
http {
    limit_conn_zone $binary_remote_addr zone=conn_limit1:12m;
    [...]
}
```

To enable simultaneous connection limiting in a location, use the `limit_conn` directive:
```nginx
location /download {
    limit_conn conn_limit1 5;
}
```

The first argument of the `limit_conn` directive specifies the zone used to store connection limiting state information, and the second argument is the maximum number of simultaneous connections.

For each connection with an active request routed to *location /download*, the `<key>` argument is evaluated. If the number of simultaneous connections sharing the same value of the key surpasses 5, the server will reply with HTTP status 503 (Service Temporarily Unavailable).

#### Note
The size of the shared memory segment that the `limit_conn_zone` directive allocates is fixed. When the allocated shared memory segment gets filled, Nginx returns HTTP status 503 (Service Temporarily Unavailable). Therefore, you have to adjust the size of the shared memory segment to account for the potential inbound traffic of your server.

---
### Limiting the transfer rate of a connection
The transfer rate of a connection can also be limited. Nginx has a number of options for this purpose. The `limit_rate` directive limits the transfer rate of a connection in a location to the value specified in the first argument:
```nginx
location /download {
    limit_rate 100k;
}
```

The preceding configuration will limit the download rate of any request for location /download to 100 KBps. The rate limit is set per request. Therefore, if a client opens multiple connections, the total download rate will be higher.

Setting the rate limit to 0 switches off transfer rate limiting. This is helpful when a certain location needs to be excluded from the rate limit restriction:
```nginx
server {
    [...]
    limit_rate 1m;
    location /fast {
        limit_rate 0;
    }
}
```
The preceding configuration limits the transfer rate of each request to a given virtual host to 1 MBps, except for `location /fast`, where the rate is unlimited.

The transfer rate can also be limited by setting the value of the variable `$limit_rate`. This option can be elegantly used when rate-limiting needs to be enabled upon a particular condition:
```nginx
if ($slow) {
    set $limit_rate 100k;
}
```
There is also an option to postpone the rate restriction until a certain amount of data has been transferred. This can be achieved by using the `limit_rate_after` directive:
```nginx
location /media {
    limit_rate 100k;
    limit_rate_after 1m;
}
```
The preceding configuration will enforce the rate limit only after the first megabyte of request data has been sent. 
Such behavior is useful, for example, when streaming video, as the initial part of the stream is usually prebuffered by the video player. Returning the initial part faster improves video startup time without clogging the disk I/O bandwidth of the server.

---
### Applying multiple limitations
The limitations described above can be combined to produce more sophisticated traffic management strategies. For example, you can create two zones for limiting the number of simultaneous connections with different variables and apply multiple limits at once:

```nginx
http {
    limit_conn_zone $binary_remote_addr zone=conn_limit1:12m;
    limit_conn_zone $server_name zone=conn_limit2:24m;
    […]
    server {
        […]
        location /download {
            limit_conn conn_limit1 5;
            limit_conn conn_limit2 200;
        }
    }
}
```

The preceding configuration will limit the number of simultaneous connections per IP address to five; at the same time the total number of simultaneous connections per virtual host will not exceed 200.