# VLLM Distributed Inferencing and Serving on GPU cluser using Kubernetes

This repository provides how to implement vllm distributed inferencing and serving on gpu cluster using kubernetes.

## Install Microk8s

```
sudo snap install microk8s --classic
sudo usermod -aG microk8s $USER
newgrp microk8s
microk8s start
```

## Enable Essenstial Services in Microk8s

```
sudo microk8s enable dashboard cert-manager hostpath-storage ingress metrics-server nvidia
```

## Setup Kubernetes Cluster

```
microk8s add-node
```

Then you can get the "microk8s join" command and you can run it on kubernetes worker node

After that, you need to deploy pod network for communicating between containers.

```
microk8s kubectl apply -f kubernetes/calico.yaml
```

## Deploy VLLM Containers on Kubernetes Cluster

First set the role of kubernetes node

```
microk8s kubectl label nodes ip-10-0-0-29 role=master
microk8s kubectl label nodes ip-10-0-0-6 role=worker-1
```

Then deploy vllm containers on kubernetes cluster

```
microk8s kubectl apply -f kubernetes/namespace.yaml
microk8s kubectl apply -f kubernetes/head-node.yaml
microk8s kubectl apply -f kubernetes/head-service.yaml
microk8s kubectl apply -f kubernetes/worker-node.yaml
microk8s kubectl apply -f kubernetes/vllm-service.yaml
```

This deployment includes setting up Ray cluster for distributed inferencing

## Run VLLM code in container

Check the current running pods in vllm cluster namespace

```
microk8s kubectl get pods -n vllm-cluster
```

Access file system of vllm container if 2 pods are running

```
microk8s kubectl exec -it vllm-head-<xxxxxx> -n vllm-cluster -- /bin/bash
```

Then you can run distributed_batch_inference.py in container

```
python3 distributed_batch_inference.py
```
