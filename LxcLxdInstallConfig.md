## LXC/LXD: Installation and Configuration

### Installing LXC/LXD## LXC/LXD: Installation and Configuration

### Installing LXC/LXD

#### Install LXD

Next we check for the LXC and `lxc-config` apps:

    $ whereis lxd
    lxd: /usr/bin/lxd /usr/lib/lxd /usr/share/man/man1/lxd.1.gz
    $ whereis lxd-client
    lxd-client:
    $ sudo apt install lxd-client lxd
    ...
    lxd-client is already the newest version (3.0.3-0ubuntu1~18.04.1).
    ..

Setup lxd with:

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

NOTE: The above is mostly default.

Download / create a new container:

    $ lxc launch ubuntu:18.04 ubuntu


Now we'll go into the new container:

    $ lxc exec ubuntu -- /bin/bash


And let's look around:

    $ ls -al
    ...
    $ cd \
    $ df -h
    ...
    $ exit

Here we will download and create a new container:

    $ lxc launch images:alpine/3.8 alpine
    $ lxc list
    ...

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
$ lxc launch alp web
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
    ...
    ...
    ...
    $ lxc image delete alp
    ...
    ...
    ...   

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