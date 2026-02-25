# 1. About Architecture used in this project ! (A Basic Industry setup example)
This project utilizes a modular Ansible Role-based architecture. By separating system preparation, control plane initialization, and worker node integration, the setup remains reduces efforts and it can be run multiple times without causing system errors.

```
├── ansible.cfg          # Global Ansible settings
├── inventory.ini        # Host groups and variables, ie., ip, hostnames etc
├── setup_k8s.yml            # Main playbook orchestrator
└── roles/
    ├── common/          # System prep (Common setup for master and workers)
    ├── master/          # Kubeadm init and CNI setup, (Setup for master)
    └── worker/          # Kubeadm join logic, (Setup for workers)
```
if you want to know more about architecture, read from 3rd point.

# 2. How to Apply This Setup (The Workflow)
To run this on your nodes, follow these steps in order to ensure a clean deployment:

## A. Virtual machine setup
Provision your virtual machines (one Master, and at least one Worker). Update the inventory.ini file with the specific IP addresses and usernames for your environment, ie., VM's.

### Inventory edit exmaple
```
eg. 
[masters]   # master conf
k8s-master ansible_host=192.168.134.133 ansible_user=master

[workers]   # worker conf
k8s-worker-1 ansible_host=192.168.134.135 ansible_user=worker-1

``` 
## B. Create SSH key on conneecting to virtal machine in local machine and tranfer it to virtual machines
### Create ssh key
```
- ssh-keygen -t ed25519           - best option
- ssh-keygen -t rsa -b 4096       - if ed25519 doesn't works
```

## C. Initial Node Access via ssh-copy-id !
Ansible needs a way in. Ensure your users are configured on all nodes and has SSH keys.

Manual Step: Copy your SSH key to the master and worker,
Transfer ssh key to virtual machines.

```
ssh-copy-id user@hostname_or_ip           - eg. master@192.168.x.x
```

**Test Connection: Run ```ansible all -m ping -i inventory.ini```. If you see "pong," your devices are reachable.**

## D. Executing the Playbook
Run the main orchestrator file, ```setup_k8s.yml```.
```
ansible-playbook -i inventory.ini -b -K setup_k8s.yml           - after enter pass
```


How to verify verify: ```sudo kubectl cluster-info```

# 3. Kubernetes Cluster Architecture

## The Control Plane (Master)
The "Brain" of the operations. Our automation initializes the following core components:

- API Server: The central hub that all components communicate with.
- Etcd: The high-availability key-value store for all cluster data.
- Controller Manager & Scheduler: Logic that maintains cluster state and places Pods on nodes.

## The Worker Nodes
The "Muscles" of the cluster where your applications (Pods) live:

- Kubelet: The primary "node agent" that ensures containers are running correctly.
- Kube-Proxy: Maintains network rules on the host to allow communication to Pods.
- Container Runtime (Containerd): The engine that pulls and runs container images.

## The Networking Layer (Calico)
Implementing Calico as the Container Network Interface (CNI). It provides a highly scalable and secure network fabric that allows Pods across different physical/virtual nodes to communicate seamlessly.

```A basic kubernetes prod example.```