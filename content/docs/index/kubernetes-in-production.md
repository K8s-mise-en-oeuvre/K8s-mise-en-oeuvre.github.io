+++
title = "Kubernetes en Production"
description = ""
date = 2022-01-01T08:00:00+00:00
updated = 2022-01-01T08:00:00+00:00
draft = false
weight = 15
sort_by = "weight"
template = "docs/page.html"

[extra]
toc = true
top = false
+++

## Kubernetes en production

### Frontal administrable Ingress

> Ingress (ou entrée réseau), ajouté à Kubernetes v1.1, expose les routes HTTP et HTTPS de l'extérieur du cluster à des services au sein du cluster. Le routage du trafic est contrôlé par des règles définies sur la ressource Ingress.

```
    internet
        |
   [ Ingress ]
   --|-----|--
   [ Services ]
```

Un Ingress peut être configuré pour donner aux services des URLs accessibles de l'extérieur, un équilibrage du trafic de charge externe, la terminaison SSL/TLS et un hébergement virtuel basé sur le nom. Un contrôleur d'Ingress est responsable de l'exécution de l'Ingress, généralement avec un load-balancer (équilibreur de charge), bien qu'il puisse également configurer votre routeur périphérique ou des interfaces supplémentaires pour aider à gérer le trafic.

Un Ingress n'expose pas de ports ni de protocoles arbitraires. **Exposer des services autres que HTTP et HTTPS à Internet généralement utilise un service de type Service.Type=NodePort ou Service.Type=LoadBalancer.**

### Limitation de ressources

### Gestion des ressources et autoscaling

### Service Discovery (env, DNS)

### Les namespaces et les quotas

### Gestion des accès (RBAC)

### Haute disponibilité et mode maintenance 

### Déploiement de conteneurs et gestion de la montée en charge (TP)
