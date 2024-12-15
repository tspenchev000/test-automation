# Kubespray information
Kubespray is collection of Ansible script for deployment of production ready Kubernetes cluster.
Repo is cloned in subfolder of this repo to avoid future changes and misconfiguration
Officail repo can be found on [link](https://github.com/kubernetes-sigs/kubespray)
git clone https://github.com/kubernetes-sigs/kubespray.git


## Downloading entire kubernetes-iac repo (kubespray is already included in it)

```
sudo git clone https://tspenchev2023@bitbucket.org/tm-prod/kubernetes-iac.git
```
```
Cloning into 'kubernetes-iac'...
Password for 'https://tspenchev2023@bitbucket.org':
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
Receiving objects: 100% (3/3), done.
```

If you need to change permissions to current user (ansible)
bash```
sudo chown -R ansible: kubernetes-iac
cd kubernetes-iac
```


## Create new LD5 config from sample
```
cp -rfp inventory/sample inventory/ld5
(update)inventory/ld5/hosts.yml
(update)group_vars/k8s_cluster/k8s-cluster.yml by adding ld5/cluster-config.yaml
	local_release_dir: "/opt/releases"
	tls_min_version: "VersionTLS12"
	kube_service_addresses: 172.31.0.0/16
	kube_pods_subnet: 172.29.0.0/16

source /opt/k8s/kubespray-venv/bin/activate
cd /opt/kubernetes-iac/
```

## Install dependencies from requirements.txt
```
cd kubespray
pip install -r requirements.txt

VENVDIR=kubespray-venv
KUBESPRAYDIR=kubespray
python3.12 -m venv $VENVDIR --system-site-packages
source $VENVDIR/bin/activate
cd $KUBESPRAYDIR
pip install -U -r requirements.txt  
ansible-galaxy collection install ansible.utils --force
pip install ruamel.yaml
```


Update kubespray/inventory/ld5/hosts.yaml with nesessary host information
You can change values in kubespray/inventory/ld5/cluster-config.yaml but on your own risk. Do not touch any other file from kybespray folder.

Create new cluster using information from kubespray/inventory/ld5/hosts.yaml and kubespray/inventory/ld5/cluster-config.yaml
```
cd /opt/kubernetes-iac/kubespray
ansible-playbook cluster.yml -i inventory/ld5/hosts.yaml -e @inventory/ld5/cluster-config.yaml --user=ansible --become --become-user=root --flush-cache
```
When initial setup of kubernetes cluster finish we need to install additional modules: MetalLB, ArgoCD, Nginx Ingress Controler and Cert Manager.
```
ansible-playbook playbooks/cluster-additionalsteps.yml -i inventory/ld5/hosts.yaml -e @inventory/ld5/cluster-config.yaml --user=ansible --become --become-user=root --flush-cache
```
To reset/delete kubernetes cluster use 
```
ansible-playbook -i inventory/ld5/hosts.yaml reset.yml --become --become-user=root
```

For Celium CNI ref: [celium](https://docs.cilium.io/en/latest/installation/k8s-install-kubespray/)

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
## Random info

```
SELINUX !!!!! - just install it on target machines pip install selinux for root user  --> python3 -m pip install selinux
# python3.11 -m venv /tmp/3.11
# source /tmp/3.11/bin/activate
# python --version
Python 3.11.2
# pip --version
pip 22.3.1 from /tmp/3.11/lib64/python3.11/site-packages/pip (python 3.11)
# pip install -U ansible boto3 pip requests selinux
# pip freeze | grep selinux
selinux==0.3.0

source /opt/k8s/kubespray-venv/bin/activate

cd /opt/k8s
python3.12 -m venv kubespray-venv2 --system-site-packages
source /opt/k8s/kubespray-venv2/bin/activate
python -m ensurepip
cd /opt/kubernetes-iac/kubespray
pip install -U -r requirements.txt 
-->pip install -U boto3 pip requests selinux 

/opt/kubernetes-iac/ansible/k8s-init-vmware-config-ld5.sh 10.32.48.11 10.32.50.1

cd /opt/kubernetes-iac/kubespray;source /opt/k8s/kubespray-venv2/bin/activate
realme:ansible-playbook cluster.yml -i inventory/ld5/hosts.yaml -e @inventory/ld5/cluster-config.yaml --user=ansible --become --become-user=root --flush-cache
additionalme:ansible-playbook playbooks/cluster-additionalsteps.yml -i inventory/ld5/hosts.yaml -e @inventory/ld5/cluster-config.yaml --user=ansible --become --become-user=root --flush-cache
testme:ansible-playbook playbooks/cluster-test.yml -i inventory/ld5/hosts.yaml -e @inventory/ld5/cluster-config.yaml --user=ansible --become --become-user=root --flush-cache
resetme:ansible-playbook -i inventory/ld5/hosts.yaml reset.yml --become --become-user=root
```

## install kubectl 
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod 0755 ./kubectl
mv ./kubectl /usr/bin/kubectl

echo 'source <(kubectl completion bash)' >>~/.bashrc
echo 'alias k=kubectl --cluster=buof-k8s' >>~/.bashrc
echo 'complete -o default -F __start_kubectl k' >>~/.bashrc

(copy) ~/.kube/config from one of the nodes
kubectl config set-cluster k8sld5cluster --server=https://10.32.48.11 --insecure-skip-tls-verify=true
kubectl config view

kubectl config set-context ld5-context --cluster=k8sld5cluster --user=kubernetes-admin
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
kubectl describe svc argocd-server -n argocd
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer", "externalIPs": ["10.32.50.12"]}}'
kubectl patch svc argocd-server -n argocd -p '{"metadata": {"annotations": {"metallb.universe.tf/loadBalancerIPs": "10.32.50.40"}}, "spec": {"type": "LoadBalancer"}}'


kubectl label node ld5tmlk8sw01 node-role.kubernetes.io/worker="";kubectl label node ld5tmlk8sw02 node-role.kubernetes.io/worker="";kubectl label node ld5tmlk8sw03 node-role.kubernetes.io/worker="";

kubectl get nodes --show-labels
iperf -c 10.10.10.1 -p 550001 -i1 -t5
```
--------------------------------------------------------------------------------------------------------------------------------------------------------------------

```
[root@ld5tmlk8sc03 metallb]# kubectl get configmap kube-proxy -n kube-system -o yaml | grep mode
    mode: ipvs
[root@ld5tmlk8sc03 metallb]# kubectl get configmap kube-proxy -n kube-system -o yaml | grep strictARP
      strictARP: true
```
--------------------------------------------------------------------------------------------------------------------------------------------------------------------
```
kubectl api-resources --namespaced=true
kubectl -n metallb-system get IPAddressPool
kubectl -n metallb-system get L2Advertisement
kubectl describe IPAddressPool -n metallb-system
#iperf -c 10.32.50.83
```
--------------------------------------------------------------------------------------------------------------------------------------------------------------------
```
addresspools                                   metallb.io/v1beta1             true         AddressPool
bfdprofiles                                    metallb.io/v1beta1             true         BFDProfile
bgpadvertisements                              metallb.io/v1beta1             true         BGPAdvertisement
bgppeers                                       metallb.io/v1beta2             true         BGPPeer
communities                                    metallb.io/v1beta1             true         Community
ipaddresspools                                 metallb.io/v1beta1             true         IPAddressPool
l2advertisements                               metallb.io/v1beta1             true         L2Advertisement
servicel2statuses                              metallb.io/v1beta1             true         ServiceL2Status
```
--------------------------------------------------------------------------------------------------------------------------------------------------------------------
```
[root@ld5tmlk8sc01 kubernetes]# kubectl describe svc argocd-server -n argocd
Name:                     argocd-server
Namespace:                argocd
Labels:                   app.kubernetes.io/component=server
                          app.kubernetes.io/name=argocd-server
                          app.kubernetes.io/part-of=argocd
Annotations:              metallb.universe.tf/ip-allocated-from-pool: pool1
                          metallb.universe.tf/loadBalancerIPs: 10.32.50.40
Selector:                 app.kubernetes.io/name=argocd-server
Type:                     LoadBalancer
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       172.31.245.161
IPs:                      172.31.245.161
LoadBalancer Ingress:     10.32.50.40 (VIP)
Port:                     http  80/TCP
TargetPort:               8080/TCP
NodePort:                 http  32411/TCP
Endpoints:                172.29.3.24:8080
Port:                     https  443/TCP
TargetPort:               8080/TCP
NodePort:                 https  32141/TCP
Endpoints:                172.29.3.24:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Internal Traffic Policy:  Cluster
Events:
  Type    Reason        Age    From                Message
  ----    ------        ----   ----                -------
  Normal  IPAllocated   5m13s  metallb-controller  Assigned IP ["10.32.50.40"]
  Normal  nodeAssigned  5m13s  metallb-speaker     announcing from node "ld5tmlk8sw02" with protocol "layer2"
[root@ld5tmlk8sc01 kubernetes]#
```
--------------------------------------------------------------------------------------------------------------------------------------------------------------------
```
[root@ld5tmlk8sc01 ~]# kubectl version -o yaml
clientVersion:
  buildDate: "2024-09-11T21:28:00Z"
  compiler: gc
  gitCommit: 948afe5ca072329a73c8e79ed5938717a5cb3d21
  gitTreeState: clean
  gitVersion: v1.31.1
  goVersion: go1.22.6
  major: "1"
  minor: "31"
  platform: linux/amd64
kustomizeVersion: v5.4.2
serverVersion:
  buildDate: "2024-09-11T21:22:08Z"
  compiler: gc
  gitCommit: 948afe5ca072329a73c8e79ed5938717a5cb3d21
  gitTreeState: clean
  gitVersion: v1.31.1
  goVersion: go1.22.6
  major: "1"
  minor: "31"
  platform: linux/amd64
```
--------------------------------------------------------------------------------------------------------------------------------------------------------------------
```
helm repo add jetstack https://charts.jetstack.io --force-update
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.15.0/cert-manager.crds.yaml


ref: https://github.com/kubernetes-sigs/kubespray/issues/6054
```
