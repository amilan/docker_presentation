# Playing with Docker

Antonio Milán Otero
KITS - 2016

---

# What Docker is

 ... but first ...

---

# What I'm not.

# **I'm not a Docker expert!!**

 ... Just to let it clear.

 ---

# OK, let's continue. What is Docker?

Docker containers wrap a piece of software in a complete filesystem that contains everything needed to run:

  - Code
  - Runtime
  - System tools
  - System libraries

And everything could be installed on a server.

This guarantees that __the software will always run the same, regardless of its environment.__

Yes, I have ctrl+C ctrl+V this info from [here][1]

[1] http://www.docker.com/what-docker

---

# What Docker is NOT

## It's not a Virtual Machine

> Add Picture here comparing VM and Containers.

Virtual Machines include: Application, binaries, libraries and the entire guest OS.
Containers include the application and its dependencies. Run in an **isolated process** in user space on the host operating system.

---

# Maybe no you feel like ...

> Add WTF gif

---

# How we used it for MxCUBE
... but first let's put you on context ...

A couple of months ago we run a workshop in Hamburg. We wanted to let the people experiment with our application, but at the end we discover that they were having plenty of problems to install it and run it.

> Add use cases in vertical slides

--

# Use case 1: Distributing a mockup version for newcomers

So, we decided to distribute a VM to make easier for newcomers to test the app.

--

# Use case 2: Development environment.

It's quite nice that we were thinking in using it as our development environment.

> add fragment

But we are not there yet.

--

# Use case 3: Deployment environment.

It will be not so complex to use it for deployment.

> add fragment

---

But we are not there yet ... sorry again.

> Add Sorry gif

---

# MxCUBE overview

MxCUBE is composed by 3 main services:

  - Redis
  - NPM
  - MxCUBE

So in order to have it running you need to run those 3.

---

## First thing: Dockerfile

Here we define everything needed in our container to run the app, and also the configuration that has to be done.

```
##############################################################################
# Dockerfile to run MXCuBE web server
##############################################################################

FROM centos:7
MAINTAINER Antonio Milan Otero <antonio.milan_otero.maxiv.lu.se>
```

> Vertical slides

--

Then you install whatever is needed in your container.

```
# Install ####
RUN yum install -y curl
RUN curl --silent --location https://rpm.nodesource.com/setup_4.x | bash -

RUN yum install -y gcc-c++ make
# or: yum groupinstall 'Development Tools'

# Install nodejs from epel repository ####
RUN yum install -y nodejs npm

# Install more dependencies ####
RUN yum install -y \
        python-devel \
        openldap-devel \
        lapack-devel \
        zlib-devel \
        libjpeg-turbo-devel \
        libxml2-devel \
        libxslt-devel \
        openssl-devel \
        libgfortran \
        cyrus-sasl-devel
```
--

Then we get the latest version of the app.
 > In the future it will be pointing official releases.

```
# Install git ####
RUN yum install -y git

# Get MxCUBE code ####
RUN mkdir /mxcube
WORKDIR /mxcube
RUN git clone https://github.com/mxcube/mxcube3.git --recursive
WORKDIR mxcube3
```
--

After that, more installations

```
# Install EPEL repository ####
RUN yum install -y epel-release
RUN yum makecache # && yum update -y

# Install python pip ####
RUN yum install -y python-pip
RUN yum install -y redis

# Install requirements ####
RUN pip install -r requirements.txt

# Install supervisor
RUN pip install supervisor

# Install npm ####
RUN npm install
RUN npm install fabric
RUN npm install --dev
```
--

Some configuration now.

```
RUN cp backend_server.js.example backend_server.js

COPY supervisord.conf /etc/supervisor/supervisord.conf

COPY run_mxcube /usr/local/bin/

EXPOSE 8090

CMD ["/usr/bin/supervisord"]
```

--

We use supervisor in order to run and monitor all the processes.
More details here:
http://supervisord.org/

Supervisor configuration example:

```
[supervisord]
nodaemon=true

[program:redis]
command=redis-server

[program:mxcube]
command=python mxcube3-server -r test/HardwareObjectsMockup.xml --log-file mxcube.log

[program:npm]
command=npm start
```

--

# And run it

```
docker build -t mxcube_web .
docker run -i -p 8090:8090 -t mxcube_web
```

> Add screenshot of mxcube running.

--

If you don't want to build it, it's already available in docker hub.
So, execute:

```
docker run -i -p 8090:8090 -t amilan/mxcube_web
```
And enjoy it.

> Add like_magic.gif

---

# Docker compose

A nice tool to orchestrate several docker containers.

Configurable with a .yml file.

```
version: '2'
services:
  mxcube:
    build: .
    expose:
      - "8081"
    ports:
      - "8081:8081"
    links:
      - redis
    command: python mxcube3-server -r test/HardwareObjectsMockup.xml --log-file /tmp/mxcube.log
    volumes:
      - ./tmp:/tmp
  redis:
    image: redis
    expose:
      - "6379"
    ports:
      - "6379:6379"
    volumes:
      - ./data:/data
```

--

And then:

```
docker-compose build
docker-compose up
```

And the magic happens.

---

Repo available here:
https://github.com/amilan/mxcube_web


---

# Docker registry

Basically it's like a repository of images.
Docker hub is the official one, but we can have our own one in Max IV.

Some good things:
  - Storage for our containers.
  - Push and pull commands (for maintenance).
  - Notification system (Webhooks)
  - Able to connect to a Continuous Integration / Delivery system.

---

# Ansible integration?

Why not?

It will provide:
  - Flexibility:
    - Portable playbooks
    - Reproducible environments (using docker, vagrant, etc.)

  - Auditability: Simple to review content in a container.

  - Ubiquity: You can manage containers environments and also host environments.

---

# Questions?
