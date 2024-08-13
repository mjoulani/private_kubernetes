create our oun cluster with our own Proxmox server.

create minimum two instances Ubuntu
create ssh key:
1. open in your host machine a terminal     in my case ubuntu
2. Run the command ssh keygen -b 4096 give name for key and the path in my case I stored in ~/.ssh/

Example:

   linux ubuntu : ssh-keygen -b 4096 -f ~/.ssh/<give name for file>
   windows: ssh-keygen -b 4096 -f C:\Users<your user name>\.ssh\<give name for file>
   note: in linux or windows must install 
   openssh-client:

   *.sudo apt update
   *.sudo apt install openssh-server
   *.sudo apt install openssh-client

   *.sudo systemctl start ssh
   *.sudo systemctl enable ssh
   *.sudo systemctl status ssh



3. after the pair key created public and private we need to copy the public key to our instances the master and the node1 ... the command to do this : ssh-copy-id <<instances name>></instances>>@<the ip for the instances> in my case mjoulani@192.168.68.124
4. make backup or snapshot before you start use the Proxmox console  

Let's start creating the cluster:
1. connect to the master node also the node1
2. Run the command disable swap for all instances : sudo swapoff -a 
3. edit the /etc/fstab to cancel the swap: sudo nano /etc/fstab 
commit this line : #/swap.img
4. install docker : sudo apt  install docker.io  -y
5. make docker to sudo group in my case : newgrp docker and sudo usermod -aG docker mjoulani and sudo chmod 666 /var/run/docker.sock
6. install curl: in my case it is install check by running : curl --version
7. Merging  Kubernetes community(master controller and node):
   you find all the Documentation by visiting https://kubernetes.io/blog/2023/08/15/pkgs-k8s-io-introduction/

   a. echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

   b. curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
   c. sudo apt update 

8. installing kubernetes component:
   sudo apt install kubeadm kubelet kubectl kubernetes-cni -y
9. Creating a cluster with kubeadm:
10. visit the docuamention wedsit for more information --- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
11. Setup the master controller node:
    a. ip route show # Look for a line starting with "default via"

    b. sudo kubeadm init 
    
    c. mkdir -p $HOME/.kube

    d. sudo cp -i /etc/kubernetes/admin.   conf $HOME/.kube/config
    e. sudo chown $(id -u):$(id -g) $HOME/.kube/config

    Alternatively, if you are the root user, you can run:

    export KUBECONFIG=/etc/kubernetes/admin.conf
    f. kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
    kubectl get pods -n kube-system
    kubectl get nodes
    note: wait for 5 min less or more until the nodes be ready
    visit https://docs.tigera.io/

    https://github.com/projectcalico/calico/blob/master/manifests/calico.yaml

12. Setup the working node:
    Let's join the cluster:
    sudo  kubeadm join <control-plane-host>:<control-plane-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash> 
    how i cna get this info ?
     when you run the init command you will get the join command with the token.
13. Managed node labels:
14. 
    a.  kubectl label nodes node1 role=worker-one
    
    b. kubectl get nodes --show-labels

    alternative:
    add labels:
    kubectl label nodes node1 node-role.kubernetes.io/<name of the labels>=

    show labels:
    kubectl get nodes --show-labels

    remove labels:
    kubectl label nodes node1 node-role.kubernetes.io/<name of the labels>-



