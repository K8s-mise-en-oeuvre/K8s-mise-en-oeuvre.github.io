+++
title = "Architecture Kubernetes"
description = ""
date = 2022-01-01T08:00:00+00:00
updated = 2022-01-01T08:00:00+00:00
draft = false
weight = 13
sort_by = "weight"
template = "docs/page.html"

[extra]
toc = true
top = false
+++

## Architecture Kubernetes 

Fonctionnalités:

<img src="https://K8s-mise-en-oeuvre.github.io/docs/feature-of-Kubernetes-Kubernetes-Architecture.png" alt="feature-of-Kubernetes-Kubernetes-Architecture" width="900" height="720">



<img src="https://K8s-mise-en-oeuvre.github.io/docs/kube-functionnalities.webp" alt="kube-functionnalities" width="900" height="720">



<img src="https://K8s-mise-en-oeuvre.github.io/docs/kubernetes-features.png" alt="kubernetes-features width="900" height="720"

### Composants du Control Plane

*Vue macro*
<img src="https://K8s-mise-en-oeuvre.github.io/docs/control-plane.png" alt="Simple_Control_Plane_Schema" width="900" height="720"> 



<img src="https://K8s-mise-en-oeuvre.github.io/docs/kube-basic-archi.png" alt="Kubernetes basic architecture" width="900" height="720"> 



<img src="https://K8s-mise-en-oeuvre.github.io/docs/kube-multi-master.png" alt="Kubernetes multi master" width="900" height="720">

Le master Kubernetes qui est un ensemble de trois processus qui s'exécutent sur un seul node de votre cluster, désigné comme master node. Ces processus sont: kube-apiserver, kube-controller-manager et kube-scheduler.
Chaque node non master de votre cluster exécute deux processus:
- kubelet, qui communique avec le Kubernetes master.
- kube-proxy, un proxy réseau reflétant les services réseau Kubernetes sur chaque node.

Les différentes parties du control plane Kubernetes, telles que les processus Kubernetes master et kubelet, déterminent la manière dont Kubernetes communique avec votre cluster. Le control plane conserve un enregistrement de tous les objets Kubernetes du système et exécute des boucles de contrôle continues pour gérer l'état de ces objets. À tout moment, les boucles de contrôle du control plane répondent aux modifications du cluster et permettent de faire en sorte que l'état réel de tous les objets du système corresponde à l'état souhaité que vous avez fourni.

Par exemple, lorsque vous utilisez l'API Kubernetes pour créer un objet Deployment, vous fournissez un nouvel état souhaité pour le système. Le control plane Kubernetes enregistre la création de cet objet et exécute vos instructions en lançant les applications requises et en les planifiant vers des nodes de cluster, afin que l'état actuel du cluster corresponde à l'état souhaité.

#### Composants du master node

<img src="https://K8s-mise-en-oeuvre.github.io/docs/Kubernetes-101-Architecture-Diagram.jpg" alt="Kubernetes Archi master-node"> 

- cluster etcd
> Un stockage simple et distribué de valeurs de clés qui est utilisé pour stocker les données du cluster Kubernetes (telles que le nombre de pods, leur état, l'espace de noms, etc), les objets API et les détails de la découverte de services. Pour des raisons de sécurité, il n'est accessible qu'à partir du serveur d'API. etc. etcd permet de notifier au cluster les changements de configuration à l'aide de surveillants. Les notifications sont des requêtes API sur chaque node du cluster etcd pour déclencher la mise à jour des informations dans le stockage du node.

- kube-apiserver
> Le serveur API Kubernetes est l'entité de gestion centrale qui reçoit toutes les demandes REST de modifications (aux pods, services, ensembles de réplication/contrôleurs et autres), servant de front-end au cluster. C'est également le seul composant qui communique avec le cluster etcd, s'assurant que les données sont stockées dans etcd et sont en accord avec les détails des services des pods déployés.

- kube-controller-manager
> Exécute un certain nombre de processus de contrôle distincts en arrière-plan (par exemple, le contrôleur de réplication contrôle le nombre de replica dans un pod, le contrôleur de endpoints remplit les objets endpints comme les services et les pods) pour réguler l'état partagé du cluster et effectuer des tâches de routine. Lorsqu'un changement dans la configuration d'un service se produit (par exemple, le remplacement de l'image à partir de laquelle les pods sont exécutés, ou la modification des paramètres dans le fichier yaml de configuration), le contrôleur repère le changement et commence à travailler vers le nouvel état souhaité.

- cloud-controller-manager
> Le responsable de la gestion des processus du contrôleur qui dépendent du cloud provider sous-jacent (le cas échéant). Par exemple, lorsqu'un contrôleur doit vérifier si un node a été résilié ou configurer des routes, des équilibreurs de charge ou des volumes dans l'infrastructure du cloud provider, tout cela est géré par le cloud provider.

- kube-scheduler
> Aide à planifier les pods (un groupe de conteneurs co-localisés à l'intérieur desquels nos processus d'application sont exécutés) sur les différents nodes en fonction de l'utilisation des ressources. Il lit les exigences opérationnelles du service et le planifie sur le node le mieux adapté. Par exemple, si l'application a besoin de 1 Go de mémoire et de 2 cœurs CPU, les pods de cette application seront planifiés sur un node disposant au moins de ces ressources. Le planificateur s'exécute chaque fois qu'il est nécessaire de planifier des pods. Le planificateur doit connaître les ressources totales disponibles ainsi que les ressources allouées aux charges de travail existantes sur chaque node.

#### Architecture d'un minion

Un node est une machine de travail dans Kubernetes, connue auparavant sous le nom de minion. Un node peut être une machine virtuelle ou une machine physique, selon le cluster. 

Chaque node contient les services nécessaires à l'exécution de pods et est géré par les composants du master. **Les services présents sur un node incluent le container runtime, kubelet et kube-proxy.**

<img src="node-archi.png" alt="Architecture Minion" width="900" height="720">

Pour effectuer l'auto-enregistrement des nodes:

Lorsque l'indicateur de kubelet **--register-node est à true** (valeur par défaut), le kubelet tente de s'enregistrer auprès du serveur d'API. C'est le modèle préféré, utilisé par la plupart des distributions Linux.

Pour l'auto-enregistrement (self-registration en anglais), le kubelet est lancé avec les options suivantes:

- --kubeconfig - Chemin d'accès aux informations d'identification pour s'authentifier auprès de l'apiserver.
- --cloud-provider - Comment lire les métadonnées d'un fournisseur de cloud sur lui-même.
- --register-node - Enregistrement automatique avec le serveur API.
- --register-with-taints - Enregistrez le noeud avec la liste donnée de marques (séparés par des virgules <key>=<value>:<effect>). Sans effet si register-node est à false.
- --node-ip - Adresse IP du noeud.
- --node-labels - Labels à ajouter lors de l’enregistrement du noeud dans le cluster (voir Restrictions des labels appliquées par le plugin NodeRestriction admission dans les versions 1.13+).
- --node-status-update-frequency - Spécifie la fréquence à laquelle kubelet publie le statut de nœud sur master.

Quand le mode autorisation de node et plugin NodeRestriction admission sont activés, les kubelets sont uniquement autorisés à créer / modifier leur propre ressource de node.

### Concepts: objets stateful, stateless

Un objet dit stateful est capable de maintenir l'état d'un processus ou d'une transaction.

StatefulSet est l'objet de l'API de charge de travail utilisé pour gérer des applications stateful.

Il gère le déploiement et la mise à l'échelle d'un ensemble de Pods, et fournit des garanties sur l'ordre et l'unicité de ces Pods.

Comme un Déploiement, un StatefulSet gère des Pods qui sont basés sur une même spécification de conteneur. Contrairement à un Deployment, un StatefulSet maintient une identité pour chacun de ces Pods. Ces Pods sont créés à partir de la même spec, mais ne sont pas interchangeables: **chacun a un identifiant persistant qu'il garde à travers tous ses re-scheduling.**

Si vous voulez utiliser des volumes de stockage pour fournir de la persistance à votre charge de travail, vous pouvez utiliser un StatefulSet comme partie de la solution. Même si des Pods individuels d'un StatefulSet sont susceptibles d'échouer, les identifiants persistants des Pods rendent plus facile de faire correspondre les volumes existants aux nouveaux Pods remplaçant ceux ayant échoué.

Ce qui caractérisera la persistence de notre objet stateful reste toutefois lié à notre politique de storage. Un StatefulSet dont le stockage n'est pas persistant n'aura que peu de tolérance à la panne sur des applications critiques ou en hautes disponibilités. C'est dans cette perspective que nous introduirons ensuite les concepts de PersistentVolume et de PersistentVolumeClaim qui permettent de tirer parti de toutes les fonctionnalités du StatefulSet.

A l'inverse, les processus pour lesquels il n'est pas souhaité que l'état soit maintenu sont dits stateless. Il s'agit par défaut dans Kubernetes de tout objet qui ne soit pas un StatefulSet ou ne disposant pas d'un storage  persistant.

Ces concepts seront revus et pratiqués dans le cadre du TP du chapitre "Exploiter Kubernetes".

### Quelques exemples d'architecture cloud hybrides ou on-premise

<img src="https://K8s-mise-en-oeuvre.github.io/docs/oc-archi.jpg" alt="OC Archi" width="900" height="720">



<img src="https://K8s-mise-en-oeuvre.github.io/docs/possible-oc-archi.jpg" alt="Possible archi OC" width="900" height="720">



<img src="https://K8s-mise-en-oeuvre.github.io/docs/possible-oc-archi.2jpg.webp" alt="Possible archi OC 2" width="900" height="720">


<img src="https://K8s-mise-en-oeuvre.github.io/docs/redhat-openshift-on-aws-architecture.png" alt="redhat-openshift-on-aws-architecture width="900" height="720"


<img src="https://K8s-mise-en-oeuvre.github.io/docs/architecture-on-premise.jpeg" alt="Archi on-premise" width="900" height="720">


<img src="https://K8s-mise-en-oeuvre.github.io/docs/archi-hybrid.png" alt="Archi Hybrid" width="900" height="720">

### Quelques exemples d'architecture cloud public

*Azure*
<img src="https://K8s-mise-en-oeuvre.github.io/docs/aks.png" alt="AKS" width="900" height="720">



<img src="https://K8s-mise-en-oeuvre.github.io/docs/aks-production-deployment.png" alt="AKS Productionc Deployment" width="900" height="720">



<img src="https://K8s-mise-en-oeuvre.github.io/docs/azure-application-architecture.png" alt="Azure Application Architecture" width="900" height="720">


*GCP*

<img src="https://K8s-mise-en-oeuvre.github.io/docs/gke-archi.png" alt="GKE Archi" width="900" height="720">



<img src="https://K8s-mise-en-oeuvre.github.io/docs/gke-archi2.png" alt="GKE Archi 2" width="900" height="720">



<img src="https://K8s-mise-en-oeuvre.github.io/docs/google-search-assistant-diagram-gcp.webp" alt="Google assistant" width="900" height="720">
