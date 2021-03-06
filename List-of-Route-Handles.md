### AllAsyncRoute
Immediately sends the same request to all child route handles. Does not wait for response. Returns the default reply for the request right away as returned by [NullRoute](#nullroute).

Properties:
 * `children`: list of child route handles.

### AllFastestRoute
Immediately sends the same request to all child route handles. Returns the first non-error reply to come back, or if all replies are errors, then the last error reply. Other requests complete in the background and their replies are silently ignored.

Properties:
 * `children`: list of child route handles.


### AllInitialRoute
Immediately sends the same request to all child route handles. Waits for the reply from first route handle in the `children` list and returns it. Other requests are completed asynchronously in the background.

Properties:
 * `children`: list of child route handles.


### AllMajorityRoute
Immediately sends the same request to all child route handles. Waits for replies until a non-error result appears (half + 1) times (or all replies if that never happens), and then returns the latest reply that it sees (the effect is that typically the reply with the most common result is returned).

Properties:
 * `children`: list of child route handles.


### AllSyncRoute
Immediately sends the same request to all child route handles. Collects all replies and responds with the "worst" reply (i.e., the error reply, if any).

Properties:
 * `children`: list of child route handles.


### DevNullRoute
Same as [NullRoute](#nullroute), but with mcrouter stats reporting. See [Stats list](Stats-list#stats-logged-to-file).


### ErrorRoute
Immediately returns the error reply for each request. You can specify the error value to return: `"ErrorRoute|MyErrorValueHere"`


### FailoverRoute
Sends the request to the first child in the list and waits for the reply. If the reply is a non-error, returns it immediately. Otherwise, depending on the settings, sends the request to the second child, and so on. If all children respond with errors, returns the last error reply.
_Note_: miss (key not found) doesn't count for an error. See [MissFailoverRoute](#missfailoverroute) to failover misses.

Properties:
 * `children`  
   List of child route handles
 * `failover_errors` (object or array, optional, default: all errors)  
   This property allows specifying for which errors to failover. If this property is missing, we assume default behavior: failover on every error received, independent of the operation. The following example shows how to failover only on `connect_timeout`, `timeout`, `connect_error` and `tko` errors (for a full list of errors, refer to [Errors List](Error-Handling#list-of-errors)).

   ```javascript
   [ "connect_timeout", "timeout", "connect_error", "tko" ]
   ```

   The above config is valid for every operation (`get`, `set`, `delete`, ...). It is possible to go further and tweak the failover behavior per operation type:

   ```javascript
   {
     "gets": [ "connect_timeout", "timeout", "connect_error", "tko" ],
     "updates": [], // empty array: will not failover.
     // "deletes" is missing, default behavior (all errors) will be assumed.
   }
   ```
   To find out more about errors and operations, see [Routing](Routing.md).
 * `failover_limit` (object, optional, default: no limit)  
   ```javascript
   {
     "rate": double, /* 0.0 <= rate <= 1.0 */
     "burst": integer /* 1 <= burst <= 1000000000 */ 
   }
   ```
   We rate limit failover requests over normal requests using [token bucket
   algorithm](http://en.wikipedia.org/wiki/Token_bucket). Rate 0.5 means we'll failover at most 50% of all requests, doesn't matter how many requests we send per second. If we hit the limit, error received from first destination in `children` list be returned immediately.

### FailoverWithExptimeRoute
[FailoverRoute](#failoverroute) with additional settings. Sends request to `normal` route handle.
If it responds with an error, checks `settings` and failovers to `failover`
if necessary.

Properties:
 * `normal`  
   All requests sent here first.
 * `failover`  
   List of route handles to which to route in case of error reply from `normal`.
 * `failover_exptime` (int, optional, default 60)  
   TTL (in seconds) for update requests sent to failover. Useful when failing
   over "sets" to a special pool with a small TTL to protect against
   temporary outages.
 * `settings` (**deprecated**, use `failover_errors` instead)  
 * `failover_errors` (object or array, optional, default: all errors)  
   Same as in [FailoverRoute](#failoverroute). This option replaces `settings`, which is now deprecated.

### HashRoute
Routes to the destination based on key hash.

Properties:
 * `children`  
   List of child route handles.
 * `salt` (string, optional, default empty)  
   Specifies salt to be added to key before hashing
 * `hash_func` (string, optional, default `Ch3`)  
   Which hashing function to use: `Ch3`, `Crc32` or `WeightedCh3`.
   More about hash functions see [here](Pools#hash-functions)
 * `weights` (list of doubles, valid only when `hash_func` is `WeightedCh3`)  
   Weight for each destination. If `weights` has more elements than `children`,
   extra values are ignored. If `weights` has fewer elements than number of
   children, missing values are assumed to be `0.5`.


### HostIdRoute
Routes to one destination chosen based on client host ID.

Properties:
 * `children`: list of child route handles.
 * `salt` (string, optional): if present, mcrouter will use `hash(salt, client host ID)` instead of client host ID.

### LatencyInjectionRoute
Injects latency before and/or after sending the request down to it's child. 
This route handle can be used for disaster simulation scenrios

Properties:
 * `child`: Child route handle.
 * `before_latency_ms`(int, optional): Latency to inject before sending the request to child
 * `after_latency_ms`(int, optional): Latency to inject after sending the request to child

### LatestRoute
Attempts to "behave well" in how many new targets it connects to.
Creates a FailoverRoute with at-most `failover_count` child handles chosen
pseudo-randomly based on client host ID.

Properties:
 * `children`  
   List of child route handles.
 * `failover_count` (int, optional, default 5)  
   Number of route handles to route to.
 * `failover_errors` (object or array, optional, default: all errors)  
   Same as described in [FailoverRoute](#failoverroute).
 * `salt` (string, optional): if present, mcrouter will use `hash(salt, client host ID)` instead of client host ID.

### LoadBalancerRoute
Load balances to a destination from a list of children. The two supported load balancing algorithms are “weighted-hashing” and “two-random-choices”.

In weighted-hashing, requests are hashed between the children taking into account a cached server load for each child.

In two-random-choices, two children are chosen at random and the child with the lowest load is selected.

In both algorithms, the load for each child is cached from a previous response and has a configurable expiry time. On expiry it will be reset to the configured default server load.
 
Properties:
 * `children`
   List of child route handles.
 * `salt` (string, optional, default empty)
   Specifies salt to be added to key before hashing
 * `load_ttl_ms` (int, optional, default 100ms)
   Expiration time in ms before a childs cached server load will be reset to default_server_load_percent
 * `default_server_load_percent` (int, optional, default 50)
   Default server load of each child
 * `failover_count` (int, optional, default 1)
   Number of times to route the request. This is currently only supported in weighted-hashing.
 * `algorithm` (string, optional, default weighted-hashing)
   Algorithm that will be used to load balance. Supports “weighted-hashing” and “two-random-choices”.

### MigrateRoute
This route handle changes behavior based on Migration mode.
 1. Before migration starts, sends all requests to `from` route handle.
 2. Between start_time and (start_time + interval), sends all requests except
    deletes to `from` route handle, sends all delete requests to both
    `from` and `to` route handle. For delete requests, returns reply from
    worst of two replies (i.e., any error reply).
 3. Between (start_time + interval) and (start_time + 2*interval), sends all
    requests except deletes to `to` route handle and sends all delete requests
    to both `from` and `to` route handle. For delete requests, returns reply from
    worst among two replies.
 4. After (start_time + 2*interval), sends all requests to `to` route handle.
 
Properties:
 
 * `from`  
   Route handle that routes to destination from which we are migrating.
 * `to`  
   Route handle that routes to destination to which we are migrating.
 * `start_time` (int)  
   Time in seconds from epoch when migration starts.
 * `interval` (int, options, default 3600)  
   Duration of migration (in seconds)


### MissFailoverRoute
For get-like requests, sends the same request sequentially to each route
handle in the list, in order, until the first hit reply.
If all replies result in errors/misses, returns the reply from the
last destination in the list.

Properties:
 * `children`: list of child route handles.
 * `return_best_on_error` (optional, default is `false`): if `true`, returns the "best" reply if all replies are errors/misses (e.g. miss is better than error), instead of returning the reply from the last destination.

### ModifyExptimeRoute
Modifies exptime (TTL) for all requests.

Properties:
 * `target`: child route handle.
 * `exptime` (int)  
   New TTL (in seconds) for requests.
 * `action` (optional, default is `set`)
   Valid options: `set` or `min`. If action is `set`, always uses new exptime. If action is `min`, uses new exptime only if it's lower than one specified in request, or the one specified in the request is 0.

### NullRoute
Returns the default reply for each request right away. Default replies are:
 
 * `delete` - not found
 * `get` - not found
 * `set` - not stored
 
No properties.

### OperationSelectorRoute (previously: PrefixPolicyRoute)
Sends to different targets based on specified operations.

Properties:

 * `default_policy`  
   Default route handle.
 * `operation_policies`  
   Object, with operation name as key and route handle for specified operation as
   value. Example:

 ```JavaScript
  {
    "type": "OperationSelectorRoute",
    "default_policy": "PoolRoute|A",
    "operation_policies": {
      "delete": {
        "type": "AllSyncRoute",
        "children": [ "PoolRoute|A", "PoolRoute|B" ]
      }
    }
  }
 ```

 Sends gets and sets to pool A, sends deletes to pool A and pool B.
 Valid operations are 'add', 'get', 'set', 'delete', 'incr', 'decr'.
 Route handles for `operation_policies` are parsed in alphabetical order (delete, get, set).

### PoolRoute
Route handle that routes to a pool. With different settings, it provides the same
functionality as [HashRoute](#hashroute), but also allows rate limiting, shadowing, et cetera.

Properties:

 * `pool` (string or object)  
   If it is a string, routes to the pool with the specified name. If it is an object, creates a pool on the fly.
   This object has the same format as item of the `pools` property described earlier, but with
   additional `name` property.
   Example:

 ```JavaScript
  {
    "type": "PoolRoute",
    "pool": {
      "name": "MyPool",
      "servers": [ /* pool hosts here */ ]
    }
  }
 ```

 * `shadows` (optional, default empty)  
   List of objects that define additional route handles to which mcrouter should duplicate routed data (shadow).
   Each object in `shadows` has following properties:
   * `target`  
     Route handle for shadow requests.
   * `index_range` (array of two integers, both in [0, number of servers in pool - 1])  
     Only requests sent to servers from `index_range` will be sent to `target`,
   * `key_fraction_range` (array of two doubles, both in [0, 1])  
     Only requests with key hash from `key_fraction_range` will be sent to
     `target`. The hash used for this purpose is [SpookyHashV2](http://burtleburtle.net/bob/hash/spooky.html).
 
 * `hash` (optional, default `Ch3`)  
   String or object that defines hash function, same as in [HashRoute](#hashroute).
 * `rates` (optional, default no rate limiting)  
   If set, enables rate limiting requests to prevent server overload.
   The object that defines rate and burst are parameters of a [token bucket
   algorithm](http://en.wikipedia.org/wiki/Token_bucket):

 ```JavaScript
  {
    "type": "PoolRoute",
    "pool": "MyPool",
    "rates": {
      "gets_rate": double (optional)
      "gets_burst": double (optional)
      "sets_rate": double (optional)
      "sets_burst": double (optional)
      "deletes_rate": double (optional)
      "deletes_burst": double (optional)
    }
  }
 ```

* `slow_warmup` (optional, default empty)  
   Enables slow warm up for this pool. If enabled, as soon the hit rate of a server in this pool drops bellow a certain threshold, a percentile of the requests will be sent to `failoverTarget` straight away.  
   The slow warmup route has the following properties:
   * `failoverTarget`  
     Route handle for requests when the server is cold. 
   * `settings` (optional)  
     Settings allow customization of slow warm up logic. It has the following properties:
     * `enable_threshold` (number between 0.0 and 1.0)  
     Slow warm up will be enabled as soon as the hit rate of the server drops bellow this threshold.
     * `disable_threshold` (number between 0.0 and 1.0, must be greater than `enable_threshold`)  
     Slow warm up will be disabled as soon as the hit rate of the server goes above this threshold.
     * `start` (optional, default 0.1, must be a number between 0.0 and 1.0)  
     Fraction of requests to send to the server being warmed up when its hit rate is 0.
     * `step` (optional, default 1.0, must be a number between 0.0 and 100.0)  
     Step by which the percentage of requests sent to the server should be incremented when its hit rate increases.  
     * `min_requests` (optional, default 100)  
     Minimum number of requests necessary to start calculating the hit rate. Before this number is reached, the server is considered warm. 

### PrefixSelectorRoute
Sends to different targets based on specified key prefixes.

Properties:

* `wildcard`  
    Default route handle if the key prefix does not match any other specified policies.

* `policies`  
   Object, with operation name as key and route handle for specified operation as
   value. Example:

 ```JavaScript
"route": {
        "type": "PrefixSelectorRoute",
        "policies": {
          "shr": "PoolRoute|shared_pool"
        },
        "wildcard": "PoolRoute|local_pool_in_second_cluster"
      }
```
See [[Prefix routing setup]] for a more detailed example.

**Note:** PrefixSelectorRoute can be only used as a topmost route handle in config tree. It's the only route handle with such restriction.

### RandomRoute
Routes to one random destination from list of children.

Properties:
 * `children`: list of child route handles.

### WarmUpRoute
Allows for substantial changes to the number of boxes in a pool
without increasing the miss rate and, consequently, to the load on the
underlying storage or service.

All sets and deletes go to the target ("cold") route handle.
Gets are attempted on the "cold" route handle and, in case of a miss, data is
fetched from the "warm" route handle (where the request is likely to result
in a cache hit). If "warm" returns a hit, the response is forwarded to
the client and an asynchronous request, with the configured expiration time,
updates the value in the "cold" route handle.

Properties:

 * `cold`  
   Route handle that routes to "cold" destination (with empty cache).
 * `warm`  
   Route handle that routes to "warm" destination (with filled cache).
 * `exptime` (int, optional, in seconds)  
   Expiration time for warm up requests.