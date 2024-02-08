# Chapter 1: Getting started with Nginx

Nginx has emerged as a robust and scalable general-purpose web server in the last decade. 

It is a choice of many webmasters, startup founders, and site reliability engineers because of its simple yet scalable and expandable architecture, easy configuration, and light memory footprint. Nginx offers a lot of useful features, such as on-the-fly compression and caching out of the box.


## 1.1 Installing Nginx

- It is strongly recommended that you use `prebuilt` binary packages of Nginx if they are available in your distribution. This ensures best integration of Nginx with your system
and reuse of best practices incorporated into the package by the package maintainer.

- Prebuilt binary packages of Nginx automatically maintain dependencies for you and package maintainers are usually fast to include security patches, so you don't get any complaints from security officers. In addition to that, the package usually provides a distribution-specific startup script, which doesn't come out of the box.

- Refer to your distribution package directory to find out if you have a prebuilt
package for Nginx. Prebuilt Nginx packages can also be found under the **download**
link on the official Nginx.org site.

### Installing Nginx on Ubuntu
```
sudo apt-get install nginx
```
The preceding command will install all the required files on your system, including
the `logrotate` script and service autorun scripts.
| Description                                              | Path/Folder             |
| -------------------------------------------------------- | ----------------------- |
| Nginx configuration files                                | /etc/nginx              |
| Main configuration file                                  | /etc/nginx/nginx.conf   |
| Virtual hosts configuration files (includingdefault one) | /etc/nginx/sitesenabled |
| Custom configuration files                               | /etc/nginx/conf.d       |
| Log files (both access and error log)                    | /var/log/nginx          |
| Temporary files                                          | /var/lib/nginx          |
| Default virtual host files                               | /usr/share/nginx/html   |

**Note** Default virtual host files will be placed into `/usr/share/nginx/html`.
Please keep in mind that this directory is only for the default virtual host. For deploying your web application, use folders recommended by **Filesystem Hierarchy Standard (FHS)**

Now you can start the Nginx service with:
```
 sudo service nginx start
```

### Installing Nginx on Red Hat Enterprise Linux or CentOS/Scientific Linux
Nginx is not provided out of the box in Red Hat Enterprise Linux or CentOS/Scientific Linux. 
Instead, we will use the Extra Packages for Enterprise Linux (EPEL) repository.

**EPEL** is a repository that is maintained by Red Hat Enterprise Linux maintainers, but contains packages that are not a part of the main distribution for various reasons. You can read more about EPEL at https://fedoraproject.org/wiki/EPEL.

further info in the book on page 20

### Installing Nginx from source files
Traditionally, Nginx is distributed in the source code. In order to install Nginx from the source code, you need to download and compile the source files on your system.

It is not recommended that you install Nginx from the source code. Do this only if you have a good reason, such as the following scenarios:
- You are a software developer and want to debug or extend Nginx
- You feel confident enough to maintain your own package
- A package from your distribution is not good enough for you
- You want to fine-tune your Nginx binary

further info in the book on page 21

----
## The structure of the Nginx installation

For each installation method, we have a set of generic locations and default paths.

### 1- The Nginx configuration folder (/etc/nginx) (/etc/nginx/nginx.conf)
This folder contains the main configuration file and a set of parameter files. 
The following table describes the purpose of each of the default parameter files:

| File name                     | Description                                                                                                                                                                                |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| mime.types                    | This contains the default MIME type map for converting file extensions into MIME types.                                                                                                    |
| fastcgi_params                | This contains the default FastCGI parameters required for FastCGI to function.                                                                                                             |
| scgi_params                   | This contains the default SCGI parameters required for SCGI to function.                                                                                                                   |
| uwsgi_params                  | This contains the default UWCGI parameters required for UWCGI to function.                                                                                                                 |
| proxy_params                  | This contains the default proxy module parameters. This parameter set is required for certain web servers when they are behind Nginx, so that they can figure out they are behind a proxy. |
| naxsi.rules (optional)        | This is the main rule set for the NAXSI web application firewall module.                                                                                                                   |
| koi-utf, koi-win, and win-utf | These are the Cyrillic character set conversion tables.                                                                                                                                    |


### 2- The default virtual host folder
- The default configuration contains references to this site as root. 
- It is not recommended that you use this directory for real sites, as `it is not a good practice` for the Nginx folders hierarchy to contain the site hierarchy. 
- Use this directory for testing purposes or for serving auxiliary files.

### 3- The virtual hosts configuration folder
- This is the location of virtual host configuration files. 
- The `recommended structure` of this folder is to have `one file per virtual host` in this folder - Or one `folder per virtual host, containing all files related to this virtual host`. 
- In this way, you will always know which files were used and which are now being used, and what each of the files contain and which files can be purged.

### 3- The log folder
This is the location for Nginx log files. 
The default access log file and error log file will be written to this location. 

### 4- The temporary folder
Nginx uses temporary files for receiving large request bodies, and proxies large files from upstream. Files that are created for this purpose can be found in this folder.

----
## Configuring Nginx

In a nutshell, Nginx configuration files are simply `sequences of directives` that can
take up to `eight space-separated arguments`, for example:
```nginx
gzip_types text/plain text/css application/x-javascript text/xml application/xml application/xml+rss text/javascript;
```
- In the configuration file, the directives are delimited by a semicolon (;) from one
another.
- Some of the directives may have a block instead of a semicolon. 
- A block is delimited by curly brackets `({})`. 
- A block can contain arbitrary text data.

```nginx
types {
 text/html html htm shtml;
 text/css css;
 text/xml xml;
 image/gif gif;
 image/jpeg jpeg jpg;
 application/x-javascript js;
 application/atom+xml atom;
 application/rss+xml rss;
}
```

- A block can also contain a list of other directives. In this case, the block is called a `section`. 
- A section can enclose other sections, thus establishing a hierarchy of sections.

Most important directives have short names; this reduces the effort required to maintain the configuration file.

### Value types
In general, a directive can have arbitrary quoted or unquoted strings as arguments.
But many directives have arguments that have common value types.

| Value type     | Format                    | Example of a value |
| -------------- | ------------------------- | ------------------ |
| Flag           | [on\|off]                 | on, off            |
| Signed integer | -?[0-9]+                  | 1024               |
| Size           | [0-9]+([mM]\|[kK])?       | 23M, 12348k        |
| Offset         | [0-9]+([mM]\|[kK]\|[gG])? | 43G, 256M          |
| Milliseconds   | [0-9]+[yMwdhms]?          | 30s, 60m           |


### Variables
- Variables are named objects that can be assigned a textual value. 
- Variables can only appear inside the `http` section. 
- A variable is referred to by its name, prefixed by the dollar `($)` symbol. 
- Alternatively, a variable reference can enclose a variable name in curly brackets to prevent merging with surrounding text `${}`.
- Examples:
  - `proxy_set_header Host $http_host;`
  -  `proxy_set_header Host ${http_host}_squirrel;`

- Variables from `$1` to `$9` refer to the capture arguments in the regular expressions, as shown here: 

```nginx
location ~ /(.+)\.php$ {
    [...]
    proxy_set_header X-Script-Name $1;
 }
```
The preceding configuration will set the HTTP header X-Script-Name in the forwarded request to the name of the PHP script in the request URI.

The captures are specified in a regular expression using round brackets.

- Variables that start with `$arg_` refer to the corresponding query argument in the original HTTP request, as shown here:
```nginx
proxy_set_header X-Version-Name $arg_ver;
```
The preceding configuration will set the HTTP header X-Version-Name in the forwarded request to the value of the ver query argument in the `original request`.

- Variables that start with `$http_` refer to the corresponding HTTP header line in the `original request`.
- Variables that start with `$sent_http_` refer to the corresponding HTTP header line in the outbound HTTP request.
- Variables that start with `$upstream_http_` refer to the corresponding HTTP header line in the response received from an upstream.
- Variables that start with `$cookie_` refer to the corresponding cookie in the `original request`.
- Variables that start with `$upstream_cookie_` refer to the corresponding cookie in the response received from an upstream.

- Variables must be declared by Nginx modules before they can be used in the configuration. 
- Built-in Nginx modules provide a set of core variables that allow you to operate with the data from HTTP requests and responses. Refer to the Nginx documentation for the complete list of core variables and their functions.
- Third-party modules can provide extra variables. These variables have to be described in the third-party module's documentation.

### Inclusions
Any Nginx configuration section can contain inclusions of other files via the `include` directive. This directive takes a single argument containing a path to a file to be included, as shown here:
```nginx
/*
 * A simple relative inclusion. The target file's path
 * is relative to the location of the current configuration file.
 */
include mime.types;
/*
 * A simple inclusion using an absolute path.
 */
include /etc/nginx/conf/site-defaults.conf;
```