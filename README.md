
# GCP Kubernetes ElasticSearch,Kibana Installation

This file contains a set of scripts and commands used to successfully deploy ElasticSearch with Kibana on Kubernetes. These commands should be run from the google cloud shell terminal


## Kubernetes Setup on GCP 

### Create Kubernetes Cluster with 3 nodes 
```bash
gcloud config set project projet-id-name  
gcloud config set compute/zone us-central1-c
gcloud container clusters create cluster-name --num-nodes=3
```

### Connect to cluster and ensure nodes are running 
```bash
gcloud container clusters get-credentials cluster-name 
kubectl get nodes 
```

## ElasticSerach and Kibana Deployment

### ElasticSearch Deployment

```bash
kubectl apply -f https://download.elastic.co/downloads/eck/1.4.1/all-in-one.yaml

cat <<EOF | kubectl apply -f -
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: quickstart
spec:
  version: 7.11.2 #Version of your choice 
  http: 
    service: 
      spec: 
        type: LoadBalancer #Adds a External IP 
  nodeSets: 
  - name: default 
    count: 1 
    config: 
      node.master: true 
      node.data: true 
      node.ingest: true 
      node.store.allow_mmap: false 
EOF
```

**Verify Elastic Search is successfully running (Will take a couple minutes at most)** 

```bash
kubectl get elasticsearch
kubectl get service quickstart-es-http
```

Make note of External-IP and Port(should be 9200).

**Generate password and curl into elasticSearch** 

```bash
PASSWORD=$(kubectl get secret quickstart-es-elastic-user -o=jsonpath='{.data.elastic}' | base64 -d)

curl -u elastic:$PASSWORD -k https://34.72.231.252:9200  #Use External-IP and Port
```
ElasticSearch is now successfully deployed 

## Kibana Deployment

```bash
cat <<EOF | kubectl apply -f -
apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
  name: quickstart
spec:
  version: 7.11.2 #Make sure Kibana and Elasticsearch are the same version. 
  http: 
    service: 
      spec: 
        type: LoadBalancer #Adds a External IP 
  count: 1 
  elasticsearchRef: 
    name: quickstart 
EOF

```

**Verify if Kibana is successfully created (Health should be green. It will take a minute or two)**

```bash
kubectl get kibana
kubectl get service quickstart-kb-http
```
Make Note of External-IP and Port(Should be 5601)

## Accessing Kibana 

1. Type in `$PASSWORD` in Cloud Shell and copy the password
2. Open a new tab in your browser and put in the External IP address followed by port 5601 (Ensure you use the following format: `https://1.1.1.1:5601`)
3. login using user as `elastic` and password is `$PASSWORD` you copied


