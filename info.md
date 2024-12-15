## Nginx Ingress Controler information

### Download specific version of nginx ingress controller

CHART_VERSION="4.4.0"
APP_VERSION="1.5.1"

```
mkdir -p ingress/controller/nginx/manifests/

helm template ingress-nginx ingress-nginx \
--repo https://kubernetes.github.io/ingress-nginx \
--version ${CHART_VERSION} \
--namespace ingress-nginx \
> ingress/controller/nginx/manifests/nginx-ingress.${APP_VERSION}.yaml
```
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### Since we have problem with certificate creation via cert-manager in kubernetes cluster, we configure certbot as standalone application on c01 node and use files to create secrets in the kubernetes cluster 

Issue wildcard certificate for tfxcorp.com domain. 
```
certbot certonly --dns-cloudflare --dns-cloudflare-credentials /root/.secrets/cloudflare.ini -d *.tfxcorp.com --preferred-challenges dns-01
```
Create secret for wildcard ftxcorp.com in cert-test namespace of the cluster
bash```
kubectl create -n cert-test secret tls secret-tfxcorp-com-tls   --cert=/etc/letsencrypt/live/tfxcorp.com/fullchain.pem   --key=/etc/letsencrypt/live/tfxcorp.com/privkey.pem
```
[root@ld5tmlk8sc01 cert]# kubectl get ing -A
NAMESPACE   NAME                CLASS   HOSTS           ADDRESS   PORTS     AGE
cert-test   service-a-ingress   nginx   a.tfxcorp.com             80, 443   6s

After certbot creates new certificate to renew renew secret-tfxcorp-com-tls in kubernetes cluster in cert-test namespace use below command
bash``` 
kubectl create -n cert-test secret tls secret-tfxcorp-com-tls --save-config --dry-run=client --key=/etc/letsencrypt/live/tfxcorp.com/privkey.pem --cert=/etc/letsencrypt/live/tfxcorp.com/fullchain.pem -o yaml | kubectl apply -f -
```
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### List all ingress rules
bash```
kubectl get ing -A
```
[root@ld5tmlk8sc01 cert]# kubectl get ing -A
NAMESPACE   NAME                CLASS    HOSTS           ADDRESS   PORTS   AGE
cert-test   service-a-ingress   <none>   a.tfxcorp.com             80      7m50s
[root@ld5tmlk8sc01 cert]#
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### sample ingress-nginx scripts

bash```
kubectl apply -f ingress/controller/nginx/manifests/nginx-ingress.${APP_VERSION}.y
kubectl -n ingress-nginx logs -l app.kubernetes.io/instance=ingress-nginx
kb create secret tls star.tfxcorp.com --key /opt/tls/star.tfxcorp.com.key --cert /opt/tls/star.tfxcorp.com.crt --namespace monitoring
kb create secret tls star.tfxcorp.com --key /opt/tls/star.tfxcorp.com.key --cert /opt/tls/star.tfxcorp.com.crt  (for default namespace)
```
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
