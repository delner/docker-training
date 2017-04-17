# TBD:

Following section is some misc content. Move it somewhere else?

### Configuring containers

6. Some Docker images, especially full fledged applications, require additional configuration to run properly.

    To this end, the `docker run` command offers many options to customize configuration for your container, which you can see by running `--help`.

    ```
    $ docker run --help

    Usage:  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

    Run a command in a new container

    Options:
          --add-host list              Add a custom host-to-IP mapping (host:ip) (default [])
      -a, --attach list                Attach to STDIN, STDOUT or STDERR (default [])
          --blkio-weight uint16        Block IO (relative weight), between 10 and 1000, or 0 to disable (default 0)
          --blkio-weight-device list   Block IO weight (relative device weight) (default [])
          --cap-add list               Add Linux capabilities (default [])
          --cap-drop list              Drop Linux capabilities (default [])
          --cgroup-parent string       Optional parent cgroup for the container
          --cidfile string             Write the container ID to the file
          --cpu-period int             Limit CPU CFS (Completely Fair Scheduler) period
          --cpu-quota int              Limit CPU CFS (Completely Fair Scheduler) quota
          --cpu-rt-period int          Limit CPU real-time period in microseconds
          --cpu-rt-runtime int         Limit CPU real-time runtime in microseconds
      -c, --cpu-shares int             CPU shares (relative weight)
          --cpus decimal               Number of CPUs (default 0.000)
          --cpuset-cpus string         CPUs in which to allow execution (0-3, 0,1)
          --cpuset-mems string         MEMs in which to allow execution (0-3, 0,1)
      -d, --detach                     Run container in background and print container ID
          --detach-keys string         Override the key sequence for detaching a container
          --device list                Add a host device to the container (default [])
          --device-cgroup-rule list    Add a rule to the cgroup allowed devices list (default [])
          --device-read-bps list       Limit read rate (bytes per second) from a device (default [])
          --device-read-iops list      Limit read rate (IO per second) from a device (default [])
          --device-write-bps list      Limit write rate (bytes per second) to a device (default [])
          --device-write-iops list     Limit write rate (IO per second) to a device (default [])
          --disable-content-trust      Skip image verification (default true)
          --dns list                   Set custom DNS servers (default [])
          --dns-option list            Set DNS options (default [])
          --dns-search list            Set custom DNS search domains (default [])
          --entrypoint string          Overwrite the default ENTRYPOINT of the image
      -e, --env list                   Set environment variables (default [])
          --env-file list              Read in a file of environment variables (default [])
          --expose list                Expose a port or a range of ports (default [])
          --group-add list             Add additional groups to join (default [])
          --health-cmd string          Command to run to check health
          --health-interval duration   Time between running the check (ns|us|ms|s|m|h) (default 0s)
          --health-retries int         Consecutive failures needed to report unhealthy
          --health-timeout duration    Maximum time to allow one check to run (ns|us|ms|s|m|h) (default 0s)
          --help                       Print usage
      -h, --hostname string            Container host name
          --init                       Run an init inside the container that forwards signals and reaps processes
          --init-path string           Path to the docker-init binary
      -i, --interactive                Keep STDIN open even if not attached
          --ip string                  IPv4 address (e.g., 172.30.100.104)
          --ip6 string                 IPv6 address (e.g., 2001:db8::33)
          --ipc string                 IPC namespace to use
          --isolation string           Container isolation technology
          --kernel-memory bytes        Kernel memory limit
      -l, --label list                 Set meta data on a container (default [])
          --label-file list            Read in a line delimited file of labels (default [])
          --link list                  Add link to another container (default [])
          --link-local-ip list         Container IPv4/IPv6 link-local addresses (default [])
          --log-driver string          Logging driver for the container
          --log-opt list               Log driver options (default [])
          --mac-address string         Container MAC address (e.g., 92:d0:c6:0a:29:33)
      -m, --memory bytes               Memory limit
          --memory-reservation bytes   Memory soft limit
          --memory-swap bytes          Swap limit equal to memory plus swap: '-1' to enable unlimited swap
          --memory-swappiness int      Tune container memory swappiness (0 to 100) (default -1)
          --name string                Assign a name to the container
          --network string             Connect a container to a network (default "default")
          --network-alias list         Add network-scoped alias for the container (default [])
          --no-healthcheck             Disable any container-specified HEALTHCHECK
          --oom-kill-disable           Disable OOM Killer
          --oom-score-adj int          Tune host's OOM preferences (-1000 to 1000)
          --pid string                 PID namespace to use
          --pids-limit int             Tune container pids limit (set -1 for unlimited)
          --privileged                 Give extended privileges to this container
      -p, --publish list               Publish a container's port(s) to the host (default [])
      -P, --publish-all                Publish all exposed ports to random ports
          --read-only                  Mount the container's root filesystem as read only
          --restart string             Restart policy to apply when a container exits (default "no")
          --rm                         Automatically remove the container when it exits
          --runtime string             Runtime to use for this container
          --security-opt list          Security Options (default [])
          --shm-size bytes             Size of /dev/shm
          --sig-proxy                  Proxy received signals to the process (default true)
          --stop-signal string         Signal to stop a container (default "SIGTERM")
          --stop-timeout int           Timeout (in seconds) to stop a container
          --storage-opt list           Storage driver options for the container (default [])
          --sysctl map                 Sysctl options (default map[])
          --tmpfs list                 Mount a tmpfs directory (default [])
      -t, --tty                        Allocate a pseudo-TTY
          --ulimit ulimit              Ulimit options (default [])
      -u, --user string                Username or UID (format: <name|uid>[:<group|gid>])
          --userns string              User namespace to use
          --uts string                 UTS namespace to use
      -v, --volume list                Bind mount a volume (default [])
          --volume-driver string       Optional volume driver for the container
          --volumes-from list          Mount volumes from the specified container(s) (default [])
      -w, --workdir string             Working directory inside the container
      $
    ```

    Typically you won't need to use many of these options, so we'll focus on the most commonly used ones instead.

    ##### Environment variables

    The most important of these options for configuring new containers is the `-e` flag, which allows you to specify environment variables to set in the container. Most Docker images use environment variables to drive their configuration, which can be customized per container.

    Let's try it out. Just pass the `-e` into a run command to start a container:

    ```
    $ docker run -it --rm -e 'MESSAGE=Hello world!' ubuntu:16.04 /bin/bash
    root@59e07d26092c:/# echo $MESSAGE
    Hello world!
    root@59e07d26092c:/# exit
    $
    ```

    Although the value of setting an environment variable is limited in this example, it really shines through in other images you can find on DockerHub.

    For example, with the `postgres` image, you can use environment variables to specify the default user/password:

    ```
    $ docker run --rm -e POSTGRES_USER=archer -e POSTGRES_PASSWORD=guest postgres
    ```

    For your own custom images, you might also use environment variables for setting sensitive configuration, such as API keys and database URLS, for your application.

    ##### Ports

    In many cases, you'll want to connect to processes running on your Docker containers, such as a web server on port 80.

    By default, however, Docker containers are network inaccessible from your local machine. We'll cover this more in detail in the *Networking* tutorial, but for now, it's important to know you must expose and map ports between Docker containers and your host machine.

