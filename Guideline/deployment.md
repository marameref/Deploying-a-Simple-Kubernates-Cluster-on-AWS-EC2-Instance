
# Deploying a Simple Kubernetes Cluster on AWS EC2

This guide will walk you through setting up a simple Kubernetes cluster on AWS EC2 instances using **kubeadm**. This setup is suitable for learning and experimentation purposes. We'll use the free tier eligible EC2 instances to avoid costs.

## Prerequisites

1. **AWS Account**: Ensure you have an AWS account.
2. **AWS CLI**: Install and configure the AWS CLI.
3. **kubectl**: Install `kubectl` on your local machine.

## Step 1: Launch EC2 Instances

1. **Login to AWS Management Console**
2. **Launch EC2 Instances**
    - **Instance Type**: Select `t2.micro` (free tier eligible).
    - **Operating System**: Choose Ubuntu 20.04 LTS (free tier eligible).
    - **Number of Instances**: Launch at least 2 instances (1 Master, 1 Worker).
    - **Security Group**: Create a security group with the following rules:
      - Allow SSH (port 22) from your IP.
      - Allow all traffic (port 0-65535) between the instances.
    - **Key Pair**: Create or select an existing key pair to access the instances.

## Step 2: Connect to Your Instances

Use your SSH key to connect to the instances.

```sh
ssh -i /path/to/your-key.pem ubuntu@<EC2-Instance-Public-IP>
```

## Step 3: Install Docker

Install Docker on both Master and Worker nodes.

```sh
sudo apt-get update
sudo apt-get install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
```

## Step 4: Install kubeadm, kubelet, and kubectl

Install the necessary Kubernetes components on both Master and Worker nodes.

```sh
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

## Step 5: Initialize the Kubernetes Master Node

Run the following commands on the Master node to initialize the cluster:

```sh
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

Set up the local kubeconfig:

```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Step 6: Deploy a Pod Network

Install a Pod network add-on so that your pods can communicate with each other. We'll use Flannel for this tutorial.

```sh
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

## Step 7: Join the Worker Node to the Cluster

On the Master node, obtain the join command to add the Worker node:

```sh
kubeadm token create --print-join-command
```

On the Worker node, run the join command obtained from the Master node.

```sh
sudo kubeadm join <Master-Node-IP>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

## Step 8: Verify the Cluster

On the Master node, check the status of the nodes:

```sh
kubectl get nodes
```

You should see both the Master and Worker nodes listed.

## Step 9: Deploy a Simple Application

Create a deployment:

```sh
kubectl create deployment nginx --image=nginx
```

Expose the deployment:

```sh
kubectl expose deployment nginx --port=80 --type=NodePort
```

Get the NodePort assigned to the service:

```sh
kubectl get svc
```

Access the application using the NodePort and the EC2 instance's public IP.

```sh
http://<EC2-Instance-Public-IP>:<NodePort>
```

## Conclusion

Congratulations! You have successfully deployed a simple Kubernetes cluster on AWS EC2 instances using `kubeadm`. This setup provides a basic environment for you to learn and experiment with Kubernetes.

---

Feel free to copy and paste this Markdown into your GitHub repository. Happy learning!
