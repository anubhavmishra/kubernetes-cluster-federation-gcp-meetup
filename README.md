# Kubernetes Cluster Federation - Google Cloud Platform Meetup Vancouver

This tutorial assumes that you have Kubernetes cluster federation already setup using Kelsey Hightower's guide [Kubernetes Cluster Federation (The Hard Way)](https://github.com/kelseyhightower/kubernetes-cluster-federation).

In this tutorial we will create the following:

* A federated config map
* A federated secret
* A federated service
* A federated replica set

## Prerequisite

```bash
kubectl config use-context federation-cluster
```

## List All Clusters

```bash
kubectl get clusters
```

```bash
NAME             STATUS    AGE
asia-east1-b     Ready     14m
europe-west1-b   Ready     14m
us-central1-b    Ready     14m
us-east1-b       Ready     14m
```

## List All Pods

```bash
CLUSTERS="asia-east1-b europe-west1-b us-east1-b us-central1-b"
```

```bash
for cluster in ${CLUSTERS}; do
  echo ""
  echo "${cluster}"
  echo "---------------"
  kubectl --context=${cluster} get pods
done
```

## Config Map

```bash
kubectl apply -f configmap.yaml
```

List all config maps in all clusters

```bash
CLUSTERS="asia-east1-b europe-west1-b us-east1-b us-central1-b"
```

```bash
for cluster in ${CLUSTERS}; do
  echo ""
  echo "${cluster}"
  echo "---------------"
  kubectl --context=${cluster} get configmaps
done
```

## Secret

```bash
echo global-secrets-are-awesome > secret.txt
```

```bash
kubectl create secret generic gcp-secret --from-file=password=./secret.txt
```

List `gcp-secret` secret in all clusters

```bash
CLUSTERS="asia-east1-b europe-west1-b us-east1-b us-central1-b"
```

```bash
for cluster in ${CLUSTERS}; do
  echo ""
  echo "${cluster}"
  echo "---------------"
  kubectl --context=${cluster} get secrets gcp-secret
done
```

## Replica Set

```bash
kubectl create -f replicaset.yaml
```

List replica set

```bash
kubectl get rs
```

Get pods in replica set

```bash
CLUSTERS="asia-east1-b europe-west1-b us-east1-b us-central1-b" 
```

```bash
for cluster in ${CLUSTERS}; do
  echo ""
  echo "${cluster}"
  echo "---------------"
  kubectl --context=${cluster} get pods
done
```

## Service

```bash
kubectl create -f service.yaml
```

List service

```bash
kubectl describe services gcp-meetup-global
```

List services in all clusters

```bash
for cluster in ${CLUSTERS}; do
  echo ""
  echo "${cluster}"
  echo "---------------"
  kubectl --context=${cluster} describe services nginx
done
```

## Ingress

```bash
kubectl create -f ingress.yaml
```

List ingress

```bash
kubectl describe ingress gcp-meetup-global
```

List ingresses in all clusters

```bash
for cluster in ${CLUSTERS}; do
  echo ""
  echo "${cluster}"
  echo "---------------"
  kubectl --context=${cluster} describe ingress gcp-meetup-global
done
```