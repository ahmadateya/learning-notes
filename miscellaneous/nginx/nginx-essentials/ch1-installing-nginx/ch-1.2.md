# Chapter 1: Getting started with Nginx (continued)

## 1.2 Sections
A section is a directive that encloses other directives in its block. Each section's delimiters must be located in the same file, while the content of a section can span multiple files via the include directive.

- **Note** It is not possible to describe every possible configuration directive in this chapter.
Refer to the [Nginx](nginx.com) documentation for more information. 

However, this is a quick go over the Nginx configuration section types so that you can orient in the structure of the Nginx configuration files.

### The http section

The `http` section enables and configures the HTTP service in Nginx. It has the server and upstream declarations. 

As far as individual directives are concerned, the `http` section usually contains those that specify defaults for the entire HTTP service.

The `http` section must contain `at least one server section` in order to process HTTP requests. 

Here is a typical layout of the http section:
```nginx
 http {
    [...]
    server {
      [...]
    }
 }
```

**Note** => […] refer to omitted irrelevant parts of the configuration.

### The server section
The `server` section configures an HTTP or HTTPS virtual host and specifies listening addresses for them using the `listen` directive. 

At the end of the configuration stage, all listening addresses are grouped together and all listening addresses are activated at startup.

The `server` section contains the `location` sections, as well as sections that can be enclosed by the location section (see description of other sections types for details).

Directives that are specified in the server section itself go into the so-called `default location`. 
In that regard, the server section serves the purpose of the location section itself.

When a request comes in via one of the listening addresses, it is routed to the server sections that match a virtual host pattern specified by the `server_name` directive.

The request is then routed further to the location that matches the path of the request URI or processed by the default location if there is no match.

### The upstream section
The `upstream` section configures a `logical server` that Nginx can pass requests to for further processing. 

This logical server can be configured to be backed by one or more physical servers external to Nginx with concrete domain names or IP addresses.

**Upstream** can be referred to by name from any place in the configuration file where a reference to a physical server can take place. In this way, your configuration can be made independent of the underlying structure of the upstream, while the upstream structure can be changed without changing your configuration.


### The location section
The `location` section is one of the workhorses in Nginx. The location directive takes parameters that specify a pattern that is matched against the path of the request URI. 

When a request is routed to a location, Nginx activates configuration that is enclosed by that location section.

There are three types of location patterns: `simple`, `exact`, and `regular expression location` patterns.

#### 1- Simple
A simple location has a string as the first argument. When this string matches the initial part of the request URI, the request is routed to that location. 
Here is an example of a simple location:
```nginx
 location /images {
    root /usr/local/html/images;
 }
```
Any request with a URI that starts with /images, such as /images/powerlogo.png, or /images/social/github-icon.png will be routed to this location. A URI with a path that equals to /images will be routed to this location as well.

#### 2- Exact
Exact locations are designated with an equals (=) character as the first argument and have a string as the second argument, just like simple locations do. 

Essentially, exact locations work just like simple locations, except that the path in the request URI has to match the second argument of the location directive exactly in order to be routed to that location:
```nginx
location = /images/empty.gif {
  emptygif;
}
```
The preceding configuration will return an empty GIF file `if and only if` the URI /images/empty.gif is requested.

#### 3- Regular expression locations
Regular expression locations are designated with a tilde `(~)` character or `~*` (for case-insensitive matches) as the first argument and have a regular expression as the second argument. 

- Regular expression locations are processed after both simple and exact locations. 

The path in the request URI has to match the regular expression in the second argument of the location directive in order to be routed to that location. 

A typical example is as follows:
```nginx
location ~ \.php$ {
  [...]
}
```
- According to the preceding configuration, requests with URIs that end with .php will be routed to this location.
- The location sections can be nested. For that, you just need to specify a location section inside another location section.

### The if section
The `if` section encloses a configuration that becomes active once a condition specified by the `if` directive is satisfied. 
The if section can be enclosed by the server and `location` sections, and is only available if the `rewrite` module is present.

A condition of an if directive is specified in round brackets and can take the following forms:
- A plain variable, as shown here:
```nginx
if ($file_present) {
  limit_rate 256k;
}
```
If the variable evaluates to true value in runtime, the configuration section activates.

- A `unary expression` that consists of an operator and a string with variables, as shown here:
```nginx
if ( -d "${path}" ) {
  try_files "${path}/default.png" "${path}/default.jpg";
}
```
The following `unary operators` are supported:

| Operator | Description                                          | Operator | Description                                                     |
| -------- | ---------------------------------------------------- | -------- | --------------------------------------------------------------- |
| -f       | True if specified file exists                        | !-f      | True if specified file does not exist                           |
| -d       | True if specified directory exists                   | !-d      | True if specified directory does not exist                      |
| -e       | True if specified file exists and is a symbolic link | !-e      | True if specified file does not exist or is not a symbolic link |
| -x       | True if specified file exists and is executable      | !-x      | True if specified file does not exist or is not executable      |

- A `binary expression` that consists of a variable name, an operator, and a string with variables. The following binary operators are supported:

| Operator | Description                                                                   | Operator | Description                                                                          |
| -------- | ----------------------------------------------------------------------------- | -------- | ------------------------------------------------------------------------------------ |
| =        | True if a variable matches a string                                           | !=       | True if a variable does not match a string                                           |
| ~        | True if a regular expression matches the value of a variable                  | !~       | True if a regular expression does not match the value of a variable                  |
| ~*       | True if a case-insensitive regular expression matches the value of a variable | !~*      | True if a case-insensitive regular expression does not match the value of a variable |


- The `if` directive seems like a powerful instrument, but it must be used with caution.
- This is because the configuration inside the if section `is not imperative`, that is, it does not alter the request processing flow according to the order of the if directives.

- **Note** Conditions are not evaluated in the order they are specified in the configuration file. They are merely applied simultaneously and configuration settings from the sections for which conditions were satisfied are merged together and applied at once.

### The limit_except section
The `limit_except` section activates the configuration that it encloses if the request method does not match any from the list of methods specified by this directive.

Specifying the GET method in the list of methods automatically assumes the HEAD method. 

This section can only appear inside the location section, as shown here:
```nginx
limit_except GET {
  return 405;
}
```
The preceding configuration will respond with HTTP status 405 ("Method Not Allowed") for every request that is not made using the GET or HEAD method.

### Other section types
Nginx configuration can contain other section types, such as main and server in the main section, as well as section types provided by third-party modules. 

In this book, we will not pay close attention to them.
Refer to the documentation of the corresponding modules for information about these types of configuration sections.