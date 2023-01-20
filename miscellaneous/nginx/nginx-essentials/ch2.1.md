# Chapter 2: Managing Nginx

In this chapter, we will see how to manage an Nginx instance in operation, and discuss the following topics:
- Reloading and reconfiguring processes
- Starting and stopping Nginx
- Allocating worker processes
- Other management questions

---
## The Nginx connection processing architecture
In the full-scale mode, a single Nginx instance consists of the master process and worker processes, as shown in the following figure:

![Nginx connection processing architecture](/assets/images/nginx/nginx-connection-processing-architecture.png)

The master process spawns worker processes and controls them by sending and forwarding signals and listening for quit notifications from them.

Worker processes wait on listening sockets and accept incoming connections. 

The operating system distributes incoming connections among worker processes in a round-robin fashion.

#### Master process 
The master process is responsible for all startup, shutdown, and maintenance tasks such as the following:
- Reading and re-reading configuration files
- Opening and reopening log files
- Creating listening sockets
- Starting and restarting worker processes
- Forwarding signals to the worker processes
- Starting a new binary

The master process thus ensures continuous operation of an Nginx instance in the face of various changes in the environment and occasional crashes of worker processes.

#### Worker processes 
Worker processes are responsible for serving connections and accepting new ones.

Worker processes can run certain maintenance tasks as well. For instance, they reopen log files on their own after the master process has ensured that this operation is safe. 

Each worker process handles multiple connections. This is achieved by running an event loop that pulls events that occurred on open sockets from the operating system via a special system call, and quickly processing all pulled events by reading from and writing to active sockets. 

Resources required to maintain a connection are allocated when a worker process starts. The maximum number of connections that a worker process can handle simultaneously is configured by the `worker_connections` directive and defaults to `512`.

In a clustered setup, a special routing device such as a load balancer or another Nginx instance is used to balance incoming connections among a set of identical Nginx instances, each of them consisting of a master process and a collection of worker processes.

![cluster setup](/assets/images/nginx/cluster-setup.png)

In this setup, the load balancer routes connections only to those instances that are listening for incoming connections. 

The load balancer ensures that each of the active instances gets an approximately equal amount of traffic, and routes traffic away from an instance if it shows any connectivity problems.

Because of the difference in the architecture, the management procedures for a clustered setup are slightly different than for a **standalone** instance.


#### Note
Every Nginx process sets its process title such that it conveniently reflects the role of the process. 

![nginx processes](/assets/images/nginx/nginx-processes.png)

Here, for example, you see the master process of the instance with process ID 2324 and four worker processes with process IDs 2325, 2326, 2327, and 2328.

Note how the **parent process ID (PPID)** column points at the master process.

---
### Control signals and their usage

Nginx, like any other Unix background service, is controlled by signals. 

**Signals** *are asynchronous events that interrupt normal execution of a process and activate certain functions*. 

The following table lists all signals that Nginx supports and the functions that they trigger:

| Signal    | Function                 |
| --------- | ------------------------ |
| TERM, INT | Fast shutdown            |
| QUIT      | Graceful shutdown        |
| HUP       | Reconfiguration          |
| USR1      | Log file reopening       |
| USR2      | Nginx binary upgrade     |
| WINCH     | Graceful worker shutdown |


All signals must be sent to the master process of an instance. The master process of an instance can be located by looking it up in the process list by `ps -C nginx -f`

---
### Fast shutdown

The `TERM` and `INT` signals are sent to the master process of an Nginx instance to trigger the fast shutdown procedure. All resources such as connections, open files and log files that each worker process is in possession of are immediately closed.


After that, each worker process quits and the master process gets notified. Once all worker processes quit, the master process quits and shutdown is completed.

A fast shutdown obviously causes visible service outage. Therefore, it must be used either in emergency situations or when you are absolutely sure that nobody is using your instance.

---
### Graceful shutdown

Once Nginx receives the `QUIT` signal, it enters graceful shutdown mode. Nginx closes listening sockets and accepts no new connections from then on. 

Existing connections are still served until no longer needed. Therefore, graceful shutdown might take a `long time` to complete, especially if some of the connections are in the middle of a long download or upload.

After you have signaled graceful shutdown to Nginx, you can monitor your process list to see which Nginx worker processes are still running and keep track of the progress of your shutdown procedure.

Once all connections handled by a worker are closed, the worker process quits and the master process gets notified. Once all worker processes quit, the master process quits and shutdown is completed.

**In a clustered or load-balanced setup**, graceful shutdown is a typical way of putting an instance out of operation. Using graceful shutdown ensures that there are no visible outages of your service due to server reconfiguration or maintenance.

**In a single instance**, graceful shutdown can only make sure that existing connections are not closed abruptly. Once graceful shutdown is triggered on a single instance, the service will immediately be `unavailable for new visitors`. To ensure continuous availability on a single instance, use maintenance procedures such as reconfiguration, log file reopening, and Nginx binary update.

---
### Reconfiguration

The `HUP` signal can be used to signal Nginx to reread the configuration files and restart worker processes. This procedure cannot be performed without restarting worker processes, as configuration data structures cannot be changed while a worker process is running.

Once the master process receives the `HUP` signals, it tries to reread the configuration files. If the configuration files can be parsed and contain no errors, the master process signals all the existing worker process to gracefully shut down. After signaling, it starts new worker processes with the new configuration.

As with graceful shutdown, the reconfiguration procedure might take a long time to complete. After you have signaled the reconfiguration to Nginx, you can monitor your process list to see which old Nginx worker processes are still running and keep track of the progress of your reconfiguration.

**Note** If another reconfiguration is triggered during a running reconfiguration procedure, Nginx will start a new collection of worker processes—even though worker processes from the past two rounds have not finished. 
This, in principle, might lead to excessive process table usage, so it's recommended that you wait until the current reconfiguration procedure is finished before starting a new one.

---
### Reopening the log file

Reopening the log file is simple yet extremely important for the continuous operation of your server. When log file reopening is triggered with the `USR1` signal, the master process of an instance takes the list of configured log files and opens each of them. If successful, it closes the old log files and signals worker processes to reopen the log files. Worker processes can now safely repeat the same procedure, and after that the log output is redirected to the new files. After that, worker processes close all old log file descriptors that they currently hold open.

The steps of the log file reopening procedure are as follows:
1. Log files are renamed or moved to new locations via an external tool.
2. You send Nginx the USR1 signal. Nginx closes the old files and opens new ones.
3. Old files are now closed and can be archived.
4. New files are now active and being used.

A typical tool for managing Nginx log files is `logrotate`. The logrotate tool is a quite common tool that can be found in many Linux distributions. Here is an example configuration file for logrotate that automatically performs the log file rotation procedure:

```nginx
/var/log/nginx/*.log {
    daily
    missingok
    rotate 7
    compress
    delaycompress
    notifempty
    create 640 nginx adm
    sharedscripts
    postrotate
    [ -f /var/run/nginx.pid ] && kill -USR1 `cat
    /var/run/nginx.pid`
    endscript
}
```

The preceding script daily rotates each log file it can find in the `/var/log/nginx` folder. The log files are kept until seven files have accumulated. 

The `delaycompress` options specify that the log files should not be compressed immediately after rotation to avoid a situation where Nginx keeps writing to a file being compressed.

Problems in log file rotation procedure can lead to losses of data. Here is a checklist that will help you to configure your log file rotation procedure correctly:
- Make sure the USR1 signal is delivered only after log files are moved. Failure to do so will make Nginx write to rotated files instead of new ones.
- Make sure Nginx has enough rights to create files in the log folder. If Nginx is not able to open new log files, the rotation procedure will fail.

---

### Nginx binary upgrade

Nginx is capable of updating its own binary while operating. This is done by passing listening sockets to a new binary and listing to them in a special environment variable.

further info in the book at pages 35-40