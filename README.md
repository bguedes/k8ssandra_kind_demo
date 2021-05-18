- Delete Helm repository

helm repo remove k8ssandra
helm repo remove traefik
helm repo list

- Start Kind Cluster

kind create cluster --config ./kind.config.yaml

- Validate the available Kubernetes StorageClasses

kubectl get storageclasses

kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml

kubectl get storageclasses

NAME                 PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local-path           rancher.io/local-path   Delete          WaitForFirstConsumer   false                  35m
standard (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  37m

- Configure the K8ssandra Helm repository

helm repo add k8ssandra https://helm.k8ssandra.io/stable
helm repo add traefik https://helm.traefik.io/traefik
helm repo update

helm install traefik traefik/traefik --create-namespace -f traefik.yaml
helm install -f k8ssandra-1node.yaml k8ssandra k8ssandra/k8ssandra

kubectl get pods

- Set up port forwarding

kubectl get services

kubectl port-forward svc/k8ssandra-dc1-stargate-service 8080 8081 8082 9042 &
kubectl port-forward svc/k8ssandra-grafana 9191:80 &
kubectl port-forward svc/prometheus-operated 9292:9090 &
kubectl port-forward svc/k8ssandra-reaper-reaper-service 9393:8080 &

kubectl port-forward svc/kubernetes 9000 &

jobs -l

Stargate swagger UI: http://localhost:8082/swagger-ui/
GraphQL Playground: http://localhost:8080/playground
Prometheus: http://localhost:9292
Grafana: http://localhost:9191
Reaper: http://localhost:9393/webui


- Copy Stockwatcher files

kubectl cp cql k8ssandra-dc1-default-sts-0:/home/cassandra/


- Access Cassandra using the Stargate APIs

curl -L -X POST 'http://localhost:8081/v1/auth' -H 'Content-Type: application/json' --data-raw '{"username": "k8ssandra-superuser", "password": "0fx00j4Y4US5iWgKqtH9"}'


- Access the Apache Cassandra

- Retrieve K8ssandra superuser credentials

kubectl get secret k8ssandra-superuser -o jsonpath="{.data.username}" | base64 --decode ; echo
kubectl get secret k8ssandra-superuser -o jsonpath="{.data.password}" | base64 --decode ; echo

username: k8ssandra-superuser
password: sOzUCJDPQ1ceOTQ6M50K


kubectl exec -it k8ssandra-dc1-default-sts-0 -c cassandra -- nodetool -u k8ssandra-superuser -pw sOzUCJDPQ1ceOTQ6M50K status

cqlsh -u k8ssandra-superuser -p sOzUCJDPQ1ceOTQ6M50K


- Upgrade notice for K8ssandra 1.1.0

cassandra:
  datacenters:
  - name: dc1
    size: 3

helm upgrade -f k8ssandra-3node.yaml k8ssandra k8ssandra/k8ssandra



kubectl get deployment k8ssandra-dc1-stargate

kubectl scale deployment k8ssandra-dc1-stargate  --replicas 3

- delete cluster
