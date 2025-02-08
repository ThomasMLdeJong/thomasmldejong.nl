---
title: Kubernetes Pods & Containers
date: 2024-07-05T00:00:00+02:00
type: post
authors:
  - Thomas de Jong
categories: 
  - Kubernetes
  - CKA
draft: false
---
In deze post worden de volgende onderdelen behandeld: hoe applicatieconfiguratie gerealiseerd kan worden via ConfigMaps en Secrets en hoe deze data gekoppeld kan worden aan containers, het beheren van container resources middels resource requests en limits, de inspectie van container health via liveness, startup en readiness probes, de aanpak van restart en self-healing met behulp van restart policies en een eerste kennismaking met multi-container pods.
<!--more-->
## Application Configuration 
Application conifguration is het geven van dynamische waarden naar een applicatie of container gedurende de runtime. 
### Configmap
Application Configuration kan worden gerealiseerd via een ConfigMap, deze slaan data op in de vorm van een key-value map.
```yaml
apiVersion: v1 
kind: ConfigMap
metadata:
  name: my-configmap
data:
  key1: value1
  key2: value2
  key3:
    subkey: 
      morekeys: data
      evenmore: some more data
  key4: | 
    multi line data
    is also possible
```
### Secrets
Ook is het mogelijk om key values door te geven via een secret, secrets zijn in essentie hetzelfde als een ConfigMap maar zijn ontwikkeld om vertrouwelijke data zoals API keys veiliger op te slaan
```yaml
apiVersion: v1 
kind: Secret
metadata: 
  name: my-secret
type: Opaque
data: 
  username: user
  password: mypass
```
### Link
Om een container of pod vervolgens deze data te laten gebruiken kan de volgende template worden gebrukt: 
```yaml
spec:
  containers:
  - . . .
    env:
    - name: ENVVAR 
      valueFrom:
        configMapKeyRef:
          name: my-configmap
          key: mykey
```
Het is ook mogelijk om de link uit te voeren door middel van een volume mount, deze ziet er als volgt uit: 
```yaml
volumes: 
- name: secret-vol 
  secret: 
    secretName: my-secret
```

## Container Resources
Resource Requrest maken het mogelijk om een hoeveelheid resources te definieren, die een pod kan en mag gebruiken. een ResourceRequest is geen minimum of maximum.
```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: my-pod 
spec: 
  containers:
 - name: busybox
   image: busybox
   resources: 
     requests: 
       cpu: "250m" 
       memory: "128Mi" 
```
### Resource Limits
Resource Limits kunnen zoals de naam zegt gebruikt worden om de hoeveelheid beschikbare resource voor een container te limiteren. Dit wordt gewaardborgt door de container runtime.
```yaml
apiVersion: v1
kind: Pod 
metadata: 
  name: my-pod 
spec: 
  containers: 
  - name: busybox
    image: busybox 
    resources: 
      limits: 
        cpu: "250m"
        memory: "128Mi"
```
## Container Health
Kubernetes biedt de mogelijkheid tot het inspecteren van de container health, om zo downtime te minimaliseren. 
### Liveness Probe 
Een Liveness Probe kijkt automatisch of de container nog in een gezonde staat is. Op default settings zou Kubernetes een container alleen als down zien als een proces stopt, dit kan worden aangepast met een Liveness Probe. 
```yaml
livenessProbe:
  exec:
    command:
      - cat
      - /tmp/healthy
    initialDelaySeconds: 5
    periodSeconds: 5
```
### Startup Probes
Vrijwel gelijk aan een Liveness probe, echter werken deze niet consistent, maar alleen tijdens het opstarten, waarna de startup probe afsluit als de container succesvol is opgestart.
```yaml
startupProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 30
  periodSeconds: 10
```
### Readiness Probe
Een Readiness probe wordt gebruikt om te kijken wanneer eeen container klaar is om requests te ontvangen, dit houdt in dat een service pas requests naar een pod zal sturen wanneer de readiness probe heeft aangegeven dat de pod klaar is.
```yaml
readinessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```

## Restart and Self-Healing
Wanneer een container crashed kan Kubernetes automatisch een container restarten. dit wordt vastgesteld op basis van Restart policies. 
### Always
Always is de default policy, deze houdt in dat een container altijd zal restarten wanneer de pod uit gaat, ook wanneer een container afsluit omdat deze succesvol zijn doel heeft berijkt. 
### OnFailure
OnFailure kan gebruikt worden om een pod te restarten wanneer deze crashed,  dit houdt in dat er niets wordt gerestart op het moment dat een container succesvol zijn doel heeft berijkt.
### Never
De container zal hoe dan ook niet restarten.

## Multi-container pod 
Een multi-container pod heeft meerdere containers, diue dezelfde resources delen, dit kan gaan over dingen als netwerk en opslag. 