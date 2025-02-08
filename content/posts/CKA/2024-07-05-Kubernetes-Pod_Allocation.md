---
title: Kubernetes Pod Allocation
date: 2024-07-05T00:00:00+02:00
type: post
authors:
  - Thomas de Jong
categories: 
  - Kubernetes
  - CKA
draft: false
---
In deze post worden de verschillende mechanismen voor het schedulen binnen Kubernetes toegelicht. Er wordt uitgelegd hoe de Kubernetes scheduler een geschikte node kiest voor pods, hoe met behulp van nodeSelector en nodeName de plaatsing kan worden beperkt of specifiek aangewezen, en hoe DaemonSets zorgen voor een consistente deployment van pods op alle nodes. Daarnaast komt ook aan bod hoe static pods en de daarmee samenhangende mirror pods bijdragen aan een alternatieve benadering van podbeheer binnen het cluster.
<!--more-->
## Scheduling
### Kubernetes Scheduler
De Kubernetes scheduler zoekt een geschikte node uit voor elke pod, waarbij deze rekening houdt met:
- Resource Requests vs. Node Resources
- Configuratie die scheduling aanpast door node labels

### nodeSelector
Een nodeSelector kan worden geconfigureerd om te limiteren op welke nodes een pod geplaatst kan worden. dit gebeurt op basis van labels:
```yaml
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    mylabel: myvalue
```
Nu zal de nodeSelector deze pod alleen schedulen op een node met het label: mylabel: myvalue

### nodeName
nodeName wordt gebruikt om de scheduler volledig te bypassen, door de node waar je een pod wilt hebben staan specifiek te definieren.
```yaml
spec:
  containers: 
  - name: nginx
    image: nginx 
  nodeName: k8s-worker1 
```

## DaemonSet
Een DaemonSet draait automatisch een kopie van een pod op elke node, ook wanneer er een nieuwe node wordt toegevoegd terwijl de daemonset al bestaat.
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

## Pods
### Static Pods
Een Static Pod is een pod die direct wordt gemanaged door een kubelet op een node, en niet door de K8S API.
### Mirror Pods
Een mirror pod wordt aangemaakt met elke static pod, deze mirror pods maken het mogelijk om de status te zien via de K8S API.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```
