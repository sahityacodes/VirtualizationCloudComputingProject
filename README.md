# VirtualizationCloudComputingProject

Objective: 

To build a distributed architecture consisting of 3 Virtual Machines (Ubuntu Server 16.04).  The three machines will be placed in a Docker Swarm and will share a file system thanks to GlusterFS technology. 
The Swarm will house the following containers:
 træfik, Joomla, Prometheus, MariaDB, Grafana

Setup commands

sudo apt install curl
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-get install software-properties-common
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
sudo usermod -aG docker [user]
sudo systemctl restart docker
Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

Clone -> VM2 and VM3 from VM1

![Image](https://drive.google.com/file/d/1jDpqFHpVF9In5qHod_ZDKoOZBqt1vOJX/view?usp=sharing)

sudo nano /etc/network/interfaces
auto ens38
iface ens38 inet static
address 10.0.0.11
netmask 255.255.255.0
network 10.0.0.0
gateway 10.0.0.2 
 
sudo nano /etc/hosts
VMware GlusterFS
10.0.0.11 vm1g
10.0.0.12 vm2g
10.0.0.13 vm3g

sudo reboot
ping vm2g

sudo apt update; 
sudo apt -y install glusterfs-server glusterfs-client

Start the service:
sudo service glusterfs-server start

Peer the service
sudo gluster peer probe vm2g
sudo gluster peer probe vm3g

Check the status
sudo gluster peer status

Bricks folder
sudo mkdir -p /gluster/bricks/1 (on vm1)
sudo mkdir -p /gluster/bricks/2 (on vm2)
sudo mkdir -p /gluster/bricks/3 (on vm3)

Add the line to the / etc / fstab file on the VMs  for mount the bricks
sudo on 
echo '/dev/sdb /gluster/bricks/1 ext4 defaults 0 0' >> /etc/fstab (on vm1)
echo '/dev/sdb /gluster/bricks/2 ext4 defaults 0 0' >> /etc/fstab (on vm2)
echo '/dev/sdb /gluster/bricks/3 ext4 defaults 0 0' >> /etc/fstab (on vm3)

Mount filesystem
sudo mount -a (on vm1, vm2, vm3)

Create gfs volume
sudo gluster volume create gfs replica 3 \
vm1g:/gluster/bricks/1/brick \
vm2g:/gluster/bricks/2/brick \
vm3g:/gluster/bricks/3/ brick 

Start  volume
sudo gluster volume start gfs

Check the status
sudo gluster volume info gfs
sudo gluster volume status gfs

Three VM to connect to gfs on each node
sudo gluster volume set gfs auth.allow 10.0.0.11, 10.0.0.12,10.0.0.13
sudo gluster volume set gfs nfs.disable Off

Mount the volume 
sudo mkdir /gfs
sudo su
echo 'localhost:/gfs /gfs glusterfs defaults,_netdev,backupvolfile-server=localhost 0 0' >> /etc/fstab
sudo mount -a
 
Set current user as owner to /gfs
sudo su
chown -R sahitya : sahitya /gfs/

Create a file on a virtual machine
echo “Hello world”> /gfs/hello.txt

Verify on other two VM
cat /gfs/hello.txt

Create folders
mkdir /gfs/joomla
mkdir /gfs/phpmyadmin
mkdir /gfs/mysql
mkdir /gfs/grafana
chmod -R a=rwx /gfs/grafana/

Swam Manager
docker swarm init --advertise-addr ens33
docker swarm join-token manager

Swam worker 
docker node promote <node name>
docker node demote <node name>

Building images
docker build . -t joomla_img
docker build . -t pmy_img

Docker stack Deploy
docker stack deploy joomlastack -c docker-compose.yml
docker stack ps joomlastack
docker stack ps joomlastack --no-trunc
docker stack services joomlastack
docker service logs <service name>
docker stack rm joomlastack

Scaling up and down
docker node update --availability Drain vm2
docker node update --availability Active vm2
docker service scale joomlastack_joomla=2 











