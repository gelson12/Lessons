# How to Install MongoDB on Kubernetes?

You can find tutorial [here]().

- Install Prometheus and Grafana on Kubernetes
- Install MongoDB Kubernetes Operator
- Install MongoDB on Kubernetes (Standalone/Single Replica)
- (Create a database user with SCRAM authentication.)
- Install MongoDB on Kubernetes (Replica Set)
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

## Install MongoDB Kubernetes Operator

kubectl apply -f k8s/mongodb/namespace.yaml
kubectl apply -f k8s/mongodb/crd.yaml
kubectl apply -f k8s/mongodb/rbac
kubectl apply -f k8s/mongodb/operator.yaml
kubectl get pods -n mongodb

## Install MongoDB on Kubernetes (Standalone/Single Replica)

kubectl apply -f k8s/mongodb/standalone/