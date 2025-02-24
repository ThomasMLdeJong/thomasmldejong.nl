---
title: Kubernetes Threat Model
date: 2025-02-24T00:00:00+02:00
type: post
authors:
  - Thomas de Jong
categories: 
  - Kubernetes
  - KCSA 
draft: false
---
<!--more--->
Threat modelling kan gebruikt worden om mogelijke risico's binnen een systeem en/of infrastructuur te vinden, de impact hiervan in te schatten, en op basis hiervan maatregelen te nemen. 

## Trust Boundaries
Trust boundaries kunnen worden gedefinieerd om ervoor te zorgen dat een breach op 1 component van de applicatie, niet de gehele applicatie compromised maakt.

### Cluster Trust Boundaries
Trust boundaries kunnen al beginnen bij het cluster, je kan er namelijk voor kiezen om gebruik te maken van apparte clusters voor elke fase van een applicatie (development, staging en productie). Hierdoor wordt voorkomen dat een kwetsbaarheid in bijvoorbeeld het development cluster, impact heeft op het productie cluster.

### Node Trust Boundaries
Elke node binnen een cluster vormt zijn eigen trust boundary, gezien het niet zo hoort te zijn dat wanneer 1 node binnen het cluster gecompromiteerd raakt, andere nodes dit ook raken. Hierom is het belangrijk dat er gebruik gemaakt van Network policies en Pod Security Standards om verkeer tussen nodes te beperken.

### Namespace Trust Boundaries
Elk cluster bevat verschillende namespaces om applicaties, maar ook teams te onderscheiden. Hierin dient Role Based Access Control (RBAC) toegepast te worden om toegang tot resources binnen een namespace te beperken. Ook dienen ResourceQuotas ingesteld te worden, om overmatig resourcegebruik te voorkomen. 

### Pod Trust Boundary 
Pods vormen de kleinste trust boundary binnen een cluster. Om de trust boundary van een pod goed af te kaderen is het belangrijk dat er gebruik gemaakt wordt van Serice Accounts, om zo het least privillege principe te kunnen volgen. Ook is het belangrijk dat de communicatie mogelijkheden van pods worden beperkt met network policies. Ten slotte kunnen Security Contexts en de Pod Security Admissions geconfigureerd worden om dit nog beter te kunnen controleren. 

## Persistence
Persistence is de mogelijkheid van een aanvaller om toegang te behouden tot een systeem, zelfs na het herstarten of patchen van een systeem. Om persistence risico's tegen te gaan kunnen een aantal maatregelen genomen in Kubernetes, namelijk het gebruik van RBAC, waarbij deze zo strak mogelijk wordt afgesteld; het gebruik maken van Secrets waarin alleen de pods die de secret nodig hebben hierbij kunnen; en Pod Hardening, door bijvoorbeeld geen privileged containers te gebruiken, en read-only root filesystems te forceren op pods. Ook is het belangrijk om regulier updates uit te voeren op containers, en patches uit te voeren wanneer dit nodig is, als laatste is het belangrijk dat er een goed monitoring en logging systeem wordt opgezet, dit kan met tools zoals POrometheus en Loki. 
