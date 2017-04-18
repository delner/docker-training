# Exercise 4: Sharing images

In this exercise, we'll learn how to share Docker images using DockerHub. DockerHub is GitHub for Docker: a great place to find community images, and upload your own to.

We'll need our `ping` image from the "Building Images" exercise, so be sure to complete that exercise first so you have an image to share.

### Getting Started

To share images on DockerHub, you'll need a DockerHub account. You can sign up [here](https://hub.docker.com/).

Most of the features on DockerHub should be pretty similar to GitHub: there's a search function for finding new images, repositories for your images, and organizations.

The `docker` CLI tool also has integration with DockerHub. In order to use certain features, you'll need to login first:

```
$ docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: delner
Password: 
Login Succeeded
$
```

### Finding images

Use the `docker search` command to search for new images:

```
$ docker search kafka
NAME                        DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
wurstmeister/kafka          Multi-Broker Apache Kafka Image                 319                  [OK]
spotify/kafka               A simple docker image with both Kafka and ...   200                  [OK]
ches/kafka                  Apache Kafka. Tagged versions. JMX. Cluste...   70                   [OK]
sheepkiller/kafka-manager   kafka-manager                                   61                   [OK]
$
```

Once you found an image you like, you can pull it locally with `docker pull`, covered in the "Running Containers" exercise.

### Tagging images

In the previous exercise "Building Images", we built and tagged an image as `<DockerHub username>/ping`. If you mistagged this image with something other than your DockerHub user name, it's no problem: we can re-tag it.

Just use `docker tag` to add a new tag with your DockerHub user name, and give it a version:

```
$ docker images
REPOSITORY                                                TAG                 IMAGE ID            CREATED             SIZE
ping                                                      latest              a980ae1c79ea        2 minutes ago       121MB
ubuntu                                                    16.04               6a2f32de169d        5 days ago          117MB
$ docker tag ping delner/ping:1.0
$ docker images
REPOSITORY                                                TAG                 IMAGE ID            CREATED             SIZE
delner/ping                                               1.0                 a980ae1c79ea        5 minutes ago       121MB
ping                                                      latest              a980ae1c79ea        5 minutes ago       121MB
ubuntu                                                    16.04               6a2f32de169d        5 days ago          117MB
$ 
```

You can see the same image ID now maps to both the old and new tag. To remove the old tag, run `docker rmi` with the old image tag:

```
$ docker rmi ping
Untagged: ping:latest
$ docker images
REPOSITORY                                                TAG                 IMAGE ID            CREATED             SIZE
delner/ping                                               1.0                 a980ae1c79ea        6 minutes ago       121MB
ubuntu                                                    16.04               6a2f32de169d        5 days ago          117MB
$ 
```

### Pushing images

To push an image, all you need to do is call `docker push` with the tag.

```
$ docker push delner/ping:1.0
The push refers to a repository [docker.io/delner/ping]
3b372b8ab44b: Pushed 
ab4b9ad8d212: Mounted from library/ubuntu 
57e913ee49e5: Mounted from library/ubuntu 
2ea6deead2b0: Mounted from library/ubuntu 
7cbd4b94e525: Mounted from library/ubuntu 
e86a0c422723: Mounted from library/ubuntu 
1.0: digest: sha256:1881fd1efdde061d3aede939b8696c0e6d0b36e9f8b38abccb1074bc60592a60 size: 1568
$
```

It will automatically create a new public repository on DockerHub under the organization in the tag. By providing it with your user name, it creates a new repository at `https://hub.docker.com/r/<USERNAME>/ping/`. Feel free to check out your new repo page!

It's also possible push images to non-DockerHub repositories, such as ones provided by AWS Elastic Container Registry, by changing the repository part of the tag to match the appropriate URL.

# END OF EXERCISE 4