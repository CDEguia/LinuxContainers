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