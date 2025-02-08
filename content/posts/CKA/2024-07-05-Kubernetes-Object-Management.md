---
title: Kubernetes Object Management
date: 2024-07-05T00:00:00+02:00
type: post
authors:
  - Thomas de Jong
categories: 
  - Kubernetes
  - CKA
draft: false
---
In deze post worden de volgende onderdelen behandeld: hoe kubectl gebruikt kan worden om Kubernetes objecten op te halen, aan te maken, aan te passen, te verwijderen en commando's binnen containers uit te voeren; het beheer van toegangsrechten via RBAC, waarbij zowel roles als cluster roles en de bijbehorende bindings aan bod komen; het aanmaken en inzetten van service accounts voor authenticatie bij de Kubernetes API; en het monitoren van resource usage met behulp van de Kubernetes Metrics Server.
<!--more-->
## Commando's
Kubectl is een CLI tool die gebruik maakt van de K8S API om te communiceren met een Kubernetes cluster.
### GET
```bash
Kubectl get <object type> <object name> -o <output> --sort-by <JSONPath> --selector <selector> 
```
- -O kan gebruikt worden om de output format te bepalen, dit kan bijvoorbeeld JSON of YAML zijn.
- --sort-by is in staat om op basis van een JSONPath de get command te sorteren op bijv grootte.
- --Selector  kan gebruikt worden om te filteren op een specifiek label
### CREATE 
Kubectl create kan worden gebruikt om objecten zoals een deployment of een pod aan te maken. Dit wordt gedaan op basis van een YAML file, die wordt aangegeven met de -f flag. Wanneer kubctl create gebruikt wordt op een al bestaand object, komt er een error uit.
```bash
kubectl create -f <file name>
```
### APPLY
kubectl apply wordt gebruikt om een al bestaand object aan te passen, ook is deze in staat om net zoals create een object aan te maken. 
```bash
kubectl apply -f <file name> 
```
### DELETE 
kubectl delete wordt gebruikt om objecten te verwijderen uit een cluster.
```bash
kubectl delete <object type> <object name> 
```

### EXEC 
kubectl EXEC kan gebruikt worden om commando's uit te voeren in een container, en is dus erg handig voor trouble shooting. 
```bash
kubectl exec <pod name> -c container name> -- <command>
```

## RBAC
Role Based Access Control (RBAC)  kan gebruikt worden om de toegang van gebruikers in te stellen binnen een cluster. Hierbij wordt onderscheid gemaakt tussen Roles en ClusterRoles.
### Roles
Een Role definieerd de permissies van een gebruiker binnen een namespace
### ClusterRole
Een ClusterRole beheert zoals de naam al zegt, de permissies van een gebruiker binnen het cluster.

Ook kan er gebruik gemaakt worden van RoleBinding en ClusterRoleBinding. Deze worden gebruikt om een user aan de bijbehorende Role toe te voegen. Zowel Roles, ClusterRoles, RoleBinding en ClusterRoleBindings worden aangemaakt op basis van een YAML file:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-secrets-global
subjects:
- kind: Group
  name: manager
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```
## Service Accounts
een serviceaccount is een account dat gebruikt kan worden door containers binnen pods om zich te authenticeren bij de Kubernetes API. een Service Account kan gemaakt worden met de volgende YAML: 
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  annotations:
    kubernetes.io/enforce-mountable-secrets: "true"
  name: my-serviceaccount
  namespace: my-namespace
```
Een service account kan gekoppeld worden met RoleBindings of ClusterRoleBindings. 
```yaml
subject:
- kind: ServiceAccount
  name: my-serviceaccount
  namespace: default
```

## Resource Usage
Om metrics betreffende resources van pods en containers te bekijken, zal er een Metrics Server gebruikt moeten worden, bijvoorbeeld de Kubernetes Metrics Server.

Wanneer een Kubernetes Cluster beschikt over de Kubernetes Metrics Server kan kubectl top gebruikt worden om data betreffende resource gebruik bekeken worden.
```bash
kubectl top pod --sort-by <JSONPATH> --selector <selector>
```