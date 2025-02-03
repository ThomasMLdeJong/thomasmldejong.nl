---
title: Kubernetes Orchestration Security
date: 2025-02-03T00:00:00+02:00
type: post
authors:
  - Thomas de Jong
categories: 
  - Kubernetes
  - KCNA 
draft: false
---

<!--more-->
## Basic Security
Security van een Kuberneters cluster begint bij de Nodes waar het cluster op draait. Denk hierbij aan passwordless authentication, opzetten van een zo strak mogelijk afgestelde firewall en dergelijken. Als een node compromised raakt, is het cluster namelijk compromised, en dat wil je altijd voorkomen

Ook is het belangrijk om te bepalen wie bij het cluster mag, en wat ze mogen doen, om zo de KubeAPI te beschermen. 
Wie er toegang kan krijgen tot een cluster kan worden geregeld met de volgende resources:
- Gebruikersnaam en Wachtwoord
- Gebruikersnaam en Tokens
- Certificaten 
- LDAP 
- Service Account

Wat ze kunnen doen wordt gedefinieerd via: 
- RBAC 
- ABAC
- Node Authorization
- Webhook modes

## TLS 
Om een Certificate Authority (CA) voor Kubernetes op te zetten, begin je met het genereren van een private key: 
```bash 
openssl genrsa -out ca.key 2048
```
vervolgens maak je een Certificate Signing Request (CSR): 
```bash
openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA"
```
tenslote teken je het certificaat: 
```bash
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```
De CA vormt vervolgens de basis voor de beveiliging met TLS in je cluster, en is belangrijk voor authenticatie van gebruikers in het cluster. Admins en Users ontvangen certificaten van de CA, waarmee ze de rechten en toegang tot de Kubernetes API ontvangen. Tijdens het authenticatieproces gebruikt de API server het certificaat van de gebruiker en controleert of deze is ondertekend door de CA. 
## Kubeconfig 
Kubeconfig gebruikt het kubeconfig bestand in ~/.kube/config om te bepalen welk cluster er gebruikt moet worden. Dit bestand bestaat ui drie onderdelen: Clusters, Users en Contexts.
### Clusters 
In de clusters worden de kubernetes clusters gedefinieerd zoals dfevelopment, staging en productie. Voor elk cluster is een naam, URL en certificaat nodig, zodat kubectl weet waar de verbinding gezogd kan worden
### Users
De Users bevat de authenticatiegegevens voor de gebruikers die toegang hebbebn tot de clusters. Hierin worden credentails opgeslagen, zoals tokens, gebruikersnaam/wachtwoorden of client-certificaten met bijbehorende sleutels. Met deze informatie kan de authenticatie uitgevoerd worden op het cluster
### Contexts
Een context is de samenvoeging van de clusters en users. De context bepaalt welke gebruiker toegang krijgt tot welk cluster. Bijvoorbeeld, de context `Thomas@Development` geeft aan kubectl door dat de user Thomas verbinding moet maken met het development-cluster. 
```yaml
apiVersion: v1
kind: Config
clusters:
  - name: Development
    cluster:
      server: https://development.thomasmldejong.nl
      certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0t...
  - name: Staging
    cluster:
      server: https://staging.thomasmldejong.nl
      certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0t...
  - name: Production
    cluster:
      server: https://production.thomasmldejong.nl
      certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0t...
contexts:
  - name: Thomas@Development
    context:
      cluster: Development
      user: Thomas
      namespace: default
  - name: Remco@Production
    context:
      cluster: Production
      user: Remco
      namespace: default
  - name: Jasper@Staging
    context:
      cluster: Staging
      user: Jasper
      namespace: default
current-context: Thomas@Development
users:
  - name: Thomas
    user:
      token: "abc123def456"
  - name: Remco
    user:
      token: "def456ghi789"
  - name: Jasper
    user:
      token: "ghi789jkl012"
```
## API group
De Kubernetes API is een REST API, die bereikbaar is via de URL `https://<masternode>:6443`. De API bestaat uit verschillende paden, met elk hun eigen doel.
- **/:** De root PAI en levert een overzicht van de API-versies en discovery-informatie.
- **/metrics:** Bevat  metrics over performance en resource gebruik. monitoring tools zoals Prometheus maken hier vaak gebruik van om gegevens over de API-server te verzamelen
- **/healthz:** Dit endpoint geeft de status van de API-server, dit is zo simpel als een 'ok'
- **/version:** Geeft inforamtie over de versie van de API-server.
- **/api:** Legacy gedeelte van de kubernetes API en bevat resources zoals namespaces, pods, services, secrets en events. 
- **/apis:** Bevat de named API-groepen, waarbij elke set weer gerelateerde resources bevat. de API groepen zijn apps, batch, networking.k8s.io en nog een aantal anderen

## Authorizatie
In Kubernetes bestaand verschillende manieren om te bepalen welke rechten een gebruiker of component heeft binnen het cluster. De drie belangrijkste methodes worden in dit kopje behandeld.=
### Node Authorizer
Deze methode beperct de acties van kubelets tot alleen het noodzakelijke. Kubelets mogen bijvoorbeeld alleen lezen en schrijven aan hun eigen node-objecten en de pods die op hun node draaien.
### ABAC
ABAC verleent toegang op basis van vooraf gedefinieerde regels die dingen zoals gebruikersnaam, groep, resource type en namespaces kunnen bevatten. Deze regels worden vastgelegd in JSON format.
### Webhook
Bij webhook authenticatie stuurt Kubernetes autorisatieverzoeken naar een externe service. De externe service beslist of een actie is toegestaan, wat dynamisch en centraal beheer van autorisatieregels mogelijk maakt.

## RBAC
Role Based Access Control (RBAC) in Kubernetes maakt het mogelijk om rechten binnen het cluster te beheren door rollen te definiëren en deze aan gebruikers of groepen toe te wijzen. Hierdoor kunnen de rechten van bijvoorbeeld developers, admins en manager gescheiden worden.

### Roles
Een rol wordt aangemaakt met een lijst aan rechten binnen een namespace. In het manifest hieronder is de rol developer te zien, dit in dit geval de mogelijk heeft tot het beheren van pods en deployments.
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "get", "create", "update", "delete"]
- apiGroups: ["apps"] 
  resources: ["deployments"]
  verbs: ["list", "get", "create", "update", "delete"]
```
### RoleBinding
Om een rol aan een gebruiker te koppelen, is een RoleBinding nodig. Hieronder is een manifest te zien die de rol developer linkt aan de gebruiker Thomas.
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Rolebinding
metadata: 
  name: thomas-developer-binding
subjects:
- kind: User
  name: Thomas
  apiGroup: rbac.authirzation.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

## ClusterRole
Een role en rolebinding zijn namespace specifiek, maar al wil je rechten aan een user toevoegen voor over het gehele cluster, bestaan er ClusterRoles en ClusterRoleBindings. Deze werken met hezelfde principe als roles, maar zijn niet gebonden aan een namespace.
### ClusterRole
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list", "get", "create", "delete"]
```

### ClusterRoleBinding
```yaml
apiVersion: rbac.authorization.k8s.io/v1`
kind: ClusterRole
metadata: 
  name: thomas-developer-cluster-role-binding
subjects:
- kind: User
  name: Thomas
  apiGroup: rbac.authoirzation.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```
bovenstaande manifests maken een clusterrole aan voor een clusteradmin, en linken deze clusterrole aan de user Thomas.

## ServiceAccounts
Een ServiceAccount is een account gemaakt voor bijvoorbeeld applicaties om met de KubeAPI-server te communiceren.
Wanneer er een namespace wordt aangemaakt wordt er automatisch een `default` serviceaccount aangemaakt in deze namespace, aan dit serviceaccount worden alle pods gewezen die niet beschikken over een eigen serviceaccount name. De rechten van een serviceaccount worden beheerd via (Cluster)Roles en de bijbehorende bindings.
```yaml
apiVersion: v1 
kind: ServiceAccount
metadata: 
  name: nginx-serviceaccount
  namespace: webserver
```
Om een service accoutn aan een pod toe te voegen, kan de `serviceAccountName` geconfigureerd worden in het manifest.
```yaml
apiVersion: v1 
kind: Pod
metadata:
  name: nginx
spec:
  serviceAccountName: nginx-serviceaccount
  containers:
  - name: nginx-container
    image: nginx:latest
```

## Network Policies
Network policies worden gebruik om netwerkverkeer tuseen pods en dergelijken te beheren. Een Network policie kan gedefinieerd worden welk inkomend en uitgaand verkeer wel en niet mag. Standaard zijn er geen network policies gedefinieerd en kan een pod dus communiceren met alles binnen het cluster, zodra een policy wordt aangemaakt kan alleen de communicatie nog die wordt toegestaan, een network policy werkt dus met een default deny.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
  namespace: webserver
spec:
  podSelector:
    matchLabels:
      app: nginx
      type: frontend
  policyTypes:
  - Ingress
  ingress:
  - from: 
    - podSelector:
       matchLabels:
         app: nginx
         type: frontend
    ports:
    - protocol: TCP 
      port: 80