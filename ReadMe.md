Install k3s on oprangePi
---


1) install dietpi on 2 orangepi 16 gb RAM nodes using SD cards
2) Install no-ip DUC client on master node. 
https://www.noip.com/support/knowledgebase/installing-the-linux-dynamic-update-client-on-ubuntu/
3) use
https://gist.github.com/NathanGiesbrecht/da6560f21e55178bcea7fdd9ca2e39b5/raw/b5594a39e908548f4319294553497d2db3053e0a/noip2.service
4) make a service 
https://www.noip.com/support/knowledgebase/installing-the-linux-dynamic-update-client-on-ubuntu/
5) Install k3s on master node with appropriate --tls-san values that correspond to domain purchased on no-ip.com dynDNS and the home router IP. Node IP being host IP of orangePI master

```
 curl -sfL https://get.k3s.io | INSTALL_K3S_MIRROR=cn INSTALL_K3S_EXEC="--node-ip=192.168.0.46  --tls-san="192.168.0.46" --tls-san="*.systemtracker.no-ip.org" --tls-san="84.214.176.152" --disable servicelb,traefik --flannel-backend=none --disable-network-policy --cluster-cidr=10.16.0.0/16  --service-cidr=10.128.0.0/16" K3S_NODE_NAME="$(hostname)" sh -s - server --cluster-init
```
6) cat the node token out 
```
cat /var/lib/rancher/k3s/server/node-token
```
7) use the token to install the other k8s nodes. For eg.
```
curl -sfL https://get.k3s.io | K3S_URL=https://192.168.0.46:6443 K3S_TOKEN=K10debc9eca32634213415ad9e66c0eda94ecf7383aa997c6079f772efc63e84828::server:b00401254d9c4817f4a9e584c19be5cf sh -
```
8) install calico on the master node
```
 kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.1/manifests/calico.yaml
```
9) get the kubeconfig and change the hostname to the domain you chose to have for your cluster e.g. systemtracker.no-ip.org
```
cat /etc/rancher/k3s/k3s.yaml
```
use it in the lens
```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tL...=
    server: https://systemtracker.no-ip.org:6443
  name: default
contexts:
- context:
    cluster: default
    user: default
  name: default
current-context: default
kind: Config
preferences: {}
users:
- name: default
  user:
    client-certificate-data: LS0tLS..g==
    client-key-data: LS0t..LQo=

```