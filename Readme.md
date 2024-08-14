# Create Our Own Cluster with Proxmox Server

## Create Minimum Two Instances of Ubuntu

### Create SSH Key:

1. Open a terminal on your host machine (Ubuntu in my case).
2. Run the command `ssh-keygen -b 4096` and provide a name for the key and the path. In my case, I stored it in `~/.ssh/`.

**Example:**

- **Linux Ubuntu:**
    <pre><code>ssh-keygen -b 4096 -f ~/.ssh/&lt;give-name-for-file&gt;</code></pre>
	<style>
	.custom-style {
		font-size: 18px;
		color: red;
	}
    </style>


- **Windows:**
-   <pre>The key will be created in the default setting in the folder C:\Users\&lt;your-username&gt;\.ssh</pre>
-   <pre class="custom-style"><code>ssh-keygen -t  ed25519 </code></pre>
-   
-   <pre style="font-size: 16px; color: blue;">Create in specific name and path</pre>  
    <pre><code>ssh-keygen -t  ed25519 -f C:\Users\&lt;your-username&gt;\.ssh\&lt;give-name-for-file&gt;</code></pre>
    <pre><code>ssh-keygen -t  ed25519 -f C:\Users\&lt;your-username&gt;\.ssh\&lt;give-name-for-file&gt;</code></pre>
    <pre><code>ssh-keygen -t  ed25519 -f C:\Users\&lt;your-username&gt;\.ssh\&lt;give-name-for-file&gt;</code></pre>

**Note:** Make sure OpenSSH client is installed on both Linux and Windows:

<pre><code>sudo apt update
sudo apt install openssh-server
sudo apt install openssh-client

sudo systemctl start ssh
sudo systemctl enable ssh
sudo systemctl status ssh</code></pre>

### Copy the Public Key to Your Instances:

After creating the public and private key pair, copy the public key to your instances (master and node1) using the following command:

<pre><code>ssh-copy-id &lt;instance-name&gt;@&lt;instance-ip&gt;</code></pre>

For example:

<pre><code>ssh-copy-id mjoulani@192.168.68.124</code></pre>

### Backup or Snapshot:

Make a backup or snapshot before you start using the Proxmox console.

## Start Creating the Cluster:

1. Connect to the master node and node1.
2. Disable swap for all instances:
    <pre><code>sudo swapoff -a</code></pre>

3. Edit `/etc/fstab` to disable swap:
    <pre><code>sudo nano /etc/fstab</code></pre>
    Comment out the swap line by adding `#` at the beginning:
    <pre><code>#/swap.img</code></pre>

4. Install Docker:
    <pre><code>sudo apt install docker.io -y</code></pre>

5. Add your user to the Docker group:
    <pre><code>newgrp docker
sudo usermod -aG docker mjoulani
sudo chmod 666 /var/run/docker.sock</code></pre>

6. Install `curl` (if not already installed):
    <pre><code>curl --version</code></pre>

7. Merge Kubernetes Community (Master Controller and Node):
    Find the documentation here: <a href="https://kubernetes.io/blog/2023/08/15/pkgs-k8s-io-introduction/">Kubernetes Documentation</a>

    a. Add Kubernetes APT repository:
        <pre><code>echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list</code></pre>

    b. Add Kubernetes GPG key:
        <pre><code>curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg</code></pre>

    c. Update package list:
        <pre><code>sudo apt update</code></pre>

8. Install Kubernetes components:
    <pre><code>sudo apt install kubeadm kubelet kubectl kubernetes-cni -y</code></pre>

9. Create a cluster with `kubeadm`. For more information, visit: <a href="https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/">Kubeadm Documentation</a>

10. Setup the Master Controller Node:

    a. Show default route:
        <pre><code>ip route show</code></pre>

    b. Initialize the Kubernetes master:
        <pre><code>sudo kubeadm init</code></pre>

    c. Configure kubectl:
        <pre><code>mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config</code></pre>

    Alternatively, as root:
        <pre><code>export KUBECONFIG=/etc/kubernetes/admin.conf</code></pre>

    d. Apply Calico networking:
        <pre><code>kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml</code></pre>

    e. Check the status:
        <pre><code>kubectl get pods -n kube-system
kubectl get nodes</code></pre>
        Note: Wait for 5 minutes or more until the nodes are ready. Visit <a href="https://docs.tigera.io/">Tigera Documentation</a> and <a href="https://github.com/projectcalico/calico/blob/master/manifests/calico.yaml">Calico Manifest</a>.

11. Setup the Worker Node:

    Join the cluster using the command provided by `kubeadm init`:
    <pre><code>sudo kubeadm join &lt;control-plane-host&gt;:&lt;control-plane-port&gt; --token &lt;token&gt; --discovery-token-ca-cert-hash sha256:&lt;hash&gt;</code></pre>

12. Manage Node Labels:

    a. Add a label to a node:
        <pre><code>kubectl label nodes node1 role=worker-one</code></pre>

    b. List nodes with labels:
        <pre><code>kubectl get nodes --show-labels</code></pre>

    Alternatively, you can:

    - **Add Labels:**
        <pre><code>kubectl label nodes node1 node-role.kubernetes.io/&lt;name-of-label&gt;=</code></pre>

    - **Show Labels:**
        <pre><code>kubectl get nodes --show-labels</code></pre>

    - **Remove Labels:**
        <pre><code>kubectl label nodes node1 node-role.kubernetes.io/&lt;name-of-label&gt;-</code></pre>

**Notes:**
- Ensure to replace placeholders like `<control-plane-host>`, `<control-plane-port>`, `<token>`, and `<hash>` with your actual values.




