canary: a music streaming server/client with DAAP support
=========================================================

`canary` is a package of a music streaming server and its companion iOS client
that run upon
[DAAP](http://en.wikipedia.org/wiki/Digital_Audio_Access_Protocol). Employing
DAAP for streaming and [Avahi](http://www.avahi.org/) for service advertisement
let `canary` work perfectly with [iTunes](https://www.apple.com/itunes/).

This document explains the server. See the files in the `client` directory for
the client.

The server supports, among other things:

- iTunes as a client,
- rescan of songs based on a schedule,
  - it cleverly does nothing unless files or directories change
- authorization via a password,
- delievery of mp3/ogg files if your client can play them and
- multiple paths to contain your songs

but does not support yet:

- adding or editing smart playlists and
- what I don't know yet but you do
  - please let me know about them!

The initial scan of songs takes more time than you might expect; 20 mins with
3,500+ songs on my [Gentoo](https://www.gentoo.org/) machine with
[Intel Atom D525](http://ark.intel.com/products/49490/Intel-Atom-Processor-D525-1M-Cache-1_80-GHz),
4GB RAM and a 5400-rpm HDD. This is mostly because of modules that
`canary-server` depends on to collect metadata from files. Once the database
has been built, however, rescanning is fairly fast; 20 secs on the same
condition. The server remembers the mtime, modification time of files and
reads only added or modified files.

This server implementation makes use of a modified version of
[avahi_pub](https://www.npmjs.com/package/avahi_pub) by
[d1plo1d](https://www.npmjs.com/~d1plo1d) distributed under the
[MIT license](http://opensource.org/licenses/MIT).


#### Prerequisites

- [MongoDB](https://www.mongodb.org/)
- Avahi and its library

Even if Avahi is meant to be compatible with Apple's
[Bonjour](http://www.apple.com/macosx/features/bonjour/) and there are
[node.js modules](https://www.npmjs.com/package/mdns) for
[DNS-SD](http://www.dns-sd.org), the module canary chooses to hire for service
discovery requires a library compatible with Avahi, which prevents you from
installing `canary` on Mac or Linux machines with a mDNSResponder-compatible
library. I'm figuring out an elegant way to resolve this incompatibility.


#### Configuration

Two configuration files need to be provided for the server, one for its
database and the other for the server itself.

The server configuration, `config/server.json` looks like:

```
{
    "name":     "canary music",
    "port":     3689,
    "password": "password",
    "scan": {
        "path":  [ "/path/to/mp3/files" ],
        "cycle": [ "17:00:00" ],
        "utc":   false
    },
    "debug": false
}
```

- the `name` of the server will be published and broadcast via Avahi. If your
  client knows DNS-SD, it will appear on it;
- the server will run on `port`; it must be set to the default port 3689 for
  iTunes to work with the server;
- if `password` is a non-empty string, the server requires a client to send the
  password on every request. This, for example, forces iTunes ask a password on
  its initial connection to the server;
- `scan` specifies the schedule for rescanning files:
  - `path` is an array of directories of music files to serve;
  - `cycle` and `utc`: clear what these mean from their names but you can refer
    to [`ontime`](https://www.npmjs.com/package/ontime) for how to specify the
    rescanning schedule. `canary-server` accepts other options for `ontime`
    except `single` that is always set to true;
- `debug` controls the server's log level. Setting this to true makes the
  server verbose.

`config/db.json` contains:

```
{
    "host":          "localhost",
    "port":          27017,
    "db":            "canary",
    "user":          "user",
    "password":      "password",
    "reconnectTime": 2
}
```

The options from `host` to `password` inclusive specify basic information for
DB connection. If no authentication is required, `user` and `password` can be
omitted.

`reconnectTime` specifies a time interval in _seconds_ for which the server
waits before trying to reconnect when disconnected from the DB.


#### How to run

As other node.js programs, you can run `canary-server` with

    node server.js -c config/

where the `-c` option (or `--config`) specifies a configuration directory the
server will use.


#### Help needed

`canary-server` is implemented in a very short time. It already works well but
needs many improvements that include, but not limited to:

- support for other DBs, especially [MySQL](https://www.mysql.com) and
  [sqlite](http://www.sqlite.org); MongoDB is getting popular but there are
  still many who don't have or are not familiar with it;
- testing for files with various and sometimes weird meta data; metadata of my
  files are normalized so not enough samples to push the server's metadata
  handling.


-------------------------------------------------------------------------------

`INSTALL.md` explains how to build and install the library. For the copyright
issues, see the accompanying `LICENSE.md` file.

If you have a question or suggestion, do not hesitate to contact me via email
(woong.jun at gmail.com) or web (http://code.woong.org/).