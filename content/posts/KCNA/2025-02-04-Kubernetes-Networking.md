---
title: Kubernetes Networking
date: 2025-02-04T00:00:00+02:00
type: post
authors:
  - Thomas de Jong
categories: 
  - Kubernetes
  - KCNA 
draft: false
---
Deze post bespreekt de netwerkarchitectuur van Kubernetes en hoe communicatie binnen en buiten het cluster wordt geregeld. Er wordt aandacht besteed aan de standaardpoorten voor een cluster, de Container Network Interface (CNI), en de werking van DNS binnen Kubernetes.
<!--more-->

## Cluster Networking
De master node heeft de volgende poorten open:
| Poort     | Functie                                                                               |  
| --------- | ------------------------------------------------------------------------------------- |  
| 6443      | Communicatie van de worker nodes met de master node via de Kube-API                   |  
| 10250     | Poort voor de Kubelet                                                                 |  
| 10259     | Poort voor communicatie van de worker nodes met de Kube-Scheduler                     |  
| 10257     | Poort voor de Kube Controller Manager                                                 |  
| 2379      | Poort voor de etcd-server, die de clusterstatus opslaat                               |  

De worker node heeft de volgende poorten open:  

| Poort             | Functie                                                                                 |  
| ----------------- | --------------------------------------------------------------------------------------- |  
| 30000 - 32767     | Dynamisch toegewezen poorten voor applicaties die draaien op de worker node             |  
| 10250             | Poort voor de Kubelet                                                                   |  

## Pod Networking
Binnen een Kubernetes-cluster beschikt elke Pod over een eigen IP-adres. Standaard kan elke Pod direct communiceren met elke andere Pod binnen het cluster, zonder gebruik te maken van NAT. Dit is mogelijk door het netwerkmodel van Kubernetes. Kubernetes implementeerd het netwerkmodel via een Container Network Interface (CNI), een aantal hiervan zijn Calico, Flannel, Cilium en Weave. 

## Container Network Interface
De CNI is een plogun waarmee Kubernetes zijn networkfunctionaliteit kan beheren. De CNI regelt hoe containers zowel intern als extern met netwerken communiceren. Elke CNI ondersteunt andere functionaliteiten zoals overlay-netwerken, netwerkbeveiliging en load balancing. 

De CNI kan geconfigureerd worden door in de Kubelet service de volgende drie flags aan te passen: 
- `--network-plugin=cni` - Zet het gebruik van de CNI aan
- `--cni-bin-dir=/opt/cni/bin` - De locatie van de CNI binaries
- `--cni-conf-dir=/etc/cni/net.d` - De locatie van de CNI-config

## DNS 
De Domain Name System (DNS) wordt gebruikt om services aan elkaar te koppelen binnen het cluster. Kubernetes gebruikt een eigen DNS-service om pods en services automatisch een DNS naam te geven, zodat ze elkaar kunnen vinden zonder een IP-adres te gebruiken, dit helpt wanneer er met een microservice architectuur gewerkt wordt en een pod overnieuw opstart met een nieuw IP adres. 

Elke pod/service krijgt een DNS naam op basis van het volgende template: 
```txt 
<servicenaam>.<namespace>.<pod/service>.cluster.local
```

De DNS wordt in Kubernetes geregeld door CoreDNS, die automatisch de DNS records beheert. 

## Ingress
Een Kubernetes Cluster maakt gebruik van een Ingress om externe toegang tot interne services te leveren. Een Ingress biedt routing mogelijkheden voor zowel HTTP als HTTPS verkeer, op basis van regels zoals hostnames en paths die op hun beurt naar de juiste service wijzen. Een Ingress is in staat om SSL/TLS uit te voeren, waardoor de Ingress encryptie kan uitvoeren, en de achterliggende services alleen HTTP hoeven te verwerken. 

Om een Ingress te gebruiken is er een Ingress Controller nodig. De controller leest de regels van het Ingress object en configureert de Ingress naar behoren. Twee Ingress controllers zijn `Ingress-nginx` en `Traefik`. 
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata: 
  name: thomasmldejong-ingress
spec:
  rules:
  - host: thomasmldejong.nl 
    http:
      paths:
      - path: / 
        pathType: Prefix
        backend:
          service:
            name: webserver-service
            port: 
              number: 80
```
Bovenstaande manifestlaat een ingress zien voor thomasmldejong.nl, in dit manifest wordt al het inkomende verkeer "/" doorgestuurd naar de webserver-service.