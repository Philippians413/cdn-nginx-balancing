### Preparations

Check config: `sudo nginx -t -c ${PWD}/nginx.conf`

Start test environment: `docker-compose up -d`

### Test default load balancing configuration

```
$ curl localhost:80
<h4>This is node1 server</h4>
$ curl localhost:80
<h4>This is node2 server</h4>
$ curl localhost:80
<h4>This is node3 server</h4>
```

### Test least connected load balancing

Let's add `least_conn;` configuration to `upstream {}` block and reload nginx in lb container to update config.

We don't have static sessions in our test environment and we have the same result as with previous configuration.
```
$ curl localhost
<h4>This is node2 server</h4>
$ curl localhost
<h4>This is node3 server</h4>
$ curl localhost
<h4>This is node1 server</h4>
```

### Test session persistence load balancing

Let's add `ip_hash;` configuration to `upstream {}` block and reload nginx in `lb` container to update config and test.
```
$ curl localhost:80
<h4>This is node1 server</h4>
$ curl localhost:80
<h4>This is node1 server</h4>
$ curl localhost:80
<h4>This is node1 server</h4>
```
All requests from machine to one server.
Let's log in to `lb` container and send request from it.
```
# curl localhost
<h4>This is node3 server</h4>
# curl localhost
<h4>This is node3 server</h4>
# curl localhost
<h4>This is node3 server</h4>
```
The same situation.

### Pros and cons of each approach (some point were found [here](https://www.nginx.com/blog/choosing-nginx-plus-load-balancing-techniques/#Pros-Cons-and-Use-Cases))

#### Default load balancing
+ Easy setup, no additional configuration.
- Bad solution for dynamic content/application servers.

#### Least connected load balancing
+ Nginx will try not to overload a busy application server with excessive requests, distributing the new requests to a less busy server instead.
- With each connection you can receive content from a different server. It's also bad for dynamic content.

#### Session persistence load balancing
+ Requests from the same client will always be directed to the same server except when this server is unavailable.
- The biggest drawback of these methods is that they are not guaranteed to distribute requests in equal numbers across servers, let alone balance load evenly. 
