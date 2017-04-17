# Exercise 3: Building images

In this exercise, we'll learn how to create a new Docker image.

To do this, like in "Changing Images", we'll add the `ping` utility to the `ubuntu:16.04` image. The outcome from each of these exercises should be equivalent.

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

If you still have the image from the previous "Changing Images", let's also remove that now. Be sure to `docker rm` any containers based on that image first, otherwise deleting the image will fail.

```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
delner/ping         latest              78ba830008a6        45 minutes ago      159MB
ubuntu              16.04               6a2f32de169d        4 days ago          117MB
$ docker rmi 78b
Untagged: delner/ping:latest
Deleted: sha256:78ba830008a61a09f9eae8ca4ead0966ff501457c23df0f635e0651253b3d0e3
Deleted: sha256:94b1207f7fb25468a3e1c16e604f8e65d9ed4df783fa057c368b635d7b086e39
$
```


### Creating a Dockerfile

Like the "Changing Images" exercise, we'll install `ping` on top of `ubuntu` to create a new image. Unlike that exercise, however, we will not run or modify any containers to do so.

Instead, we'll use `Dockerfile`. The `Dockerfile` is a "recipe" of sorts, that contains a list of instructions on how to build a new image.

1. Create a new file named `Dockerfile` in your working directory:

    ```
    $ touch Dockerfile
    $
    ```

2. Open `Dockerfile` with your favorite text editor.

2. Inside the file, we'll need to add some important headers.

    The `FROM` directive specifies what base image this new image will be built upon. (Ubuntu in our case.)

    The `LABEL` directive adds a label the image. Useful for adding metadata.

    Add the following two lines to the top of your file:

    ```
    FROM ubuntu:16.04
    LABEL author="David Elner"
    ```

3. Then we'll need to add some commands to modify the image.

    The `RUN` directive runs a command inside the image, and rolls any changes to the filesystem into a commit. A typical Dockerfile will contain several `RUN` statements, each committing their changes on top of the previous.

    To install `ping`, we'll need to run `apt-get update` and `apt-get install`. First, add the `apt-get update` command:

    ```
    RUN apt-get update
    ```

    Then add the `apt-get install` command:

    ```
    RUN apt-get install -y iputils-ping
    ```

    Notice we added the `-y` flag. When building Docker images, these commands will run non-interactively. Normally the `apt-get` command will prompt you "Y/n?" if you want to proceed. The `-y` flag avoids that prompt by always answering "Y".

    Our file should now look something like this:

    ```
    FROM ubuntu:16.04
    LABEL author="David Elner"

    RUN apt-get update

    RUN apt-get install -y iputils-ping
    ```

    And with that, we should be ready to build our image.

### Building the Dockerfile

To build Docker images from Dockerfiles, we use the `docker build` command. The `docker build` command reads a Dockerfile, and runs its instructions to create a new image.

1. Let's build our image.

    Running the following builds and tags the image:

    ```
    $ docker build -t 'delner/ping' .
    Sending build context to Docker daemon    190kB
    Step 1/4 : FROM ubuntu:16.04
     ---> 6a2f32de169d
    Step 2/4 : LABEL author "David Elner"
     ---> Running in 50f765b29144
     ---> 27bfa513216f
    Removing intermediate container 50f765b29144
    Step 3/4 : RUN apt-get update
     ---> Running in ae8647e54bd1
    Get:1 http://security.ubuntu.com/ubuntu xenial-security InRelease [102 kB]
    Get:2 http://security.ubuntu.com/ubuntu xenial-security/universe Sources [30.0 kB]
    ...
     ---> 1f4fe5596cb1
    Removing intermediate container ae8647e54bd1
    Step 4/4 : RUN apt-get install -y iputils-ping
     ---> Running in e6a838d41cef
    Reading package lists...
    Building dependency tree...
    ...
     ---> 918648f00b92
    Removing intermediate container e6a838d41cef
    Successfully built 918648f00b92
    $
    ```

    The use of `.` in the arguments is significant here. `docker build` looks for files named `Dockerfile` by default to run. So by giving it a `.`, we're telling Docker to use the `Dockerfile` in our current directory. If this Dockerfile was actually named anything else, you'd change this to match the name of the file.

    Also notice the output about "steps" here. Each directive in your Dockerfile maps to a step here, and after each step is completed, it becomes a commit. Why does this matter though?

    Docker layers each commit on top of the other, like an onion. By doing so, it can keep image sizes small, and when rebuilding images, it can even reuse commits that are unaffected by changes to make builds run quicker.

    We can see this caching behavior in action if we simply rerun the same command again:

    ```
    $ docker build -t 'delner/ping' .
    Sending build context to Docker daemon    191kB
    Step 1/4 : FROM ubuntu:16.04
     ---> 6a2f32de169d
    Step 2/4 : LABEL author "David Elner"
     ---> Using cache
     ---> 27bfa513216f
    Step 3/4 : RUN apt-get update
     ---> Using cache
     ---> 1f4fe5596cb1
    Step 4/4 : RUN apt-get install -y iputils-ping
     ---> Using cache
     ---> 918648f00b92
    Successfully built 918648f00b92
    $
    ```

    Running `docker images`, we can see our newly built image.

    ```
    $ docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    delner/ping         latest              918648f00b92        9 minutes ago       159MB
    ubuntu              16.04               6a2f32de169d        4 days ago          117MB
    ```

### Optimizing the Dockerfile

Looking at new image, we can see it is `159MB` in size versus its base image of `117MB`. That's a pretty big change in size for installing some utilities. That will take more disk space and add additional time to pushes/pulls of this image.

But why is it that much bigger? The secret is in the `RUN` commands. As mentioned before, any filesystem changes are committed after the `RUN` command completes. This includes any logs, or temporary data written to the filesystem which might be completely inconsequential to our image.

In our case, the use of `apt-get` generates a lot of this fluff we don't need in our image. We'll need to modify these `RUN` directives slightly.

We can start with removing any old logs after the install completes. Adding the following to the bottom of the Dockerfile:

```
RUN apt-get clean \
    && cd /var/lib/apt/lists && rm -fr *Release* *Sources* *Packages* \
    && truncate -s 0 /var/log/*log
```

Then running `build` again yields...

```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
delner/ping         latest              912bc1c7c059        4 seconds ago       159MB
ubuntu              16.04               6a2f32de169d        4 days ago          117MB
$
```

...yields no improvement? What's going on here?

Turns out because how commits are layered one upon the other, if there's fluff hanging around from a previous commit, it won't matter if you clean it up in a future `RUN` directive. It will be permanently apart of the history, thus the image size.

Our fluff actually happens to originate from the `apt-get update` command, which leaves a bunch of package lists around that we don't need. The easiest way to deal with this is to collapse all of the related `RUN` directives together.

The rewritten Dockerfile should like:

```
FROM ubuntu:16.04
LABEL author="David Elner"

RUN apt-get update \
    && apt-get install -y iputils-ping \
    && apt-get clean \
    && cd /var/lib/apt/lists && rm -fr *Release* *Sources* *Packages* \
    && truncate -s 0 /var/log/*log
```

Then after rerunning `build`, our images now like:

```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
delner/ping         latest              622e555950e0        12 seconds ago      121MB
ubuntu              16.04               6a2f32de169d        4 days ago          117MB
$
```

Now the new image is only 4MB larger in size. A major improvement.

### Other Dockerfile directives

There are [many other useful directives](https://docs.docker.com/engine/reference/builder/) available in the Dockerfile.

Some important ones:

 - `COPY`: Copy files from your host into the Docker image.
 - `WORKDIR`: Specify a default directory to execute commands from.
 - `CMD`: Specify a default command to run.
 - `ENV`: Specify a default environment variable.
 - `EXPOSE`: Expose a port by default.
 - `ARG`: Specify a build-time argument (for more configurable, advanced builds.)

Since our Dockerfile is build for `ping`, let's add the `ENV` and `CMD` directives.

```
FROM ubuntu:16.04
LABEL author="David Elner"

ENV PING_TARGET "google.com"

RUN apt-get update \
    && apt-get install -y iputils-ping \
    && apt-get clean \
    && cd /var/lib/apt/lists && rm -fr *Release* *Sources* *Packages* \
    && truncate -s 0 /var/log/*log

CMD ["sh", "-c", "ping $PING_TARGET"]
```

These new directives mean that our image when run with `docker run -it delner/ping` will automatically run `ping google.com`.

```
$ docker run -it delner/ping
PING google.com (172.217.10.46) 56(84) bytes of data.
64 bytes from lga34s13-in-f14.1e100.net (172.217.10.46): icmp_seq=1 ttl=37 time=0.300 ms
64 bytes from lga34s13-in-f14.1e100.net (172.217.10.46): icmp_seq=2 ttl=37 time=0.373 ms
64 bytes from lga34s13-in-f14.1e100.net (172.217.10.46): icmp_seq=3 ttl=37 time=0.378 ms
^C
--- google.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2078ms
rtt min/avg/max/mdev = 0.300/0.350/0.378/0.038 ms
$
```

### Learning more about Dockerfiles

Most images in the Docker world are built from Dockerfiles, and looking at other Dockerfiles from your favorite repos can be a wonderful source of information about how they function, and how you can improve your own images. Seek them out on both DockerHub and GitHub!

# END OF EXERCISE 3