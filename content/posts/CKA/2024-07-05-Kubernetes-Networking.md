---
title: Kubernetes Networking
date: 2024-07-05T00:00:00+02:00
type: post
authors:
  - Thomas de Jong
categories: 
  - Kubernetes
  - CKA
draft: false
---
In deze post worden de volgende onderdelen behandeld: hoe het netwerkmodel van Kubernetes de communicatie tussen pods regelt door hen ieder een uniek IP-adres toe te kennen, de rol van CNI plugins in het realiseren van de benodigde netwerkconnectiviteit, de inzet van DNS voor het vereenvoudigen van pod-discovery via domeinnamen, en het gebruik van NetworkPolicies om via pod selectors het verkeer tussen pods te isoleren en te beheren.
<!--more-->
Het netwerk model van Kubernetes definieerd hoe pods met elkaar communiceren, ongeacht  van welke node ze op draaien. Om dit waar te maken draait elke pod met een eigen uniek IP adres binnen het cluster. Dit zorgt er voor dat elke pod elke ander pod kan berijken via het virtuele netwerk. 

## CNI plugin
een CNI plugin is een netwerk plugin. Deze bieden netwerk connectivity tussen pods op basis van de standaard vastgesteld door een netwerk model. 

## DNS 
het netwerk van Kubernetes maakt gebruik van een DNS, om zo pods in staat te stellen om andere pods te vinden via een domein in plaats van een IP adres.  De DNS draait van zichzelf in de kube-system namespace.

De domein namen zijn als volgt geformateerd: 
```
pod-ipv4-address.my-namespace.pod.cluster-domain.example.
```

## NetworkPolicies
### PodSelector
Een pod selector bepaalt op welke pod in een namespace de NetworkPolicy in gaat. Dit wordt gedaan op basis van labels.
```yaml
spec:
  podSelector:
    matchLabels:
      role: db
```
Pods zijn van zichzelf non-isolated, todat een network policy wordt toegepast. 