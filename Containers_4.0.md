# Linux Containers LXC/LXD Deep Dive

- [Linux Containers LXC/LXD Deep Dive](#linux-containers-lxclxd-deep-dive)
  - [Getting Started With LXC/LXD](#getting-started-with-lxclxd)
    - [Into to Containers](#into-to-containers)
  - [LXC/LXD: Installation and Configuration](#lxclxd-installation-and-configuration)
    - [Setting up the RaspberryPi for running Containers](#setting-up-the-raspberrypi-for-running-containers)
    - [Installing LXC/LXD](#installing-lxclxd)
      - [Install LXD](#install-lxd)
        - [Back to Linux Academy](#back-to-linux-academy)
      - [Lecture: Troubleshooting LXD Installations](#lecture-troubleshooting-lxd-installations)
    - [Launching Your First Container](#launching-your-first-container)
      - [Launching Your First Container, Part 1](#launching-your-first-container-part-1)
      - [Launching Your First Container, Part 2](#launching-your-first-container-part-2)
    - [Lecture: Launching Your First Container, Part 3](#lecture-launching-your-first-container-part-3)
    - [Exercise: Launching Your First Container](#exercise-launching-your-first-container)
  - [LXC/LXD Images](#lxclxd-images)
    - [LXC Images](#lxc-images)
      - [LXC Images Part 1: Remotes](#lxc-images-part-1-remotes)
      - [LXC Images Part 2: Remotes](#lxc-images-part-2-remotes)

> 5-4-2019

[LXC/LXD Deep Dive](https://linuxacademy.com/cp/modules/view/id/140): goes into the Linux containers. Runs Machine containers,

## Getting Started With LXC/LXD 

### Into to Containers

+

## LXC/LXD: Installation and Configuration

### Setting up the RaspberryPi for running Containers

> `Raspbian 2019-4-8` currently doesn't allow containerization.

+ Downloaded [ubuntu 18.04](https://wiki.ubuntu.com/ARM/RaspberryPi#Booting_generic_arm64_ISO_images) RPi 3B iso image.
+ Installed with the Etcher app on the microSD
+ Booted the RaspberryPi and ran the initial setup.
+ used `ssh ubuntu@<ipaddress>` to get access into the Pi.

### Installing LXC/LXD

#### Install LXD

Next we check for the LXC and `lxc-config` apps:
```
$ whereis lxd
lxd: /usr/bin/lxd /usr/lib/lxd /usr/share/man/man1/lxd.1.gz
$ whereis lxd-client
lxd-client:
$ sudo apt install lxd-client lxd
...
lxd-client is already the newest version (3.0.3-0ubuntu1~18.04.1).
...
```

##### Back to Linux Academy


> Started the RPi and did `update` and `upgrade`

Setup lxd with:
```
$ sudo lxd init
Would you like to use LXD clustering? (yes/no) [default=no]: 
Do you want to configure a new storage pool? (yes/no) [default=yes]:
Would you like to connect to a MAAS server? (yes/no) [default=no]: 
Would you like to create a new local network bridge? (yes/no) [default=yes]: 
What should the new bridge be called? [default=lxdbr0]: 
The requested network bridge "lxdbr0" already exists. Please choose another name.
What should the new bridge be called? [default=lxdbr0]: lsdbr0
What IPv4 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 
What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 
Would you like LXD to be available over the network? (yes/no) [default=no]: yes
Address to bind LXD to (not including port) [default=all]: 
Port to bind LXD to [default=8443]: 
Trust password for new clients: 
Again: 
Would you like stale cached images to be updated automatically? (yes/no) [default=yes] 
Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]:
```
NOTE: The above is mostly default.

Download / create a new container:
```
lxc launch ubuntu:18.04 ubuntu
```

Now we'll go into the new container:
```
$ lxc exec ubuntu -- /bin/bash
```

And let's look around:
```
$ ls -al
...
$ cd \
$ df -h
...
$ exit
```

Here we will download and create a new container:
```
$ lxc launch images:alpine/3.8 alpine
$ lxc list
...
```

Now Alpine is a very small container and doesn't have alot of functionality so to look around we'll use `ash`:
```
$ lxc exec alpine -- ash
~ # ls -al
...
~ # cd \
~ # df -h
...
~ # exit
```

#### Lecture: Troubleshooting LXD Installations

I didn't have any problems. and most of what he did here was what was done in the beginning. The only note would be to re-run the `dpkg-reconfigure`:
```
$ sudo dpkg-reconfigure -p medium lxd
```

### Launching Your First Container

#### Launching Your First Container, Part 1 

Now create a local copy of a container and name it alp: 
```
$ lxc image copy images:alpine/3.8 local: --alias alp
```

Launch a new container
```
lxc launch alp web
```

Check for updates out side the container
```
$ lxc exec web -- apk update
```

Move into the container with /bin/bash or ash or bashsh
```
$ lxc exec web -- ash
```

add nginx to the `web` container
```
$ apk add nginx
```

Next is neat, we will edit a config file inside the `web` container from the host PC while `ssh`ing into the host PC:
```
$ lxc file edit web/etc/nginx/conf.d/default.conf
```

Remove most of the file, in the end it should look like:
```
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www;
}
```

#### Launching Your First Container, Part 2

Enter the container and take a look at the web/etc/nginx/conf.d/default.conf and contents of /var/www

Create a index.html inside the `web` container without ever entering it (i.e. from outside the container)

        $ lcx file edit web/var/www/index.html
        Served from the web container!
        ...

Check the web

        $ curl 10.186.94.33


Create a snapshot of the container

        $ lxc snapshot web initialconfig


Restore a snapshot

        $ lxc restore web initialconfig


Test restore!!!

### Lecture: Launching Your First Container, Part 3

**Snapshots!!!** can be used to spin up containers when using **load balancers**

Create a new container from the initial config snapshot of `web`:

        $ lxc copy web/initialconfig web2


Check and then start the container:

        $ lxc list
        ...
        $ lxc start web2
        $ lxc list
        ...
 

> 5-5-2019

Edit HTML pages of both the `web2` and `web` containers from outside. Again these container are running on the RaspberryPi, hardwired into my local network.

        $ lcx file edit web2/var/www/index.html
        Served from the web2 container!

        $ lcx file edit web2/var/www/index.html
        Served from the first web container!

Now we'll take an other snapshot:

        $ lxc snapshot web 1.1
        $ lxc snapshot web2 1.1
        
Calling them both `1.1` will show that they are synchronized.

Now we'll `delete` a snapshot:

        $ lxc delete web/1.1

Note that trying to delete a running container will:

        $ lxc delete web
        Error: The container is currently running, stop it first or pass --force


Remember to check on what containers are running:

        $ lxc list

Lets cleanup by stopping the running containers and then deleting them:

        $ lxc stop web
        $ lxc stop web2
        $ lxc delete web
        $ lxc delete web2
        $ lxc list

Now lets list and delete some of the images we have saved:

        $ lxc image list
        .
        .
        .
        $ lxc image delete alp
        .
        .
        .   

### Exercise: Launching Your First Container

1. Launch a container named 'web' based on the Alpine v3.5 image.
2. Install and configure nginx in your new container without logging in to the container's console.
3. Create a test page called "index.html" and place it in the web root with the following content:  "Web page served out of an LXC container!"
4. Log into the container's console and set nginx to automatically start when the container starts (The command is rc-update add nginx default.)
5. Start nginx in your container.
6. Exit your container and verify that everything is functioning as expected using the curl command with your container's IP address. Note that this is a private IP address and will only be accessible from the console of the LXD host.
7. Create a snapshot of your container called 1.0.
8. Change the contents of the index.html in the web root to read, "Typos are horrible!" and verify the changes took place with curl.
9. Create a snapshot of your container called 1.1.
10. Restore your container to the 1.0 state and verify that the changes took place.
11. Delete the 1.1 snapshot.


## LXC/LXD Images

### LXC Images

#### LXC Images Part 1: Remotes 

5/5/2019 5:00pm
NOTE: upon restart of the RaspberryPi `$ lxc list` would hang, as would all commands to `lxd`. Removed and reinstalled `lxd` and `lxd-client`.
Then ran through starting up the lxd services.

Where do these images come from? **LXC remotes** use the following to see what remote repositories currently setup on the machine:

```
$ lxc remote list 
...
```

To create your own remote (I used a container on the Pi to be the host of a repository):

Log into your **remote** machine you'd like to use as a repository and locate the ip address listed as `inet addr: <ipaddress>`

```
$ ifconfig
... inet addr: 
...
```

Set the port and address to listen to:

        $ lxc config set core.https_address "[::]:8443"


Set the password so that other `lxd daemons` can use to access the lxd server as a remote.

        $ lxc config set core.trust_password <password>
        exit

Now return to the main machine and add the remote:
```
$ lxc remote add second 172.31.23.109:8443 --password=secret
$ lxc list second:
```

To see images available that are currently installed on on other lxd servers:
```
$ lxc image list
$ lxc image list second:
```

#### LXC Images Part 2: Remotes