# Docker != Virtual Machine

Despite the growing use of containers over recent years there is still some confusion as to the difference between a container and a virtual machine. Most people understand they are both virtualization technologies but are a little hazy on the differences.

The aim of this repo is to help clarify the difference. In a nutshell, the difference is a VM seperates the kernel, file system and networking stack from the host machine. Docker separates only the file system, networking stack and uses a thing called cgroups to isolate the process(es) from the host machine. With Docker both the container and the host use the same kernel.

You are welcome to clone/fork and play with the [code examples](https://github.com/trepanning-oi/docker-aint-a-vm) as you see fit. Some people (myself included) learn best by playing with things themselves.

Contained in the repo are multiple containers that all implement the classic "Hello World!" program. It's so classic even Docker uses it as the validation step to prove Docker has installed correctly.

Each container implements the "Hello World!" program in a different language, or a different version of a language. By looking at the differences of each container I'll explain what a container is and clear up the confusion.

Let's begin by seeing what these containers do. This isn't a primer for Docker, so I'm not going to explain how to install or use it. By using `docker-compose` we can run all the containers in parallel and get the following output:

```bash
$ docker-compose up
...
Attaching to golang-1.10, node, golang-1.10-stripped, python3, python2, nasm, bash
golang-1.10             | Hello world!
node                    | Hello World!
golang-1.10 exited with code 0
golang-1.10-stripped    | Hello world!
python3                 | Hello world!
python2                 | Hello world!
node exited with code 0
golang-1.10-stripped exited with code 0
python3 exited with code 0
nasm                    | Hello world!
bash                    | Hello world!
python2 exited with code 0
nasm exited with code 1
bash exited with code 0
```

What we see from each container is two things, that it prints "Hello World!" and the exit code for that container. Eagle-eyes readers will note that `nasm` is the only one with an exit code of 1, I'll explain that later.

There are 7 implementations of the "Hello World!" program written in Python2, Python3, Golang 1.10, Node, Bash and ASM (aka `nasm` which is assembler). They are all literally or effectively one line programs and **only** print the string "Hello World!". The only thing each container uses is 12 bytes for the string and however many bytes for the print instruction. 

## Docker Image Sizes

Let's take a look at the size of each docker image.

```bash
$ docker images | grep size
size/node           9.11                5ca049eb2135        15 minutes ago      68.5MB
size/asm            2.10                52cb0ac37af4        15 minutes ago      356B
size/golang         1.10-stripped       8df3b4dc0dc4        15 minutes ago      758kB
size/golang         1.10                7c293b9dbccf        15 minutes ago      1.26MB
size/python         3                   d4019d0cdd0a        15 minutes ago      81.3MB
size/python         2                   bca1a2cfd9fe        15 minutes ago      58.2MB
size/bash           alpine3.7           76e2115e34dc        15 minutes ago      4.21MB
```

The last column shows the size of each docker image and there is quite a range. The smallest is 356B, the largest is 81.3MB! Why such a big range?

It's the programming language runtime and it's dependencies. For each programming language there is code automatically added to the code your wrote to make your code run, that's called the runtime. That runtime needs certain things from the operating system to run, so those need to be included too.

Remember, all that is happening is we are printing a 12 byte string. OK, you can add a byte or 2 for the line feed (and/or carrage return). Add in the print instruction and the total size is a 10s of bytes. Maybe a 100 bytes. Why are these containers so big?

Lets look at each programming language to see what is being included.

### Python

Remember that each container has a separate file system from it's host. That literally means there are two copies of `/etc`, `/usr`, `/var`, and so on, one for the host and one for the container. Now, the container doesn't have to contain **any** files or directories. But it must contain the ones it depends on.

Python depends on a lot. For all practical purposes it depends on a full OS. It certainly depends on a lot of system libraries. Python 3 and Python 2 rank #1 & #3 for the biggest containers. A possible reason for this is that the container is based on a "full" OS. You can use `docker run --rm -it size/python:2 bash -l` to take a look at what files are in this container. The OS this image is based on is Alpine 3.7. How big is that image?

```bash
$ docker images alpine:3.7
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
alpine              3.7                 bc8fb6e6e49d        31 hours ago        4.21MB
```

So Alpine is just over 4MB. Where does the other 54MB (Python2) and 77MB (Python3) come from? That is the Python runtime.

To be fair Python does describe itself as a "batteries included" programming language. And it is a syntactically light and very flexible programming language. Python does a lot of heavy lifting for the programming and the runtime is the muscle that does that lifting. Python sure has some big :muscle:

### Node

Node takes the #2 spot in our Docker image leader board. Like Python, the Node container is based on Alpine 3.7. And like Python the vast majority of the image size is the Node runtime. 

Node, like Python, does a lot for the programmer and does so using Javascript! :dizzy_face: 

### Bash

Compared to the previous scripting languages Bash is tiny. It's no bigger than the Alpine image it is based on. But it's hardly a fair comparison to Python and Node. To do much more with Bash would require installing additional applications and using Bash to interact with their CLI.

### Golang

This is the first of the compiled languages. It's the first language to leave the MB territory and enter the KB. But to do that there are a few tricks.

Docker isolates one or more processes from the rest of the host, making only the kernel - and only some of it system calls - available. Because Golang is compiled we can drastically reduce the dependencies using a multi-stage Docker files. This means using one "fat" container to build the executable and then copy that exectuable into an empty container.

But wait, if all Golang is doing is printing a string to the screen, and that should take 100 bytes or so, why are the Golang containers 1.2MB and 750KB?

Once again, it is our old friend the runtime. While Golang does not do "as much" for the programmer as Python or Node, it still does a lot. Those Go routines don't run in thin air! The stripped version of the container is using certain compiler flags to remove unnecessary code. 

### Assembler

And finally the winner for the smallest Docker image. 300B almost 1000x smaller than the next smallest. Like the Golang images, this image is built using a multi-stage Dockerfile so that only the executable goes into the final image. Because it is assembler the programmer has to do **everything**. On the flip side that means our container has almost nothing more than the instruction to print and the 12(-ish) bytes to print.

Now to explain that exit code. There are 3 things you need to know.

1. I am not a software developer by trade (although I can code in multiple languages with the help of Stack Overflow :trollface:)
2. The last time I wrote any assembler was back in University, over 20 years ago :scream:
3. I had no idea what I was doing in assembler then and still have no idea now!

It is for these reasons that the `nasm` container exits with a code of 1. It should not, but I have no idea how to fix it. Fortunatly it's does not matter.

## What does this all mean and why should you care

By looking at different programming languages we have learned that a container does not need to contain anything but the exact code it needs to operate. It needs no files (other than the executable), directories or anything else supporting it. Depending on the programming language you maybe forced to included an OS. Scripting languages like Python and Node can suffer from needing to "download half the Internet" before they can do anything useful. 

The programming language you choose can have a massive impact on your containerized infrastructure. A system based on images that are MB will always be cheaper and more responsive than a system based on images that are GB. Storage maybe cheap, but would you rather be waiting to download a 1GB image or a 1MB image before your service is online?

To be clear. I am not advocating you must use a compiled language to get the "full benefit" of a container. Node and Python have their place, just as Golang does. And there are techniques you can use in the Dockerfile to minimise total image storage (and transfer). But if your containers are so big you can't tell the difference from a VM are you sure you're using the technology?

