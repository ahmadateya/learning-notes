# Chapter 4: Rewrite Engine and Access Control (continued)

### Patterns
The following table gives a brief overview of regular expression syntax used in rewrite rules:
| Pattern                        | Examples                  | Description                                  |
| ------------------------------ | ------------------------- | -------------------------------------------- |
| \<pattern A\> \<pattern B\>    | Ab, (aa)(bb)              | Following                                    |
| \<pattern A\> \| \<pattern B\> | a    \| b  , (aa) \| (bb) | Alternative                                  |
| \<pattern\>                    | ? (\.gz)?                 | Option                                       |
| \<pattern\>                    | * A*, (aa)*               | Repetition of \<pattern\> from 0 to infinity |
| \<pattern\>                    | + a+, (aa)+               | Repetition of \<pattern\> from 1 to infinity |
| \<pattern\>                    | {n} a{5}, (aa){6}         | Repetition of \<pattern\> n times            |
| \<pattern\>                    | {n,} a{3,}, (aa){7,}      | Repetition of \<pattern\> from n to infinity |
| \<pattern\>                    | {,m} a{,6}, (aa){,3}      | Repetition of \<pattern\> from 0 to m        |
| \<pattern\>                    | {n,m} a{5,6}, (aa){1,3}   | Repetition of \<pattern\> from n to m        |
| ( \<pattern\> )                | (aa)                      | Grouping or parameter capture                |
| .                              | .+                        | Any character                                |
| ^                              | ^/index                   | Start of line                                |
| $                              | \.php$                    | End of line                                  |
| [\<characters\>]               | [A-Za-z]                  | Any character from the specified set         |
| [^\<characters\>]              | [^0-9]                    | Any character outside of the specified set   |

The patterns are listed in increasing priority order. That is, the pattern `aa|bb` will be interpreted as `a(a|b)b`, while the pattern `a{5}aa{6}` will be interpreted as `(a{5}) (a)(a{6})` and so on.

To specify characters that are themselves part of regular expression syntax, you can use the backslash character \, for example \* will match an asterisk *, \. will match a dot character ., \\ will match the backslash character itself and \{ will match an opening curly bracket {.

### Captures and positional parameters
Captures are designated with round brackets and mark sections of matched URLs that need to be extracted. Positional parameters refer to substrings of the matched URLs extracted by corresponding capture, that is, if the pattern is as follows:
```nginx
    ^/users/(.+)/(.+)/$
```
Also, if the request URL is like this:
```nginx
    /users/id/23850/
```
The positional parameters `$1` and `$2` will evaluate to id and `23850`, respectively.

**Positional parameters** can be used in any order within the substitution string and this is how you connect it with the match pattern.

### Other functionalities of the rewrite engine

The rewrite engine can also be used to perform other tasks:
- Assigning variables
- Evaluating predicates using the if directive
- Replying with specified HTTP status code

A combination of these operations and rewrite rules can be performed at every pass of the rewrite engine. Note that if sections are separate locations, so it is still possible that the location will change at the location rewrite pass.

#### Assigning variables
Variables can be assigned using the `set` directive:
```
set $fruit "apple";
```
Variable values can be scripts with text and other variables:
```
set $path "static/$arg_filename";
```
Once set on the rewrite phase, variables can be used in any directive in the rest of the configuration file.

#### Evaluating predicates using if sections
You have probably figured out from the title that if sections are part of the rewrite engine. This is true. The if sections can be used to conditionally apply selected rewrite rules:
```nginx
if ( $request_method = POST ) {
    rewrite ^/media/$ /upload last;
}
```

In the preceding configuration, any attempt to make a POST request to the URL `/media/` will result in rewriting it to the URL `/upload`, while requests with other methods to the same URL will result in no rewrites. Multiple conditions can also be combined. Let's look at the following code:
```nginx
set $c1 "";
set $c2 "";

if ( $request_method = POST ) {
    set $c1 "yes";
}

if ( $scheme = "https" ) {
    set $c2 "yes";
}

set $and "${c1}_${c2}";
if ( $and = "yes_yes" ) {
    rewrite [...];
}
```

The preceding configuration applies the rewrite only when both if conditions are fulfilled, that is, when the request method is POST and the request URL scheme is https.

Now that you know how you can use the if section, let's talk about its side effects.

Remember that conditions in the if directives are evaluated in the course of the rewrite directive processing. What it means is that when the if section contains directives that are not part of the rewrite engine, the behavior of the if section becomes non-intuitive. 

Consider the following configuration:
```nginx
if ( $request_method = POST ) {
    set $c1 "yes";
    proxy_pass http://localhost:8080;
}
if ( $scheme = "https" ) {
    set $c2 "yes";
    gzip on
}
```
- Each individual `if` section contains an atomic set of configuration settings.
- Assume Nginx receives a `POST` request with the https URL scheme such that both conditions evaluate to true. 
- All set directives will be correctly processed by the rewrite engine and will be assigned to proper values. However, Nginx cannot merge other configuration settings and cannot have multiple configurations active at once.
- When rewrite processing is finished, Nginx simply switches configuration to the last `if` section with its conditions evaluated to true. Because of that, in the preceding configuration, compression will be switched on but the request will not be proxied according to `proxy_pass` directive. This is not something you might expect.

To avoid this non-intuitive behavior, stick to the following best practices:
- Minimize the usage of the `if` directive
- Combine the `if` evaluations using the set directive
- Take actions only in the last `if` section.

### Replying with a specified HTTP status code
If a definite reply with a specified HTTP status code is required in a certain location, you can use the `return` directive to enable this behavior and specify the status code, a reply body, or a redirect URL. 

Let's look at the following code:
```nginx
location / {
    return 301 "https://www.example.com$uri";
}
```

The preceding configuration will execute a permanent redirect (301) to the secure part of domain `www.example.com` and the URI path identical to the URI path in the original request. Thus, the second argument of the return directive will be treated as a redirect URI. 

The other status codes that treat the second argument of the return directive as a redirect URI are `302`, `303` and `307`.

Performing a redirect with the `return` directive is much faster than doing so with the rewrite directive, because it does not run any regular expressions. Use the `return` directive in your configuration instead of the `rewrite` directive whenever possible

The status code 302 is quite common, so the return directive has a simplified syntax for temporary redirects:
```nginx
location / {
    return "https://www.example.com$uri";
}
```

As you can see, if the return directive has a single argument, it is treated as redirect URI and makes Nginx perform a temporary redirect. This argument must start from http:// or https:// to trigger such behavior.


The `return` directive can be used to return a reply with a specified body. To trigger such behavior, the status code must simply be other than 301, 302, 303 or 307. The second argument of the return directive specified the content of the response body:
```nginx
location /disabled {
    default_type text/plain;
    return 200 "OK";
}
```

The preceding configuration will return HTTP status 200 (OK) with the specified response body. To assert correct processing of the body content, we set response content type to `text/plain` using the `default_type` directive.
