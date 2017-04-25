# subr

A subdomain router.

```sh
npm install -g subr
```

```sh
subr -h
```

```
Usage: subr [options]

Options:
  -p, --port    Port(s) to use, comma separated
  -d, --dir     Directory to look for unix domain sockets                             [default: "."]
  -k, --key     File containing ssl key
  -c, --cert    File containing ssl cert
  -t, --tunnel  Tunnel to request
  -h, --help    Show help                                                                  [boolean]

Examples:
  subr                               Connects http://*.localtest.me:<random> to ./*
  subr -p 80,443 -k <key> -c <cert>  Connects http(s)://*.localtest.me to ./*
  subr -p 1234 -k <key> -c <cert>    Connects http(s)://*.localtest.me:1234 to ./*
  subr -t bob.tunnelprovider.com     Connects http(s)://*.bob.tunnelprovider.com to ./*
```

The idea is for your http servers to listen on unix domain sockets. Nodejs allows you to do this simply by specifying a filesystem path where you would usually specify a port. The http library will create the unix domain socket for you:

```js
// hello.js

'use strict';

const fs = require('fs');
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello World\n');
});

const path = `./sockets/hello`;

server.listen(path, (err) => {
  if (err) {
    throw err;
  }

  console.log(`Server running at ${path}`);

  // If you don't do this, you'll get EADDRINUSE after the first run because
  // the file will still be there after this process exits.
  process.on('SIGINT', () => { server.close(); });
});
```

```sh
node hello.js &
subr ./sockets &
```

Then go to http://hello.localtest.me.

Don't forget to cleanup:

```
$ jobs
[1]-  Running                 node hello.js &
[2]+  Running                 subr -d sockets/ &
$ kill %1 %2
```