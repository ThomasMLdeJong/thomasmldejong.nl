---
title: Kubernetes Service Mesh
date: 2025-02-05T00:00:00+02:00
type: post
authors:
  - Thomas de Jong
categories: 
  - Kubernetes
  - KCNA 
draft: false
---
Deze post behandeldt de services in Kubernetes, waaronder NodePort, ClusterIP en LoadBalancers. Daarnaast wordt uitgeloegd hoe sidecars extra functionaliteiten kunnen bieden binnen een pod. Het verschil tussen monolitische applicaties, en microservices komen aan bod. en ten slotte de rol van een service mesh in het beheren van communicatie tussen microservices.
<!--more-->
## Services
Pods in Kubernetes hebben een IP adres, wanneer ze resetten verandert dit IP adres. Om de verbinding niet te verliezen kan er een service aangemaakt worden zodat elke applicatie de service kan gebruiken om te verbinden met de pods. Een service bepaalt op basis van labels met welke pods er verbinding gemaakt mag worden.

Er zijn in Kubernetes drie verschillende soorten services: 
| Service      | Functie                                                                                                                                            | 
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| NodePort     | Deze service is in staat om beschikbaar te zijn buiten het cluster                                                                                 | 
| ClusterIP    | De default service die alleen beschikt over interne communicatie mogelijkheden                                                                     | 
| LoadBalancer | Alleen ondersteund bij een aantal cloud providers, werkt hetzelfde als een NodePort, maar gebruikt een externe LoadBalancer van de cloud provider. |

## SideCar
Een sidecar is een container die ondersteuning bied aan de hoofdcontainer binnen dezelfde pod. De sidecar voorziet de hoofdcontainer van extra functionaliteiten zonder dat de hoofdcontainer daarvoor aangepast hoeft te worden. Denk hierbij aan functionaliteiten zoals het verwerken van logs, doorsturen van monitoring gegevens of het injecteren van data. 
Een sidecar is mogelijk doordat ze binnen dezelfde pod draaien, wat ervoor zorgt dat ze dezelfde netwerk en storage resources gebruiken. 

## Monolith & Microservices
Een monolitische applicatie is een applicatie waarbij alle functionaliteiten binnen één codebase en deployment bestaan. Dit betekent dat alle componenten zoals de userinterface, logica en database één geheel zijn, dus ook als geheel worden ontwikkeld, getest en opgezet. 

Een microservice architectuur is wanneer een applicatie wordt opgesplitst in meerdere kleine services die afzonderlijk van elkaar kunnen worden ontwikkeld. Elke microservice beheert zijn eigen data en functionaliteit en communiceert dit met de andere service via calls. 

### Belangrijkste verschillen:
- Monolithische applicaties zijn eenvoudiger te ontwikkelen en beheren in kleine teams, maar kunnen moeilijk schaalbaar en onderhoudbaar worden naarmate de codebase groeit.
- Microservices bieden flexibiliteit, schaalbaarheid en technologische diversiteit, maar brengen extra complexiteit met zich mee door onderlinge communicatie en beheer van meerdere services.

## Service Mesh
Een service mash is een laag in de infrastructuur van het Kubernetes cluster dat de communicatie tussen microservices regels. Dit gebeurt door het inzetten van sidecar-proxies naast de applicatie. Door deze taken buiten de applicatielogica te houden, kunnen developers zich richten op de kernfunctionaliteit, terwijl de service mesh zorgt voor een betrouwbare en veilige communicatie tussen de services.
