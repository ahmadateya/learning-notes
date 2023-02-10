# Chapter 4: Rewrite Engine and Access Control (continued)

### Using basic authentication for access restriction
You can configure Nginx to allow access only to those users who can provide the correct combination of a username and a password. Username/password verification is enabled using the `auth_basic` directive:
```
auth_basic <realm name> | off;
```
Realm name specifies the name of a realm (an authenticated area). This argument is usually set to a string that helps users to identify the area they are trying to access (for example Administrative area, Web mail, and so on). 

This string will be passed to the browser and displayed in the username/password entry dialog. In addition to the realm name, you need to specify a file containing a user database using the `auth_basic_user_file` directive:
```
auth_basic_user_file <path to a file>;
```
This file must contain authentication information with a username and a password in each line:
```nginx
username1:encrypted_password1
username2:encrypted_password2
username3:encrypted_password3
username4:encrypted_password4
username5:encrypted_password5
```
This file presumably must be placed outside of document root of any website you are hosting. The access rights must be set up such that Nginx can only read this file, never write or execute.

Passwords must be encrypted using one of the following algorithms:
| Algorithms | Comments                                     |
| ---------- | -------------------------------------------- |
| CRYPT      | Unix DES-based password encryption algorithm |
| SSHA       | Salted Secure Hash Algorithm 1               |

Deprecated: Do not use
MD5 Message Digest 5 algorithm
SHA Unsalted Secure Hash Algorithm 1

The password file can be managed using the `htpasswd` utility from Apache web server. Here are some examples:

- **Instruction:** Create a password file and add user john to the password file
```
htpasswd -b -d -c /etc/nginx/auth.d/auth.pwd john test
```
- **Instruction:** Add user thomas to the password file
```
htpasswd -b -d /etc/nginx/auth.d/auth.pwd thomas test
```
- **Instruction:** Replace John's password
```
htpasswd -b -d /etc/nginx/auth.d/auth.pwd john test
```
- **Instruction:** Remove user john from the password file
```
htpasswd -D /etc/nginx/auth.d/auth.pwd john
```
#### Further information in the book page 87

#### Good Note
Nginx reads and parses the password file every time a request to protected resources is made. This is scalable only when the number of entries in the password file does not exceed a few hundred

---

### Authenticating users with a subrequest
User authentication can be delegated to another web server using the auth request module. This module must first be enabled at the source code configuration stage using the `–with-http_auth_request_module` command-line switch:
```
$ ./configure –with-http_auth_request_module
$ make
$ make install
```
Now the `auth_request` module is ready to be used

#### #### Further information in the book pages 88, 89

---

### Combining multiple access restriction methods
Multiple access restriction methods can be combined together. For that, they must be both configured and enabled. By default, all configured access restriction methods
must be satisfied in order to allow the request. If any of the access restriction methods
are not satisfied, Nginx rejects the request with 403 Forbidden HTTP status.
This behavior can be changed using the satisfy directive:
satisfy all | any;
Specifying satisfy any in a location makes Nginx accept the request if any of the
enabled access restriction methods are satisfied, while specifying satisfy all (the
default) makes Nginx accept the request only if all enabled access restriction methods
are satisfied.