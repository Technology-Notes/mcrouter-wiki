Mcrouter is a Memcache protocol routing layer that enables the easy expansion of memecached deployments to distributed pools of arbitrary side, providing request routing,
connection pooling, failover, and many other features. Because the routing and feature logic are abstracted from the client in mcrouter deployments, the client does may 
simply communicate with destination hosts thorugh mcrouter over a TCP connection using standard Memcache protocol. 
Typically, little or or no client modification is needed to use mcrouter, which was designed to be a drop-in proxy between the client and memcached
hosts. At Facebook and Instagram, mcrouter is a core component of a distributed cache
infrastructure that spans a very large number of individual memcached boxes.

For information on installing mcrouter, see [Installation](Installation.md).
[There’s a lot more to be added here about significant features that mcrouter easily provides, like migration support, failover, shadow testing production hosts, et cetera]

###Overview

Essentially, mcrouter takes in Memcache requests, applies rules defined by 
mcrouter configs, and sends them on. Configuration allows for flexible
specification of special handling and failover behavior.
[Definitely recommend expanding this section with diagrams and more on system architecture, more of an explanation of design goals, details (what language is mcrouter written in? is it used as a library?)]

###Command line arguments

Assuming you have a memcached instance on the local host running on port 5001,
the simplest mcrouter setup is <code>(::1,/code>, which is the IPv6 loopback address. IPv6 addressesmust be specified in square brackets. You may also use IPv4 addressing, e.g, "127.0.0.1:5001" or
specify a port on localhost, e.g., <code>"localhost:5001"):</code>.
```Shell
./mcrouter --config-str='{"pools":{"A":{"servers":["[::1]:5001"]}},"route":"PoolRoute|A"}' -p 5000
```

To verify mcrouter works, send a request to port 5000. For example, using
Netcat (http://netcat.sourceforge.net/):
```Shell
echo -ne "get key\r\n" | nc 0 5000
```

For a complete list of command line arguments, see `./mcrouter --help`.

###Routing

Mcrouter supports typical Memcache protocol commands like get, set, delete, et cetera,
and commands to access stats, version and so on.
For more information on routing and other Mcrouter commands, see [here](Routing.md).

###Configuration

Mcrouter is configured with JSON files called mcrouter configs, which primarily consist of pool and
route information.  See [Configuration](Configuration.md) for information. 