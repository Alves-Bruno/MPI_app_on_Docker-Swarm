### How to run MPI application on docker-swarm

#### Step 1: Install docker

You can click [here](https://docs.docker.com/install/#supported-platforms) to go to the Docker's documentation on how to install Docker.

If you are using Mac, you can follow [these](https://docs.docker.com/docker-for-mac/install/) steps.  
If you are using Windows, you can follow [these](https://docs.docker.com/docker-for-windows/install/) steps.

For this tutorial we recommend a Linux based OS:  
If you are using Debian, you can follow [these](https://docs.docker.com/install/linux/docker-ce/debian/#set-up-the-repository) steps.  
If you are using Fedora, you can follow [these](https://docs.docker.com/install/linux/docker-ce/fedora/) steps.  
If you are using CentOS, you can follow [these](https://docs.docker.com/install/linux/docker-ce/centos/) steps.  
If you are using Ubuntu, you can follow [these](https://docs.docker.com/install/linux/docker-ce/ubuntu/) steps.

---

#### Step 2: Install docker-machine

You can follow this [guide](https://docs.docker.com/machine/install-machine/) to install docker-machine on your plataform.

If you are using Linux, you can follow the steps bellow:

```
base=https://github.com/docker/machine/releases/download/v0.16.0 &&
  curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
  sudo install /tmp/docker-machine /usr/local/bin/docker-machine
```

---

#### Step 3: Configure docker-swarm

Before you start configuring your scenario, we strongly recommend the reading of the articles listed bellow:

[https://docs.docker.com/engine/swarm/](https://docs.docker.com/engine/swarm/ "Swarm mode overview.")

[https://docs.docker.com/engine/swarm/key-concepts/](https://docs.docker.com/engine/swarm/key-concepts/ "Swarm mode key concepts.")

##### Scenario I: Run MPI-docker-swarm in 2 PCs \(or more\)

Note: We will configure two PCs to run MPI-docker-swarm, and we call them PC\_man \(configured as manager\) and PC\_wkr \(configured as worker\). The steps 1, 2, 3 must be done on PC\_man and PC\_wkr.

First, you need to start the swarm on the PC\_man.

Acces the PC\_man and execute:

```
ifconfig # To get the PC's IP
docker swarm init --advertise-addr <PC_man-IP>
```

```
$ sudo ifconfig

enp63s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.1.111  netmask 255.255.252.0  broadcast 10.0.3.255
        inet6 fe80::121f:74ff:fe40:e43b  prefixlen 64  scopeid 0x20<link>
        ether 10:1f:74:40:e4:3b  txqueuelen 1000  (Ethernet)
        RX packets 2513891  bytes 979328016 (933.9 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 117109  bytes 36316092 (34.6 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device interrupt 18  

$ docker swarm init --advertise-addr 10.0.1.111
Swarm initialized: current node (nexohc9t9ul0id76jdb7cpbs4) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-1ji7n3wibaud6ojqc0tgprnys5d0j7olqjm43cc261etvjots7-ecdpajicjtf6pjz5zv3ez0aqs 10.0.1.111:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

To add a worker to this swarm, run the following command \(on PC\_wkr\):

```
# Here you execute the 'docker swarm join' command outputed by the 'docker swarm init'
docker swarm join --token SWMTKN-1-1ji7n3wibaud6ojqc0tgprnys5d0j7olqjm43cc261etvjots7-ecdpajicjtf6pjz5zv3ez0aqs 10.0.1.111:2377
```

Ok, now you can execute the docker swarm join command on the PC\_wkr:

```
# In my case:
    docker swarm join \
    --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
    192.168.99.100:2377
```

To see if your configuration is running, you can check the nodes connected to swarm with \(on PC\_man\):

`docker node ls`

You should see 2 nodes, the PC\_man as manager and the PC\_wkr as worker.

##### Scenario II: Run MPI-docker-swarm with virtualbox \(only one PC needed\)

For this scenario you will need the VirtualBox installed. You can download it [here](https://www.virtualbox.org/wiki/Downloads).

Note: We will create 2 VirtualBox machines, they will be called VM-man \(configured as manager\) and VM-wkr \(configured as a worker\).

First, we need to create the VBox machines:

```
# Create the VM_man
docker-machine create --driver virtualbox VM-man
# Create the VM_wkr
docker-machine create --driver virtualbox default VM-wkr
```

You can use the `docker-machine ls` command to see the available machines on your PC.

```
$ docker-machine ls
NAME      ACTIVE   DRIVER       STATE     URL   SWARM   DOCKER    ERRORS
default   -        virtualbox   Stopped                 Unknown
VM-man    -        virtualbox   Running   tcp://192.168.99.100:2376           v18.09.0   
VM-wkr    -        virtualbox   Running   tcp://192.168.99.101:2376           v18.09.0
```

If the machines are not **Running**, you can start them with: `docker-machine start VM-man VM-wkr.`

OK, you can access the VM-man with:

```
$ docker-machine ssh VM-man
   ( '>')
  /) TC (\   Core is distributed with ABSOLUTELY NO WARRANTY.
 (/-_--_-\)           www.tinycorelinux.net

docker@VM-man:~$
```

Now, configure swarm on VM-man. The command bellow will initialize the docker swarm mode.

```
docker@VM-man:~$ docker swarm init --advertise-addr <VM-man eth1 IP>
Swarm initialized: current node (3pvpn4omithj20sbrkl91q69q) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-2wbhfili4q0obtd60wckdt730bhzcxin9g0i8ggos0rp4y527p-besp1mf5py9sb8yvt0igtrljx 192.168.99.100:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

On the VM-wkr machine \(acessed by: `docker-machine ssh VM-wkr`\), join the swarm mode with:

```
# Execute the token generate on the command 'docker swarm init',
# this command is just for an example.
docker@VM-wkr:~$ docker swarm join --token SWMTKN-1-2wbhfili4q0obtd60wckdt730bhzcxin9g0i8ggos0rp4y527p-besp1mf5py9sb8yvt0igtrljx 192.168.99.100:2377
This node joined a swarm as a worker.
```

On VM-man execute: `docker node ls`, if everything is working fine, you should see the nodes VM-man and VM-wkr with the status 'Ready'.

```
docker@VM-man:~$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
3pvpn4omithj20sbrkl91q69q *   VM-man              Ready               Active              Leader              18.09.0
r1nqe8hrx0d5rs4y2uyertgvz     VM-wkr              Ready               Active                                  18.09.0
```

---

#### Step 4: Configure docker-swarm network

On docker you can configure networks to be connect to the containers, as MPI uses the network to connect and run proccess we need all containers connected to the same network.

This [article](https://docs.docker.com/network/overlay/) explaines why we are using overlay network to connect the nodes on docker-swarm.

To create the mpi-swarm-net, execute this command on VM-man \(if you have configured the scenario II\) or on PC-man \(if you have configured the scenario I\):

```
docker network create -d overlay --attachable mpi-swarm-net
```

```
:~$ docker network create -d overlay --attachable mpi-swarm-net
xlbv8gxy4aa9mk2ljjxoufn6s

:~$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
5811ee5404cc        bridge              bridge              local
f2d4e91d04f7        docker_gwbridge     bridge              local
2ba9cedb420a        host                host                local
9t2hf2bcozd5        ingress             overlay             swarm
xlbv8gxy4aa9        mpi-swarm-net       overlay             swarm
```

---

#### Step 5: Configure the MPI service

We are going to use the files on that GitHub repository: [https://github.com/dispel4py/docker.openmpi](https://github.com/dispel4py/docker.openmpi), to run a MPI application.

Note: Execute the commands listed here \(Step 5\) on VM-man \(if you have configured the scenario II\) or on PC-man \(if you have configured the scenario I\).

First, we are going to download the container image that will run on the containers at the nodes.

```
docker pull dispel4py/docker.openmpi
# You can download the container image on the worker node too (PC-wkr or VM-wkr), 
# that will make the service start faster.
```

To deploy the MPI service on docker-swarm, you should run:

`docker service create --replicas 10 --network mpi-swarm-net --name mpi-service dispel4py/docker.openmpi`

To see the nodes running your service, you can execute: `docker service ps mpi-service`

```
~$ docker service ps mpi-service 
ID                  NAME                IMAGE                             NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
pmgrzmq1n3e5        mpi-service.1       dispel4py/docker.openmpi:latest   VM-man              Running             Running 32 minutes ago                       
yhv8qfav43eu        mpi-service.2       dispel4py/docker.openmpi:latest   VM-wkr              Running             Running 32 minutes ago                       
me3k8utqbqzt        mpi-service.3       dispel4py/docker.openmpi:latest   VM-wkr              Running             Running 32 minutes ago                       
0u8hd0o1qhs7        mpi-service.4       dispel4py/docker.openmpi:latest   VM-man              Running             Running 32 minutes ago                       
x7z9c5kvb4op        mpi-service.5       dispel4py/docker.openmpi:latest   VM-wkr              Running             Running 32 minutes ago                       
pc0cdxrx5c7p        mpi-service.6       dispel4py/docker.openmpi:latest   VM-man              Running             Running 32 minutes ago                       
xxjojgq7wv1e        mpi-service.7       dispel4py/docker.openmpi:latest   VM-man              Running             Running 32 minutes ago                       
uyhz1os73g3h        mpi-service.8       dispel4py/docker.openmpi:latest   VM-wkr              Running             Running 32 minutes ago                       
5zw1boy44mel        mpi-service.9       dispel4py/docker.openmpi:latest   VM-wkr              Running             Running 32 minutes ago                       
uj47souatq4w        mpi-service.10      dispel4py/docker.openmpi:latest   VM-man              Running             Running 32 minutes ago
```

---

#### Step 6: Run the MPI application

\(On VM-man or PC-man\) First you need to download the files from [https://github.com/dispel4py/docker.openmpi](https://github.com/dispel4py/docker.openmpi):

```
git clone https://github.com/dispel4py/docker.openmpi
cd docker.openmpi/
```

Get the IP list of the running containers \(you have to execute this command on both nodes: manager and worker\):

```
docker@VM-man:~$ docker ps -q | xargs -n 1 docker inspect --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' | sed 's/ \// /'
10.0.1.7
10.0.1.8
10.0.1.11
10.0.1.5
10.0.1.10

docker@VM-wkr:~$ docker ps -q | xargs -n 1 docker inspect --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}} ' | sed 's/ \// /'
10.0.1.12 
10.0.1.4 
10.0.1.3 
10.0.1.6 
10.0.1.9
```

Create a file called "machines" on the manager and past the listed IPs:

```
# You can create the file with the vi editor
docker@VM-man:~$ cat machines  
10.0.1.12 
10.0.1.4 
10.0.1.3 
10.0.1.6 
10.0.1.9 
10.0.1.7
10.0.1.8
10.0.1.11
10.0.1.5
10.0.1.10
```

Start the container \(mpi-head\) that will command the others containers:

`docker run --expose 22 --name mpi-head dispel4py/docker.openmpi`

Connect the mpi-head to the mpi-swarm-net:

```
docker network connect mpi-swarm-net mpi-head
```

OK, now copy the machines file into the mpi-head container:

```
docker cp machines mpi-head:/home/tutorial/mpi4py_benchmarks
```

Connect to the mpi-head container:

```
# Change the ssh key access permission
chmod 400 ssh/id_rsa.mpi
# Get the mpi-head IP:
docker@VM-man:~/docker.openmpi$ docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mpi-head    
172.17.0.210.0.1.15
# Connect to the mpi-head:
docker@VM-man:~/docker.openmpi$ ssh -i ssh/id_rsa.mpi -p 22 tutorial@172.17.0.2    
Welcome to Ubuntu 14.04 LTS (GNU/Linux 3.13.0-92-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

tutorial@24577f1b15b6:~$
```

Finally, execute the MPI application:

```
tutorial@24577f1b15b6:~/mpi4py_benchmarks$ cd mpi4py_benchmarks
tutorial@24577f1b15b6:~/mpi4py_benchmarks$ mpiexec -hostfile machines -n 10 python helloworld.py 
Hello, World! I am process 7 of 10 on dea8e62d250d.
Hello, World! I am process 9 of 10 on 92827432adda.
Hello, World! I am process 5 of 10 on 773a46000572.
Hello, World! I am process 8 of 10 on 9257099ba41b.
Hello, World! I am process 6 of 10 on 8fcf45bdb646.
Hello, World! I am process 4 of 10 on bc6a67cc3793.
Hello, World! I am process 1 of 10 on 620a5056e4bc.
Hello, World! I am process 2 of 10 on b6121a9df0ed.
Hello, World! I am process 0 of 10 on 6cd4ab98b500.
Hello, World! I am process 3 of 10 on b7ba567407b9.
```



