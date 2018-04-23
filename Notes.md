# Docker notes from a poor SysAdmin
---

## Build Dockerfile

    sudo docker build -t centos7/nonroot:v1 .

## Create container interactive and attach terminal from image

    sudo docker run -it --name mycontainer centos7/nonroot:v1 /bin/bash

## Create container from image as daemon

    sudo docker run -d --name mycontainer centos7/nonroot:v1

## Start non running container

    sudo docker start sleepy_allen

## Check running containers

    sudo docker ps

## Exec on container as root user id (-u 0)

    sudo docker exec -u 0 -it sleepy_allen /bin/bash

    sudo docker exec mycontainer /bin/cat /var/www/html/index.html

## Expose all exposed ports on the image with EXPOSE

    sudo docker run -d --name mycontainer -P centos7/nonroot:v1

## Force expose of ports even if no port exposed on the image

    sudo docker run -d --name mycontainer -p 8080:80 centos7/nonroot:v1

## Create volume on container, you will get the path /mydata from host (/var/lib/docker/volumes) inside the container

    sudo docker run -it --name mycontainer -v /mydata centos:latest /bin/bash 

## Map local path to container

    sudo docker run -it --name mycontainer -v /home/oscar:/mydata centos:latest /bin/bash

## Restart container

    sudo docker restart mycontainer

## Stop container

    sudo docker stop mycontainer

## Create without run a container

    sudo docker create -it --name="mycontainer" ubuntu:latest /bin/bash

## Attach to running container

    sudo docker attach my_container

## List networks

    sudo docker network ls

## Create network

    sudo docker network create --subnet 10.1.0.0/24 --gateway 10.1.0.1 mynetwork01 

## Delete network, do not delete default networks

    sudo docker rm mynetwork01

## Create network with IP range to assign to containers

    sudo docker network create --subnet 10.1.0.0/16 --gateway 10.1.0.1 --ip-range=10.1.4.0/24 --drive=bridge --label=host4network MyNetworkBridge04

## Add container to network

    sudo docker run -it --name nettest1 --net MyNetworkBridge04 centos:latest /bin/bash

## Add container to network and assign specific IP address

    sudo docker run -it --name nettest2 --net MyNetworkBridge04 --ip 10.1.4.100 centos:latest /bin/bash

## Inspect docker container

    sudo docker inspect nettest2 | grep IPAddress

## Inspect container processes

    sudo docker exec mycontainer01 /bin/ps aux | grep bash

    sudo docker top mycontainer01 

    sudo docker exec -it mycontainer01 /bin/bash

## Get container statistics

    sudo docker stats mycontainer01

## Get non running container IDs

   sudo docker ps -a -q

## Get last non running container ID

    sudo docker ps -a -l -q

## Force running container deletion

    sudo docker rm -f mycontainer01

## Delete containers using file system

    sudo systemctl stop docker

    cd /var/lib/docker/
    cd containers && ls -alrth

    rm -rf <container_string_path> 

## Controlling Port Exposure on Containers

### The exposed ports on the Dockerfile image are not exposed to the host automatically

    sudo docker -itd nginx:latest #Not mapping to the host

    sudo docker ps

    CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS               NAMES

    992fd75a8b59        nginx:latest        "nginx -g 'daemon of…"   About a minute ago   Up 59 seconds       80/tcp              laughing_kare

### With -p port the ports are mapped to the host

    sudo docker run -itd -p 80 nginx:latest #Mapping to the host

    sudo docker run -itd -p 8080:80 nginx:latest #Specifying mapping port on host

    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                   NAMES

    d93cc46cfcf6        nginx:latest        "nginx -g 'daemon of…"   5 seconds ago       Up 2 seconds        0.0.0.0:32768->80/tcp   boring_swanson

### To expose automatically from the Dockerfile you must define them as, this way you won't need to use -p flag:

    EXPOSE 8080:80

    EXPOSE 4343:443

### Expose all the ports on the Dockerfile/Image and map to the host

    sudo docker run -itd -P nginx:latest

### Define also host:port:container_port

    sudo docker run -itd -p 127.0.0.1:8081:80 nginx:latest #In this case only mapped to 127.0.0.1 interface like 127.0.0.1:8081->80/tcp

    sudo docker run -itd -p 127.0.0.1:8081:80/udp nginx:latest

    sudo docker run -itd -p 127.0.0.1:8081:80/tcp nginx:latest

## Naming containers

    sudo docker run -itd -P --name MyArchContainer01 base/archlinux /bin/bash

### Rename container

    sudo docker rename MyArchContainer01 MyArch01

## Docker Events

    sudo docker events

    sudo docker events --since '1h'

    sudo docker events --filter event=attach #container|event|image|type|volume|network|daemon

## Docker Images

    sudo docker images

    sudo docker rmi mycontainer

## Saving and Loading images

### Commit container to image

    sudo docker commit mycontainer oscargcervantes:mycontainer #commit a container to a docker image

### Save container to tar file

    sudo docker save oscargcervantes:mycontainer > mycontainer_latest.tar #save image to tar file

    sudo docker save -o mycontainer_latest.tar oscargcervantes:mycontainer

    sudo docker save --output mycontainer_latest.tar oscargcervantes:mycontainer

### Load container fron tar file

    sudo docker load < mycontainer_latest.tar

    sudo docker load --input mycontainer_latest.tar

### Compress the tar image

    gzip mycontainer_latest.tar

## Image history

    sudo docker history oscargcervantes:mycontainer

    sudo docker history --no-trunc oscargcervantes:mycontainer

    sudo docker history --quiet --no-trunc oscargcervantes:mycontainer

## Delete docker images and containers

    sudo docker stop $( sudo docker ps -q)

    sudo docker rm $( sudo docker ps -a -q )

    sudo docker rmi -f $( sudo docker images -q )

## Docker tags

    sudo docker tag <imageID> mine/centos:v1.0

    sudo docker tag <imageName> mine/centos:v2.0

## Dockerhub push images

    sudo docker tag <imageID or imageName> oscargcervantes/mycontainers

    sudo docker push oscargcervantes/mycontainers

    sudo docker pull oscargcervantes/mycontainers:latest

## Options to create container

    sudo docker run -it --dns=8.8.8.8 --dns-search="mydomain.local" --name="mycontainer2" docker.io/ubuntu:latest /bin/bash

    sudo docker run -it --dns=8.8.8.8 --dns-search="mydomain.local" --name="mycontainer3" -v /local_vol -v /home/tcox:/remote_vol docker.io/ubuntu:latest /bin/bash

# Building a web farm for Dev and Test with Docker

## Pull the image

    sudo docker pull centos:centos6

## Create container

    sudo docker run -it centos:centos6 /bin/bash #Install epel repo

### Inside container install epel repo

    yum -y update
    yum -y install which sudo httpd php openssh-server git

#### Add to .bashrc

    /sbin/service httpd start
    /sbin/service sshd start

## Commit the changes to a new image

    sudo docker commit <containerID or Name> centos6:baseweb

## Create new container based on centos6:baseweb to check changes (not required)

    sudo docker run -it centos6:baseweb /bin/bash

## Download web server code on local computer

    cd /home/oscar
    mkdir docker
    cd docker

    mkdir dockerwww
    cd dockerwww

    wget http://static.oswd.org/designs/3682/bluefreedom3.zip

    unzip bluefreedom3.zip
    mv bluefreedom3/* .

    rm -rf bluefreedom3/

    git init .
    git status
    git add -A
    git status
    git commit -m "Initial commit"
    git push -u origin master

## Create container to make more changes

    sudo docker run --name=webtest -p 8443:443 -p 8080:80 -v /home/oscar/docker/dockerwww:/var/www/html -it centos6:baseweb /bin/bash

## Commit all the changes

    sudo docker commit webtest centos6:finalwebv1

## Rename dockerwww to dockergit to have the repo separated

    cd /home/oscar/docker

    mv dockerwww dockergit

    git clone root@localhost:/home/oscar/docker/dockergit dockerwww

## Start docker containers from last commited image

    sudo docker run -itd --name=devweb1 -p 8081:80 -v /home/oscar/docker/dockerwww:/var/www/html centos6:finalwebv1 /bin/bash
    sudo docker run -itd --name=devweb2 -p 8082:80 -v /home/oscar/docker/dockerwww:/var/www/html centos6:finalwebv1 /bin/bash
    sudo docker run -itd --name=devweb3 -p 8083:80 -v /home/oscar/docker/dockerwww:/var/www/html centos6:finalwebv1 /bin/bash

## Inspect containers

    sudo docker inspect devweb1 | grep IpAddr
    sudo docker inspect devweb2 | grep IpAddr
    sudo docker inspect devweb3 | grep IpAddr

## Install nginx on the host

    sudo yum install nginx
    sudo service nginx start

## Create a nginx proxy

    vi /etc/nginx/sites-available/default.conf

#### 192.168.0.5 is my local host IP

Add:

    //Nginx configuration
   
    upstream containerapp {
        server 192.168.0.5:8081;
        server 192.168.0.5:8082;
        server 192.168.0.5:8083;
    }
    
    server {
        listen *:80;
        
        servername 192.168.0.5;
        index index.html index.htm index.php
        
        acces_log /var/log/nginx/localweb.log;
        error_log /var/log/nginx/localerr.log;
        
        location / {
        
            proxy_pass http://containerapp;
        }
    }

service nginx restart

## Custom Network and IP addresses range

    ip link add br10 type bridge
    ip addr 10.10.100.1/24 dev br10
    ip link set br10 up
    ifconfig or ip addr show

    sudo docker -d -b br10 & #To launch docker service with custom created network
    sudo docker run -itd centos:centos6 /bin/bash #It will be started on the custom created network, only temporal not persistent

#### Edit /etc/network or /etc/sysconfig/network-scripts/...... and edit the configuration files to have the bridge br10 as persistent

#### Ubuntu:

    auto br10
    iface br10 inet static
            address 10.10.100.1
            netmask 255.255.255.0
            bridge_ports dummy0
            bridge_stp off
            bridge_fd 0
    
#### RHEL/CentOS/Fedora:

    Edit, copy and rename eth0 config file





