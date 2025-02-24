---
title: Kubernetes Security Fundamentals 
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
## Pod Security Admission
Een Pod Security Admission (PSA) is geimplementeerd met als doel om security configuratie simpeler te maken. Met deze reden komt de PSA met drie default profiles, namelijk pivileged, baseline en restricted. Hierbij kan er ook een admission mode op worden toegepast, namelijk  enfore, audit of warn. De Pod Security Admission kan worden toegewezen aan namespaces dmv een label

## Authentication
Standaard is Kubernetes niet in staat om zelf gebruikers te beheren, het gaat uit van bestanden met gebruikers, of LDAP. Hierdoor is het niet mogelijk om bijvoorbeeld `kubectl create user user1` uit te voeren. De toegang tot het cluster wordt ten aller tijden beheerd via de kube-apiserver, en is dus verantwoordelijk voor authenticatie en processing. De authenticatie kan doormiddel van een wachtwoord, token, certificaat of LDAP. 

Het bestand waar de users met bijbehorende wachtwoord in staan wordt aangegeven in het manifest van de kube-apiserver met de flag `--basic-auth-file`. Wanneer het om een token gaat is het `--token-auth-file`.

## Authorization
Kubernetes ondersteunt vier vormen van authorisatie, namelijk: Node, ABAC, RBAC en Webhook

### Node
Node authorisatie wordt gebruikt voor de Kubelet, en zorgt ervoor dat de Kubelet kan Lezen en schrijven naar de Kube-apiserver

### Attribute Based Access Control (ABAC)
Bij Attribute Based Access Control krijgt elke individuele gebruiker of groepen permissies toegewezen, dit zorgt ervoor dat authorisatie op basis van ABAC op grotere schaal lastig te beheren wordt. 

### Role Based Access Control (RBAC)
Role based access control zorgt ervoor dat je inpalats van authorisatie binden aan groepen of gebruikers, authorisatie kan toepassen op een rol. Op deze manier is het mogelijk om permissies toe te kennen aan een grote groep gebruikers, op een beter te beheren manier.

RBAC wordt gebasseerd op twee componenten, namelijk een role en een rolebinding: 
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "get", "create", "update", "delete"]
- apiGroups: [""]
  resources: ["ConfigMap"]
  verbs: ["create"]
```
bovenstaande role zorgt ervoor dat elke user met de rol developer in staat is om:
- Alle pods op te vragen, lezen, aanmaken, aanpassen en te verwijderen
- Een Configmap kan aanmaken

De role moet dan nog gebonden worden aan een user, dit kan met een rolebinding: 
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: thomas-developer-binding 
subjects:
- kind: User
  name: Thomas
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorizatin.k8s.io
```
bovenstaande rolebinding linkt de role developer aan de user Thomas.

Vervolgens kan er gecontroleerd worden of het mogelijk is om bijvoorbeeld een ConfigMap aan te maken, dit kan met het commando `kubectl auth can-i create configmaps`

### Webhook
Webhook werkt met externe applicaties zoals Open Policy Agent (OPA), waarbij de communicatie met de Kubernetes API wordt doorgezet naar OPA om te controleren of de user in de request de juist permissies heeft.

## Segmentation & Isolation
Segmentatie binnen het Kubernetes kan worden gerealiseerd d.m.v. namespaces. Door een Namespace te gebruiken kan een admin resource limits toepassen per namespace, om zo het cluster tolleranter te maken voor demand fluctuations. Door segmentatie is het niet meer mogelijk om een service in een ander namespace (dev) aan te spreken met "mysql", maar zal dit moeten met "mysql.dev.svc.cluster.local".

Isolatie in Kubernetes wordt voor een stuk al gedaan door namespaces toe te passen, maar om dit helemaal goed in te richten kan er bovenop nog gebruik gemaakt worden van network policies. Deze zorgen ervoor dat een pod alleen nog mag communiceren met pods die worden gedefinieerd in de network policy.

Het wisselen tussen namespaces kan met het volgende commando: `kubectl config set-context $(kubectl config current-context) --namespace=<Namespace>`
