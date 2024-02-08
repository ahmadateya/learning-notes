# Chapter 1: Getting started with Nginx (continued)

## 1.3 Configuration settings' inheritance rules

Many Nginx configuration settings can be inherited from a section of outer level to a section of inner level. This saves a lot of time when you configure Nginx.

The following figure illustrates how inheritance rules work:

![Configuration settings' inheritance rules](/assets/images/nginx/configuration-settings-inheritance-rules.png)

All settings can be attributed to three categories:
- Those that make sense only in the entire HTTP service `(marked red)`
- Those that make sense in the virtual host configuration `(marked blue)`
- Those that make sense on all levels of configuration `(marked green)`

#### First category (Those that make sense only in the entire HTTP service)
The settings from the first category do not have any inheritance rules, because they cannot inherit values from anywhere. 
- They can be specified in the http section only and can be applied to the entire HTTP service. 
- These are settings set by directives, such as `variables_hash_max_size`, `variables_hash_bucket_size`, `server_names_hash_max_size`, and `server_names_hash_bucket_size`.


#### Second category (Those that make sense in the virtual host configuration)
The settings from the second category can inherit values only from the http section.
- They can be specified both in the http and server sections, but the settings applied to a given virtual host are determined by inheritance rules. 
- These are settings set by directives, such as `client_header_timeout`, `client_header_buffer_size`, and `large_client_header_buffers`.


#### Third category (Those that make sense on all levels of configuration)
The settings from the third category can inherit values from any section up to `http`. 

They can be specified in any section inside the HTTP service configuration, and the settings applied to a given context are determined by inheritance rules.

The arrows on the figure illustrate value propagation paths. 
The colors of the arrows specify the scope of the setting. 

The propagation rules along a path are as follows:
- When you specify a value for a parameter at a certain level of the configuration, it overrides the value of the same parameter at the outer levels if it is set, and automatically propagates to the inner levels of the configuration. 
  
Let's take a look at the following example:

```nginx
location / {
 # The outer section

 root /var/www/example.com;
 gzip on;
 location ~ \.js$ {
    # Inner section 1
    gzip off;
 }
 location ~ \.css$ {
    # Inner section 2
 }
 [...]
}
```

The value of the root directive will propagate to the inner sections, so there is no need to specify it again. 

The value of the gzip directive in the outer section will propagate to the inner sections, but will be overridden by the value of the `gzip` directive inside the first inner section. 

The overall effect of that will be that gzip compression will be enabled everywhere in the other section, except for the first inner section.

When a value for some parameter is not specified in a given configuration section, it is inherited from a section that encloses the current configuration section. 

If the enclosing section does not have this parameter set, the search goes to the outer level and so on. 

If a value for a certain parameter is not specified at all, a built-in default value is used.

----
### First sample configuration
Lets study a short but functioning configuration that will give you an idea of what a complete configuration file might look like:

```nginx
error_log logs/error.log;
events {
    use epoll;
    worker_connections 1024;
}
http {
    include mime.types;
    default_type application/octet-stream;
    server {
        listen 80;
        server_name example.org www.example.org;
        location / {
            proxy_pass http://localhost:8080;
            include proxy_params;
        }
        location ~ ^(/images|/js|/css) {
            root html;
            expires 30d;
        }
    }
}
```

- This configuration first instructs Nginx to write the error log to logs/error.log.
- Then, it sets up Nginx to use the epoll event processing method (`use epoll`) and allocates memory for 1024 connections per worker (`worker_connections 1024`).
- After that, it enables the HTTP service and configures certain default settings for the HTTP service (`include mime.types, default_type application/octet-stream`).
- It creates a virtual host and sets its names to example.org and www.example.org (`server_name example.org www.example.org`). 
- The virtual host is made available at the default listening address 0.0.0.0 and port 80 (listen 80).

- We then configure two locations. The first location passes every request routed to it into a web application server running at http://localhost:8080 (`proxy_pass http://localhost:8080`). 
- The second location is a regular expression location. By specifying it we effectively exclude a set of paths from the first location. 
- We use this location to return static data such as images, JavaScript files, and CSS files. We set the base directory for our media files as html (`root html`). 
- For all media files, we set the expiration date as 30 days (`expires 30d`).

----
### Configuration best practices
Here is a list of recommendations that will help you to maintain your configuration more efficiently and make it more robust and manageable:
- **Structure your configuration well**. Observe which common parts of the configuration are used more often, move them to separate files, and reuse them using the include directive. In addition to that, try to make each file in your configuration file hierarchy of a reasonable length, ideally no more than two screens. This will help you to read your files quicker and navigate over them efficiently.
- **Minimize use of the `if` directive**. The `if` directive has a nonintuitive behavior. Try to avoid using it whenever possible to make sure configuration settings are applied to the incoming requests as you expect.
- **Use good defaults**. Experiment with inheritance rules and try to come up with defaults for your settings so that they result in the least number of directives to be configured. 
 
This includes moving common settings from location to the server level and further to the HTTP level.