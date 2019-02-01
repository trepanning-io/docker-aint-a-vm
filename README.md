# Docker != Virtual Machine

Despite the growing use of containers over recent years there is still some confusion as to the difference between container and a virtual machine. Most people understand they are both virtualization technologies but are a little hazy on the differences.

The aim of this repo is to help clarify the difference. In a nutshell the difference is a VM seperates the kernel, file system and networking stack from the host machine. Docker separates only the file system, networking stack and uses a thing called cgroups to isolate the process(es) from the host machine. With Docker both the container and the host use the same kernel.

You are welcome to clone/fork and play with the (code examples)[https://github.com/trepanning-oi/docker-aint-a-vm] as you see fit. Some people (myself included) learn best by playing with things themselves.

Contained in the repo are multiple containers that all implement the classic "Hello World!" program. It's so classic even Docker uses it as the validation step to prove Docker has installed correctly.

Each container implements the "Hello World!" program in a different language, or a different version of the language. By looking at the differences of each container I'll explain what a container is and why I think there is – understandably – some confusion.

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

What we see from each container is two things, that it prints "Hello World!" and the exit code for that container. Eagle-eyes readers will not that `nasm` is the only one with an exit code of 1, I'll explain that later.

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

## Python

Remember that each container has a separate file system from it's host. That literally means there are two copies of `/etc`, `/usr`, `/var`, and so on, one for the host and one for the container. Now, the container doesn't have to contain **any** files or directories. But it must contain the ones it depends on.

Python depends on a lot. For all practical purposes it depends on a full OS. It certainly depends on a lot of system libraries. Python 2 and Python 3 rank #1 & #3 for the biggest containers. The main reason for this is that the container is based on a "full" OS. In this case Alpine 3.7. How big is that image?

```bash
$ docker images alpine:3.7
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
alpine              3.7                 bc8fb6e6e49d        31 hours ago        4.21MB
```

So Alpine is just over 4MB. Where does the other 54MB (Python2) and 77MB (Python3) come from? That is the Python runtime.

To be fair Python does describe itself as a "batteries included" programming language. And it is a syntactically light and very flexible programming language. Python does a lot of heavy lifting for the programming and the runtime is the muscle that does that lifting. Python sure has some big :muscle:
