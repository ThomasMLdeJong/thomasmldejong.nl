---
title: Kubernetes Resources
date: 2025-02-01T00:00:00+02:00
type: post
authors:
  - Thomas de Jong
categories: 
  - Kubernetes
  - KCNA 
draft: false
---

Deze post biedt een overzicht van de kerncomponenten binnen Kubernetes, waaronder YAML, Pods, Replication Controller, ReplicaSets, Deployments, Strategiën en imperative & declarative resourcebeheer.
<!--more-->
## YAML
YAML is de template language die Kubernetes gebruikt om resources aan te maken. Hieronder staat een voorbeeld van een YAML-manifest voor het aanmaken van een Pod:
```yaml
apiVersion: v1 
kind: pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers: 
    - name: nginx-container
      image: nginx:latest
```
Een belangrijk aspect van bovenstaande manifest is de apiVersion, er bestaan namelijk meerdere api's. Kubernetes maakt gebruik van meerdere api versies om ervoor te zorgen dat functionaliteiten los van elkaar kunnen worden doorontwikkeld. 
| Kind                  | API Version | 
| :-------------------- | ----------- |
| Pod                   | v1          |
| Service               | v1          |
| ReplicationController | v1          |
| Replicaset            | apps/v1     | 
| Deployment            | apps/v1     | 
Wanneer een manifest klaar is kan deze worden toegepast op twee manieren
```bash
kubectl apply -f nginx-pod.yaml
```
Het apply commando kan een resource bijwerken als deze al bestaat, of een nieuwe resource aanmaken als deze nog niet bestaat. Daarentegen zorgt het create commando er voor dat een resource wordt aangemaakt, en geeft het een foutmelding als de resource al bestaat
```bash 
kubectl create -f nginx-pod.yaml
```

## Pods 
Een Pod is een instantie van een applicatie en vormt het kleinste object dat in Kubernetes kan worden aangemaakt. Pods worden ingezet om applicaties in containers te draaien. Wanneer een nieuwe instantie van een applicatie nodig is, wordt er niet simpelweg een extra exemplaar binnen een bestaande Pod gestart, maar wordt er een geheel nieuwe Pod aangemaakt, hetzij op dezelfde node, hetzij op een andere. Bovendien kan een Pod meerdere containers bevatten. Deze containers kunnen identiek zijn, maar vaak bevat een Pod ook een andere applicatie, bijvoorbeeld in de vorm van een sidecar-container die specifieke taken, zoals het verwerken van data, uitvoert.


## Replication Controller
Wanneer een pod crasht, krijgen gebruikers geen toegang meer tot de applicatie. Om dit te voorkomen, is het wenselijk om meerdere instanties van dezelfde pod te draaien. De Replication Controller zorgt ervoor dat er altijd het gewenste aantal pods actief is in het cluster. Daarnaast draagt deze controller bij aan load balancing, zodat de applicatie zich kan aanpassen aan wisselende gebruikersaantallen.

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-rc
  labels:
    app: nginx
    type: front-end
spec:
  replicas: 3
  template:
    metadata:
      name: nginx-pod
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
```

## ReplicaSet
Een ReplicaSet is de opvolger van de Replication Controller en biedt vergelijkbare functionaliteiten, maar met extra flexibiliteit. Zo ondersteunt een ReplicaSet set-gebaseerde selectors, wat meer mogelijkheden biedt bij het specificeren van de pods die beheerd moeten worden. Een ReplicaSet is in staat om ook pods te beheren die niet aangemaakt zijn door de ReplicaSet, zolang ze maar over de juiste selector beschikken. Hoewel de ReplicaSet tegenwoordig de aanbevolen methode is voor het beheren van pods in Kubernetes, blijft de Replication Controller nog steeds een valide manier van replicatie.
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
  labels:
    app: nginx
    type: front-end
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
      type: front-end
  template:
    metadata:
      labels:
        app: nginx
        type: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
```

## Deployments
Een deployment is een decleratieve manier om applicaties te beheren en te updaten. Met een Deployment definieer je de staat van de applicatie, waarna Kubernetes automatisch de benodigde acties uitvoert om de staat te realiseren. Dit omvat het aanmaken van nieuwe pods, het vervangen van bestaande pods en het terugdraaien van wijzigingen wanneer dit nodig is. Een deployment levert op deze manier een minimale downtime op. 
```yaml
apiVersion: v1 
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
    type: front-end
spec:
  replicas: 3
  selector: 
    matchLabels:
      app: nginx
      type: front-end
  template: 
    metadata: 
      labels:
        app: nginx
        type: front-end 
    spec: 
      containers: 
      - name: nginx-container
        image: nginx:latest
```
### Deployment strategies
In Kubernetes zijn er verschillende deploymentstrategieën om applicaties bij te werken en uit te rollen. Hier zijn enkele veelgebruikte methoden:
- **Rolling Update:** Standaardstrategie waarbij oude pods geleidelijk worden vervangen door nieuwe, waardoor de applicatie beschikbaar blijft tijdens de update.

- **Recreate Deployment:** Alle bestaande pods worden verwijderd voordat nieuwe pods worden aangemaakt, wat resulteert in een korte downtime.

- **Blue/Green Deployment:** Twee omgevingen ('blauw' en 'groen') worden gebruikt; de nieuwe versie wordt in de groene omgeving geïmplementeerd en na testen wordt het verkeer omgeschakeld.

- **Canary Deployment:** Een nieuwe versie wordt uitgerold naar een klein deel van de gebruikers om prestaties te evalueren voordat volledige implementatie plaatsvindt.

## Namespaces
In Kubernetes bieden namespaces een manier om resources binnen een cluster te scheiden. Ze creëren omgevingen waardoor meerdere teams of proejcten hetzelfde cluster kunnen gebruiken, maar elkaar niet in de weg zitten.

Er zijn een aantal namespaces die standaard worden aangemaakt door Kubernetes, namelijk de `default` en de `kube-system` namespace.

Het gebruik van namespaces is metname handig wanneer er een omgeving bestaat met veel gebruikers en/of projecten, of bij het scheiden van verschillende omgevingen zoals ontwikkeling, test en productie. 

## Imperative & Declarative
In Kubernetes zijn er twee manieren om resources te beheren: Imperative en Declarative
### Imperative
Deze methode geef je directe opdrachten via de commandline om de status van je cluster te wijzigen. Bijvoorbeeld, om een pod te creëren met een Nginx-image, gebruik je: 
```bash
kubectl run nginx-pod --image=nginx:latest
```
Deze manier van resource beheer is goed voor snelle tests en eenmalige acties.
### Declarative
Bij deze methode definieer je de gewenste toestand van je cluster met YAML of JSON bestanden. Vervolgens pas je deze toe met: 
```bash
kubectl apply -f nginx-pod.yaml
```
Deze methode werkt ideaal voor productieomgeving, omdat het ook versiebeheer en reproduceerbaarheid met zich mee neemt. 