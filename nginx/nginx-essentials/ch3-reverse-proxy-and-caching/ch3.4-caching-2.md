# Chapter 3: Proxying and Caching (continued)

### Improving cache efficiency and availability
The efficiency and availability of the cache can be improved. You can prevent an item from being cached until it gets a certain minimum number of requests.
This could be achieved using the `proxy_cache_min_uses` directive:
```nginx
location @proxy {
    proxy_pass http://192.168.0.1:8080;
    proxy_cache my_cache;
    proxy_cache_min_uses 5;
}
```
In the preceding example, the response will be cached once the item gets no less than five requests. This prevents the cache from being populated by infrequently used items, thus reducing the disk space used for caching.

Once the item has expired, it can be revalidated without being evicted. To enable revalidation, use the `proxy_cache_revalidate` directive:

```nginx
location @proxy {
    proxy_pass http://192.168.0.1:8080;
    proxy_cache my_cache;
    proxy_cache_revalidate on;
}
```

In the preceding example, once a cache item expires, Nginx will revalidate it by making a conditional request to the upstream server. This request will include the `If-Modified-Since` and/or `If-None-Match` headers as a reference to the cached version. If the upstream server responds with a 304 Not Modified response, the cache item remains in the cache and the expiration time stamp is reset.

Multiple simultaneous requests can be prohibited from filling the cache at the same time. Depending on the upstream reaction time, this might speed up cache population while reducing the load on the upstream server at the same time. 

To enable this behavior, you can use the `proxy_cache_lock` directive:
```nginx
location @proxy {
    proxy_pass http://backend;
    proxy_cache my_cache;
    proxy_cache_lock on;
}
```
Once the behavior is enabled, only one request will be allowed to populate a cache item it is related to. The other requests related to this cache item will wait until either the cache item is populated or the lock timeout expires. The lock timeout can be specified using the `proxy_cache_lock` directive.

If higher `availability` of the cache is required, you can configure Nginx to reply with stale data when a request refers to a cached item. This is very useful when Nginx acts as an edge server in a distribution network. 

The users and search engine crawlers will see your web site available, even though the main site experiences connectivity problems. 

To enable replying with stale data, use the `proxy_cache_use_stale` directive:
```nginx
location @proxy {
    proxy_pass http://backend;
    proxy_cache my_cache;
    proxy_cache_use_stale error timeout http_500 http_502 http_503 http_504;
}
```
The following table lists all possible values for arguments of the `proxy_cache_use_stale` directive:

| Value                                                                               | Meaning                                                                                                                         |
| ----------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| error                                                                               | A connection error has occurred or an error during sending a request or receiving a reply has occurred                          |
| timeout A connection timed out during setup, sending a request or receiving a reply |
| invalid_header                                                                      | The upstream server has returned an empty or invalid reply updating Enables stale replies while the cache item is being updated |
| http_500                                                                            | The upstream server returned a reply with HTTP status code 500 (Internal Server Error)                                          |
| http_502                                                                            | The upstream server returned a reply with HTTP status code 502 (Bad Gateway)                                                    |
| http_503                                                                            | The upstream server returned a reply with HTTP status code 503 (Service Unavailable)                                            |
| http_504                                                                            | The upstream server returned a reply with HTTP status code 504 (Gateway Timeout)                                                |
| http_403                                                                            | The upstream server returned a reply with HTTP status code 403 (Forbidden)                                                      |
| http_404                                                                            | The upstream server returned a reply with HTTP status code 404 (Not Found)                                                      |
| off                                                                                 | Disables use of stale replies                                                                                                   |


### Handling exceptions and borderline cases
When caching is not desirable or not efficient, it can be bypassed or disabled.

This can happen in the following instances:
- A resource is dynamic and varies depending on external factors
- A resource is user-specific and varies depending on cookies
- Caching does not add much value
- A resource is not static, for example a video stream

When bypass is forced, Nginx forwards the request to the backend without looking up an item in the cache. The bypass can be configured using the `proxy_cache_bypass` directive:
```nginx
location @proxy {
    proxy_pass http://backend;
    proxy_cache my_cache;
    proxy_cache_bypass $do_not_cache $arg_nocache;
}
```
This directive can take one or more arguments. When any of them evaluate to true (nonempty value and not 0), Nginx does not look up an item in the cache for a given request. Instead, it directly forwards the request to the upstream server. The item can still be stored in the cache.

To prevent an item from being stored in the cache, you can use the `proxy_no_cache` directive:
```nginx
location @proxy {
    proxy_pass http://backend;
    proxy_cache my_cache;
    proxy_no_cache $do_not_cache $arg_nocache;
}
```

This directive works exactly like the `proxy_cache_bypass` directive, but prevents items from being stored in the cache. When only the `proxy_no_cache` directive is specified, the items can still be returned from the cache. The combination of both `proxy_cache_bypass` and `proxy_no_cache` disables caching completely.

#### real-world example 
Assume that you have a website powered by WordPress and you want to enable caching for all pages but disable caching for all customized or user-specific pages. To implement this, you can use a configuration similar to the following:

```nginx
    location ~* wp\-.*\.php|wp\-admin {
        proxy_pass http://backend;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    location / {
        if ($http_cookie ~* "comment_author_|wordpress_|wp-postpass_" ) {
        set $do_not_cache 1;
    }
    proxy_pass http://backend;
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_cache my_cache;
    proxy_cache_bypass $do_not_cache;
    proxy_no_cache $do_not_cache;
}
```

In the preceding configuration, we first delegate all requests pertaining to the WordPress administrative area to the upstream server. We then use the if directive to look up WordPress login cookies and set the `$do_not_cache` variable to 1 if they are present. Then, we enable caching for all other locations but disable caching whenever the `$do_not_cache variable` is set to 1 using the `proxy_cache_bypass` and `proxy_no_cache` directives. This disables caching for all requests with WordPress login cookies.

The preceding configuration can be extended to extract no-cache flags from arguments or HTTP headers, to further tune your caching