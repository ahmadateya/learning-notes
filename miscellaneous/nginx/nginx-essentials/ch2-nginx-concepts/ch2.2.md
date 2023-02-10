# Chapter 2: Managing Nginx (continued)

### Allocating worker processes

Nginx is an `asynchronous web server`, which means actual input/output operations run asynchronously with the execution of a worker process. 

Each worker process runs an event loop that fetches all file descriptors that need processing using a special system call, and then services each of these file descriptors using nonblocking I/O operations. Hence, each worker process serves multiple connections.

In this situation, the time between an event occurs on a file descriptor, and this file descriptor can be serviced (that is latency) depends on how soon a full event processing cycle can be completed. 

Therefore, in order to achieve higher latency it makes sense to penalize the competition for CPU resources between worker processes in favor of more connections per process, because this would reduce the number of context switches between worker processes.

Therefore, `on the systems that are CPU-bound`, it makes sense to allocate as many worker processes as there are CPU cores in the system. 

For example, if your system has eight independent CPU cores. The maximum number of worker processes that will not compete for CPU cores on this system is therefore eight. 

To configure Nginx to start a specified number of worker processes, you can use the `worker_processes` directive in the main configuration file:
```nginx
worker_processes 8;
```

The preceding command will instruct Nginx to start eight worker processes to serve the incoming connections.

**Note** If the number of worker processes is set to a number lower than the number of CPU cores, Nginx will not be able to take advantage of all parallelism available in your system.

To extend the maximum number of connections that can be processed by a worker process, use the `worker_connections` directive:
```nginx
events {
    worker_connections 10000;
}
```

The preceding command will extend the total number of connection that can be allocated to 10,000. This includes both `inbound` (connections from clients) and `outbound` connections (connections to proxied servers and other external resources).


`On disk I/O-bound systems`, in the absence of the AIO facility, additional latency might be introduced into the event cycle due to blocking disk I/O operations.

While a worker process is waiting for a blocking disk I/O operation to complete on a certain file descriptor, the other file descriptors cannot be serviced. 

However, other processes can use the available CPU resources. Therefore, adding worker processes past the number of available I/O channels might not lead to an improvement in performance.

`On systems with mixed resource demands`, a worker process allocation strategy other than the previously mentioned two might be needed to achieve better performance. 

Try varying the numbers of workers in order to obtain the configuration that works best. This can range from one worker to hundreds of workers.

---
### Managing temporary files

Managing temporary files is usually not a big deal, but you must be aware of it. Nginx uses temporary files to store transient data such as the following:
- Large request bodies received from users
- Large response bodies received from proxied servers or via FastCGI, SCGI, or UWCGI protocols.

In the Installing Nginx section of *Chapter 1, Getting Started with Nginx*, you saw the default location of temporary folders for these files. The following table lists the configuration directives that specify temporary folders for various Nginx core modules:

| Directive             | Purpose                                                     |
| --------------------- | ----------------------------------------------------------- |
| client_body_temp_path | Specifies temporary path for client request body data       |
| proxy_temp_path       | Specifies temporary path for responses from proxied servers |
| fastcgi_temp_path     | Specifies temporary path for responses from FastCGI servers |
| scgi_temp_path        | Specifies temporary path for responses from SCGI servers    |
| uwsgi_temp_path       | Specifies temporary path for responses from UWCGI servers   |

The arguments of the preceding directives are as follows
```
proxy_temp_path <path> [<level1> [<level2> [<level3>]]]
```
In the preceding code, `<path>` specifies the path to the directory that contains temporary files, and the levels specify the number of characters in each level of hashed directories.

#### What is a hashed directory? 
In UNIX, a directory in the file system is essentially a file that simply contains a list of entries of that directory. 
So, imagine one of your temporary directories contains 100,000 entries. Each search in this directory routinely scans all of these 100,000 entries, which is not very efficient. To avoid this, you can split your temporary directory into a number of subdirectories, each of them containing a limited set of temporary files.

By specifying levels, you instruct Nginx to split your temporary directory into a set of subdirectories, each having a specified number of characters in its name, for example, a directive:

```
proxy_temp_path /var/lib/nginx/proxy 2;
```
The preceding line of code instructs Nginx to store a temporary file named 3924510929 under the path `/var/lib/nginx/proxy/29/3924510929`.

Likewise, the directive `proxy_temp_path /var/lib/nginx/proxy 1 2` instructs Nginx to store a temporary file named 1673539942 under the path `/var/lib/nginx/proxy/2/94/1673539942`.

As you can see, the characters that constitute the names of the intermediary directories are extracted from the tail of the temporary file name.

Both hierarchical and nonhierarchical temporary directory structures have to be purged from time to time. This could be achieved by walking the directory tree and removing all files residing in those directories. 

#### Side Notes
The commands like `find` and `rm` are dangerous as it blindly removes a broadly-specified set of files. To avoid data loss, stick to the following principles when managing temporary directories:

- Never store anything but temporary files inside temporary directories
- Always use absolute paths in the first argument of a `find` command
- If possible, check what you are about to remove by substituting rm with echo in order to print the list of files to be supplied to `rm`
- Make sure Nginx stores temporary files under a specially-designated user such as nobody or www-data, and never under the superuser
- Make sure the command above runs under a specially-designated user such as nobody or www-data, and never under the superuser

---
### Communicating issues to developers
Developers usually don't have access to production systems, but knowing the environment your Nginx instance is running in is crucial to trace the cause of the problem.

Therefore, you need to provide detailed information about the issue. Detailed information about a crash can be found in the core file that was created after the crash.

#### Warning!
The core file contains a memory dump of a worker process at the moment of a crash and therefore can contain sensitive information, such as passwords, keys, or private data. Therefore, never share core files with people you don't trust.

Instead, use the following procedure to obtain detailed information about a crash:

1. Get a copy of the Nginx binary that you run with debugging information (see following instructions)
2. If a core file is available, run gdb on the binary with the debugging information:
``# gdb ./nginx-binary core``
3. If the run is successful, this will open the gdb prompt. Type bt full in it: 
```
(gdb) bt full
[… produces a dump … ]
```

The preceding command will produce a long dump of the stack at the moment of the crash and it's usually sufficient to debug a wide variety of problems. 

Make a summary of the configuration that resulted in a crash and send it over to the developer along with the full stack trace.

