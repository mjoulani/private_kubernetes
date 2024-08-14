# Create Our Own Cluster with Proxmox Server

## Create Minimum Two Instances of Ubuntu

### Create SSH Key:

1. Open a terminal on your host machine (Ubuntu in my case).
2. Run the command `<font color="blue"><code>ssh-keygen -b 4096 -f ~/.ssh/&lt;give-name-for-file&gt;</code></font>`. 

   **Example:**

   - **Linux Ubuntu:**
       <pre><font color="blue"><code>ssh-keygen -b 4096 -f ~/.ssh/&lt;give-name-for-file&gt;</code></font></pre>

   - **Windows:**
       - The key will be created in the default setting in the folder `C:\Users\&lt;your-username&gt;\.ssh`
       - Create in a specific name and path:
           <pre><font color="blue"><code>ssh-keygen -t ed25519 -f C:\Users\&lt;your-username&gt;\.ssh\&lt;give-name-for-file&gt;</code></font></pre>

**Note:** Make sure OpenSSH client is installed on both Linux and Windows:

<pre><font size="4" color="green"><code>sudo apt update
sudo apt install openssh-server
sudo apt install openssh-client

sudo systemctl start ssh
sudo systemctl enable ssh
sudo systemctl status ssh</code></font></pre>

### Copy the Public Key to Your Instances:

After creating the public and private key pair, copy the public key to your instances (master and node1) using the following command:

<pre><font color="blue"><code>ssh-copy-id &lt;instance-name&gt;@&lt;instance-ip&gt;</code></font></pre>

For example:

<pre><font color="blue"><code>ssh-copy-id mjoulani@192.168.68.124</code></font></pre>

### Backup or Snapshot:

Make a backup or snapshot before you start using the Proxmox console.

## Start Creating the Cluster:

1. Connect to the master node and node1.
2. Disable swap for all instances:
    <pre><font color="red"><code>sudo swapoff -a</code></font></pre>

3. Edit `/etc/fstab` to disable swap:
    <pre><font color="red"><code>sudo nano /etc/fstab</code></font></pre>
    Comment out the swap line by adding `#` at the beginning:
    <pre><font color="red"><code>#/swap.img</code></font></pre>

4. Install Docker:
    <pre><font color="red"><code>sudo apt install docker.io -y</code></font></pre>

5. Add your user to the Docker group:
    <pre><font color="red"><code>newgrp docker
sudo usermod -aG docker mjoulani
sudo chmod 666 /var/run/docker.sock</code></font></pre>

6. Install `curl` (if not already installed):
    <pre><font color="red"><code>curl --version</code></font></pre>

7. Merge Kubernetes Community (Master Controller and Node):
    Find the documentation here: <a href="https://kubernetes.io/blog/2023/08/15/pkgs-k8s-io-introduction/">Kubernetes Documentation</a>

    a. Add Kubernetes APT repository:
        <pre><font color="blue"><code>echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list</code></font></pre>

    b. Add Kubernetes GPG key:
        <pre><font color="blue"><code>curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg</code></font></pre>

    c. Update package list:
        <pre><font color="blue"><code>sudo apt update</code></font></pre>

8. Install Kubernetes components:
    <pre><font color="blue"><code>sudo apt install kubeadm kubelet kubectl kubernetes-cni -y</code></font></pre>

9. Create a cluster with `kubeadm`. For more information, visit: <a href="https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/">Kubeadm Documentation</a>

10. Setup the Master Controller Node:

    a. Show default route:
        <pre><font color="blue"><code>ip route show</code></font></pre>

    b. Initialize the Kubernetes master:
        <pre><font color="blue"><code>sudo kubeadm init</code></font></pre>

    c. Configure kubectl:
        <pre><font color="blue"><code>mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config</code></font></pre>

    Alternatively, as root:
        <pre><font color="blue"><code>export KUBECONFIG=/etc/kubernetes/admin.conf</code></font></pre>

    d. Apply Calico networking:
        <pre><font color="blue"><code>kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml</code></font></pre>

    e. Check the status:
        <pre><font color="blue"><code>kubectl get pods -n kube-system
kubectl get nodes</code></font></pre>
        Note: Wait for 5 minutes or more until the nodes are ready. Visit <a href="https://docs.tigera.io/">Tigera Documentation</a> and <a href="https://github.com/projectcalico/calico/blob/master/manifests/calico.yaml">Calico Manifest</a>.

11. Setup the Worker Node:

    Join the cluster using the command provided by `kubeadm init`:
    <pre><font color="blue"><code>sudo kubeadm join &lt;control-plane-host&gt;:&lt;control-plane-port&gt; --token &lt;token&gt; --discovery-token-ca-cert-hash sha256:&lt;hash&gt;</code></font></pre>

12. Manage Node Labels:

    a. Add a label to a node:
        <pre><font color="blue"><code>kubectl label nodes node1 role=worker-one</code></font></pre>

    b. List nodes with labels:
        <pre><font color="blue"><code>kubectl get nodes --show-labels</code></font></pre>

    Alternatively, you can:

    - **Add Labels:**
        <pre><font color="blue"><code>kubectl label nodes node1 node-role.kubernetes.io/&lt;name-of-label&gt;=</code></font></pre>

    - **Show Labels:**
        <pre><font color="blue"><code>kubectl get nodes --show-labels</code></font></pre>

    - **Remove Labels:**
        <pre><font color="blue"><code>kubectl label nodes node1 node-role.kubernetes.io/&lt;name-of-label&gt;-</code></font></pre>

**Notes:**
- Ensure to replace placeholders like `<control-plane-host>`, `<control-plane-port>`, `<token>`, and `<hash>` with your actual values.
