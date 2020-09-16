# VirtualizationCloudComputingProject

Objective: 
</br>
To build a distributed architecture consisting of 3 Virtual Machines (Ubuntu Server 16.04).  The three machines will be placed in a Docker Swarm and will share a file system thanks to GlusterFS technology. </br>
The Swarm will house the following containers:</br>
 træfik</br> Joomla</br> Prometheus</br> MariaDB</br> Grafana</br>
</br>
Setup commands
</br>
sudo apt install curl </br>
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - </br>
sudo apt-get install software-properties-common</br>
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"</br>
sudo apt-get update</br></br>
sudo apt-get install docker-ce docker-ce-cli containerd.io</br>
sudo usermod -aG docker [user]</br>
sudo systemctl restart docker</br>
Docker Compose</br>
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose</br>
sudo chmod +x /usr/local/bin/docker-compose</br>
<br /> 
Clone -> VM2 and VM3 from VM1
<br /> 
![Image](https://github.com/sahityacodes/VirtualizationCloudComputingProject/blob/master/Network.PNG?raw=true)
<br /> 
sudo nano /etc/network/interfaces</br>
auto ens38</br>
iface ens38 inet static</br>
address 10.0.0.11</br>
netmask 255.255.255.0</br>
network 10.0.0.0</br>
gateway 10.0.0.2 </br>
<br /> 
sudo nano /etc/hosts</br>
VMware GlusterFS</br>
10.0.0.11 vm1g</br>
10.0.0.12 vm2g</br>
10.0.0.13 vm3g</br>

sudo reboot</br>
ping vm2g</br>

sudo apt update; </br>
sudo apt -y install glusterfs-server glusterfs-client</br>

Start the service:</br>
sudo service glusterfs-server start</br>

Peer the service</br>
sudo gluster peer probe vm2g</br>
sudo gluster peer probe vm3g</br>
</br>
Check the status</br>
sudo gluster peer status</br>
</br>
Bricks folder</br>
sudo mkdir -p /gluster/bricks/1 (on vm1)</br>
sudo mkdir -p /gluster/bricks/2 (on vm2)</br>
sudo mkdir -p /gluster/bricks/3 (on vm3)</br>

Add the line to the / etc / fstab file on the VMs  for mount the bricks</br>
echo '/dev/sdb /gluster/bricks/1 ext4 defaults 0 0' >> /etc/fstab (on vm1)</br>
echo '/dev/sdb /gluster/bricks/2 ext4 defaults 0 0' >> /etc/fstab (on vm2)</br>
echo '/dev/sdb /gluster/bricks/3 ext4 defaults 0 0' >> /etc/fstab (on vm3)</br>

Mount filesystem</br>
sudo mount -a (on vm1, vm2, vm3)</br>

Create gfs volume</br>
sudo gluster volume create gfs replica 3 \</br>
vm1g:/gluster/bricks/1/brick \</br>
vm2g:/gluster/bricks/2/brick \</br>
vm3g:/gluster/bricks/3/ brick </br>
</br>
Start  volume</br>
sudo gluster volume start gfs</br>
</br>
Check the status</br>
sudo gluster volume info gfs</br>
sudo gluster volume status gfs</br>
</br>
Three VM to connect to gfs on each node</br>
sudo gluster volume set gfs auth.allow 10.0.0.11, 10.0.0.12,10.0.0.13</br>
sudo gluster volume set gfs nfs.disable Off</br>
</br>
Mount the volume </br>
sudo mkdir /gfs</br>
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
 </br>
Bonus:
</br>
Split-Brain condition</br>
gluster volume heal <VOLNAME> info split-brain












