Kubernetes Cluster Setup using Ansible on AWS EC2
📖 Overview

This project automates the setup of a Kubernetes (v1.29) cluster on AWS EC2 instances using Ansible.

The cluster consists of:

1 Master Node

1 Worker Node

The setup includes containerd runtime, kubeadm initialization, worker node join, and Calico CNI installation.

🏗 Architecture
                         +----------------------+
                         |  Ansible Control Node |
                         |  (Local Machine)      |
                         +----------+------------+
                                    |
                                    | SSH (Port 22)
                                    v
        -----------------------------------------------------
        |                     AWS VPC                      |
        |                                                   |
        |   +-------------------+      +-------------------+ |
        |   |   Master Node     |      |   Worker Node     | |
        |   |   EC2 Instance    |      |   EC2 Instance    | |
        |   |-------------------|      |-------------------| |
        |   | kube-apiserver    |      | kubelet           | |
        |   | controller-mgr    |      | kube-proxy        | |
        |   | scheduler         |      | containerd        | |
        |   | etcd              |      |                   | |
        |   | containerd        |      |                   | |
        |   +-------------------+      +-------------------+ |
        |                                                   |
        -----------------------------------------------------
AWS EC2 (Ubuntu)

Separate Security Groups for Master and Worker

Kubernetes v1.34

Containerd (configured with systemd cgroup driver)

Calico CNI

Ansible for automation

Master Node:

kube-apiserver

kube-controller-manager

kube-scheduler

etcd

Worker Node:

kubelet

kube-proxy

containerd

🔐 Security Group Configuration

Separate security groups were created for Master and Worker nodes.

Master Node Inbound Rules:

22 (SSH)

6443 (Kubernetes API Server)

10250 (kubelet)

Worker Node Inbound Rules:

22 (SSH)

10250 (kubelet)

30000-32767 (NodePort services)

Internal communication allowed between security groups.

⚙️ Key Configuration Fix

During setup, the Kubernetes API server failed to start due to container runtime misconfiguration.

🔴 Issue:
Kubelet failed because containerd was using the default cgroup driver.

✅ Fix:
Updated containerd configuration:

File:

/etc/containerd/config.toml.. please find this Option  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options] inside this file and 
SystemdCgroup = true

Then restarted:
sudo systemctl restart containerd
sudo systemctl restart kubelet

🚀 Setup Steps

Launch EC2 instances (Ubuntu)

Configure Security Groups

Update Ansible inventory

Run Playbook and Store Execution logs in a file:
ansible-playbook -i inventory site.yml | tee execution.log

CNI Installation
Calico was installed using:
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/calico.yaml

TroubleShooting Steps:
🔹 Step A — Completely Reset First
On master:
sudo kubeadm reset -f
sudo rm -rf /etc/kubernetes
sudo rm -rf /var/lib/etcd
sudo rm -rf /var/lib/kubelet
sudo systemctl restart containerd

Step B — Fix containerd Properly:
As mentioned above as:- sudo nano /etc/containerd/config.toml

🔹 Step C — Restart Runtime
sudo systemctl restart containerd
sudo systemctl restart kubelet
Check:
sudo systemctl status containerd
sudo systemctl status kubelet
Both must be:
active (running)

🔹 Step D — Verify containerd is Using systemd
containerd config dump | grep SystemdCgroup
It must show:
SystemdCgroup = true

🔹 Step E — Run kubeadm init Manually First (Test)
Before using Ansible again, test manually:
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
If this succeeds → API server will start.

sudo crictl ps
You should now see:--- all running...
kube-apiserver
kube-controller-manager
kube-scheduler
etcd

🧠 What I Learned

Kubernetes control plane internals
Container runtime & cgroup drivers
Troubleshooting kubelet issues
Security group design
Infrastructure automation with Ansible

What I actually did in this project:-
Provisioned EC2 instances
Configured separate security groups
Fixed containerd cgroup driver mismatch
Deployed Kubernetes 1.29
Successfully joined worker to master
Installed Calico
Automated everything using Ansible

Author!
Vanshika Jaiswal



