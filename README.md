# Graceful cluster

    npm install --save graceful-cluster
    
## How to use

### 1. Enable graceful server shutdown

This patch will preve
  
Example 'server.js':

    // Example server with 'express'.
    var express = require('express');
    var app = express();
    var listener = app.listen(8000);

    var GracefulServer = require('graceful-cluster').GracefulServer;
    new GracefulServer({
        server: listener,
        shutdownTimeout: 10 * 1000,             // 10 sec.
    });

GracefulServer options description:

 - 'server'                - required, http server instance.
 - 'log'                   - function, custom log function, console.log used by default.
 - 'shutdownTimeout'       - ms, force worker shutdown on SIGTERM timeout.

### 2. Use simplified cluster initialization.

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

 - 'serverFunction'        - required, function with worker logic.
 - 'log'                   - function, custom log function, console.log used by default.
 - 'shutdownTimeout'       - ms, force worker shutdown on SIGTERM timeout.
 - 'disableGraceful'       - disable graceful shutdown for faster debug.
 - 'restartOnMemory'       - bytes, restart worker on memory usage.
 - 'restartOnTimeout'      - ms, restart worker by timer.

### Gracefully restart cluster

Graceful restart performed by USR2 signal:

    pkill -USR2 <cluster_process_name>

or

    kill -s SIGUSR2 <cluster_pid>