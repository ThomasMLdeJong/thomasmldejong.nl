---
title: Kubernetes Autoscaling
date: 2025-02-05T00:00:00+02:00
type: post
authors:
  - Thomas de Jong
categories: 
  - Kubernetes
  - KCNA 
draft: false
---
In deze post wordt het concept van autoscaling in Kubernetes uitgelicht, waarbij zowel het op- en afschalen van pods als het automatisch aanpassen van het aantal nodes aan bod komt. Eerst wordt toegelicht hoe horizontale scaling zorgt voor het dynamisch toevoegen of verwijderen van pods op basis van de load, terwijl verticale scaling inhoudt dat bestaande pods meer resources toegewezen krijgen. Vervolgens worden de drie autoscaling methoden besproken: de Horizontal Pod Autoscaler, de Vertical Pod Autoscaler en de Cluster Autoscaler, die samen zorgen voor een efficiënte en flexibele inzet van resources binnen een Kubernetes cluster.
<!--more-->
## Autoscaling
Het autoscalen van een pod houdt in dat wanneer er een verhoging is in het gebruik van een service, er een extra pod wordt aangemaakt om deze vraag te kunnen onderhouden. Op deze manier kan Kubernetes efficient met resources werken aangezien het minder pods gebruikt wanneer er weinig verkeer is, en meer pods wanneer er spikes in verkeer plaatsvinden. 

Meer resources toekennen aan een pod staat bekend als vertical scaling. Wanneer je een extra pod opspint naast de al bestaande pod is dit Horizontal scaling. 

Kubernetes beschikt over drie verschillende vormen van scaling, namelijk de Horizontal Pod autoscaler, Vertical Pod autoscaler en de Cluster AUtoscaler. 

## Horizontal Pod Autoscaler
De horizontal pod autoscaler (HPA) zorgt ervoor dat wanneer de load op het systeem toeneemt, er meer pods worden gedeployed op het cluster, en wanneer het afneemt worden er pods verwijderd. 
DE HPA past de hoeveelheid replica's aan op een deployment, de trigger voor scalen in de HPA is het resourcegebruik, dit leest de HPA af van de ingebouwde metrics server van Kubernetes.
```yaml 
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa 
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
  minReplicas: 1 
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu 
      target:
        type: Utilization
        avarageUtilization: 50
``` 

## Vertical Pod Autoscaler
De vertical pod autoscaler (VPA) past automatisch de CPU en RAM aan op basis van het resource gebruik van een pod. Dit verbeterd de resourcebenutting binnen een cluster aangezien het dan gebruik maakt van alleen de resources die het nodig heeft, dit heet ook wel right-sizen.

De modusen waarmee de VPA werken zijn als volgt: 
| Modus    | Effect                                                                                                                 | 
| -------- | ---------------------------------------------------------------------------------------------------------------------- |
| Recreate | Veranderd de resourcerequirements van pods aan door ze te herstarten, kan voor downtime zorgen wanneer er 1 replica is | 
| Initial  | Wijzigt de resourcerequirement alleen bij het opstarten van een pod                                                    |
| Off      | Geeft alleen advies zonder iets toe te passen, soort alert only                                                        | 
| Auto     | Gelijk aan recreate, bedoeld in de toekomst om automatisch te bepalen welke actie er wordt ondernomen                  | 

Een VPA kan met het volgende manifest opgezet worden: 
```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: nginx-vpa
spec:
  targetRef: 
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
  updatePolicy:
    updateMode: "Auto" 
  resourcePolicy:
    containerPolicies: 
      - containerName: '*'
        minAllowed:
          cpu: 100m 
          memory: 50Mi
        maxAllowed:
          cpu: 1
          memory: 500Mi
        controlledResources: ["cpu", "memory"]
``` 

## Cluster Autoscaler
De Cluster Autoscaler past automatisch het aantal nodes aan in een cluster op basis van de resourcerequirements van de pods. Wanneer pods niet kunen wordt geplaats wegens onvoldoende resources, voegt de Cluster Autoscaler een extra node toe. Deze node wordt vervolgens weer verwijderd wanneer de resourcerequirements afnemen. 