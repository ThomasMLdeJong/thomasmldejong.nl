---
title: Kubernetes Services
date: 2024-07-05T00:00:00+02:00
type: post
authors:
  - Thomas de Jong
categories: 
  - Kubernetes
  - CKA
draft: false
---
In deze post wordt beschreven hoe een service in Kubernetes applicaties kan exposen op basis van een verzameling pods, waarbij de service fungeert als een interne load-balancer. Verder wordt toegelicht hoe DNS-namen het mogelijk maken om services eenvoudig te benaderen binnen het cluster, en hoe Ingress objecten externe toegang beheren met extra functionaliteiten zoals SSL, load balancing en name-based virtual hosting.
<!--more-->
Een service biedt de mogelijk om een applicatie te exposed op basis van een set aan pods. Een service is eigenlijk een soort interne load-balancer.

Een service zal altijd routen richting een endpoint, elke pod beschikt automatisch over een endpoint die richt op de service.
## Service Types
Er zijn verschillende soorten services.
- ClusterIP - Exposed een applicatie binnen het cluster.
- NodePort - exposed een applicatie naar buiten het cluster.
- LoadBalancer - Exposed een applicatie naar buiten het cluster, op basis van een cloud load balancer, de cloud provider zal dus wel load-balancing moeten ondersteunen.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    run: my-nginx
```
## DNS 
net zoals pods beschikt  een service ook over een DNS naam, deze wordt als volgt geformateerd:
```yaml
service-name.namespace.svc.cluster.local
```
de DNS kan vanuit elke namespace bereikt worden, ongeacht waar deze staat. Binnen een namespace kan een pod een service bereiken op basis van alleen de service naam.

## Ingress
Een ingress is een kubernetes object dat  externe toegang tot services beheert. Een ingress bevat meer functionaliteiten dan een NodePort, zoals SSL, LoadBalancing en name-based virtual hosting.

Een Ingress doet van zichzelf niets, om ze functionaliteit te geven moeten ze een ingress controller hebben.
```yaml
apiVersion: networking.k8s.io/v1 
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
 - http:
   paths:
   - path: /somepath
     pathType: Prefix
     backend:
       service:
       name: my-service
       port:
         name: web
```