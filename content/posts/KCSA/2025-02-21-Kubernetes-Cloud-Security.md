---
title: Kubernetes Cloud Native Security
date: 2025-02-21T00:00:00+02:00
type: post
authors:
  - Thomas de Jong
categories: 
  - Kubernetes
  - KCSA 
draft: false
---
Deze post behandelt de securitymaatregelen van AWS, Azure en GCP. We kijken naar hoe deze cloudproviders hun infrastructuur beveiligen met oplossingen zoals SIEM, WAF en containerbeveiliging. Daarnaast bespreken we Kubernetes security en best practices voor veilige container images. Zo krijg je inzicht in de verschillende aanpakken en hoe je cloud security optimaal kunt toepassen.
<!--more-->
## Cloud Providers & Security
Binnen de wereld van cloud providers zijn er drie grote spelers, namelijk *AWS*, *GCP (Google Cloud)* en *Azure*. Deze cloud providers hebben hun security maatregelen elk op hun eigen manier ingeregeld.
| Provider | Oplossing               | Functie                                                                                                                                           | 
| -------- | ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| Azure    | Microsoft Sentinel      | Security Information and Event manager (SIEM) and Security Orchestration, Automation and Respone (SOAR)                                           | 
| AWS      | GuardDuty               | Een machine learning service dat automatisch security issues kan detecteren zonder user interferance                                              |
| GCP      | Security Command Center | Netzoals AWS maakt dit systeem gebruik van machine learning om security issues te detecteren en de gezondheid van de infrastructuur te waarborgen |

### Web Application Firewall (WAF)
Aan de kant van webapplicatie security heeft elke provider ook een eigen oplossing ingeregeld
| Provider | Oplossing          | Functie                                                                                                        |
| -------- | ------------------ | -------------------------------------------------------------------------------------------------------------- |
| Azure    | Azure WAF          | Beschermt tegen aanvallen zoals SQL Injections en XSS attacks                                                  | 
| AWS      | AWS WAF            | Maakt het mogelijk om custom security regels te maken en deze te linken aan de Load Balancer of AWS CloudFront |
| GCP      | Google Cloud Armor | Beschermt web tegen standaard aanvallen zoals DDOS en dergelijken.                                             |

### Container Security
| Provider | Oplossing | Functie                                                                                              |
| -------- | ----------| ---------------------------------------------------------------------------------------------------- |
| Azure    | AKS       | Wordt geleverd met standaard security maatregelen en is niet afhankelijk van tools om dit te regelen |
| AWS      | EKS       | Maakt gebruik van Bottlerocket en KubeBench om security te waarborgen. Bottlerocket is een eigen OS  | 
| GCP      | GKE       | Implementeert security policies door Google Anthos en Open Policy Agent (OPA)                        |

## Cluster Security 
Binnen een Kubernetes cluster wordt security ingeregeld via verschillend maatregelend, bijvoorbeeld; 
- **Namespace**: Het configureren van namespaces binnen een cluster maakt het mogelijk om toegang en andere maatregelen zoals network policies speicifiek toe te passen binnen een namespaces
- **Network Policy**: Network policies kunnen worden ingesteld om te definieren welke pods met elkaar mogen communiceren, binnen een namespace, maar  ook buiten een namespace. Dit gaat op basis van labels. 
- **RBAC**: Role Based Access Control maakt het mogelijk om user accounts binnen een cluster permissies te geven om bepaalde acties uit te voeren op bijvoorbeeld read, write en execute. 
- **Resource Quatas**: Resource quotas zijn in staat om te maximaliseren hoeveel resources een pod/deployment mogen gebruiken.  
- **Security Context**: Wordt gebruikt om te definieren welke dingen een cluster mag, bijvoorbeeld het draaien als een non root user, en een non-root group.

## Images
Elke container maakt gebruik van een container image. Deze containers zijn niet altijd even veilig.
De latest tag in een container houdt vaak niet in dat dit daadwerkelijk de laatste en dus de meest up-to-date image is.
Best practice is om een officiele minimal base image te gebruiken, van bijvoorbeeld ubuntu of alpine. 