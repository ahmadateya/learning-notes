# Chapter 4: Rewrite Engine and Access Control

## Basics of the rewrite engine
The rewrite engine allows you to manipulate the request URI of `inbound requests`.

The rewrite engine is configured using rewrite rules. 

**Rewrite rules** are used when the request URI needs to undergo transformation before further processing. Rewrite rules instruct Nginx to match the request URI with a regular expression and substitute the request URI with a specified pattern whenever a match has been scored.

Rewrite rules can be specified inside `server`, `location`, and `if` sections of the configuration.

Consider a simple case when one resource needs to be substituted by another:
```nginx
location / {
    rewrite ^/css/default\.css$ /css/styles.css break;
    root /var/www/example.com;
}
```

- With the preceding configuration, every request to `/css/default.css` will have its URI rewritten to `/css/styles.css` and will fetch this resource instead. 

- The `rewrite` directive specifies a pattern that has to match the request URI in order to fire the rule and a substitution string that says how the request URI must look after transformation. 

- The third argument, `break`, is a flag that instructs Nginx to stop processing rewrite rules once a match for this rule has been scored.


#### multiple resources
- The preceding configuration can be extended to work with multiple resources as well. For that, you need to use captures (round brackets in the first argument) and positional parameters (variables with numbers that refer to captures):

```nginx
location / {
    rewrite ^/styles/(.+)\.css$ /css/$1.css break;
    root /var/www/example.com;
}
```
With the preceding configuration, every request to any CSS file in `/styles/` will have its URI rewritten to the corresponding resource in `/css/`.

In the last two examples, we used the `break` flag in order to stop rewrite rules from processing as soon as a match is found (assuming more rules can be added to those configurations). If we want to combine those two examples, we need to drop the `break` flag and allow the cascading application of rewrite rules:

```nginx
location / {
    rewrite ^/styles/(.+)\.css$ /css/$1.css;
    rewrite ^/css/default\.css$ /css/styles.css;
    root /var/www/example.com;
}
```

Now, every request to style sheets in `/styles/` will be redirected to the corresponding resource in `/css/`, and `/css/default.css` will be rewritten to `/css/styles.css`.

A request to `/styles/default.css` will undergo two rewrites, as it sequentially matches both rules.

Notice that all URI transformations are performed by Nginx internally. This means that for an external client, the original URIs return ordinary resources, thus the previous configurations will externally look like a series of documents with identical content (that is, `/css/default.css` will be identical to `/css/styles.css`).

This is not a desirable effect in the case of ordinary web pages, as search engines might penalize your website for duplicate content.

#### How to avoid duplicate content problem
To avoid this problem, it is necessary to replace copies of a resource with permanent
redirects to the master resource, as shown in the following configuration:
```nginx
location / {
    rewrite ^/styles/(.+)\.css$ /css/$1.css permanent;
    root /var/www/example.com;
}
```

This works well for whole sections of a website:
```nginx
location / {
    rewrite ^/download/(.+)$ /media/$1 permanent;
    root /var/www/example.com;
}
```
It also works for an entire virtual host:
```nginx
server {
    listen 80;
    server_name example.com;
    rewrite ^/(.*)$ http://www.example.com/$1 permanent;
}
```

The preceding configuration for any URL requested performs a permanent redirect from a top-level domain example.com to the www sub domain, making it the primary entry point of the website.

### Semantic URLs
The next powerful application of rewrite rules is translating a semantic URL into a URL with a query (section of a URL after the ? character). This functionality has its primary application in Search Engine Optimization (SEO) and website usability, and it is driven by a need to obtain semantic URLs for each and every resource and to deduplicate the content.

Consider the following configuration:
```nginx
server {
    [...]
    rewrite ^/products/$ /products.php last;
    rewrite ^/products/(.+)$ /products.php?name=$1 last;
    rewrite
    ^/products/(.+)/(.+)/$ /products.php?name=$1&page=$2 last;
    [...]
}
```
The preceding configuration transforms URLs consisting of a number of path sections starting with `/products` into a URL starting with `/products.php` and arguments. In this way, it is possible to hide implementation details from users and search engines, and generate semantic URLs

Note that the flags of the rewrite directives are now set to `last`. This makes Nginx seek a new location for a rewritten URL and process request with a newly-found location.

## More about rewrite rules
The complete syntax of the rewrite directive:
```nginx
rewrite <pattern> <substitution> [<flag>];
```
- **<pattern>**, is a regular expression that needs to match the request URI in order to activate the substitution.

- **<substitution>** argument is a script that is evaluated once a match has been scored and the value produced by evaluation replaces the request URI. 
  - Special variables $1...$9 can be used to cross-reference a pattern and its substitution by referring to a capture with the specified position. 
- **<flag>** argument affects the behavior of the rewrite directive. 

### List all possible flags of the rewrite directive and their functions

| Flag      | Function                                                                               |
| --------- | -------------------------------------------------------------------------------------- |
| break     | Interrupts processing of rewrite rules                                                 |
| last      | Interrupts processing of rewrite rules and looks up a location for the new request URI |
| redirect  | Returns a temporary redirect (HTTP status 302) to the new request URI                  |
| permanent | Returns a permanent redirect (HTTP status 301) to the new request URI                  |

### multiple passes
The rewrite engine makes multiple passes before a location for the request is found, and then in the request location and subsequent locations that the request is redirected to (such as those that are invoked by the `error_page` directive).

Rewrite rules specified directly in the `server` section are processed in the **first pass**, while rewrite rules in the `location`, `if`, and other sections within the `server` section are processed at subsequent passes. 

Consider the following example:
```nginx
server {
    <rewrite rules here are processed in the first pass>;
    
    location /a {
        <rewrite rules here are processed in subsequent passes>;
    }
    
    location /b {
        <rewrite rules here are processed in subsequent passes>;
    }
}

After the first pass is complete, Nginx searches for a location that matches the rewritten request URI if a rewrite was performed, or a location that matches the original request URI (if no rewrite took place). 

The subsequent passes alter the request URI without changing the location.

Rewrite rules at each pass are processed in order of appearance. Once a match is scored, the substitution is applied and processing resumes with subsequent rewrite rules—unless a flag is specified to interrupt processing.

If the resulting request URI starts with http:// or https://, it is treated as absolute and Nginx returns a temporary (302 "Found") or a permanent (301 "Moved Permanently") redirect to the resulting location.