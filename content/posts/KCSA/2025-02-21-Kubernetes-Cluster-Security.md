---
title: Kubernetes Cluster Security
date: 2025-02-21T00:00:00+02:00
type: post
authors:
  - Thomas de Jong
categories: 
  - Kubernetes
  - KCSA 
draft: false
---
<!--more-->
## Kubernetes API 
De Kubernetes API biedt de toegang tot alle functionaliteiten binnen het cluster, en is daarom het 1e wat je wilt beveiligen. Het eerste wat bepaald moet worden is hoe je authenticatie wilt afvangen, Kubernetes biedt hierin de volgende functionaliteiten:
- Username & Password
- Username & Token
- Certificate
- External Authentication - Using LDAP systems
- Service Accounts 

Wanneer authenticatie is afgehandeld is, moet authorizatie ingesteld worden, binnen Kubernetes zijn de volgende mogelijkheden: 
- Role Based Access Control - RBAC 
- Attribute Based Access Control - ABAC 
- Node Authorization
- Webhook Mode

Wat elke functionaliteit binnen het cluster doet en hoe dit ingesteld wordt, wordt later in deze post behandeld.

## CRI 
De container runtime interface wordt gebruikt om containers te draaien op het cluster, om dit te beveiligen zijn er een aantal maatregelen: 
- Updaten van je container runtime zoals docker of containerd
- Draai geen containers als root user (securityContext [runAsUser: 1000])
- readOnlyRootFilesystem via securityContext
- limiet plaatsen op resource usage van containers om zo DOS te voorkomen door resource starvation 

## Pods
### Pod Security Policy (PSP)
Een pod security policy is een admission controller, en kan controleren of een pod die aangemaakt wordt wel voldoet aan de gestelde PodSecurityPolicy.
```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata: pod-psp
spec:
  privileged: false
seLinux:
  rule: RunAsAny
supplementalGroups:
  rule: RunAsAny
runAsUser:
  rule: 'MustRunAsNonRoot'
requiredDropCapabilities:
- 'CAP_SYS_BOOT'
defaultAddCapabiltiies:
- 'CAP_SYS_TIME'
volumes:
- 'persistentVolumeClaim
```
Wanneer er een pod wordt aangemaakt wordt deze pod getoetst tegen bovenstaande yaml. In dit geval gaat het erom dat de securitycontext moet bevatten dat privileged mode op false staat, en de runAsUser moet gelijk aan 0 zijn. 

## ETCD
ETCD bevat de configuraties en states van het gehele cluster. Om ETCD te beveiligen is het belangrijk dat de data encrypted wordt op ETCD. dit kan door een EncryptionConfiguration object aan te maken. Ook is het belangrijk om regulier backups te maken van ETCD om zo downtime te voorkomen. 

## Container Networking 
Elke pod binnen een kubernetes cluster krijgt een eigen IP adres en is instaat om elke andere pod binnen een cluster te bereiken. Om dit te restricten kunnen network policies aangemaakt worden. De Network policies zorgen ervoor dat pods niet meer kunnen communiceren tenzij dit expliciet wordt gedefinieerd in de Ingress of Egress van de policy.  Zoals al eerder benoemd kunnen namespaces ook bijdragen aan de beveiliging van het netwerk. 

## KubeConfig 
De kubeconfig wordt gebruikt om meerdere kubernetes profiles op te slaan. Dit wordt gedaan door het systeem op te delen in: `Contexts`, `Clusters`, `Users`, waarbij de namen zichzelf uitleggen. 
```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/tdejong/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Fri, 07 Feb 2025 16:53:20 CET
        provider: minikube.sigs.k8s.io
        version: v1.35.0
      name: cluster_info
    server: https://192.168.49.2:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    extensions:
    - extension:
        last-update: Fri, 07 Feb 2025 16:53:20 CET
        provider: minikube.sigs.k8s.io
        version: v1.35.0
      name: context_info
    namespace: default
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /home/tdejong/.minikube/profiles/minikube/client.crt
    client-key: /home/tdejong/.minikube/profiles/minikube/client.key
``` 