# Kubernetes Cluster Federation - Google Cloud Platform Meetup Vancouver

This tutorial assumes that you have Kubernetes cluster federation already setup using Kelsey Hightower's guide [Kubernetes Cluster Federation (The Hard Way)](https://github.com/kelseyhightower/kubernetes-cluster-federation).

In this tutorial we will create the following:

* A federated replica set
* A federated service
* A federated ingress

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

## Replica Set

```bash
kubectl create -f replica-set.yaml
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
kubectl apply -f service.yaml
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
  kubectl --context=${cluster} describe services gcp-meetup-global
done
```

## Ingress

```bash
gcloud compute addresses create gcp-meetup-global-ingress --global
```

```bash
gcloud compute firewall-rules create \
  federated-ingress-firewall-rule \
  --source-ranges 130.211.0.0/22 \
  --allow tcp:30015 --network default
```

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

## Let's generate some traffic

```bash
gcloud compute ssh gcp-meetup-asia-east1 --zone=asia-east1-b

gcloud compute ssh gcp-meetup-europe-west1 --zone=europe-west1-b

gcloud compute ssh gcp-meetup-us-west1 --zone=us-west1-b

gcloud compute ssh  gcp-meetup-us-east1 --zone=us-east1-b
```

```bash
ab -n 100000 -c 10 http://35.201.87.58/
```

## Scale up

Change replica count to `4`

```bash
kubectl scale rs gcp-meetup-global --replicas=4
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