# Chapter 3: Proxying and Caching (continued)

## Caching
Once Nginx is set up as a reverse proxy, it's logical to turn it into a [`caching proxy`](https://www.techopedia.com/definition/23262/caching-proxy). Fortunately, this can be achieved very easily with Nginx.

### Configuring caches
Before you can enable caching for a certain location, you need to configure a cache.

**A cache:** is a filesystem directory containing files with cached items and a shared memory segment where information about cached items is stored.

A cache can be declared using the `proxy_cache_path` directive:
```nginx
proxy_cache_path <path> keys_zone=<name>:<size> [other parameters...];
```

The preceding command declares a cache rooted at the path `<path>` with a shared memory segment named `<name>` of the size `<size>`.

This directive has to be specified in the http section of the configuration. Each instance of the directive declares a new cache and must specify a unique name for a shared memory segment. 

Consider the following example:
```nginx
http {
    proxy_cache_path /var/www/cache keys_zone=my_cache:8m;
    [...]
}
```

The preceding configuration declares a cache rooted at `/var/www/cache` with a shared memory segment named `my_cache`, which is 8 MB in size. 

Each cache item takes around 128 bytes in memory, thus the preceding configuration allocates space for around 64,000 items.

The following table lists other parameters of `proxy_cache_path` and their meaning:

| Parameter        | Description                                                                                                            |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------- |
| levels           | Specifies hierarchy levels of the cache directory                                                                      |
| inactive         | Specifies the time after which a cache item will be removed from the cache if it was not used, regardless of freshness |
| max_size         | Specifies maximum size (total size) of all cache items                                                                 |
| loader_files     | Specifies the number of files a cache loader process loads in each iteration                                           |
| loader_sleep     | Specifies the time interval a cache loader process sleeps between each iteration                                       |
| loader_threshold | Specifies the time limit for each iteration of a cache loader process                                                  |

Once Nginx starts, it processes all configured caches and allocates shared memory segments for each of the caches.

After that, a special process called cache loader takes care of loading cached items into memory. 

`Cache loader` loads items in iterations. The parameters `loader_files`, `loader_sleep`, and `loader_threshold` define the behavior of the cache loader process.

When running, a special process called cache manager monitors the total disk space taken by all cache items and evicts less requested items if the total consumed space is larger than specified in the `max_size` parameter.

### Enabling caching
To enable caching for a location, you need to specify the cache using the `proxy_cache` directive:
```nginx
location @proxy {
    proxy_pass http://192.168.0.1:8080;
    proxy_cache my_cache;
}
```

The argument of the `proxy_cache` directive is the name of a shared memory segment that points to one of the caches configured using the `proxy_cache_path` directive. 

The same cache can be used in multiple locations. The upstream response will be cached if it is possible to determine the expiration interval for it. The primary source for the expiration interval for Nginx is the upstream itself. 

The following table explains which upstream response header influences caching and how:
| Upstream response header | How it influences caching                                                                                                                                                                                     |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| X-Accel-Expires          | This specifies the cache item expiration interval in seconds. If the value starts from @, then the number following it is UNIX timestamp when the item is due to expire. This header has the higher priority. |
| Expires                  | This specifies the cache item expiration time stamp.                                                                                                                                                          |
| Cache-Control            | This enables or disables caching                                                                                                                                                                              |
| Set-Cookie               | This disables caching                                                                                                                                                                                         |
| Vary                     | The special value * disables caching.                                                                                                                                                                         |

It is also possible to explicitly specify an expiration interval for various response codes using the `proxy_cache_valid` directive:
```nginx
location @proxy {
    proxy_pass http://192.168.0.1:8080;
    proxy_cache my_cache;
    proxy_cache_valid 200 301 302 1h;
}
```
This sets the expiration interval for responses with codes 200, 301, 302 to 1h (1 hour).

Note that the default status code list for the `proxy_cache_valid` directive is 200, 301, and 302, so the preceding configuration can be simplified as follows:
```nginx
location @proxy {
    proxy_pass http://192.168.0.1:8080;
    proxy_cache my_cache;
    proxy_cache_valid 10m;
}
```
To enable caching for negative responses, such as 404, you can extend the status code list in the `proxy_cache_valid` directive:
```nginx
location @proxy {
    proxy_pass http://192.168.0.1:8080;
    proxy_cache my_cache;
    proxy_cache_valid 200 301 302 1h;
    proxy_cache_valid 404 1m;
}
```

The preceding configuration will cache 404 responses for 1m (1 minute). The expiration interval for negative responses is deliberately set to much lower values than that of the positive responses. Such an optimistic approach ensures higher availability by expecting negative responses to improve, considering them as transient and assuming a shorter expected lifetime.

### Choosing a cache key
Choosing the right cache key is important for the best operation of the cache. The cache key must be selected such that it maximizes the expected efficiency of the cache, provided that each cached item has valid content for all subsequent requests that evaluate to the same key. This requires some explanation.

**First**, let's consider `efficiency`. When Nginx refers to the upstream server in order to revalidate a cache item, it obviously stresses the upstream server. With each subsequent cache hit, Nginx reduces the stress on the upstream server in comparison to the situation when requests were forwarded to the upstream without caching.

Thus, the efficiency of the cache can be represented as *`Efficiency = (Number hits + Number misses) / Number misses`*.

Thus, when nothing can be cached, each request leads to a cache miss and the efficiency is 1. But when we get 99 subsequent cache hits for each cache miss, the efficiency evaluates to *`(99 + 1) / 1 = 100`*, which is 100 times larger!

**Second**, if a document is cached but it is not valid for all requests that evaluate to the same key, clients might see content that is not valid for their requests.

For example, the upstream analyses the `Accept-Language` header and returns the version of the document in the most suitable language. If the cache key does not include the language, the first user to request the document will obtain it in their language and trigger the caching in that language. All users that subsequently request this document will see the cached version of the document, and thus they might see it in the wrong language.

If the cache key includes the language of the document, the cache will contain multiple separate items for the same document in each requested language, and all users will see it in the proper language.

The default cache key is `$scheme$proxy_host$request_uri`.
This might not be optimal because of the following reasons:
- The web application server at $proxy_host can be responsible for multiple domains
- The HTTP and HTTPS versions of the website can be identical (`$scheme` variable is redundant, thus duplicating items in the cache)
- Content can vary depending on query arguments

Thus, considering everything described previously and given that HTTP and HTTPS versions of the website are identical and content varies depending on query arguments, we can set the cache key to a more optimal
value `$host$request_uri$is_args$args`. 

To change the default cache item key, you can use the `proxy_cache_key` directive:
```nginx
location @proxy {
    proxy_pass http://192.168.0.1:8080;
    proxy_cache my_cache;
    proxy_cache_key "$host$uri$is_args$args";
}
```
This directive takes a script as its argument which is evaluated into a value of a cache key at runtime.

### References & more
- [Make your backend more reliable using Nginx caching proxy](https://www.sheshbabu.com/posts/nginx-caching-proxy/)