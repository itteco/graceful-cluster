# Graceful cluster

    npm install --save graceful-cluster
    
## How to use

### 1. Enable graceful server shutdown

  
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

### 2. Use simplified cluster initialization.

Example 'cluster.js':

    var GracefulCluster = require('graceful-cluster').GracefulCluster;

    process.title = 'cluster-title';
    
    GracefulCluster.start({
        shutdownTimeout: 10 * 1000,             // 10 sec.
        restartOnTimeout: 5 * 3600 * 1000,      // 5 hours.
        restartOnMemory: 150 * 1024 * 1024,     // 150 MB.
        serverFunction: function() {
            require('./server');                // Your 'server.js' code module with server logic.
        }
    });

GracefulCluster options description:

 - options.serverFunction        - required, function with worker logic.
 - options.log                   - function, custom log function, console.log used by default.
 - options.shutdownTimeout       - ms, force worker shutdown on SIGTERM timeout.
 - options.disableGraceful       - disable graceful shutdown for faster debug.
 - options.restartOnMemory       - bytes, restart worker on memory usage.
 - options.restartOnTimeout      - ms, restart worker by timer.
