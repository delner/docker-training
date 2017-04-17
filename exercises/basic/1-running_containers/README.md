# Exercise 1: Running containers

In this exercise, we'll learn the basics of pulling images, starting, stopping, and removing containers.

### Pulling an image

To run containers, we'll first need to pull some images.

1. Let's see what images we have currently on our machine, by running `docker images`:

    ```
    $ docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    ```

2. On a fresh Docker installation, we should have no images. Let's pull one from Dockerhub.

    We usually pull images from DockerHub by tag. These look like:

    ```
    # Official Docker images
    <repo>:<tag>
    # ubuntu:16.04
    # elasticsearch:5.2
    # nginx:latest

    # User or organization made images
    <user or org>/<repo>:<tag>
    # delner/ubuntu:16.04
    # bitnami/rails:latest
    ```

    We can search for images using `docker search <keyword>`

    ```
    $ docker search ubuntu
    NAME                                         DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
    ubuntu                                       Ubuntu is a Debian-based Linux operating s...   5582      [OK]       
    rastasheep/ubuntu-sshd                       Dockerized SSH service, built on top of of...   73                   [OK]
    ubuntu-upstart                               Upstart is an event-based replacement for ...   70        [OK]       
    consol/ubuntu-xfce-vnc                       Ubuntu container with "headless" VNC sessi...   43                   [OK]
    ubuntu-debootstrap                           debootstrap --variant=minbase --components...   27        [OK]       
    ```

    You can also find images online at [DockerHub](https://hub.docker.com/).

    Run `docker pull ubuntu:16.04` to pull an image of Ubuntu 16.04 from DockerHub.

    ```
    $ docker pull ubuntu:16:04
    16.04: Pulling from library/ubuntu
    8aec416115fd: Pull complete 
    695f074e24e3: Pull complete 
    946d6c48c2a7: Pull complete 
    bc7277e579f0: Pull complete 
    2508cbcde94b: Pull complete 
    Digest: sha256:71cd81252a3563a03ad8daee81047b62ab5d892ebbfbf71cf53415f29c130950
    Status: Downloaded newer image for ubuntu:16.04
    ```
3. We can also pull different versions on the same image.

    Run `docker pull ubuntu:16.10` to pull an image of Ubuntu 16.10.

    ```
    16.10: Pulling from library/ubuntu
    3a635c0fcefb: Pull complete 
    bf3f7e9b4869: Pull complete 
    ad323864e1f8: Pull complete 
    b4d3fc870200: Pull complete 
    4e69d6ff0e56: Pull complete 
    Digest: sha256:609c1726180221d95a66ce3ed1e898f4a543c5be9ff3dbb1f10180a6cb2a6fdc
    Status: Downloaded newer image for ubuntu:16.10
    ```

    Then when we run `docker images again, we should get:

    ```
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    ubuntu              16.10               31005225a745        4 weeks ago         103 MB
    ubuntu              16.04               f49eec89601e        4 weeks ago         129 MB
    ```

4. Over time, your machine can collect a lot of images, so it's nice to remove unwanted images.

    Run `docker rmi <IMAGE ID>` to remove the Ubuntu 16.10 image we won't be using.

    ```
    $ docker rmi 31005225a745
    Untagged: ubuntu:16.10
    Untagged: ubuntu@sha256:609c1726180221d95a66ce3ed1e898f4a543c5be9ff3dbb1f10180a6cb2a6fdc
    Deleted: sha256:31005225a74578ec48fbe5a833ef39a3e41ebcbf0714ad3867070405b3efd81e
    Deleted: sha256:c9fcffc56240d2382f78da3130215afcfc7130b29210184f30ffce3a3eae677d
    Deleted: sha256:7a8ffa53e9616698d138da12474f8f7441f00e129bb06c7f12b9264828bcad1e
    Deleted: sha256:c71fbd03fa070b80919b1712f7b335829fecd0157915cf4b60775988c18a5687
    Deleted: sha256:3b1d2c1b8ae337cacea8271862bded89d920bbbf5049fa1b4927169cb3b3974c
    Deleted: sha256:6b720ab3505cb593509654fac976193e75bc881d9c72abdffe4c29278396c636
    ```

    Alternatively, you can delete images by tag or by a partial image ID. In the previous example, the following would have been equivalent:

     - `docker rmi 31`
     - `docker rmi ubuntu:16.10`

    Running `docker images` should reflect the deleted image.

    ```
    $ docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    ubuntu              16.04               f49eec89601e        4 weeks ago         129 MB
    ```

    A nice shortcut for removing all images from your system is `docker rmi $(docker images -a -q)`

### Running our container

Using the Ubuntu 16.04 image we downloaded, we can run a our first container. Unlike a traditional virtualization framework like VirtualBox or VMWare, we can't just start a virtual machine running this image without anything else: we have to give it a command to run.

The command can be anything you want, as long as it exists on the image. In the case of the Ubuntu image, it's a Linux kernel with many of the typical applications you'd find in a basic Linux environment.

1. Let's do a very simple example. Run `docker run ubuntu:16.04 /bin/echo 'Hello world!'`

    ```
    $ docker run ubuntu:16.04 /bin/echo 'Hello world!'
    Hello world!
    ```

    The `/bin/echo` command is a really simple application that just prints whatever you give it to the terminal. We passed it 'Hello world!', so it prints `Hello world!` to the terminal.

    When you run the whole `docker run` command, it creates a new container from the image specified, then runs the command inside the container. From the previous example, the Docker container started, then ran the `/bin/echo` command in the container.

2. Let's check what containers we have after running this. Run `docker ps`

    ```
    $ docker ps
    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
    ```

    That's strange: no containers right? The `ps` command doesn't show stopped containers by default, add the `-a` flag.

    ```
    $ docker ps -a
    CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS                          PORTS               NAMES
    4fc37e27944a        ubuntu:16.04        "/bin/echo 'Hello ..."   About a minute ago   Exited (0) About a minute ago                       zen_swartz
    ```

    Okay, there's our container. But why is the status "Exited"?

    *Docker containers only run as long as the command it starts with is running.* In our example, it ran `/bin/echo` successfully, printed some output, then exited with status code 0 (which means no errors.) When Docker saw this command exit, the container stopped.

3. Let's do something a bit more interactive. Run `docker run ubuntu:16.04 /bin/bash`

    ```
    $ docker run ubuntu:16.04 /bin/bash
    $
    ```

    Notice nothing happened. When we run `docker ps -a`

    ```
    $ docker ps -a
    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
    654792eb403b        ubuntu:16.04        "/bin/bash"         4 seconds ago       Exited (0) 2 seconds ago                       distracted_minsky
    ```

    The container exited instantly. Why? We were running the `/bin/bash` command, which is an interactive program. However, the `docker run` command doesn't run interactively by default, therefore the `/bin/bash` command exited, and the container stopped.

    Instead, let's add the `-it` flags, which tells Docker to run the command interactively with your terminal.

    ```
    $ docker run -it ubuntu:16.04 /bin/bash
    root@5fa68739793c:/# 
    ```

    This looks a lot better. This means you're in a BASH session inside the Ubuntu container. Notice you're running as `root` and the container ID that follows.

    You can now use this like a normal Linux shell. Try `pwd` and `ls` to look at the file system.

    ```
    root@5fa68739793c:/# pwd
    /
    root@5fa68739793c:/# ls
    bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
    ```

    You can type `exit` to end the BASH session, terminating the command and stopping the container.

    ```
    root@5fa68739793c:/# exit
    exit
    $
    ```
4. By default your terminal remains attached to the container when you run `docker run`. What if you don't want to remain attached?

    By adding the `-d` flag, we can run in detached mode, meaning the container will continue to run as long as the command is, but it won't print the output.

    Let's run `/bin/sleep 3600`, which will run the container idly for 1 hour:

    ```
    $ docker run -d ubuntu:16.04 /bin/sleep 3600
    be730b8c554b69383f30f71222b9ac264367c7454790dc2a4eb0cda33c0baa2a
    $
    ```

    If we check the container, we can see it's running the sleep command in a new container.

    ```
    $ docker ps
    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
    be730b8c554b        ubuntu:16.04        "/bin/sleep 3600"    41 seconds ago      Up 40 seconds                           jovial_goldstine
    $
    ```

5. Now that the container is running in the background, what if we want to reattach to it?

    Conceivably, if this were something like a web server or other process where we might like to inspect logs while it runs, it'd be useful to run something on the container without interrupting the current process.

    To this end, there is another command, called `docker exec`. `docker exec` runs a command within a container that is already running. It works exactly like `docker run`, except instead of taking an image ID, it takes a container ID.

    This makes the `docker exec` command useful for tailing logs, or "SSHing" into an active container.

    Let's do that now, running the following, passing the first few characters of the container ID:

    ```
    $ docker exec -it be7 /bin/bash
    root@be730b8c554b:/#
    ```

    The container ID appearing at the front of the BASH prompt tells us we're inside the container. Once inside a session, we can interact with the container like any SSH session.

    Let's list the running processes:

    ```
    root@be730b8c554b:/# ps aux
    USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    root         1  0.2  0.0   4380   796 ?        Ss   15:41   0:00 /bin/sleep 3600
    root         6  0.6  0.1  18240  3208 ?        Ss   15:41   0:00 /bin/bash
    root        16  0.0  0.1  34424  2808 ?        R+   15:41   0:00 ps aux
    root@be730b8c554b:/#
    ```

    There we can see our running `/bin/sleep 3600` command. Whenever we're done, we can type `exit` to exit our current BASH session, and leave the container running.

    ```
    root@be730b8c554b:/# exit
    exit
    $ docker ps
    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
    be730b8c554b        ubuntu:16.04        "/bin/sleep 3600"   9 minutes ago       Up 9 minutes                            jovial_goldstine
    $
    ```

    And finally checking `docker ps`, we can see the container is still running.

6. Instead of waiting 1 hour for this command to stop (and the container exit), what if we'd like to stop the Docker container now?

    To that end, we have the `docker stop` and the `docker kill` commands. The prior is a graceful stop, whereas the latter is a forceful one.

    Let's use `docker stop`, passing it the first few characters of the container name we want to stop.

    ```
    $ docker stop be73
    be73
    ```

    Then checking `docker ps -a`...

    ```
    $ docker ps -a
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
    be730b8c554b        ubuntu:16.04        "/bin/sleep 600"         1 minute ago        Exited (137) 1 minute ago                       jovial_goldstine
    $
    ```

    We can see that it exited with code `137`, which in Linux world means the command was likely aborted with a `kill -9` command.

### Removing containers

7. After working with Docker containers, you might want to delete old, obsolete ones.

    ```
    $ docker ps -a
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
    be730b8c554b        ubuntu:16.04        "/bin/sleep 600"         1 minute ago        Exited (137) 1 minute ago                       jovial_goldstine
    $
    ```

    From our previous example, we can see with `docker ps -a` that we have a container hanging around.

    To remove this we can use the `docker rm` command which removes stopped containers.

    ```
    $ docker rm be73
    be73
    ```

    A nice shortcut for removing all containers from your system is `docker rm $(docker ps -a -q)`

    It can be tedious to remove old containers each time after you run them. To address this, Docker also allows you to specify the `--rm` flag to the `docker run` command, which will remove the container after it exits.

    ```
    $ docker run --rm ubuntu:16.04 /bin/echo 'Hello and goodbye!'
    Hello and goodbye!
    $ docker ps -a
    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
    $
    ```

# END OF EXERCISE 1