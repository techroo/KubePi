# KubePi
Kubernetes on Raspberry PI

For a kubernetes test setup i have been looking at cheap ways to create nodes.
The new raspberry Pi 5 seemed like an ideal candidate.
.
.
.
# Hardware
We will use the following hardware:
- Raspberry Pi 5 8Gb.
- Pimoroni NVME base for Pi 5.
- 1TB M.2 SSD.
- Generic RTL8153 USB3.0 to Gbit Ethernet converter.
- 32GB Class10 uSD card.




# Results
We will try to reach the following results:
- A High Availibility (HA) cluster.
- Booting host OS from uSD card in a lasting way.
- A distributed filesystem based on LongHorn on our M.2 SSD.
- Rancher control website installed on our HA cluster.
- Loadbalancer running on our cluster, mitigating the need for an external one. 

We will be running two networks to each node:
- internal eth0 - _Used for the container and k3s communication_
- usb converter eth1 - _Used for longhorn distributed storage sync_




# Prepare uSD
First we download the raspberry pi imager. We then install raspberry pi5 > other os > 64bit lite.  
Make sure to set custom OS settings.  
- Hostname
- SSH login with password.
- password and user, we will use pi for both.

After writing boot the raspberry pi 






# Node1 - Install K3S
We will use curl to download and install K3S.  
**INSTALL_K3S_VERSION="v1.26.10+k3s1"** _K3S version to install_  
**K3S_TOKEN="myLittleK3Cluster"**       _The cluster join token_  
**--cluster-init**                      _Indicates we will run HA and want to use EtcD_  
**--disable traefik**                   _Disables the internal ingress controller?_  
**--disable servicelb**                 _Disables the internal loadbalancer_  

```
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION="v1.26.10+k3s1" K3S_KUBECONFIG_MODE="644" K3S_TOKEN="myLittleK3Cluster" sh -s - server --cluster-init --disable traefik --disable servicelb
```

We can check the state of kubernetes using the kubectl command.  
To CHeck if this node installed correctly run:
```
kubectl get nodes
```
This should display our one node.






# Node2,3,x - Install K3S
For the other nodes we run the identical command, only without the --cluster-init and with a pointer to our Node1
**--server https://10.128.10.21:6443** _Pointer to Node1-Eth0_
```
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION="v1.26.10+k3s1" K3S_KUBECONFIG_MODE="644" K3S_TOKEN="myLittleK3Cluster" sh -s - server --server https://10.128.10.21:6443 --disable traefik --disable servicelb
```
We can check the state of our kubernetes nodes using the kubectl command again.  
```
kubectl get nodes
```
This should display our two nodes.






# Node1 - Install Helm
We will use helm to install things later on. We will install it already on node 1

```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```






# Node1 - Install metalb load balancer
Since we disabled the serviceLb during instqllqtion, we need to install our own full load balancer  
```
helm repo add metallb https://metallb.github.io/metallb
helm repo update
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
helm install metallb metallb/metallb --namespace kube-system
```

To set the config we do:
```
nano resources.yaml
```
then paste
```
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  creationTimestamp: null
  name: default
  namespace: kube-system
spec:
  addresses:
  - 10.128.10.30-10.128.10.38
status: {}
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  creationTimestamp: null
  name: l2advertisement1
  namespace: kube-system
spec:
  ipAddressPools:
  - default
status: {}
---
```
> [!TIP]
> To save and exit nano, press CTRL+X, Y, ENTER.
> 
To Install we run:
```
kubectl create -f resources.yaml
```

To verify check:   
```
kubectl get customresourcedefinitions.apiextensions.k8s.io ipaddresspools.metallb.io
```






# Node1 - Install nginx ingress controller
The ingress controller handles incoming connection and redirects them to the correct services.   
Since we disables the default treafik, we need to install nginx:   
```
 kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.5.1/deploy/static/provider/cloud/deploy.yaml
```

Our ingress controller should receive an LAN ip from the range we gave to the load balancer. We can check this with:
```
kubectl get svc -n ingress-nginx
```
An external-IP should be specified for the loadbalancer.   
Check if the address is reachable from the LAN:   
- http://10.128.10.30/   
- https://10.128.10.30/   
Ignore the 400 error, this just means we are not forwarding port 80.




# Node1 - Install cert mqnqger
To do https we need certificates. To handle these we must install cert manager
```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.3/cert-manager.yaml
```
check if the pods qre running
```
kubectl get pods --namespace cert-manager
```


# Node1 - Install longhorn

$ helm repo add longhorn https://charts.longhorn.io
$ helm repo update
$ helm install longhorn longhorn/longhorn --namespace longhorn-system --set defaultSettings.defaultDataPath="/var/lib/longhorn" --create-namespace



# Node1 - Install rancher











> [!NOTE]
> Useful information that users should know, even when skimming content.

> [!TIP]
> Helpful advice for doing things better or more easily.

> [!IMPORTANT]
> Key information users need to know to achieve their goal.

> [!WARNING]
> Urgent info that needs immediate user attention to avoid problems.

> [!CAUTION]
> Advises about risks or negative outcomes of certain actions.
