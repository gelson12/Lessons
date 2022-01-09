# How to Install MongoDB on Kubernetes?

You can find tutorial [here]().

- Install MongoDB Kubernetes Operator
- Install MongoDB on Kubernetes (Standalone/Single Replica)
- Install MongoDB on Kubernetes (Replica Set)
- Install Prometheus and Grafana on Kubernetes
- Monitor MongoDB with Prometheus
- Install Cert-Manager on Kubernetes
- Secure MongoDB with TLS/SSL
- Configure Generic External Access with Node Port
- Configure External Access on AWS
- Configure External Access on GCP

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





## Install Prometheus and Grafana on Kubernetes (v0.53.1)

Error
- The CustomResourceDefinition "prometheuses.monitoring.coreos.com" is invalid: metadata.annotations: Too long: must have at most 262144 bytes.
- That is not possible anymore since prometheus CRD is too long to fit in the "kubectl.kubernetes.io/last-applied-configuration" annotation.

kubectl create -f k8s/prometheus-operator/crds
kubectl apply -f k8s/prometheus-operator/rbac
kubectl apply -f k8s/prometheus-operator/deployment
kubectl get pods -n monitoring

(set correct persmissions otherwise you get permission denied)
kubectl apply -f k8s/prometheus
kubectl get pods,svc -n monitoring

Deploy grafana

echo -n "devops123" | base64
place one dashboard per config map, if it grows you not going to be able to apply (there is a limit for config map size)

kubectl apply -R -f k8s/grafana
kubectl port-forward service/prometheus-operated 9090 -n monitoring
- show all 3 targets in prometheus
kubectl port-forward svc/grafana 3000 -n monitoring
deploy cadvisor

## Install MongoDB Kubernetes Operator

kubectl apply -f k8s/mongodb/namespace.yaml
kubectl apply -f k8s/mongodb/crd.yaml
kubectl apply -f k8s/mongodb/rbac
kubectl apply -f k8s/mongodb/operator.yaml
kubectl get pods -n mongodb

## Install MongoDB on Kubernetes (Standalone/Single Replica)

kubectl apply -f k8s/mongodb/standalone/
kubectl get pods -n mongodb
kubectl get pvc -n mongodb
kubectl get secret mongodb-standalone-admin-admin-user -o yaml -n mongodb

kubectl get secret mongodb-standalone-admin-admin-user -n mongodb -o json | jq -r '.data | with_entries(.value |= @base64d)'


install MongoDB Shell
brew install mongosh
kubectl pod
(go over connection string formets) Connection String URI Format - https://docs.mongodb.com/manual/reference/connection-string/
mongosh
mongosh "mongodb+srv://<username>:<password>@example-mongodb-svc.mongodb.svc.cluster.local/admin?ssl=true"
kubectl port-forward mongodb-standalone-pod-0 27017 -n mongodb
(mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000)

mongosh "mongodb://admin-user:devops123@127.0.0.1:27017/admin?directConnection=true&serverSelectionTimeoutMS=2000"

show dbs
use store
db.user.insertOne({name: "Anton"})
rs.status()
rs.printSecondaryReplicationInfo()

db.getUsers()

db.createUser(
  {
    user: 'aputra',
    pwd: 'devops123',
    roles: [ { role: 'readWrite', db: 'store' } ]
  }
);
db.auth('aputra', 'devops123')

use store
db.users.insertOne({name: "Anton"})

db.users.find()

prometheus

## Install MongoDB on Kubernetes (Replica Set)

scale up from 1 to 3
explain why it's hard to connect from local host to primary
- [percona/mongodb_exporter](https://github.com/percona/mongodb_exporter)
- https://grafana.com/grafana/dashboards/7353

rs.status()
connect to secondary
try to add item
connect to primary and add item

(update time, maybe??) (Yekaterinburg Standard Time)
rs.printSecondaryReplicationInfo()

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
nslookup -q=SRV mongodb-standalone-svc.mongodb.svc.cluster.local

docker pull percona/mongodb_exporter:0.30

run application after prometheus and grafana is configured, maybe load test

- [MongoDB Connection Types](https://docs.mongodb.com/manual/reference/command/serverStatus/#connections)
- [Prometheus Old/New Metrics](https://github.com/percona/mongodb_exporter/blob/v0.30.0/exporter/v1_compatibility.go)

import mongodb grafana dashboard from json

deploy cadvisor
kubectl apply -f k8s/cadvisor
https://grafana.com/grafana/dashboards/315