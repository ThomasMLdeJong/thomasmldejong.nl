---
title: Kubernetes Scheduling
date: 2025-02-02T00:00:00+02:00
type: post
authors:
  - Thomas de Jong
categories: 
  - Kubernetes
  - KCNA 
draft: false
---
In deze post woren de verschillende manieren om pods in Kubernetes te schedulen en te beheren behandeld. Het gaat hier over handmatig pods toewijzen met nodeNames of BindingObject, en hoe je d.m.v. labels en selectors resources kunt organiseren. Daarnast worden taints en tolerations behandeld, die bepalen welke pods op welke nodes mogen draaien. Ten slotte wordt node affinity behandeld, waarna er kort gesproken wordt over de werking van DaemonSets.
<!--more-->

## Manual Scheduling
Elk manifest bevat onderwater de field `nodeName`, deze wordt vaak niet gedefinieerd omdat Kubernetes deze zelf invult. Kubernetes checkt welke nodes er zijn, en welke hiervan nog resources over hebbben om een extra pod te kunnen draaien. het veld `nodeName` kan ingesteld worden om zo handmatig pods te schedulen om opgezet te worden. De `nodeName` kan alleen bij het aanmaken worden ingesteld en kan daarna niet meer worden aangepast. 

### Binding Object
Een andere manier om een pod aan een specifieke node te koppelen, is door een Binding Object te gebruiken. Dit object linkt een pod aan een node op basis van de opgegeven gegevens.
```yaml
apiVersion: v1 
kind: Binding 
metadata: 
  name: nginx
target: 
  apiVersion: v1 
  kind: Node 
  name: worker-node1
```

## Labels & Selectors 
Labels zijn een dictionary in de metadata-tag die kan worden gebruikt op objecten. Ze bieden een manier om resources te organiseren en te groeperen. Labels bestaan uit key-values en kunnen op elk moment worden toegevoegd of gewijzigd zonder de ondeliggende objecten aan te passen

Selectors worden gebruikt om objecten te filteren op basis van labels. Ze worden toegepast bij services, deployments en replica sets om alleen de juiste resources te targeten. 
```yaml
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector: 
    matchLabels: 
      app: nginx
  template:
    metadata: 
      labels: 
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
```

## Taints & Tolerations 
Taints en Tolerations worden in Kubernetes gebruikt om te bepalen welke pods wel of niet op een bepaalde node mogen draaien. 

### Taints 
Een taint wordt toegepast op een node om te voorkomen dat pods daar worden gescheduled, tenzij ze de bijpassende toleration hebben. Hierbij is het belangrijk om te onthouden dat een node meerdere taints kan hebben, maar standaard geen taints heeft.

Er zijn in totaal drie verschillende taint effecten: 
- **NoSchedule:** Nieuwe pods zonder de bijpassende toleration worden niet gescheduled op de node
- **PreferNoSchedule:** De scheduler probeert te vermijden dat nieuwe pods op deze node worden geplaatst, maar het is geen strikte regel
- **NoExecute:** Nieuwe pods zonder een bijpassende toleration worden niet gescheduled, en bestaande pods zonder de juiste toleration worden verwijderd van de node. 

Het toevoegen van een taint kan met het volgende commando
```bash
kubectl taint nodes node01 stage=development:NoSchedule 
```
Hiermee kan geen enkele pod op node01 draaien zonder de toleration `stage=development:NoSchedule`

### Tolerations 
Een toleration wordt toegevoegd aan een pod om het toe te staan dat het mag draaien op een node met de bijpassende taint. Een toleration dwingt niet af dat een pod op een bepaalde node draait, het maakt het alleen mogelijk.

Het zetten van een toleration kan op de volgende manier: 
```yaml
apiVersion: v1 
kind: Pod
metadata: 
  name: nginx
spec: 
  containers: 
  - name: nginx 
    image: nginx:latest
  tolerations:
  - key: "stage"
    operator: "Equal"
    value: "development"
    effect: "NoSchedule"
```
Met bovenstaande manifest kan een pod draaien op elke node met de taint `stage=development:NoSchedule`

## Node Selectors
Standaard kan een pod op elke node in het cluster worden gescheduled. Om dit te beperken tot specifieke nodes, kan een NodeSelector worden gebruikt

met een NodeSelector geef je aan dat een pod alleen mag draaien op nodes met een specifiek label. Bijvoorbeeld met de volgende pod-configuratie: 
```yaml
apiVersion: v1 
kind: Pod
metadata: 
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx:latest
  nodeSelector: 
    stage: development
```

Om ervoor te zorgen dat de NodeSelector een node kan vinden, moet de betreffende node wel een bijpassend label hebben. Dit kan worden ingesteld met het volgende commando: 
```bash
kubectl label nodes node01 stage=development
```

## Node Affinity 
Node affinity is een manier om pods aan een specifieke node te koppelen, netzoals NodeSelectors. Het maakt het mogelijk om complexere regels te definiëren over waar een pod mag draaien. De node affinity kan worden aangegeven met meerdere types, namelijk: 
| Type                                             | Effect                                                                                                                        |
| ------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------- | 
| requiredDuringSchedulingIgnoredDuringExecution   | De pod moet op een node draaien die aan de voorwaarden voldoet. Als er geen geschikte node is, wordt de pod niet gescheduled. |
| preferredDuringSchedulingIgnoredDuringExecution  | Kubernetes probeert de pod op een geschikte node te plaatsen, maar als dat niet kan, wordt de pod alsnog elders gescheduled.  |
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
  affinity: 
    nodeAffinity: 
      requiredDuringSchedulingIgnoredDuringExecution: 
        nodeSelectorTerms: 
        - matchExpressions:
          - key: stage
            operator: In
            values: ["development"]
```
Zoals te zien in bovenstaande manifest wordt er gebruik gemaakt van operators, de volgende operators bestaan binnen Kubernetes:
| Operator     | Functie                                                              | 
| --------     | -------------------------------------------------------------------- |
| In           | De waarde van het label moet gelijk zijn aan de opgegeven waarde     |
| NotIn        | De waarde van het label mag niet gelijk zijn aan de opgegeven waarde |
| Exists       | Het label moest bestaan op de node                                   |
| DoesNotExist | Het label mag niet bestaan op de node                                | 
| Gt           | De waarde van het label moet groter zijn dan de opgegeven waarde     | 
| Lt           | De waarde van het label moet kleiner zijn dan de opgegeven waarde    | 

## Resource Scheduling 
In Kubernetes kunnen resource requests en limits ingesteld worden om te bepalen hoeveel CPU en geheugen een container minimaal nodig heeft en maximaal mag gebruiken. 

### Resource Request
Een resource request geeft de minimale CPU en geheugen hoeveelheid aan die een container nodig heeft om te draaien. De Kubernetes-scheduler gebruik deze waarde op zijn beurt om te bepalen op welke node de pod geplaatst kan worden. Nodes moeten ten minste de gevraagde resources hebben, anders wordt de pod niet gescheduled

### Resource Limits
Een resource limit geeft de maximale CPU en geheugen hoeveelheid aan die een container mag gebruiken. Als een container probeert om meer te gebruiken dan ingesteld, kan het cluster bepalen om de pod af te sluiten.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    resources:
    requests:
        memory: "3Gi"
        cpu: "1"
    limits:
        memory: "4Gi"
        cpu: "2"
```
In bovenstaande manifest heeft de pod minimaal 3GiB en 1 CPU nodig om te draaien, de Kubernetes scheduler gebruikt deze waardes om een bijpassende node te vinden. De pod mag maximaal 4GiB geheugen en 2 CPU's gebruiken. 

## DaemonSets
Een DaemonSet lijkt op een ReplicaSet, maar het verschil is dat een DaemonSet ervoor zorgt dat er exact één pod op elke node draait, in plaats van een specifiek aantal pods op één node. Als er een nieuwe node wordt toegevoegd aan het cluster, wordt er automatische een pod op deze node toegevoegd. Dit maakt DaemonSets erg toepasbaar voor bijvoorbeeld monitoring en logging. 

Een Daemonset kan worden gedefinieerd met het volgende manifest:
```yaml
apiVesrion: apps/v1 
kind: DaemonSet
metadata:
  name: monitoring
spec:
  selector:
    matchLabels:
      app: monitoring-agent 
  template:
    metadata: 
      labels: 
        app: monitoring-agent
    spec:
      containers:
        name: monitoring-agent
        image: prometheus
```