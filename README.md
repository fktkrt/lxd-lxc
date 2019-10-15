# lxd-lxc

## install lxd on Debian

First, install snapd and lxd

```
apt install snapd
snap install lxd
```

Your output will look like this
`lxd 3.18 from 'canonical' installed`

Update your current PATH
`. /etc/profile.d/apps-bin-path.sh`

You can initalize your lxd config now 
`lxd init`

Example config printed in yml

```
config: {}
networks:
- config:
    ipv4.address: auto
    ipv6.address: auto
  description: ""
  managed: false
  name: lxdbr0
  type: ""
storage_pools:
- config:
    size: 2GB
  description: ""
  name: default
  driver: btrfs
profiles:
- config: {}
  description: ""
  devices:
    eth0:
      name: eth0
      nictype: bridged
      parent: lxdbr0
      type: nic
    root:
      path: /
      pool: default
      type: disk
  name: default
cluster: null
```

You can use `lxd list` to view your available containers

Sampel output if you have zero containers

```
+------+-------+------+------+------+-----------+
| NAME | STATE | IPV4 | IPV6 | TYPE | SNAPSHOTS |
+------+-------+------+------+------+-----------+
```

You can create your nginx container named frontend:

```
lxc launch ubuntu:18.04 frontend
```

Note that you can list the available versions of the given distro by:

`lxc image list image:`

If you list your images now, you can see your frontend

```
+----------+---------+-----------------------+-----------------------------------------------+------------+-----------+
|   NAME   |  STATE  |         IPV4          |                     IPV6                      |    TYPE    | SNAPSHOTS |
+----------+---------+-----------------------+-----------------------------------------------+------------+-----------+
| frontend | RUNNING | 10.39.244.187 (eth0) | fd42:33bb:da01:f7f2:216:3eff:fe07:cd22 (eth0) | PERSISTENT | 0         |
+----------+---------+-----------------------+-----------------------------------------------+------------+-----------+
```

Login to the frontend with the following command

`lxc exec frontend -- sudo --login --user ubuntu`

Install nginx on the image

```
sudo apt update
sudo apt install nginx
```

Edit the config file:

`sudo nano /var/www/html/index.nginx-debian.html`

```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx on LXD!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx on LXD!</h1>
...
```

Logout

`logout`


If you curl your containers ip address by

`curl http://10.39.244.187`

You should get the content of the edited file.

Now you have to forward incoming connections to your container.

Add the following to your iptables:

```
PORT=80 PUBLIC_IP=192.168.163.128 CONTAINER_IP=10.39.244.187 \
sudo -E bash -c 'iptables -t nat -I PREROUTING -i eth0 -p TCP -d $PUBLIC_IP --dport $PORT -j DNAT --to-destination $CONTAINER_IP:$PORT -m comment --comment "forward to the Nginx container"'

```

List iptables routes by

`sudo iptables -t nat -L PREROUTING`

Output

```
target     prot opt source               destination
DNAT       tcp  --  anywhere             192.168.163.128      tcp dpt:http /* forward to the Nginx container */ to:10.39.244.187:80
```

If you curl your host's address

` curl --verbose  'http://192.168.163.128'`

You should see an output like this:

```
*   Trying 192.168.163.128...
* Connected to 192.168.163.128 (192.168.163.128) port 80 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.47.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.10.0 (Ubuntu)
...
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx on LXD!</title>
<style>
    body {
...
```

You can stop your container by

`lxc stop frontend`

And delete it by

`lxc delete fronted`
