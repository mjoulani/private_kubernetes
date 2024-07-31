create our oun cluster with our own Proxmox server.

create minimum two instances Ubuntu
create ssh key:
1. open in your host machine a terminal     in my case ubuntu
2. Run the command ssh keygen -b 4096 give name for key and the path in my case I stored in ~/.ssh/
3. after the pair key created public and private we need to copy the public key to our instances the master and the node1 ... the command to do this : ssh-copy-id <instances name>@<the ip for the instances> in my case mjoulani@192.168.68.124
4. make backup or snapshot before you start use the Proxmox console  

Let's start creating the cluster:
1. connect to the master node also the node1
2. Run the command disable swap for all instances : sudo swapoff -a 
3. edit the /etc/fstab to cancel the swap: sudo nano /etc/fstab 
commit this line : #/swap.img
4. install docker : sudo apt  install docker.io  -y
5. make docker to sudo group in my case : newgrp docker and sudo usermod -aG docker mjoulani and sudo chmod 666 /var/run/docker.sock




