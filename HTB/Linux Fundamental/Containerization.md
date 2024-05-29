# Containerization

Containerization is a process of packaging and running applications in  isolated environments, such as a container, virtual machine, or  serverless environment. Technologies like Docker, Docker Compose, and  Linux Containers make this process possible in Linux systems. Containerization is a great way to ensure that applications are managed and deployed efficiently and securely.

## Dockers

Docker is an open-source platform for automating the deployment of  applications as self-contained units called containers. It uses a  layered filesystem and resource isolation features to provide  flexibility and portability. Additionally, it provides a robust set of  tools for creating, deploying, and managing applications, which helps  streamline the containerization process.

#### Install Docker-Engine

Installing Docker is relatively straightforward. We can use the following script to install it on a Ubuntu host:

```bash
#!/bin/bash

# Preparation
sudo apt update -y
sudo apt install ca-certificates curl gnupg lsb-release -y
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt update -y
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Add user htb-student to the Docker group
sudo usermod -aG docker htb-student
echo '[!] You need to log out and log back in for the group changes to take effect.'

# Test Docker installation
docker run hello-world
```

The Docker engine and specific Docker images are needed to run a container. These can be obtained from the [Docker Hub](https://hub.docker.com/), a repository of pre-made images, or created by the user. The Docker Hub is a cloud-based registry for software repositories or a library for  Docker images. It is divided into a `public` and a `private` area. The public area allows users to upload and share images with the  community. It also contains official images from the Docker development  team and established open-source projects. Images uploaded to a private  area of the registry are not publicly accessible.

Creating a Docker image is done by creating a [Dockerfile](https://docs.docker.com/engine/reference/builder/), which contains all the instructions the Docker engine needs to create  the container. We can use Docker containers as our “file hosting” server when transferring specific files to our target systems. Therefore, we  must create a `Dockerfile` based on Ubuntu 22.04 with `Apache` and `SSH` server running. With this, we can use `scp` to transfer files to the docker image, and Apache allows us to host files and use tools like `curl`, `wget`, and others on the target system to download the required files. Such a `Dockerfile` could look like the following:

#### Dockerfile

```bash
# Use the latest Ubuntu 22.04 LTS as the base image
FROM ubuntu:22.04

# Update the package repository and install the required packages
RUN apt-get update && \
    apt-get install -y \
        apache2 \
        openssh-server \
        && \
    rm -rf /var/lib/apt/lists/*

# Create a new user called "student"
RUN useradd -m docker-user && \
    echo "docker-user:password" | chpasswd

# Give the htb-student user full access to the Apache and SSH services
RUN chown -R docker-user:docker-user /var/www/html && \
    chown -R docker-user:docker-user /var/run/apache2 && \
    chown -R docker-user:docker-user /var/log/apache2 && \
    chown -R docker-user:docker-user /var/lock/apache2 && \
    usermod -aG sudo docker-user && \
    echo "docker-user ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

# Expose the required ports
EXPOSE 22 80

# Start the SSH and Apache services
CMD service ssh start && /usr/sbin/apache2ctl -D FOREGROUND
```

After we have defined our Dockerfile, we need to convert it into an image. With the `build` command, we take the directory with the Dockerfile, execute the steps from the `Dockerfile`, and store the image in our local Docker Engine. If one of the steps  fails due to an error, the container creation will be aborted. With the  option `-t`, we give our container a tag, so it is easier to identify and work with later.

#### Docker Build

```shell
DixLan@htb[/htb]$ docker build -t FS_docker .
```

Once the Docker image has been created, it can be executed through  the Docker engine, making it a very efficient and easy way to run a  container. When a container is to be started on a system, a package with the  respective image is first loaded if unavailable locally. We can start  the container by the following command [docker run](https://docs.docker.com/engine/reference/commandline/run/):

#### Docker Run - Syntax

```shell
DixLan@htb[/htb]$ docker run -p <host port>:<docker port> -d <docker container name>
```

#### Docker Run

```shell
DixLan@htb[/htb]$ docker run -p 8022:22 -p 8080:80 -d FS_docker
```

In this case, we start a new container from the image `FS_docker` and map the host ports 8022 and 8080 to container ports 22 and 80,  respectively. The container runs in the background, allowing us to  access the SSH and HTTP services inside the container using the  specified host ports.

#### Docker Management

When managing Docker containers, Docker provides a comprehensive  suite of tools that enable us to easily create, deploy, and manage containers.

| **Command**      | **Description**               |
| ---------------- | ----------------------------- |
| `docker ps`      | List all running containers   |
| `docker stop`    | Stop a running container.     |
| `docker start`   | Start a stopped container.    |
| `docker restart` | Restart a running container.  |
| `docker rm`      | Remove a container.           |
| `docker rmi`     | Remove a Docker image.        |
| `docker logs`    | View the logs of a container. |

When working with Docker images, it's important to note that any  changes made to an existing image are not permanent. Instead, we need to create a new image that inherits from the original and includes the  desired changes.

This is done by creating a new Dockerfile that starts with the `FROM` statement, which specifies the base image, and then adds the necessary  commands to make the desired changes. Once the Dockerfile is created, we can use the `docker build` command to build the new image,  tagging it with a unique name to help identify it. This process ensures  that the original image remains intact while allowing us to create a new image with the desired changes.

It is important to note that Docker containers are designed to be  immutable, meaning that any changes made to a container during runtime  are lost when the container is stopped. Therefore, it is recommended to  use container orchestration tools such as Docker Compose or Kubernetes  to manage and scale containers in a production environment.

## Linux Containers

Linux Containers (`LXC`) is a virtualization technology  that allows multiple isolated Linux systems to run on a single host. It  uses resource isolation features, such as `cgroups` and `namespaces`, to provide a lightweight virtualization solution. LXC also provides a  rich set of tools and APIs for managing and configuring containers,  contributing to its popularity as a containerization technology.

Both LXC and Docker are containerization technologies that allow for  applications to be packaged and run in isolated environments. However,  there are some differences between the two that can be distinguished  based on the following categories:

- Approach
- Image building
- Portability
- Easy of use
- Security

LXC is a lightweight virtualization technology that uses resource  isolation features of the Linux kernel to provide an isolated  environment for applications. In LXC, images are manually built by  creating a root filesystem and installing the necessary packages and  configurations. Those containers are tied to the host system, may not be easily  portable, and may require more technical expertise to configure and  manage. LXC also provides some security features but may not be as  robust as Docker.

On the other hand, Docker is an application-centric platform that builds on top of LXC and provides a more user-friendly interface for  containerization.

#### Install LXC

```shell
DixLan@htb[/htb]$ sudo apt-get install lxc lxc-utils -y
```

Once LXC is installed, we can start creating and managing containers  on the Linux host. It is worth noting that LXC requires the Linux kernel to support the necessary features for containerization.

#### Creating an LXC Container

To create a new LXC container, we can use the `lxc-create` command followed by the container's name and the template to use. For example, to create a new Ubuntu container named `linuxcontainer`, we can use the following command:

```shell
DixLan@htb[/htb]$ sudo lxc-create -n linuxcontainer -t ubuntu
```

#### Managing LXC Containers

When working with LXC containers, several tasks are involved in  managing them. These tasks include creating new containers, configuring  their settings, starting and stopping them as necessary, and monitoring  their performance.

| Command                                       | Description                                                  |
| --------------------------------------------- | ------------------------------------------------------------ |
| `lxc-ls`                                      | List all existing containers                                 |
| `lxc-stop -n <container>`                     | Stop a running container.                                    |
| `lxc-start -n <container>`                    | Start a stopped container.                                   |
| `lxc-restart -n <container>`                  | Restart a running container.                                 |
| `lxc-config -n <container name> -s storage`   | Manage container storage                                     |
| `lxc-config -n <container name> -s network`   | Manage container network settings                            |
| `lxc-config -n <container name> -s security`  | Manage container security settings                           |
| `lxc-attach -n <container>`                   | Connect to a container.                                      |
| `lxc-attach -n <container> -f /path/to/share` | Connect to a container and share a specific directory or file. |

As penetration testers, we may encounter situations where we must test  software or systems with dependencies or configurations that are  difficult to reproduce on our machines. This is where Linux containers  come in handy. Since a Linux container is a lightweight, standalone  executable package containing all the necessary dependencies and  configuration files to run a specific software or system, it provides an isolated environment that can be run on any Linux machine, regardless  of the host's configuration.

We can also use them to test exploits or malware in a controlled  environment where we create a container that simulates a vulnerable  system or network and then use that container to safely test exploits  without risking damaging our machines or networks. However, it is  important to configure LXC container security to prevent unauthorized  access or malicious activities inside the container. This can be  achieved by implementing several security measures, such as:

- Restricting access to the container
- Limiting resources
- Isolating the container from the host
- Enforcing mandatory access control
- Keeping the container up to date

LXC containers can be accessed using various methods, such as SSH or  console. It is recommended to restrict access to the container by  disabling unnecessary services, using secure protocols, and enforcing  strong authentication mechanisms. For example, we can disable SSH access to the container by removing the `openssh-server` package or by configuring SSH only to allow access from trusted IP addresses.  Those containers also share the same kernel as the host system, meaning  they can access all the resources available on the system. We can use  resource limits or quotas to prevent containers from consuming excessive resources. For example, we can use `cgroups` to limit the amount of CPU, memory, or disk space that a container can use.

#### Securing LXC

Let us limit the resources to the container. In order to configure `cgroups` for LXC and limit the CPU and memory, a container can create a new configuration file in the `/usr/share/lxc/config/<container name>.conf` directory with the name of our container. For example, to create a configuration file for a container named `linuxcontainer`, we can use the following command:

```shell
DixLan@htb[/htb]$ sudo vim /usr/share/lxc/config/linuxcontainer.conf
```

In this configuration file, we can add the following lines to limit the CPU and memory the container can use.

```txt
lxc.cgroup.cpu.shares = 512
lxc.cgroup.memory.limit_in_bytes = 512M
```

When working with containers, it is important to understand the `lxc.cgroup.cpu.shares` parameter. This parameter determines the CPU time a container can use  in relation to the other containers on the system. By default, this  value is set to 1024, meaning the container can use up to its fair share of CPU time. However, if we set this value to 512, for example, the  container can only use half of the CPU time available on the system.  This can be a useful way to manage resources and ensure all containers  have the necessary access to CPU time.

One of the key parameters in controlling the resource allocation of a container is the `lxc.cgroup.memory.limit_in_bytes` parameter. This parameter allows you to set the maximum amount of  memory a container can use. It's important to note that this value can  be specified in a variety of units, including bytes, kilobytes (K),  megabytes (M), gigabytes (G), or terabytes (T), allowing for a high  degree of granularity in defining container resource limits. After  adding these two lines, we can save and close the file by typing:

- `[Esc]`
- `:`
- `wq`

To apply these changes, we must restart the LXC service.

```shell
DixLan@htb[/htb]$ sudo systemctl restart lxc.service
```

LXC use `namespaces` to provide an isolated environment  for processes, networks, and file systems from the host system.  Namespaces are a feature of the Linux kernel that allows for creating  isolated environments by providing an abstraction of system resources.

Namespaces are a crucial aspect of containerization as they provide a high degree of isolation for the container's processes, network  interfaces, routing tables, and firewall rules. Each container is  allocated a unique process ID (`pid`) number space, isolated  from the host system's process IDs. This ensures that the container's  processes cannot interfere with the host system's processes, enhancing  system stability and reliability. Additionally, each container has its  own network interfaces (`net`), routing tables, and firewall  rules, which are completely separate from the host system's network  interfaces. Any network-related activity within the container is  cordoned off from the host system's network, providing an extra layer of network security.

Moreover, containers come with their own root file system (`mnt`), which is entirely different from the host system's root file system.  This separation between the two ensures that any changes or  modifications made within the container's file system do not affect the  host system's file system. However, it is important to remember that  while namespaces provide a high level of isolation, they do not provide  complete security. Therefore, it is always advisable to implement  additional security measures to further protect the container and the  host system from potential security breaches.