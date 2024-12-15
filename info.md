# ArgoCD information 
----------------------------------------------------------------------------------------------------------------------------------------------------------
## Install ArgoCD CLI

```bash
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod 0755 argocd-linux-amd64
mv argocd-linux-amd64 /usr/bin/argocd
argocd version --client
```

argocd: v2.13.2+dc43124
  BuildDate: 2024-12-11T19:01:33Z
  GitCommit: dc43124058130db9a747d141d86d7c2f4aac7bf9
  GitTreeState: clean
  GoVersion: go1.22.9
  Compiler: gc
  Platform: linux/amd64
----------------------------------------------------------------------------------------------------------------------------------------------------------
## ArgoCD UI: admin/Change-me-tomorrow-2
----------------------------------------------------------------------------------------------------------------------------------------------------------
## ArgoCD bitbucket information
https://bitbucket.org/tm-prod/kubernetes-iac.git
tmbuildjenkins@thinkmarkets.com 
k8s-argocd-readonly/<tmbuildjenkins-read-key.secret>
----------------------------------------------------------------------------------------------------------------------------------------------------------
## ArgoCD is checking every 1-3 min git repositories for configuration changes. Because of this for ArgoCD is dedicated separate IP different from Nginx Ingress Controller, which can help in firewall/acl definitions 
inventory/<ld5>/cluster-config.yaml : {{ argocd_externalip }}
----------------------------------------------------------------------------------------------------------------------------------------------------------
## CLI sample commands
```bash
argocd login --core
kubectl config set-context --current --namespace=argocd
```
----------------------------------------------------------------------------------------------------------------------------------------------------------
##  Creating new argocd flow 
```bash
argocd proj create ld5-app-uat -d https://kubernetes.default.svc,ld5-app-uat -s https://bitbucket.org/tm-prod/kubernetes-iac.git
argocd repo add https://bitbucket.org/tm-prod/kubernetes-iac.git --username tmbuildjenkins --password <tmbuildjenkins-read-key.secret>

argocd app create uat-test \
    --repo https://bitbucket.org/tm-prod/kubernetes-iac.git \
    --path service-a \
    --revision uat-test \
    --dest-server  https://kubernetes.default.svc \
    --directory-recurse \
    --sync-policy automated \
    --self-heal \
    --sync-option Prune=true \
    --sync-option CreateNamespace=true \
(ooptional)    --dest-namespace uat-test
```
----------------------------------------------------------------------------------------------------------------------------------------------------------
# Listing information

[root@ld5tmlk8sc01 ~]# argocd proj list
NAME         DESCRIPTION  DESTINATIONS                                SOURCES                                           CLUSTER-RESOURCE-WHITELIST  NAMESPACE-RESOURCE-BLACKLIST  SIGNATURE-KEYS  ORPHANED-RESOURCES  DESTINATION-SERVICE-ACCOUNTS
default                   *,*                                         *                                                 */*                         <none>                        <none>          disabled            <none>
ld5-app-uat               https://kubernetes.default.svc,ld5-app-uat  https://bitbucket.org/tm-prod/kubernetes-iac.git  <none>                      <none>                        <none>          disabled            <none>
[root@ld5tmlk8sc01 cert]# argocd repo list
TYPE  NAME  REPO                                              INSECURE  OCI    LFS    CREDS  STATUS      MESSAGE  PROJECT
git         https://bitbucket.org/tm-prod/kubernetes-iac.git  false     false  false  true   Successful
[root@ld5tmlk8sc01 cert]# argocd app list
NAME             CLUSTER                         NAMESPACE  PROJECT  STATUS  HEALTH   SYNCPOLICY  CONDITIONS  REPO                                              PATH       TARGET
argocd/uat-test  https://kubernetes.default.svc  uat-test   default  Synced  Healthy  Auto        <none>      https://bitbucket.org/tm-prod/kubernetes-iac.git  service-a  uat-test

----------------------------------------------------------------------------------------------------------------------------------------------------------
Create TLS:
```bash
/usr/local/bin/kubectl create -n argocd secret tls argocd-server-tls --cert=/etc/letsencrypt/live/argocd.tfxcorp.com/fullchain.pem --key=/etc/letsencrypt/live/argocd.tfxcorp.com/privkey.pem
```
Renew TLS:
```bash
/usr/local/bin/kubectl create -n argocd secret tls argocd-server-tls --save-config --dry-run=client --key=/etc/letsencrypt/live/argocd.tfxcorp.com/privkey.pem --cert=/etc/letsencrypt/live/argocd.tfxcorp.com/fullchain.pem -o yaml | kubectl apply -f -
```
----------------------------------------------------------------------------------------------------------------------------------------------------------

cd /etc/kubernetes

[root@ld5tmlk8sc01 ~]# kubectl describe svc argocd-server -n argocd
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
IP:                       172.31.115.36
IPs:                      172.31.115.36
LoadBalancer Ingress:     10.32.50.40 (VIP)
Port:                     http  80/TCP
TargetPort:               8080/TCP
NodePort:                 http  32672/TCP
Endpoints:                172.29.0.29:8080
Port:                     https  443/TCP
TargetPort:               8080/TCP
NodePort:                 https  31525/TCP
Endpoints:                172.29.0.29:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Internal Traffic Policy:  Cluster
Events:
  Type    Reason        Age   From                Message
  ----    ------        ----  ----                -------
  Normal  IPAllocated   44s   metallb-controller  Assigned IP ["10.32.50.40"]
  Normal  nodeAssigned  44s   metallb-speaker     announcing from node "ld5tmlk8sw02" with protocol "layer2"
