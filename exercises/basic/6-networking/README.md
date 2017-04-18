# Exercise 6: Networking

In this exercise, we'll learn to work with Docker networks, and interconnect containers.

To accomplish this, we'll setup two Postgres databases, and query between them.

### Listing networks

Docker defines networks, which groups containers for interoperability and DNS functions.

To list networks, run `docker network ls`:

```
$ docker network ls
NETWORK ID          NAME                         DRIVER              SCOPE
99cbf6b3d074        bridge                       bridge              local
4a2071abf006        host                         host                local
3789e459bd9c        none                         null                local
$
```

There are 3 default networks: `bridge`, `host`, and `none` listed here. Any other custom networks will also be listed here. The `host` and `none` networks are not important for this exercise, but `bridge` is of interest.

### The default `bridge` network

All new containers, if given no other configuration, will be automatically added the `bridge` network. This network acts as a pass through to your host's ethernet, so your Docker containers can access the internet.

We can inspect the `bridge` network by running `docker network inspect bridge`:

```
$ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "99cbf6b3d07476bca2aaca71413b1a7609338d7d8deae9d4af77b062a98672de",
        "Created": "2017-04-17T20:27:36.319424753-04:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
$
```

There's some miscellaneous information about the network, but notice the `"Containers": {}` entry. You can see any containers that are currently connected to the network here.

Let's start a `ping` container, and inspect this again:

```
$ docker run --rm -d --name dummy delner/ping:1.0
104633917dbfe00843722336838f163b800dde46e632e47470b204c21fc44f21
$ docker network inspect bridge
...
        "Containers": {
            "104633917dbfe00843722336838f163b800dde46e632e47470b204c21fc44f21": {
                "Name": "dummy",
                "EndpointID": "38f01d182b8d55de5f8ed3221f12086dd2eac3426b159cc8e6bda0075dbd0f47",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
...
$
```

You can see the container was added to the default network. Now let's add another `ping` container, and set it to ping our first.

```
$ docker run --rm -d -e PING_TARGET=172.17.0.2 --name pinger delner/ping:1.0
3a79f28b8ac36c0e7aae523c4831c9405c110d593c15a30639606250595b245b
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS               NAMES
3a79f28b8ac3        delner/ping:1.0     "sh -c 'ping $PING..."   4 seconds ago        Up 3 seconds                            pinger
104633917dbf        delner/ping:1.0     "sh -c 'ping $PING..."   About a minute ago   Up About a minute                       dummy
$ docker logs pinger
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.171 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.100 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.098 ms
64 bytes from 172.17.0.2: icmp_seq=4 ttl=64 time=0.098 ms
$
```

Inspecting the logs for `pinger` we can see it was able to successfully ping the other container in the network. While IP address does work, it's very cumbersome and prone to error if addresses change. It would be better to use a hostname, specifically the container name `dummy`, to always resolve to the correct container.

Running `ping` with the `dummy` as the target:

```
$ docker run --rm -d -e PING_TARGET=dummy --name pinger delner/ping:1.0
3a79f28b8ac36c0e7aae523c4831c9405c110d593c15a30639606250595b245b
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS               NAMES
104633917dbf        delner/ping:1.0     "sh -c 'ping $PING..."   About a minute ago   Up About a minute                       dummy
$
```

...results in failure. The host name couldn't be resolved, thus causing the command to error and the container exit and autoremove.

The default `bridge` network will not automatically allow you to network containers by container name. We can, however, easily accomplish host resolution using a custom network.

Stop and remove the `dummy` container by running `docker stop dummy`.

### Managing custom networks

To create a new network, use the `docker network create` command and provide it a network name.

```
$ docker network create skynet
c234438e88ab579be943859dbc0a89788563226c3a9a13b4f1a2c78d1d8000c9
$ docker network ls
NETWORK ID          NAME                         DRIVER              SCOPE
99cbf6b3d074        bridge                       bridge              local
4a2071abf006        host                         host                local
3789e459bd9c        none                         null                local
c234438e88ab        skynet                       bridge              local
$ docker network inspect skynet
[
    {
        "Name": "skynet",
        "Id": "c234438e88ab579be943859dbc0a89788563226c3a9a13b4f1a2c78d1d8000c9",
        "Created": "2017-04-18T01:28:56.982335289-04:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.26.0.0/16",
                    "Gateway": "172.26.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
$
```

To remove networks, run `docker network rm` and provide it the network name.

### Adding containers to a network

Let's rerun the `ping` container, this time assigning it a network:

```
$ docker run --rm -d --network skynet --name dummy delner/ping:1.0
```

Then the pinger, targeting the dummy `ping` container:

```
$ docker run --rm -d --network skynet -e PING_TARGET=dummy --name pinger delner/ping:1.0
28e68fed9fe28a4346951fa8b6f4147a16f2afec8671357f1ed5f27425914b0a
$ docker logs pinger
PING dummy (172.26.0.2) 56(84) bytes of data.
64 bytes from dummy.skynet (172.26.0.2): icmp_seq=1 ttl=64 time=0.101 ms
64 bytes from dummy.skynet (172.26.0.2): icmp_seq=2 ttl=64 time=0.102 ms
64 bytes from dummy.skynet (172.26.0.2): icmp_seq=3 ttl=64 time=0.116 ms
$
```

Notice this time the host name resolves successfully. This is Docker's Embedded DNS in action. It's most useful when orchestrating multiple containers in a single application, such as a web server, database, and cache. Instead of using IP addresses, you can define each of the respective connection strings using container names to leverage DNS resolution.

Stop and remove the containers by running `docker stop pinger` and `docker stop dummy`.

### Connecting between containers in a network

We can resolve host names and ping, but this isn't the same as connecting with TCP/UDP between containers.

Let's setup two `postgres` databases to connect to one another: a `widget` database, and `gadget` database.

Start each of the databases and add them to the network:

```
$ docker run --rm -d --name widgetdb --network skynet -p 5432 postgres
7f0248e3c0f4f03159ef966fd9767a4c7e3412801f8b0445cebb933d1e84e020
$ docker run --rm -d --name gadgetdb --network skynet -p 5432 postgres
8dc66701837c695728abb9046db71924112a9b8f2f1e096094ab5b5d631e2f73
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                     NAMES
8dc66701837c        postgres            "docker-entrypoint..."   11 seconds ago      Up 10 seconds       0.0.0.0:32769->5432/tcp   gadgetdb
7f0248e3c0f4        postgres            "docker-entrypoint..."   40 seconds ago      Up 39 seconds       0.0.0.0:32768->5432/tcp   widgetdb
$
```

By default, port 5432 is blocked and inaccessible. However, by adding `-p 5432`, we are permitting other containers to access it through port 5432, the default Postgres port. 

Now that they're running, start a shell session in the `widgetdb` using `docker exec`:

```
$ docker exec -it widgetdb /bin/bash
root@7f0248e3c0f4:/#
```

You can then connect to the local database using `psql`. (End the `psql` session by entering `\q`.)

```
root@7f0248e3c0f4:/# psql -U postgres
psql (9.6.2)
Type "help" for help.

postgres=# \q
root@7f0248e3c0f4:/#
```

Or to the `gadget` database by referring to it by name:

```
root@7f0248e3c0f4:/# psql -U postgres -h gadgetdb
psql (9.6.2)
Type "help" for help.

postgres=# \q
root@7f0248e3c0f4:/#
```

Type `exit` to end the session, then `docker stop widgetdb gadgetdb` to stop and remove the containers.

### Binding ports to the host

Sometimes its useful to access an application running in a Docker container directly, as if it were running on your host machine.

To this end, you can bind ports from a container to a port on your host machine. To do this, the altered command from our previous Postgres example would look like:

```
$ docker run --rm -d --name widgetdb --network skynet -p 5432:5432 postgres
```

The `-p` flag given `<host port>:<container port>` does this mapping, making the server available as `localhost:5432`:

You can then run `psql` (if the utility is installed) on your host machine to access the Postgres database 

```
$ psql -U postgres -h localhost
psql (9.6.2)
Type "help" for help.

postgres=# \q
$
```

It's important to keep in mind that you can only bind one application to a host port at a time. If you try to start any applications on your host machine, or other Docker containers that want to bind to a port already in use, they will fail to do so. 

Type `docker stop widgetdb` to stop and remove the container.

# END OF EXERCISE 6