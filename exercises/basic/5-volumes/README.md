# Exercise 5: Volumes

In this exercise, we'll learn to work with Docker volumes, for persisting data between containers.

To accomplish this, we'll setup an Apache HTTPD web server, and persist some HTML files in a volume.

### Setting up the server

To run our Apache HTTPD server, run this command:

```
$ docker run --rm -d --name apache -p 80:80 httpd:2.4
d87e0a193dde5652ac762d8849983c2cadb5116b80c8a61a4180e350d678b4d2
```

This command will start a new container from HTTP 2.4, name it `apache`, bind port `80` to the host machine (more on this later), and set a flag to delete the container when it stops.

After it starts, we can run `curl localhost` to query the web server for the default page:

```
$ curl localhost
<html><body><h1>It works!</h1></body></html>
$
```

This is the default `index.html` file included with a new Apache 2.4 installation. Let's replace this HTML file with new content.

To do so, we'll use the `docker cp` command, similar to `scp`, which copies files between the host and containers. Let's give it the `index.html` file from the directory this README is sitting in:

```
$ docker cp index.html apache:/usr/local/apache2/htdocs/
$
```

The first path is the source path, representing our new file on our host machine, and the second path our destination. `apache` is the name of the container we want to copy into, and `/usr/local/apache2/htdocs/` is where the web server serves HTML from.

Running `curl` again now looks a little different:

```
$ curl localhost
<html><body><h1>It works in Docker!</h1></body></html>
$
```

##### A possible data problem

This container, for its lifetime, will continue to serve our new HTML file.

However, containers in Docker are, in practice, considered ephemeral. They can die unexpectedly, and in certain deployments, be removed without warning. If you're depending upon the container state for your application, you might lose important data when such containers die. This is especially a concern for applications like databases, which are supposed to be considered permanent datastores.

In the case of our HTTPD server, simply stopping the container will cause it to be autoremoved. We can bring another container back up in it's place, but it won't have our changes any more.

```
$ docker stop apache
apache
$ docker run --rm -d --name apache -p 80:80 httpd:2.4
9bd0620e3d8464456c368b1fe9b82733282d980a7c3f854b8cba7726f0a02958
$ curl localhost
<html><body><h1>It works!</h1></body></html>
$
```

To preserve our data between outages or system upgrades, we can use volumes to persist our data across generations of containers.

### Managing volumes

Volumes in Docker are file stores, which sit independently of your Docker containers. The function like Amazon Web Services' EBS volumes, and other mountable media like USB thumb drives. They can be created, deleted, and mounted on containers at specific locations within an image, like you would with `mount` command in Linux.

To list your volumes, run `docker volume ls`:

```
$ docker volume ls
DRIVER              VOLUME NAME
$
```
To create a new volume, run `docker volume create` and give it a volume name.

```
$ docker volume create myvolume
myvolume
$ docker volume ls
DRIVER              VOLUME NAME
local               myvolume
$
```

To remove a volume, run `docker volume rm` and give it the volume name.

```
$ docker volume rm myvolume
myvolume
$ docker volume ls
DRIVER              VOLUME NAME
$
```

### Mounting volumes on containers

First create a new volume named `httpd_htdocs`:

```
$ docker volume create httpd_htdocs
httpd_htdocs
$
```

Then re-run our `docker run` command, providing the `-v` mount flag.

```
$ docker run --rm -d --name apache -p 80:80 -v httpd_htdocs:/usr/local/apache2/htdocs/ httpd:2.4
c21dd93fea83d710b4d4c954911862760030723df6a5b42650e462e388fe6049
$
```

And re-copy in our modified HTML file.

```
$ docker cp index.html apache:/usr/local/apache2/htdocs/
$
```

And run `curl` to verify it worked.

```
$ curl localhost
<html><body><h1>It works in Docker!</h1></body></html>
$
```

Now to see the volume in action, let's stop the container. By providing the `--rm` flag during `run`, it should remove the container upon stopping.

```
$ docker stop apache
apache
$
```

Then once again start httpd with the same run command as last time. This time, however, we can `curl` and see our file changes are still there from before.

```
$ docker run --rm -d --name apache -p 80:80 -v httpd_htdocs:/usr/local/apache2/htdocs/ httpd:2.4
c21dd93fea83d710b4d4c954911862760030723df6a5b42650e462e388fe6049
$ curl localhost
<html><body><h1>It works in Docker!</h1></body></html>
$
```

We can take this volume and mount it on any HTTPD container now, which gives us flexibility in swapping out our container for newer versions without losing our data, if we wish.

Go ahead and run `docker stop apache` to stop and remove the container, then `docker volume rm httpd_htdocs` to remove the volume.

### Mounting host directories on containers

As an alternative to using volumes, if you have a directory on your host machine you'd like to use like a volume, you can mount those too. This is technique is useful in development environments, where you might want to mount your local repo onto a Docker image, and actively modify the contents of a Docker container without rebuilding or copying files to it.

The `-v` flag to accomplish this is almost identical to the previous one. Simply specify an absolute path to a local directory instead. In our case, we'll pass `.` to specify the `5-volumes` directory in this repo, which conveniently contains a modified version of the HTML file.

```
$ pwd
/home/david/src/docker-training/exercises/basic/5-volumes/
$ docker run --rm -d --name apache -p 80:80 -v /home/david/src/docker-training/exercises/basic/5-volumes/:/usr/local/apache2/htdocs/ httpd:2.4
0d91516b20ea6113b5dcca08ada6465095dc68663b3d2201dc0490165764f842
$ curl localhost
<html><body><h1>It works in Docker!</h1></body></html>
$
```

With the host directory mount in place, modify the `index.html` file in this directory with whatever message you like, then save the file and re-run `curl`.

```
$ curl localhost
<html><body><h1>It works quite well in Docker!</h1></body></html>
$
```

You can see file changes take place immediately on the Docker container without any need to run `docker cp`.

Go ahead and run `docker stop apache` to stop and remove the container.

# END OF EXERCISE 5