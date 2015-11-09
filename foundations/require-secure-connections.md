#### Require Secure Connections

Require secure connections with TLS to access the API, without exception.
It’s not worth trying to figure out or explain when it is OK to use TLS
and when it’s not. Just require TLS for everything.

Ideally, simply reject any non-TLS requests by not responding to requests for
http or port 80 to avoid any insecure data exchange. In environments where this
is not possible, respond with `403 Forbidden`.

Redirects are discouraged since they allow sloppy/bad client behaviour without
providing any clear gain.  Clients that rely on redirects double up on
server traffic and render TLS useless since sensitive data will already
 have been exposed during the first call.
