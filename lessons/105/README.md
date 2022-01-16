# How to Install MongoDB on Kubernetes?

You can find tutorial [here]().

- Install MongoDB Kubernetes Operator
- Install MongoDB on Kubernetes (Standalone/Single Replica)
- Install MongoDB on Kubernetes (Replica Set)
- Install Cert-Manager on Kubernetes
- Secure MongoDB with TLS/SSL
- Configure Generic External Access with Node Port
- Configure External Access on AWS
- Install Prometheus and Grafana on Kubernetes
- Monitor MongoDB with Prometheus

Include:
- test leader election (Arbiters)
- monitor with prometheus
- scale up/down
- upgrade
- Create users with SCRAM authentication
- Create custom roles
- Show reference to terraform code AWS & GCP
- pod antiaffinity for external access port forward
- Deploy a Sharded Cluster (https://docs.mongodb.com/kubernetes-operator/master/tutorial/deploy-sharded-cluster/)
- include load balancer price

- [MongoDB Community Kubernetes Operator](https://github.com/mongodb/mongodb-kubernetes-operator)
- [Example](https://www.mongodb.com/blog/post/run-secure-containerized-mongodb-deployments-using-the-mongo-db-community-kubernetes-oper)
- [Deploy and Configure a MongoDB Resource](https://github.com/mongodb/mongodb-kubernetes-operator/blob/master/docs/deploy-configure.md)
- [Kubernetes Operator Docs](https://github.com/mongodb/mongodb-kubernetes-operator/tree/master/docs)
- [Kubernetes Operator Samples](https://github.com/mongodb/mongodb-kubernetes-operator/tree/master/config/samples)
- [Question about configuring a simple external access](https://github.com/mongodb/mongodb-kubernetes-operator/issues/634)


## Install MongoDB Kubernetes Operator

kubectl apply -f k8s/mongodb/namespace.yaml
kubectl apply -f k8s/mongodb/crd.yaml
kubectl apply -f k8s/mongodb/rbac
kubectl apply -f k8s/mongodb/operator.yaml
kubectl get pods -n mongodb
kubectl logs \
  -l name=mongodb-kubernetes-operator \
  -n mongodb \
  -f

## Install MongoDB on Kubernetes (Standalone/Single Replica)

kubectl apply -f k8s/mongodb/database/secret.yaml
kubectl apply -f k8s/mongodb/database/mongodb.yaml
kubectl get pods -n mongodb
kubectl get pvc -n mongodb
kubectl get secret my-mongodb-admin-admin-user -o yaml -n mongodb

kubectl get secret my-mongodb-admin-admin-user -n mongodb -o json | jq -r '.data | with_entries(.value |= @base64d)'


install MongoDB Shell
brew install mongosh

(go over connection string formets) Connection String URI Format - https://docs.mongodb.com/manual/reference/connection-string/


kubectl port-forward my-mongodb-0 27017 -n mongodb


mongosh "mongodb://admin-user:admin123@127.0.0.1:27017/admin?directConnection=true&serverSelectionTimeoutMS=2000"

show dbs
db.createUser(
  {
    user: 'aputra',
    pwd: 'devops123',
    roles: [ { role: 'readWrite', db: 'store' } ]
  }
);
db.auth('aputra', 'devops123')

use store
db.employees.insertOne({name: "Anton"})

db.employees.find()

## Install MongoDB on Kubernetes (Replica Set)

scale up from 1 to 3
kubectl apply -f k8s/mongodb/database/mongodb.yaml
kubectl get pods -n mongodb
explain why it's hard to connect from local host to primary

rs.status()
rs.printSecondaryReplicationInfo()


## Install Cert-Manager on Kubernetes

<!-- kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.6.1/cert-manager.yaml -->

helm repo add jetstack \
    https://charts.jetstack.io

helm repo update

kubectl create -f k8s/prometheus-operator/crds

helm install cert-105 jetstack/cert-manager \
  --namespace cert-manager \
  --version v1.6.1 \
  --create-namespace \
  --values cert-manager-values.yaml


## Secure MongoDB with TLS/SSL

kubectl exec -it my-mongodb-0 -- bash

mongosh \
  --tls \
  --tlsCAFile /var/lib/tls/ca/ca.crt \
  --tlsCertificateKeyFile /var/lib/tls/server/*.pem \
  "mongodb+srv://admin-user:devops123@my-mongodb-svc.mongodb.svc.cluster.local/admin?ssl=true"

mongosh \
  --tls \
  --tlsCAFile /var/lib/tls/ca/ca.crt \
  --tlsCertificateKeyFile /var/lib/tls/server/*.pem \
  "mongodb://admin-user:devops123@my-mongodb-0.devopsbyexample.io:27017,my-mongodb-1.devopsbyexample.io:27017,my-mongodb-2.devopsbyexample.io:27017/admin?directConnection=true&serverSelectionTimeoutMS=2000"

Create SRV record

_mongodb._tcp.my-mongodb

0 50 27017 my-mongodb-0.devopsbyexample.io.
0 50 27017 my-mongodb-1.devopsbyexample.io.
0 50 27017 my-mongodb-2.devopsbyexample.io.

cat /var/lib/tls/server/*.pem
cat /var/lib/tls/ca/ca.crt
code certificateKey.pem
code ca.pem

kubectl exec -n mongodb my-mongodb-0 -c mongod -- cat /var/lib/tls/ca/ca.crt > ca.crt
cat ca.crt

kubectl get secrets -n mongodb my-mongodb-server-certificate-key -o yaml
echo "ksfb" | base64 -d > certificateKey.pem
cat certificateKey.pem

mongosh \
  --tls \
  --tlsCAFile ca.pem \
  --tlsCertificateKeyFile certificateKey.pem \
  "mongodb://admin-user:devops123@my-mongodb-0.devopsbyexample.io:27017,my-mongodb-1.devopsbyexample.io:27017,my-mongodb-2.devopsbyexample.io:27017/admin?serverSelectionTimeoutMS=2000"

mongosh \
  --tls \
  --tlsCAFile ca.pem \
  --tlsCertificateKeyFile certificateKey.pem \
  "mongodb+srv://admin-user:admin123@my-mongodb.devopsbyexample.io/admin?ssl=true&serverSelectionTimeoutMS=2000"

## Install Prometheus and Grafana on Kubernetes (v0.53.1)

Error
- The CustomResourceDefinition "prometheuses.monitoring.coreos.com" is invalid: metadata.annotations: Too long: must have at most 262144 bytes.
- That is not possible anymore since prometheus CRD is too long to fit in the "kubectl.kubernetes.io/last-applied-configuration" annotation.

kubectl apply -f k8s/prometheus-operator/rbac
kubectl apply -f k8s/prometheus-operator/deployment
kubectl get pods -n monitoring

(set correct persmissions otherwise you get permission denied)
kubectl apply -f k8s/prometheus

kubectl apply -f k8s/mongodb/exporter

kubectl get pods,svc -n monitoring
kubectl port-forward svc/prometheus-operated 9090 -n monitoring

Deploy grafana

echo -n "devops123" | base64
place one dashboard per config map, if it grows you not going to be able to apply (there is a limit for config map size)

kubectl apply -f k8s/cadvisor


kubectl port-forward service/prometheus-operated 9090 -n monitoring
- show all 3 targets in prometheus

kubectl apply -R -f k8s/grafana
kubectl port-forward svc/grafana 3000 -n monitoring




- [percona/mongodb_exporter](https://github.com/percona/mongodb_exporter)
- https://grafana.com/grafana/dashboards/7353

Create user woth k8s object
      {
         "role":"clusterMonitor",
         "db":"admin"
      },
      {
         "role":"read",
         "db":"local"
      }


kubectl run -i --tty --rm busybox --image=busybox -- sh
nslookup -q=SRV my-mongodb-svc.mongodb.svc.cluster.local
nslookup -q=SRV _mongodb._tcp.my-mongodb.devopsbyexample.io

my-mongodb-0.my-mongodb-svc.mongodb.svc.cluster.local

docker pull percona/mongodb_exporter:0.30

run application after prometheus and grafana is configured, maybe load test

- [MongoDB Connection Types](https://docs.mongodb.com/manual/reference/command/serverStatus/#connections)
- [Prometheus Old/New Metrics](https://github.com/percona/mongodb_exporter/blob/v0.30.0/exporter/v1_compatibility.go)

import mongodb grafana dashboard from json

kubectl delete -R k8s