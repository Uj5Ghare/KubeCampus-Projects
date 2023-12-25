# Kubernetes Installation and Configuration Guide With Kubeadm

This guide provides step-by-step instructions for installing and configuring Kubernetes.

## Installing Kubernetes (Both Master & Slave)

1. Download the Google Cloud public signing key:
    ```bash
    sudo curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg add -
    ```

2. Add the Kubernetes apt repository:
    ```bash
    sudo echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
    ```

3. Update apt package index with the new repository and install the utilities:
    ```bash
    sudo apt update
    sudo apt install -y kubelet=1.20.0-00 kubeadm=1.20.0-00 kubectl=1.20.0-00
    sudo apt-mark hold kubelet kubeadm kubectl
    ```

4. Install Docker:
    ```bash
    sudo export VERSION=19.03 && curl -sSL get.docker.com | sh
    ```

5. Setup the Kubernetes Control Plane:
    ```bash
    sudo mkdir /etc/docker
    cat <<EOF | sudo tee /etc/docker/daemon.json
    {
      "exec-opts": ["native.cgroupdriver=systemd"],
      "log-driver": "json-file",
      "log-opts": {
        "max-size": "100m"
      },
      "storage-driver": "overlay2"
    }
    EOF

    sudo systemctl enable docker
    sudo systemctl daemon-reload
    sudo systemctl restart docker
    ```

## Master Node/Control Plane Configuration (Master Node)

On the Control Plane node only, run this command to initialize the Kubernetes control plane:
```bash
sudo kubeadm init --ignore-preflight-errors=all
sudo mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
sudo kubeadm token create --print-join-command
```
Copy the output and run it on all worker nodes

## Installing the Weave Net Add-On

Note:- Open these ports TCP 6783 and UDP 6783/6784.
```bash 
sudo kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s-1.11.yaml
sudo watch kubectl get pods --all-namespaces
```

## Demo Application with Microservices
```bash
sudo git clone https://github.com/microservices-demo/microservices-demo.git
cd microservices-demo/deploy/kubernetes
sudo kubectl apply -f complete-demo.yaml
sudo watch kubectl get pods --namespace sock-shop
```

