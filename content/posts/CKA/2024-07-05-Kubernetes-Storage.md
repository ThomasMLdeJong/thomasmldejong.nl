---
title: Kubernetes Storage
date: 2024-07-05T00:00:00+02:00
type: post
authors:
  - Thomas de Jong
categories: 
  - Kubernetes
  - CKA
draft: false
---
In deze post worden de opslagconcepten binnen Kubernetes uitgebreid behandeld. Er wordt toegelicht hoe volumes en volumeMounts de basis vormen voor het koppelen van fysieke opslag aan pods, en welke verschillen er bestaan tussen veelgebruikte volume types zoals hostPath en emptyDir. Daarnaast wordt uitgelegd hoe PersistentVolumes en StorageClasses een abstractielaag creëren bovenop de fysieke opslag, zodat opslag efficiënt kan worden beheerd en gedeeld. Ook komen belangrijke configuratie-opties zoals persistentVolumeReclaimPolicy en allowVolumeExpansion aan bod.
<!--more-->
## Volumes and volumeMounts
### Volumes
Een volume in een Kubernetes-pod is een specificatie die aangeeft welke opslagvolumes beschikbaar zijn voor die pod. In de gegeven specificatie wordt bijvoorbeeld een volume met de naam "my-volume" gedefinieerd. Dit volume wordt gemount vanaf het pad "/data" op de host.
```yaml
spec:
  volumes:
  - name: my-volume
    hostPath:
      path: /data
```
### volumeMounts
Een volumeMount in een Kubernetes-pod specificeert het mountPath waar het volume van de pod moet worden gemount in het bestandssysteem van de container. In de gegeven specificatie wordt bijvoorbeeld een volumeMount gedefinieerd met de naam "my-volume" en het mountPath "/output". Dit betekent dat het volume met de naam "my-volume", dat is gedefinieerd in de volumespecificatie, zal worden gemount op het pad "/output" binnen de container.
```yaml
spec:
  containers:
    volumeMounts:
    - name: my-volume
      mountPath: /output
```
### Common Volume Types
- **hostPath** \
Slaat data op in een specifieke directory in een Kubernetes node
- **emptyDir** \
slaat data op in een dynamische locatie op een node, deze directory bestaat alleen zolang als dat de pod bestaat.

## PersistentVolumes
Een PersistentVolume (PV) in Kubernetes is inderdaad een object dat het mogelijk maakt om opslag als een abstracte resource te behandelen. PV's bieden een abstractielaag bovenop de details van de opslag-implementatie, waardoor gebruikers zich minder zorgen hoeven te maken over de specifieke kenmerken van de fysieke opslaginfrastructuur.
```yaml
kind: PersistenVolume
apiVersion: v1
metadata:
  name: my-pv
spec:
  storageClassName: localdisk
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /var/output
```
### StorageClasses
Een StorageClass maakt het mogelijk om een type van storage service te definieren.
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: localdisk
provisioner: kubernetes.io/no-provisioner
```
Deze classes kunnen beschikken over meerdere eigenschappen
### Metadata
- low: low performance, inexpensive storage
- fast: high-performance, costly storage
### persistentVolumeReclaimPolicy
Het is mogelijk om een ReclaimPolicy aan te geven:
- Retain, dit houdt in dat alle data behouden wordt, om dit op te schonen is er een admin nodig.
- Delete, dit verwijderd de opslag automatisch
```yaml
spec:
  persistentVolumeReclaimPolicy: <policy>
```
### allowVolumeExpansion
De flag allowVolumeExpansions bepaald of een StorageClass volumes mag resizen nadat deze zijn aangemaakt. Als deze niet op true staat, zal een poging tot resizen resulteren in een error.