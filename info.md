# VMWare environment
Script should be executed with admin credentials. You can find instructions how to setup your in terraform/vmware/config-ld5/terraform.tfvars
Machine and environment configuration is stored in terraform/vmware/config-ld5/terraform.tfvars
NB: General rule of thumb: You should not change any other file than terraform/vmware/config-ld5/terraform.tfvars and kubespray/inventory/ld5/hosts.yaml

## Initial setup:
### Step 1: Terraform 

```bash
cd ./kubernetes-iac/terraform/vmware/config-ld5
terraform init --reconfigure
terraform plan
```
To create environment execute and confirm
```bash
terraform apply
```
To destroy environment execute and confirm
```bash
terraform destoy 
```

### Step 2: Kubespray via Ansible
Update kubespray/inventory/ld5/hosts.yaml with nesessary host information
You can change values in kubespray/inventory/ld5/cluster-config.yaml but on your own risk. Do not touch any other file from kybespray folder.

```bash
cd /opt/kubernetes-iac/kubespray
(optional) you can create custom python environment refer to docs/k8s-kubespray.md
pip install -r requirements.txt
ansible-playbook cluster.yml -i inventory/ld5/hosts.yaml -e @inventory/ld5/cluster-config.yaml --user=ansible --become --become-user=root --flush-cache
```
When initial setup of kubernetes cluster finish we need to install additional modules: MetalLB, ArgoCD, Nginx Ingress Controler and Cert Manager.
```bash
ansible-playbook playbooks/cluster-additionalsteps.yml -i inventory/ld5/hosts.yaml -e @inventory/ld5/cluster-config.yaml --user=ansible --become --become-user=root --flush-cache
```
To reset/delete kubernetes cluster use 
```bash
ansible-playbook -i inventory/ld5/hosts.yaml reset.yml --become --become-user=root
```

Now you should be able to access ArgoCD at https://<argocd_externalip>  with password provided in cluster-config.yaml

### Step 3: Use ArgoCD to configure rest of the cluster . For more detailed information ref: [docs/k8s-argocd.md](./docs/k8s-argocd.md)
NB: do not use main branch because Argo will pull data every 3 min and will look for changes

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
    --sync-option CreateNamespace=true
```

### Additional installation instructions, cli commands and configuration parameters can be found in dos folder in respective files
