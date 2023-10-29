## Prerequisites
- JDK 1.8 or later
- Maven 3 or later
- MySQL 5.6 or later

## Technologies 
- Spring MVC
- Spring Security
- Spring Data JPA
- Maven
- JSP
- MySQL
## Database
Here,we used Mysql DB 
MSQL DB Installation Steps for Linux ubuntu 14.04:
- $ sudo apt-get update
- $ sudo apt-get install mysql-server

Then look for the file :
- /src/main/resources/accountsdb
- accountsdb.sql file is a mysql dump file.we have to import this dump to mysql db server
- > mysql -u <user_name> -p accounts < accountsdb.sql


## Setup kops
export KOPS_STATE_STORE=s3://vprofile-kops-state-wrc
kops create secret --name mycluster.k8s.local sshpublickey admin -i ~/.ssh/id_rsa.pub
kops create cluster --name=wrccluster.k8s.local --state=s3://vprofile-kops-state-wrc --zones=ap-southeast-1a,ap-southeast-1b --node-count=2 --node-size=t3.small --master-size=t3.medium --node-volume-size=8 --master-volume-size=10  --ssh-access 172.31.37.240/32 --yes
kops update cluster --name wrccluster.k8s.local --yes
kops validate cluster --wait 10m --state=s3://vprofile-kops-state-wrc

## Setup Vault
kubectl create namespace vault
helm repo add hashicorp https://helm.releases.hashicorp.com
helm install vault hashicorp/vault --namespace vault
kubectl exec -it vault-0 --namespace=vault -- /bin/sh
vault operator init

## Unseal the first vault server until it reaches the key threshold
$ kubectl exec --stdin=true --tty=true vault-0 -- vault operator unseal # ... Unseal Key 1
$ kubectl exec --stdin=true --tty=true vault-0 -- vault operator unseal # ... Unseal Key 2
$ kubectl exec --stdin=true --tty=true vault-0 -- vault operator unseal # ... Unseal Key 3

vault login <root_token>
vault secrets enable -path=secret kv-v2
vault kv put secret/vprofile-config db.username=<username> db.password=<password>

vault write auth/kubernetes/config kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443"
vault policy write vprofile - << EOF
path "internal/*" {
  capabilities = ["read"]
}
EOF

vault write auth/kubernetes/role/vprofile \
bound_service_account_names=vprosa \
bound_service_account_namespaces=prod \
policies=vprofile \
ttl=24h

kubectl create serviceaccount vprosa --namespace prod

## Setup Nginx Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.3/deploy/static/provider/aws/deploy.yaml

## Clean up steps
helm delete vprofile-stack --namespace prod
kops delete cluster --name=wrccluster.k8s.local --state=s3://vprofile-kops-state-wrc --yes
kubectl delete -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.3/deploy/static/provider/aws/deploy.yaml