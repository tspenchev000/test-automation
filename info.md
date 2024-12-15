## Cert-Manager information

Download specific version of cert manager

helm repo add jetstack https://charts.jetstack.io --force-update

kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.16.2/cert-manager.yaml

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
## Since cert-manager can not issue certificate for some reason?! as workaround cert is issued with certbot application from OS on C01 and then imported in the namespace 

Create TLS namespace cert-test:
```
/usr/local/bin/kubectl create -n cert-test secret tls secret-tfxcorp-com-tls --cert=/etc/letsencrypt/live/tfxcorp.com/fullchain.pem --key=/etc/letsencrypt/live/tfxcorp.com/privkey.pem
```
Renew TLS namespace cert-test:
```
/usr/local/bin/kubectl create -n cert-test secret tls secret-tfxcorp-com-tls --save-config --dry-run=client --key=/etc/letsencrypt/live/tfxcorp.com/privkey.pem --cert=/etc/letsencrypt/live/tfxcorp.com/fullchain.pem -o yaml | kubectl apply -f -
```
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
cmctl:  https://cert-manager.io/docs/reference/cmctl/#installation


curl -fsSL -o cmctl https://github.com/cert-manager/cmctl/releases/download/v2.1.1/cmctl_linux_amd64
chmod 0755 cmctl
sudo mv cmctl /usr/local/bin

[root@ld5tmlk8sc01 cert]# cmctl check api
The cert-manager API is ready

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
cloudflare: https://ruanbekker.hashnode.dev/cert-manager-dns-challenge-with-cloudflare-on-kubernetes

https://cert-manager.io/docs/troubleshooting/
https://cert-manager.io/docs/troubleshooting/acme/

```
kubectl get issuer
kubectl get clusterissuer

kubectl describe issuer <letsencrypt-http>
kubectl describe clusterissuer <letsencrypt-http>

kubectl describe certificaterequest <example-com-2745722290>
kubectl describe order <example-com-2745722290-439160286>

kubectl get challenges
```

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
```
[root@ld5tmlk8sc01 kubernetes]# kubectl get challenges -n argocd -o wide
NAME                                         STATE     DOMAIN               REASON                                                                                             AGE
argocd-certificate-1-1568510716-1516881485   pending   argocd.tfxcorp.com   Waiting for DNS-01 challenge propagation: DNS record for "argocd.tfxcorp.com" not yet propagated   42m


kubectl describe challenge argocd-certificate-1-1568510716-1516881485
[root@ld5tmlk8sc01 kubernetes]# kubectl describe challenge argocd-certificate-1-1568510716-1516881485 -n argocd
Name:         argocd-certificate-1-1568510716-1516881485
Namespace:    argocd
Labels:       <none>
Annotations:  <none>
API Version:  acme.cert-manager.io/v1
Kind:         Challenge
Metadata:
  Creation Timestamp:  2024-12-13T15:40:18Z
  Finalizers:
    finalizer.acme.cert-manager.io
  Generation:  1
  Owner References:
    API Version:           acme.cert-manager.io/v1
    Block Owner Deletion:  true
    Controller:            true
    Kind:                  Order
    Name:                  argocd-certificate-1-1568510716
    UID:                   bacd7dba-7f4b-4b8e-b901-f67329ccfaea
  Resource Version:        36069
  UID:                     2fa3676d-c93f-4aeb-b34f-b03af68ffd69
Spec:
  Authorization URL:  https://acme-v02.api.letsencrypt.org/acme/authz/2111231365/444327547305
  Dns Name:           argocd.tfxcorp.com
  Issuer Ref:
    Kind:  ClusterIssuer
    Name:  letsencrypt-production
  Key:     63H0RXaoQOqvjirVo_AYyB0bfXylgzvy7IjY4riuFFM
  Solver:
    dns01:
      Cloudflare:
        API Key Secret Ref:
          Key:   api-key
          Name:  cloudflare-api-key-secret
        Email:   it-certificates@thinkmarkets.com
  Token:         5Wa9MZwpoju0PbFCBxW9mIAvL7JP_RHfxSZpLhBzB8k
  Type:          DNS-01
  URL:           https://acme-v02.api.letsencrypt.org/acme/chall/2111231365/444327547305/vUkx4g
  Wildcard:      false
Status:
  Presented:   true
  Processing:  true
  Reason:      Waiting for DNS-01 challenge propagation: DNS record for "argocd.tfxcorp.com" not yet propagated
  State:       pending
Events:
  Type    Reason     Age   From                     Message
  ----    ------     ----  ----                     -------
  Normal  Started    44m   cert-manager-challenges  Challenge scheduled for processing
  Normal  Presented  44m   cert-manager-challenges  Presented challenge using DNS-01 challenge mechanism
[root@ld5tmlk8sc01 kubernetes]#
```

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------

https://ruanbekker.hashnode.dev/cert-manager-dns-challenge-with-cloudflare-on-kubernetes
```
kubectl create secret generic cloudflare-api-key-secret \
  --from-literal=api-key=[YOUR_CLOUDFLARE_API_KEY]

apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-dns01-issuer
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: you@example.com  # your email address for updates
    privateKeySecretRef:
      name: letsencrypt-dns01-private-key
    solvers:
    - dns01:
        cloudflare:
          email: you@example.com # your cloudflare account email address
          apiTokenSecretRef:
            name: cloudflare-api-key-secret
            key: api-key
```


```
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: default-workshop-certificate
  namespace: default
spec:
  secretName: default-workshop-example-tls
  issuerRef:
    name: letsencrypt-dns01-issuer
    kind: ClusterIssuer
  commonName: workshop.example.com
  dnsNames:
  - workshop.example.com
  - '*.workshop.example.com'
```
