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
<br /> 
Clone -> VM2 and VM3 from VM1
<br /> 
![Image](https://drive.google.com/file/d/1jDpqFHpVF9In5qHod_ZDKoOZBqt1vOJX/view?usp=sharing)
<br /> 
sudo nano /etc/network/interfaces
auto ens38
iface ens38 inet static
address 10.0.0.11
netmask 255.255.255.0
network 10.0.0.0
gateway 10.0.0.2 
<br /> 
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
sudo su<br /> <br /> 
echo 'localhost:/gfs /gfs glusterfs defaults,_netdev,backupvolfile-server=localhost 0 0' >> /etc/fstab<br /> 
sudo mount -a<br /> 
 
Set current user as owner to /gfs<br /> 
sudo on<br /> 
chown -R sahitya : sahitya /gfs/<br /> 

Create a file on a virtual machine<br /> <br /> 
echo “Hello world”> /gfs/hello.txt<br /> 

Verify on other two VM<br /> 
cat /gfs/hello.txt<br /> 

Create folders<br /> 
mkdir /gfs/joomla<br /> 
mkdir /gfs/phpmyadmin<br /> 
mkdir /gfs/mysql<br /> 
mkdir /gfs/grafana<br /> 
chmod -R a=rwx /gfs/grafana/<br /> 
<br /> 
Swam Manager<br /> 
docker swarm init --advertise-addr ens33<br /> 
docker swarm join-token manager<br /> 
<br /> 
Swam worker<br /> 
docker node promote <node name><br /> 
docker node demote <node name><br /> 
<br /> 
Building images<br /> 
docker build . -t joomla_img<br /> 
docker build . -t pmy_img<br /> 
<br /> 
Docker stack Deploy<br /> 
docker stack deploy joomlastack -c docker-compose.yml<br /> 
docker stack ps joomlastack<br /> 
docker stack ps joomlastack --no-trunc<br /> 
docker stack services joomlastack<br /> 
docker service logs <service name><br /> 
docker stack rm joomlastack<br /> 
<br /> 
Scaling up and down<br /> 
docker node update --availability Drain vm2<br /> 
docker node update --availability Active vm2<br /> 
docker service scale joomlastack_joomla=2 <br /> 











