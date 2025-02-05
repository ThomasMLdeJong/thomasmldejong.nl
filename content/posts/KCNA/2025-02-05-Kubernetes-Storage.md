---
title: Kubernetes Storage
date: 2025-02-05T00:00:00+02:00
type: post
authors:
  - Thomas de Jong
categories: 
  - Kubernetes
  - KCNA 
draft: false
---

<!--more-->
## Docker
Docker maakt gebruik van een layered architecture. Dit betekent dat elke instructiue in een Dockerfile een nieuwe laag is. Het voordeel hiervan is dat wanneer meerdere Docker-images beginnen met `FROM ubuntu`, deze laag één keer wordt opgeslagen in de directory `/var/lib/docker`. Hierdoor wordt de opslagruimte minder uitgeput en blijft de grootte van Docker beperkt.

Wanneer een image is gebouwd, kan deze niet meer veranderd worden, tenzij deze opnieuw wordt gebouwd, deze laag heet de image layer. Wanneer de container wordt gestart, kan de applicatie of gebruiker data schrijven, deze data wordt opgeslagen in de container layer. Wanneer tijdens het draaien van de container een bestand uit de image layer wordt aangepast, behoudt Docker het originele bestand en slaat de gewijzigde versie op in de container layer, dit heet `copy-on-write`

### Docker Volumes
Volumes worden binnen Docker gebruikt om persistent data op te slaan. In tegenstelling tot de container layer, blijft de data in een volume bestaan wanneer de container wordt afgesloten. Volumes worden beheerd door Docker en slaan data op in de host, los van de containerlayer. Volumes kunnen ook gemount worden door meerdere containers tegelijk, wat nutrtig is voor het delen van bijvoorbeeld configuratie- en logbestanden.

## Container Storage Interface
De Container Storage Interface (CSI) is een gestandaardiseerde interface die het mogelijk maakt om externe opslag te integreren met Kubernetes. Dankzij CSI kunnen providers hun eigen drivers ontwikkelen die dynamisch opslag kunnen leveren, beheren en koppelen, onafhankelijk van de onderliggende infrastructuur. Dit maakt het voor Kubernetes mogelijk om Persistent Volumes op een uniforme en consistente wijze aan te maken en te beheren, waarbij de opslagfunctionaliteit volledig losgekoppeld wordt van de Kubernetes core. CSI werkt onder hetzelfde principe als de Container Runtime Interface (CRI), die de communicatie tussen Kubernetes en container runtimes standaardiseert, waardoor beide interfaces een modulair en vervangbaar systeem mogelijk maken.

## Volumes  
Zonder volumes werkt de opslag in Kubernetes zoals bij Docker: er is een onveranderlijke image layer en een tijdelijke container layer waarin data aangepast kan worden. Volumes definieëren opslag die losstaat van de container, zodat data behouden blijft buiten de container-levenscyclus.

### Persistent Volume  
Een Persistent Volume (PV) maakt het mogelijk om data op te slaan die blijft bestaan, ongeacht de status van de container. PV's worden beheerd door de Kubernetes Storage API en ondersteunen dynamische opslag.  

Een PV kan worden aangemaakt met het volgende manifest:  
```yaml
apiVersion: v1 
kind: PersistentVolume
metadata:
  name: pv-nginx
spec:
  accessModes:
    - ReadWriteOnce
  capacity: 
    storage: 1Gi 
```  

Om een container toegang te geven tot een PV, wordt een PersistentVolumeClaim (PVC) aangemaakt. Dit kan als volgt:  
```yaml
apiVersion: v1 
kind: PersistentVolumeClaim
metadata:
  name: pvc-nginx
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```  

Wanneer een PVC wordt verwijderd, blijft de data op het PV bewaard.
