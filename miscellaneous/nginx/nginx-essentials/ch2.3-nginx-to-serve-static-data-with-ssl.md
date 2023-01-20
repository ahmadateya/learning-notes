# Chapter 2: Managing Nginx (continued)

## Setting up Nginx to serve static data (images, CSS, or JavaScript files.)

a sample configuration from the previous chapter and make it support multiple virtual hosts using wild card inclusion

```nginx
error_log logs/error.log;
worker_processes 8;
events {
    use epoll;
    worker_connections 10000;
}
http {
    include mime.types;
    default_type application/octet-stream;
    include /etc/nginx/site-enabled/*.conf;
}
```

We have set up Nginx to take advantage of eight processor cores (check [Allocating worker processes](ch2.2.md)) and include all configurations files located in `/etc/nginx/site-enabled`.

Next, we will configure a virtual host `static.example.com` for serving static data. The following content goes into the file `/etc/nginx/site-enabled/static`.

example.com.conf:

```nginx
server {
    listen 80;
    server_name static.example.com;
    access_log /var/log/nginx/static.example.com-access.log main;
    sendfile on;
    sendfile_max_chunk 1M;
    tcp_nopush on;
    gzip_static on;
    root /usr/local/www/static.example.com;
}
```

This file configures virtual host `static.example.com`. 
- The virtual host root location is set as `/usr/local/www/static.example.com`. 
- To enable more efficient retrieval of static files, we encourage Nginx to use the `sendfile()` system call `(sendfile on)`.
- and set the maximum sendfile chunk to 1 MB. 
- We also enable the `"TCP_NOPUSH"` option to improve TCP segment utilization when using sendfile() `(tcp_nopush on)`.
- The `gzip_static` on directive instructs Nginx to check for gzipped copies of static files, such as main.js.gz for main.js and styles.css.gz for styles.css. If they are found, Nginx will indicate the presence of the .gzip content encoding, and use the content of the compressed files instead of the original one.

**This configuration** is suitable for virtual hosts that serve small-to-medium size static files.

---
## Installing SSL certificates

Nginx has high-class SSL support and makes it easy for you to configure. Let's walk over the installation procedure of an SSL virtual host.

Before we start, make sure the `openssl` package is installed on your system:
```terminal
# apt-get install openssl
```

### Creating a Certificate Signing Request

You need an SSL certificate in order to set up an SSL virtual host. In order to obtain a real certificate, you need to contact a certification authority to issue an SSL certificate. A certification authority will usually charge you a fee for that.

To issue an SSL certificate, a certification authority needs a **Certificate Signing Request (CSR)** from you. A **CSR** is a message created by you and sent to a certification authority containing your identification data, such as distinguished name, address, and your public key.

To generate a **CSR**, run the following command:
```
openssl req -new -newkey rsa:2048 -nodes -keyout your_domain_name.key -out your_domain_name.csr
```

This will start the process of generating two files: 
1. a private key for the decryption of your SSL certificate (your_domain_name.key).
2. and a certificate signing request (your_domain_name.csr) used to apply for a new SSL certificate.

This command will ask you for your identification data:
- **Country name (C):** This is a two-letter country code, for example, NL or US.
- **State or province (S):** This is the full name of the state you or your company is in, for example, Noord-Holland.
- **Locality or city (L):** This is the city or town you or your company is in, for example, Amsterdam.
- **Organization (O):** If your company or department has &, @, or any other symbol using the Shift key in its name, you must spell out the symbol or omit it to enroll. For example, XY & Z Corporation would be XYZ Corporation or XY and Z Corporation.
- **Organizational Unit (OU):** This field is the name of the department or organization unit making the request.
- **Common name (CN):** This is the full name of the host you are protecting.

The last field is of particular importance here. 

It must match the full name of the host you are protecting. For instance, if you registered a domain `example.com` and users will connect to `www.example.com`, you must enter `www.example.com` into the common name field. If you enter example.com into that field, the certificate will not be valid for www.example.com.

**Note** Do not fill in optional attributes such as e-mail address, challenge password, or the optional company name when generating the CSR. They do not add much value, but just expose more personal data.

### Installing an issued SSL certificate

Once your certificate is issued, you can proceed with setting up your SSL server. Save your certificate under a descriptive name such as your_domain_name.crt.

Move it to a secure directory that only Nginx and superuser have access to. We will use /etc/ssl for simplicity as an example of such a directory.

Now, you can start adding configuration for your secure virtual host:
```nginx
server {
    listen 443;
    server_name your.domain.com;
    ssl on;
    ssl_certificate /etc/ssl/your_domain_name.crt;
    ssl_certificate_key /etc/ssl/your_domain_name.key;
    [… the rest of the configuration ...]
}
```

The name of the domain in the server_name directive must match the value of the common name field in your certificate signing request.

After the configuration is saved, restart Nginx.

### Permanently redirecting from a nonsecure virtual host

This ensures that all requests over plain HTTP to any resource on your web site will be redirected to the identical resource on the SSL virtual host.

```nginx
server {
    listen 80;
    server_name your.domain.com;
    rewrite ^/(.*)$ https://your.domain.com/$1 permanent;
}
```