---
title: Kubernetes Cluster Management
date: 2024-07-05T00:00:00+02:00
type: post
authors:
  - Thomas de Jong
categories: 
  - Kubernetes
  - CKA
draft: false
---
In deze post worden de volgende onderdelen behandeld: de management tools voor Kubernetes (zoals Kubectl, Kubeadm, Minikube, Helm, Kompose en Kustomize), het drainen van nodes voor onderhoud, het upgraden van nodes met Kubeadm en het backuppen van etcd.
<!--more-->
## Management tools
K8s management tools voegen extra functionaliteiten toe aan Kubernetes.
- Kubectl is de officiele CLI voor Kubernetes. 
- Kubeadm is de tool om makkelijk en snel K8s clusters op te zetten
- Minikube is een tool om lokaal single-node Kubernetes clusters op te zetten
- Helm is een tool die template en package management biedt voor Kubernetes.
- Kompose is een tool die gebruikt kan worden om Docker-compose om te zetten naar k8s
- Kustomize is een tool die K8s object configuraties kan beheren. 

## Draining
Wanneer er onderhoud gepleegd moet worden aan een cluste,r kan het nodig zijn om een node te verwijderen uit de service. Om dit te doen kan er gebruik gemaakt worden van drain. Draining verplaatst waar mogelijk resources naar een nieuwe node, om zo ervoor te zorgen dat er geen downtime plaats vindt.

Een pod drainen kan met de volgende commando: 
```bash
kubectl drain <node>
```
Echter kan dat soms niet werken omdat er DaemonSet pods werken op een node, die gelinkt zijn aan deze node. Om dit te bypassen is de volgende commando beschikbaar:
```bash
kubectl drain <node> --ignore-daemonsets
```
Zodra het onderhoud aan een node is geweest, en deze node onderdeel blijft van het cluster, kan deze weer beschikbaar gesteld worden voor pods om hierop te draaien met het volgende commando: 
```bash
kubectl uncordon <node>
```

## Upgrading
Om een Kubernetes node te upgraden kan Kubeadm gebruikt worden, hiervoor moet de node wel eerst gedrained zijn.  De volgende stappen worden doorlopen om een node te upgraden:
### Control plane node
- Upgrade Kubeadm
- Drain
- Plan de upgrade 
```bash
kubeadm upgrade plan
```
- pas de upgrade toe
```bash
kubeadm upgrade apply
```
- upgrade kubelet en kubectl
- uncordon

## Backing up
etcd is de backend data storage solution voor een Kubernetes cluster. Dit houdt ook in dat alle objecten, applicaties en conifguraties hier in zijn opgeslagen. Daarom is het wenselijk om hier periodiek een backup van te maken.

Om een backup te maken van etcd kan etcdctl gebruikt worden, in dit geval het volgende commando:
```bash
ETCDCTL_API=3 etcdctl --endpoints $ENDPOINT snapshot save <file name> 
```
het is natuurlijk ook belangrijk om te weten hoe je de backup weer inlaad, hiervoor kan het volgende commando worden gebruikt:
```bash
ETCDCTL_API=3 etcdtl snapshot restore <file name> 
```