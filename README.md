# install-kubernet-opensuse
## install docker
```bash
sudo zypper update
sudo zypper refresh

#ensure the swap is disable
sudo swapon --show 


sudo zypper install docker

#enable the docker service started on boot
sudo systemctl enable docker
sudo systemctl start docker

sudo docker ps
```

## install kubernet 1.27
```bash
sudo zypper install kubernetes1.27-kubelet
sudo zypper install kubernetes1.27-kubeadm
sudo zypper install kubernetes1.27-client

#enable the kubelet service started on boot
sudo systemctl enable kubelet


```

## setup the firewall rules in both virtual machines
```bash
sudo firewall-cmd --query-port=6443/tcp

#add port in the firewall rules
sudo firewall-cmd --permanent --zone=public --add-port=6443/tcp

sudo systemctl reload firewalld

```

## init master cluster
```bash
sudo kubeadm init --kubernetes-version=1.27.4 \
    --apiserver-advertise-address=10.10.1.213 \
    --image-repository registry.aliyuncs.com/google_containers \
    --service-cidr=172.17.0.0/16 \
    --pod-network-cidr=172.18.0.0/16
```

## make kubectl work for the non-root user

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## install a Pod network add-on
```bash
sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

```
if https://raw.githubusercontent.com is blocked, just get the yaml file and copy/pates into vi
```bash
sudo vi kube-flannel.yml
sudo kubectl apply -f  kube-flannel.yml
```

## change the IP range of service-cidr
```bash
net-conf.json: |
{
    "Network": "172.17.0.0/16",
    "Backend": {
        "Type": "vxlan"
    }
}
```
## check the status, all pods should be in running status

```bash
sudo kubectl get pods --all-namespaces
```

## do in worker node, add the worker node into the cluster with kubeadm

specify the container runtime in --cri-socket, Kubernetes uses the Container Runtime Interface (CRI) to interface with your chosen container runtime, if it's using Docker Engine, then set --cri-socket=/var/run/dockershim.sock


```bash
sudo kubeadm join 10.211.55.7:6443 --cri-socket=/var/run/dockershim.sock \
    --token <your token> \
    --discovery-token-ca-cert-hash sha256:<your hash>
```

## check status in master node

```bash
sudo kubectl get nodes

```