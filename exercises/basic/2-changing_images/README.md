# Exercise 2: Changing images

In this exercise, we'll learn how to modify an existing Docker image, and commit it as a new one.

To accomplish this, we'll modify the `ubuntu:16.04` image to include the `ping` utility.

### Getting setup

First download the `ubuntu:16.04` image with `docker pull`. (You might already have this if you have completed the previous tutorial.)

```
$ docker pull ubuntu:16.04
16.04: Pulling from library/ubuntu
c62795f78da9: Pull complete 
d4fceeeb758e: Pull complete 
5c9125a401ae: Pull complete 
0062f774e994: Pull complete 
6b33fd031fac: Pull complete 
Digest: sha256:c2bbf50d276508d73dd865cda7b4ee9b5243f2648647d21e3a471dd3cc4209a0
Status: Downloaded newer image for ubuntu:16.04
$
```

### Modifying an image

Let's run the image in a new container and install the `ping` utility.

1. First start the container with `/bin/bash`:

    ```
    $ docker run -it ubuntu:16.04 /bin/bash
    root@786b94c53c6d:/#
    ```

2. Try running `ping` in the terminal.

    ```
    root@786b94c53c6d:/# ping google.com
    bash: ping: command not found
    root@786b94c53c6d:/#
    ```

    The command doesn't exist. The Ubuntu image for Docker only has the bare minimum of software installed to operate the container. That's okay though: we can install the `ping` command.

2. But first we'll update our software list.

    In Debian-based Linux environments (such as Ubuntu), you can install new software using the `apt` package manager. For those who have experience with Macs, this program is the equivalent of `homebrew`.

    By default, to reduce the image size, the Ubuntu image doesn't have a list of the available software packages. We need to update the list of available software:

    ```
    root@786b94c53c6d:/# apt-get update
    Get:1 http://security.ubuntu.com/ubuntu xenial-security InRelease [102 kB]
    Get:2 http://security.ubuntu.com/ubuntu xenial-security/universe Sources [29.6 kB]   
    Get:3 http://archive.ubuntu.com/ubuntu xenial InRelease [247 kB]    
    Get:4 http://security.ubuntu.com/ubuntu xenial-security/main amd64 Packages [308 kB]
    Get:5 http://security.ubuntu.com/ubuntu xenial-security/restricted amd64 Packages [12.8 kB]
    Get:6 http://security.ubuntu.com/ubuntu xenial-security/universe amd64 Packages [132 kB]   
    Get:7 http://security.ubuntu.com/ubuntu xenial-security/multiverse amd64 Packages [2936 B]
    Get:8 http://archive.ubuntu.com/ubuntu xenial-updates InRelease [102 kB]                 
    Get:9 http://archive.ubuntu.com/ubuntu xenial-backports InRelease [102 kB]
    Get:10 http://archive.ubuntu.com/ubuntu xenial/universe Sources [9802 kB]
    Get:11 http://archive.ubuntu.com/ubuntu xenial/main amd64 Packages [1558 kB]
    Get:12 http://archive.ubuntu.com/ubuntu xenial/restricted amd64 Packages [14.1 kB]
    Get:13 http://archive.ubuntu.com/ubuntu xenial/universe amd64 Packages [9827 kB]
    Get:14 http://archive.ubuntu.com/ubuntu xenial/multiverse amd64 Packages [176 kB]
    Get:15 http://archive.ubuntu.com/ubuntu xenial-updates/universe Sources [186 kB]
    Get:16 http://archive.ubuntu.com/ubuntu xenial-updates/main amd64 Packages [652 kB]
    Get:17 http://archive.ubuntu.com/ubuntu xenial-updates/restricted amd64 Packages [13.2 kB]
    Get:18 http://archive.ubuntu.com/ubuntu xenial-updates/universe amd64 Packages [577 kB]
    Get:19 http://archive.ubuntu.com/ubuntu xenial-updates/multiverse amd64 Packages [9809 B]
    Get:20 http://archive.ubuntu.com/ubuntu xenial-backports/main amd64 Packages [4929 B]
    Get:21 http://archive.ubuntu.com/ubuntu xenial-backports/universe amd64 Packages [2567 B]
    Fetched 23.9 MB in 5s (4409 kB/s)                
    Reading package lists... Done
    root@786b94c53c6d:/#
    ```

3. Now we can install the `ping` command.

    Call `apt-get install iputils-ping` to install the package containing `ping`:

    ```
    root@786b94c53c6d:/# apt-get install iputils-ping
    Reading package lists... Done
    Building dependency tree       
    Reading state information... Done
    The following additional packages will be installed:
      libffi6 libgmp10 libgnutls-openssl27 libgnutls30 libhogweed4 libidn11 libnettle6 libp11-kit0 libtasn1-6
    Suggested packages:
      gnutls-bin
    The following NEW packages will be installed:
      iputils-ping libffi6 libgmp10 libgnutls-openssl27 libgnutls30 libhogweed4 libidn11 libnettle6 libp11-kit0 libtasn1-6
    0 upgraded, 10 newly installed, 0 to remove and 0 not upgraded.
    Need to get 1303 kB of archives.
    After this operation, 3778 kB of additional disk space will be used.
    Do you want to continue? [Y/n] Y
    Get:1 http://archive.ubuntu.com/ubuntu xenial/main amd64 libgmp10 amd64 2:6.1.0+dfsg-2 [240 kB]
    Get:2 http://archive.ubuntu.com/ubuntu xenial-updates/main amd64 libnettle6 amd64 3.2-1ubuntu0.16.04.1 [93.5 kB]
    Get:3 http://archive.ubuntu.com/ubuntu xenial-updates/main amd64 libhogweed4 amd64 3.2-1ubuntu0.16.04.1 [136 kB]
    Get:4 http://archive.ubuntu.com/ubuntu xenial-updates/main amd64 libidn11 amd64 1.32-3ubuntu1.1 [45.6 kB]
    Get:5 http://archive.ubuntu.com/ubuntu xenial/main amd64 libffi6 amd64 3.2.1-4 [17.8 kB]
    Get:6 http://archive.ubuntu.com/ubuntu xenial-updates/main amd64 libp11-kit0 amd64 0.23.2-5~ubuntu16.04.1 [105 kB]
    Get:7 http://archive.ubuntu.com/ubuntu xenial-updates/main amd64 libtasn1-6 amd64 4.7-3ubuntu0.16.04.1 [43.2 kB]
    Get:8 http://archive.ubuntu.com/ubuntu xenial-updates/main amd64 libgnutls30 amd64 3.4.10-4ubuntu1.2 [547 kB]
    Get:9 http://archive.ubuntu.com/ubuntu xenial-updates/main amd64 libgnutls-openssl27 amd64 3.4.10-4ubuntu1.2 [21.9 kB]
    Get:10 http://archive.ubuntu.com/ubuntu xenial/main amd64 iputils-ping amd64 3:20121221-5ubuntu2 [52.7 kB]
    Fetched 1303 kB in 0s (1767 kB/s)   
    debconf: delaying package configuration, since apt-utils is not installed
    Selecting previously unselected package libgmp10:amd64.
    (Reading database ... 4764 files and directories currently installed.)
    Preparing to unpack .../libgmp10_2%3a6.1.0+dfsg-2_amd64.deb ...
    Unpacking libgmp10:amd64 (2:6.1.0+dfsg-2) ...
    Selecting previously unselected package libnettle6:amd64.
    Preparing to unpack .../libnettle6_3.2-1ubuntu0.16.04.1_amd64.deb ...
    Unpacking libnettle6:amd64 (3.2-1ubuntu0.16.04.1) ...
    Selecting previously unselected package libhogweed4:amd64.
    Preparing to unpack .../libhogweed4_3.2-1ubuntu0.16.04.1_amd64.deb ...
    Unpacking libhogweed4:amd64 (3.2-1ubuntu0.16.04.1) ...
    Selecting previously unselected package libidn11:amd64.
    Preparing to unpack .../libidn11_1.32-3ubuntu1.1_amd64.deb ...
    Unpacking libidn11:amd64 (1.32-3ubuntu1.1) ...
    Selecting previously unselected package libffi6:amd64.
    Preparing to unpack .../libffi6_3.2.1-4_amd64.deb ...
    Unpacking libffi6:amd64 (3.2.1-4) ...
    Selecting previously unselected package libp11-kit0:amd64.
    Preparing to unpack .../libp11-kit0_0.23.2-5~ubuntu16.04.1_amd64.deb ...
    Unpacking libp11-kit0:amd64 (0.23.2-5~ubuntu16.04.1) ...
    Selecting previously unselected package libtasn1-6:amd64.
    Preparing to unpack .../libtasn1-6_4.7-3ubuntu0.16.04.1_amd64.deb ...
    Unpacking libtasn1-6:amd64 (4.7-3ubuntu0.16.04.1) ...
    Selecting previously unselected package libgnutls30:amd64.
    Preparing to unpack .../libgnutls30_3.4.10-4ubuntu1.2_amd64.deb ...
    Unpacking libgnutls30:amd64 (3.4.10-4ubuntu1.2) ...
    Selecting previously unselected package libgnutls-openssl27:amd64.
    Preparing to unpack .../libgnutls-openssl27_3.4.10-4ubuntu1.2_amd64.deb ...
    Unpacking libgnutls-openssl27:amd64 (3.4.10-4ubuntu1.2) ...
    Selecting previously unselected package iputils-ping.
    Preparing to unpack .../iputils-ping_3%3a20121221-5ubuntu2_amd64.deb ...
    Unpacking iputils-ping (3:20121221-5ubuntu2) ...
    Processing triggers for libc-bin (2.23-0ubuntu7) ...
    Setting up libgmp10:amd64 (2:6.1.0+dfsg-2) ...
    Setting up libnettle6:amd64 (3.2-1ubuntu0.16.04.1) ...
    Setting up libhogweed4:amd64 (3.2-1ubuntu0.16.04.1) ...
    Setting up libidn11:amd64 (1.32-3ubuntu1.1) ...
    Setting up libffi6:amd64 (3.2.1-4) ...
    Setting up libp11-kit0:amd64 (0.23.2-5~ubuntu16.04.1) ...
    Setting up libtasn1-6:amd64 (4.7-3ubuntu0.16.04.1) ...
    Setting up libgnutls30:amd64 (3.4.10-4ubuntu1.2) ...
    Setting up libgnutls-openssl27:amd64 (3.4.10-4ubuntu1.2) ...
    Setting up iputils-ping (3:20121221-5ubuntu2) ...
    Setcap is not installed, falling back to setuid
    Processing triggers for libc-bin (2.23-0ubuntu7) ...
    root@786b94c53c6d:/#
    ```

4. Finally, we should be able to use `ping`.

    Ping your favorite website. When you've seen enough, `Ctrl+C` to interrupt, then `exit` the container.

    ```
    root@786b94c53c6d:/# ping google.com
    PING google.com (172.217.4.206) 56(84) bytes of data.
    64 bytes from lga15s48-in-f14.1e100.net (172.217.4.206): icmp_seq=1 ttl=37 time=0.936 ms
    64 bytes from lga15s48-in-f14.1e100.net (172.217.4.206): icmp_seq=2 ttl=37 time=0.367 ms
    64 bytes from lga15s48-in-f14.1e100.net (172.217.4.206): icmp_seq=3 ttl=37 time=0.258 ms
    ^C
    --- google.com ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2033ms
    rtt min/avg/max/mdev = 0.258/0.520/0.936/0.297 ms
    root@786b94c53c6d:/# exit
    exit
    $
    ```

### Committing changes

Installing `ping` isn't very special in itself. But what if you wanted to have `ping` on all of your `ubuntu` containers? You'd have to redo this installation each time you spin up a new container, and that isn't much fun.

The Docker way is to create a new image. There are two ways to do this: 1) build a new image from scratch or 2) commit a container state as a new image. We'll cover how to do #1 in the "Building Images" exercise, but we can do #2 now.

1. Let's find our container to create the new image from.

    Fortunately, we have a Docker container with our `ping` utility already installed from the previous steps. It should be stopped right now, but let's find its container ID.

    ```
    $ docker ps -a
    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
    786b94c53c6d        ubuntu:16.04        "/bin/bash"         13 minutes ago      Exited (0) 22 seconds ago                       angry_pike
    $
    ```

2. Now let's commit it as a new image.

    `docker commit` takes a container, and allows you to commit its changes as a new image.

    ```
    $ docker commit --help

    Usage:  docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

    Create a new image from a container's changes

    Options:
      -a, --author string    Author (e.g., "John Hannibal Smith <hannibal@a-team.com>")
      -c, --change list      Apply Dockerfile instruction to the created image (default [])
          --help             Print usage
      -m, --message string   Commit message
      -p, --pause            Pause container during commit (default true)
    $
    ```

    Pass the container ID, an author, commit message, and give it the name `<DockerHub username>/ping`:

    ```
    $ docker commit -a 'David Elner' -m 'Added ping utility.' 786 delner/ping
    sha256:78ba830008a61a09f9eae8ca4ead0966ff501457c23df0f635e0651253b3d0e3
    $ 
    ```

    Then check `docker images` to see your new image:

    ```
    $ docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
    delner/ping         latest              78ba830008a6        About a minute ago   159MB
    ubuntu              16.04               6a2f32de169d        4 days ago           117MB
    $
    ```

3. Finally run your new image in a new container to see it in action!

    ```
    $ docker run -it --rm delner/ping /bin/bash
    root@3ab21a456c9f:/# ping google.com
    PING google.com (172.217.4.206) 56(84) bytes of data.
    64 bytes from lga15s48-in-f14.1e100.net (172.217.4.206): icmp_seq=1 ttl=37 time=1.12 ms
    64 bytes from lga15s48-in-f14.1e100.net (172.217.4.206): icmp_seq=2 ttl=37 time=0.380 ms
    64 bytes from lga15s48-in-f14.1e100.net (172.217.4.206): icmp_seq=3 ttl=37 time=0.352 ms
    ^C
    --- google.com ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2024ms
    rtt min/avg/max/mdev = 0.352/0.620/1.129/0.360 ms
    root@3ab21a456c9f:/#
    ```

# END OF EXERCISE 2