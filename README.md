# Graceful cluster

Install:

    npm install graceful-cluster
    
## How to use

### 1. Enable graceful server shutdown

This patch will prevent active connections reset when server receives `SIGKILL` or `SIGTERM`. Idle (keep-alive) inbound connections without active requests will be destroyed.
 
Example 'server.js':

    // Example server with 'express'.
    var express = require('express');
    var app = express();
    var listener = app.listen(8000);

    var GracefulServer = require('graceful-cluster').GracefulServer;
    var gracefulServer = new GracefulServer({
        server: listener,
        shutdownTimeout: 10 * 1000,             // 10 sec.
    });
    
GracefulServer options description:

 - `server`                - required, http server instance.
 - `log`                   - function, custom log function, `console.log` used by default.
 - `shutdownTimeout`       - ms, force worker shutdown on `SIGTERM` timeout.
 
 Also you can initiate graceful shutdown when needed:
 
    gracefulServer.shutdown();

### 2. Use simplified cluster initialization

This cluster wrapper will send `SIGTERM` signal to workers and wait till they finished all requests.

Also it can gracefully restart all workers one by one with zero cluster downtime on some conditions:

  1. Worker memory used.
  2. Worker time online.
  3. Your custom condition: just call `GracefulCluster.gracefullyRestartCurrentWorker()` to restart current worker in `serverFunction`.
  4. On `SIGUSR2` signal to cluster process.

Example 'cluster.js':

    var GracefulCluster = require('graceful-cluster').GracefulCluster;

    process.title = '<your-cluster-title>';     // Note, process title must be near filename (cluster.js) length, longer title truncated.
    
    GracefulCluster.start({
        shutdownTimeout: 10 * 1000,             // 10 sec.
        restartOnTimeout: 5 * 3600 * 1000,      // 5 hours.
        restartOnMemory: 150 * 1024 * 1024,     // 150 MB.
        serverFunction: function() {
            require('./server');                // Your 'server.js' code module with server logic.
        }
    });

GracefulCluster options description:

 - `serverFunction`        - required, function with worker logic.
 - `log`                   - function, custom log function, `console.log` used by default.
 - `shutdownTimeout`       - ms, force worker shutdown on `SIGTERM` timeout.
 - `disableGraceful`       - disable graceful shutdown for faster debug.
 - `restartOnMemory`       - bytes, restart worker on memory usage.
 - `restartOnTimeout`      - ms, restart worker by timer.

### Gracefully restart cluster

Graceful restart performed by `USR2` signal:

    pkill -USR2 <cluster_process_name>

or

    kill -s SIGUSR2 <cluster_pid>