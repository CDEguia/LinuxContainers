## LXC/LXD Images

### LXC Images

#### LXC Images Part 1: Remotes 

5/5/2019 5:00pm
> NOTE: upon restart of the RaspberryPi `$ lxc list` would hang, as would all commands to `lxd`. Removed and reinstalled `lxd` and `lxd-client`.
> Then ran through starting up the lxd services.

Where do these images come from? **LXC remotes** use the following to see what remote repositories currently setup on the machine:

    $ lxc remote list 
    ...

To create your own remote (I used a container on the Pi to be the host of a repository):

Log into your **remote** machine you'd like to use as a repository and locate the ip address listed as `inet addr: <ipaddress>`

    $ ifconfig
    ... inet addr: 
    ...

> Note when using a local container, log into the container and run `$ lxc init`

Set the port and address to listen to:

    $ lxc config set core.https_address "[::]:8443"

Set the password so that other `lxd daemons` can use to access the lxd server as a remote.

     $ lxc config set core.trust_password <password>
     exit

Now return to the main machine and add the remote:

    $ lxc remote add second 172.31.23.109:8443  --password=secret
        $ lxc list second:

To see images available that are currently installed on on other lxd servers:

    $ lxc image list
    $ lxc image list second:

#### LXC Images Part 2: Publishing, Exporting, and Examining

Publish an LXC image so others can spin up the container:

    $ lxc publish [container name]/[snapshot] --alias [name]

Spin up a new one

    $ lxc launch  [container name] --alias [name]

> Note images are Tar balls and is out  of scope

But here are the basics:

Make as dir to store the image:

    $ mkdir [dir name]

Export and existing image into the folder as a tarball:

    $ lxc image export [container image] [dir name/]

Now we'll uncompress and look at the files

    $ sudo tar -xvf [tarball].tar.gz
    .
    .
    .

Three the main things are created the `metadata.yaml`, `rootfs/`, `templates/`

The metadata.yaml lists lots of information about the container.

The `templates/' contain simple text files that gets inserted into the root file system.

In the `root/` you will find the standard root operating system.

#### LXC Images Part 3: Creating From Scratch