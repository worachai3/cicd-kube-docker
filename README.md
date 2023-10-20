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
kops create cluster --name=wrccluster.k8s.local --state=s3://vprofile-kops-state-wrc --zones=ap-southeast-1a,ap-southeast-1b --node-count=2 --node-size=t3.small --master-size=t3.medium --node-volume-size=8 --master-volume-size=8
kops update cluster --name wrccluster.k8s.local --yes
kops validate cluster --name wrccluster.k8s.local

## Clean up steps
kops delete cluster --name=wrccluster.k8s.local --state=s3://vprofile-kops-state-wrc --yes
helm delete vprofile-stack --namespace prod