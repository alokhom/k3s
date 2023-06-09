Install k3s on orangePi
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
6) install calico on the master node
```
 kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.1/manifests/calico.yaml
```
7) remove all the node taints and tolerations inside the deployment of the metrics-server and keep port 4443 and readiness probe at 4443
```
kubectl taint nodes orangepi5 node.kubernetes.io/not-ready:NoExecute-
kubectl taint nodes orangepi5 node.kubernetes.io/not-ready:NoSchedule-
kubectl edit deployment/metrics-server -n kube-system
```
8) cat the node token out 
```
cat /var/lib/rancher/k3s/server/node-token
```
9) use the token to install the other k8s nodes. For eg.
```
curl -sfL https://get.k3s.io | K3S_URL=https://192.168.0.46:6443 K3S_TOKEN=K10debc9eca32634213415ad9e66c0eda94ecf7383aa997c6079f772efc63e84828::server:b00401254d9c4817f4a9e584c19be5cf sh -
```

10) get the kubeconfig and change the hostname to the domain you chose to have for your cluster e.g. systemtracker.no-ip.org
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
11) install flux. Ensure the exports are in place or in bash_profile GITHUB_USER=alok.hom@gmail.com GITHUB_TOKEN=xxxxx ( check with repo or password manager)
```
flux bootstrap github --repository=flux --branch=main --path=clusters/dev --owner=alokhom --components-extra=image-reflector-controller,image-automation-controller
```
