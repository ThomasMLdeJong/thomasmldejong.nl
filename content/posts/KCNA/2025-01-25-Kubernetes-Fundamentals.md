---
title: Kubernetes Fundamentals
date: 2025-01-25T00:00:00+02:00
type: post
authors:
  - Thomas de Jong
categories: 
  - Kubernetes
  - KCNA 
draft: false
---
In deze post leer je de basisprincipes van Kubernetes, inclusief containers, container orchestration en de Kubernetes-architectuur. We bespreken de verschillende soorten containers, hoe Kubernetes helpt bij het beheren van containers in complexe applicaties, en de rol van de control plane en worker nodes. We eindigen met een uitleg over de Container Runtime Interface (CRI) en de flexibiliteit die het biedt voor container runtimes.
<!--more-->

## Containers
Containers zijn machines die alles bevatten om een applicatie zoals nginx te draaien. Ze omvatten de code, runtime, libraries en systeemafhankelijkheden die nodig zijn om de applicatie uit te voeren. Containers worden vaak gebruikt om een applicatie consistent te laten draaien in verschillende omgevingen, zoals development, test en productie.
Er zijn verschillende soorten containers, elk met hun eigen focus en gebruiksscenario:

## Systeemcontainers (LXC, LXD)
Een systeemcontainer biedt een volledige operating system (OS) binnen een container, ze issoleren hiermee niet alleen de applicatie, maar het gehele systeem, inclusief de processen.

## Applicatiecontainers (Docker)
Een applicatiecontainer is gericht op het draiien van één applicatie en de bijbehorende afhankelijkheden. Ze delen de kernel van de host en zijn minder zwaar in resources dan een systeemcontainer.

## Container Orchestration
Container orchestration is het beheren en automatiseren van het inzetten, schalen en netwerken van applicaties die draaien in containers. Het is belangrijk voor het effectief beheren van complexere applicaties die bestaan uit meerdere containers (microservice-architecture)

## Functies
- Containers worden automatisch gestart of gestopt op basis van de vraag en workload 
- De workload wordt verdeeld over de beschikbare containers om de druk te verminderen 
- Containers worden automatisch herstart wanneer ze "unhealthy" zijn
- Regelt de interne en externe netwerkcommunicatie tussen containers en services

## Kubernetes Architecture
Kubernetes is ontworpen om container orchestration op schaal mogelijk te maken. Het bestaat uit een **control plane**  en **worker nodes**

## Control Plane
De control plane beheert het cluster en zorgt ervoor dat de gewenste status van de applicaties behouden blijft. Ook draaien er op de control plane de volgende services: 

- **API Server**\
Centrale interface waarmee gebruikers en applicaties communiceren met Kubernetes.
- **ETCD**\
Een key-value store die de configuratie en status gegevens van het Kubernetes-cluster opslaan. 
- **Scheduler**\
De scheduler is verantwoordelijk voor het toewijzen van workloads (pods) aan geschikte nodes op basis van beschikbare resources.
- **Controller**\
De Kubernetes controller zorgt ervoor dat de resources zoals pods en replica's blijven voldoen aan de configuraties gegeven door de gebruiker

## Worker node
Een worker node wordt, zoals de naam al zegt, aangestuurd door de Control Plane. Een worker node bevat hierom niets meer dan het programma Kubelet, en de Container Runtime

## Container Runtime Interface (CRI)
De CRI maakt het mogelijk voor elke container format om te werken met Kubernetes zolang ze maar compliant zijn met de **Open Container Initiative** (OCI). Dit zorgt voor flexibiliteitbij het kiezen van een container-runtime, zoals containerd of CRI-O. 
