+++
title = "Introduction à Kubernetes"
description = ""
date = 2022-01-01T08:00:00+00:00
updated = 2022-01-01T08:00:00+00:00
draft = false
weight = 11
sort_by = "weight"
template = "docs/page.html"

[extra]
toc = true
top = false
+++

## Introduction à kubernetes

Kubernetes, souvent abrégé k8s, est un orchestrateur de conteneurs conçu par Google et sorti en 2014. C’est un outil Open Source (code source disponible sur github), écrit en Go, qui trouve ses racines dans Borg le système interne de Google qui gère l’infrastructure du géant de Mountain View.

<img src="https://K8s-mise-en-oeuvre.github.io/docs/borg.png" alt="Borg" width="720" height="600">

Ce n’est pas le seul orchestrateur de conteneurs sur le marché, il en existe d’autres comme **Docker Swarm**, **Apache Marathon**, **Nomad** ou **Kontena**. Cependant, aujourd'hui K8s apparaît comme hégémonique sur le marché et ses concurrents ne se divisent que peu de parts.

> Le nom Kubernetes tire son origine du grec ancien, signifiant capitaine ou pilote ou encore timonier et est la racine de gouverneur et cybernetic. K8s est l'abbréviation dérivée par le remplacement des 8 lettres "ubernete" par "8".

Soit celui qui tient la barre d’un bateau. Si l’on reprend l’analogie de Docker avec les conteneurs maritimes, Kubernetes est le capitaine qui dirige le porte-conteneurs.

Aujourd'hui, en 2022, force est de constater que Kubernetes est devenu incontournable et la plupart des cloud providers proposent désormais des solutions Kubernetes hébergées.

Il faudra attendre **2013 pour que Google lance sa première offre IAAS** en ayant été ralenti par l'échec de son PAAS AppEngine.

**Un des enjeux de K8S fut de réussir a mixer la flexibilité du IAAS et la facilité d'accès, la descriptabilité du PAAS, ce que l'on pourrait appeler l'IPAAS.**

Il fallait donc un standard pour l'orchestration, une API qui puisse gérer l'ensemble de ces caractéristiques.

Kubernetes est strictement impératif, nous demandons un état sans avoir à connaître les étapes intermédiaires.

Par exemple:

- "J'aimerai un espace isolé dont les applications n'utilisent que 4Go de RAM et 2 CPU"
- "J'aimerai une base de données répliquée 2 fois"
- "J'aimerai un storage avec une rétention de 6 mois"
- "J'aimerai que des mises a jour soit déployées mais n'impactent que les applications du ns 'web' ou correspondant au tag 'web'"

Pour utiliser Kubernetes, vous utilisez les objets de l'API Kubernetes pour décrire l'état souhaité de votre cluster: quelles applications ou autres processus que vous souhaitez exécuter, quelles images de conteneur elles utilisent, le nombre de réplicas, les ressources réseau et disque que vous mettez à disposition, et plus encore. Vous définissez l'état souhaité en créant des objets à l'aide de l'API Kubernetes, généralement via l'interface en ligne de commande, kubectl. Vous pouvez également utiliser l'API Kubernetes directement pour interagir avec le cluster et définir ou modifier l'état souhaité.

Une fois que vous avez défini l'état souhaité, le plan de contrôle Kubernetes (control plane en anglais) permet de faire en sorte que l'état actuel du cluster corresponde à l'état souhaité. Pour ce faire, Kubernetes effectue automatiquement diverses tâches, telles que le démarrage ou le redémarrage de conteneurs, la mise à jour du nombre de réplicas d'une application donnée, etc.

**L’orchestration est gérée par des contrôleurs**. Ces contrôleurs sont compilés dans le kube-controller-manager.

Kubernetes est container-centric et va nous permettre de déployer et de gérer des conteneurs. Les conteneurs ne sont pas gérés individuellement. Au lieu de cela ils font partie d’un ensemble plus grand appelé **Pod**.

> Un Pod se compose d’un ou de plusieurs conteneurs qui partagent une adresse IP, un accès au stockage et un espace de nommage.

On va distinguer la notion de spec de la notion de statut.

S'il réussi a converger à l'issue de sa boucle de réconciliation, l'état est appliqué et le résultat retourné à l'api server.

Il était très compliqué d'atteindre l'indempotence avec Puppet ou Chef par exemple, car outre l'effet snowflake, certains composants tel le réseau on-premise, n'étaient pas prévus pour être indompotents.

Deux machines, 2 versions ne vont pas nécessairement réagir de la même façon. C'est l'effet snowflake car tous les flocons se ressemblent et il devient presque impossible distinguer/troubleshooter les différences sur des milliers de machines de production.

Kubernetes répond a ce problème car il maintient l'état et le retourne après action. On a donc un *avant* et un *après*.

Les pratiques de Kube sont basées sur Google mais également sur beaucoup d'autres suggestions de la communauté.

L'organe de certification de la CNCF permet d'assurer que toutes les solutions qui sont vendues sont compatibles.

On a un cluster ETCD qui sert a la persistence de l'état. 

L'API server est en somme un "gardien du temple" qui s'assure de la validité des requêtes émises.

### De la virtualisation à la conteneurisation

L'ancienne façon (old way) de déployer des applications consistait à installer les applications sur un hôte en utilisant les systèmes de gestions de paquets natifs. Cela avait pour principale inconvénient de lier fortement les exécutables, la configuration, les librairies et le cycle de vie de chacun avec l'OS. Il est bien entendu possible de construire une image de machine virtuelle (VM) immuable pour arriver à produire des publications (rollouts) ou retours arrières (rollbacks), mais les VMs sont lourdes et non-portables.

La nouvelle façon (new way) consiste à déployer des conteneurs basés sur une virtualisation au niveau du système d'opération (operation-system-level) plutôt que de la virtualisation hardware. Ces conteneurs sont isolés les uns des autres et de l'hôte : ils ont leurs propres systèmes de fichiers, ne peuvent voir que leurs propres processus et leur usage des ressources peut être contraint. Ils sont aussi plus faciles à construire que des VMs, et vu qu'ils sont décorrélés de l'infrastructure sous-jacente et du système de fichiers de l'hôte, ils sont aussi portables entre les différents fournisseurs de Cloud et les OS.

### Solutions d'installation (MiniKube, On-Premise, etc.)

#### K8S certified providers

[Marketplace Solutions](https://unofficial-kubernetes.readthedocs.io/en/latest/setup/pick-right-solution)

[Kubernetes Training Partners](https://kubernetes.io/training/)

<img src="https://K8s-mise-en-oeuvre.github.io/docs/ccnf-landscape.jpg" alt="CCNF Landscape" width="900" height="720">

<img src="https://K8s-mise-en-oeuvre.github.io/docs/landscape-kuber-min-1.jpg
" alt="CCNF Landscape 2" width="900" height="720">

##### Managed Solutions

Quelques solutions parmi les plus représentées: 

- GCP GKE
- AWS EKS
- Azure AKS
- Platform9 (KUBE2GO)
- Pivotal
- Rancher
- Tectonic by CoreOS
- Stackpoint.io

##### Le cas OVH: implémentation d'offre KAAS basée sur OpenStack et Platform9 solution

Dans le cadre de la Kubecon 2019, OVH annonce le déploiement de Kubernetes sur ses serveurs dédiés suite à un partenariat avec **Platform9**. Par conséquent, il est désormais le fournisseur européen qui propose le choix le plus diversifié en matière de déploiement Kubernetes.

##### Le cas Bare-metal

En bare-metal l'expérience montre qu'avec CoreOs racheté par Redhat et intégré a Openshift, cette solution ainsi que Pivotal et Rancher sont les plus représentées sur le marché français.

#### Solutions locales

##### kind: Kubernetes dans Docker

Sous licence Apache, le projet kind (« Kubernetes in Docker») est un peu différent. Il déploie les nodes sous forme de conteneurs – chaque cluster étant identifié par une étiquette Docker. Son usage est donc plutôt orienté sur le test que le développement.

Le cœur fonctionnel est codé en Go. L’installation de base se fonde sur une image minimale d’Ubuntu. Elle contient les outils statiques nécessaires à Kubernetes, ajoute un point d’entrée sur chaque nœud pour réaliser des actions avant l’amorçage et paramètre un service systemd pour gérer la journalisation. Il en existe des variantes « étendues », contenant des tarballs (archives d’images) que le cluster chargera avant d’exécuter kubeadm.

L’installation est nettement plus lourde qu’avec les options sus-évoquées. Sur Mac comme sur Windows, il faut au minimum 6 Go de RAM (8 recommandés) pour la VM Docker. Il n’existe pas encore d’images Arm.

Parmi les autres limites, kind ne gère pas les applications à état. Il ne fonctionne par ailleurs pas dans la sandbox Linux de Chrome OS et ne prend pas en charge les conteneurs Windows. Depuis peu, il permet d’utiliser, côté hôte, docker et podman en rootless. L’implémentation réseau par défaut repose sur un module propre au projet (kindnetd). Associable à MetalLB pour la répartition de charge, il supporte l’IPv6, y compris en mode dual-stack.

##### Minikube et MicroK8

Pas encore d’IPv6 sur Minikube, le « Kubernetes light officiel ». Sa première release remonte à 2016. Il couvre aujourd’hui Windows, Mac et Linux, sur x86-64. Son principe : exécuter des clusters à nœud unique dans une VM. Ses exigences : au minimum 2 CPU, 2 Go de RAM et 20 Go de disque. On peut aussi l’utiliser en remplacement de Docker Desktop, sans lancer Kubernetes.

Comme k0s, MicroK8s (AMD64, ARM64) peut faire l’objet d’un support commercial – 10 ans de maintenance – par son éditeur. Au-delà du développement local, il cible l’IoT et les environnements minimaux d’intégration continue. Canonical le distribue sous forme de Snap (et d’exécutable sur Windows) qui exécute nativement l’orchestrateur (sans VM).

De base, MicroK8s inclut le serveur d’API, le planificateur, le kubelet, l’interface CNI et kube-proxy. On peut l’étendre avec une trentaine d’add-on dont Istio, Kubeflow, OpenFaaS, Prometheus et l’opérateur NVIDIA. Configuration minimale recommandée: 4 Go de RAM et 20 Go de disque sur Linux avec l’installation par défaut (40 Go de disque sur Windows et Mac). Autre possibilité: installer MicroK8s dans un conteneur LXD. Comme Minikube, on est sur du déploiement de clusters à un seul nœud. Mais il est possible d’en ajouter et d’activer la haute disponibilité – avec DQlite en datastore.

### Installation et configuration de docker

**Ce guide part du postulat que les machines clientes pour cette formation qui seront amenées a utiliser un runtime docker tournent sous Windows, tandis que les outils seront installés dans un container linux.**

L'installation de docker réfèrera donc au docker-desktop.

Cliquez sur le lient de téléchargement ci-dessous et suivez les étapes mentionnées: [Install Docker Desktop](https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe)

### Accéder au cluster Kubernetes: CLI (kubectl), GUI (dashboard) et APIs

Pour accéder au cluster nous aurons besoin de récupérer au préalable le **KUBECONFIG** de celui-ci. La récupération des données propres a générer, reconstruire ou accéder à un kubeconfig existant dépendra de la solution via laquelle le cluster a été déployé.

Voici un ensemble d'outils nécessaires sinon indispensables à une utilisation de Kubernetes en ligne de commande: 

#### Kubectl

`kubectl` est le client CLI écrit en go pour Kubernetes. 

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

Valider le téléchargement du binaire via la checksum:

```bash
curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"

echo "$(<kubectl.sha256)  kubectl" | sha256sum --check
```

Si la vérification est conforme:

```bash
kubectl: OK
```

Si la vérification échoue, sha256 se termine avec un statut non nul et imprime une sortie similaire à:

```bash
kubectl: FAILED
sha256sum: WARNING: 1 computed checksum did NOT match
```

Installer kubectl:

```bash
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

Si vous n'avez pas l'accès root sur le système cible, vous pouvez quand même installer kubectl dans le répertoire *~/.local/bin*:

```bash
chmod +x kubectl
mkdir -p ~/.local/bin/kubectl
mv ./kubectl ~/.local/bin/kubectl
# and then add ~/.local/bin/kubectl to $PATH
```

Vérifier l'installation:

```bash
kubectl version --client
```

#### Krew, plugin manager for Kubernetes

```bash
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  KREW="krew-${OS}_${ARCH}" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
  tar zxvf "${KREW}.tar.gz" &&
  ./"${KREW}" install krew
)

cat << EOF >> ~/.bashrc 
export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
EOF 
```

#### Kubectx + Kubens

```bash
kubectl krew install ctx
kubectl krew install ns
```

#### Konfig

Le plugin konfig va nous servir a exporter une configuration cluster présente dans un fichier de structure kubeconfig et le copier dans *~/.kube/config*. De cette façon, les binaires kubectx ou kubie auront accès au contexte et nous pourrons ainsi passer d'un contexte à l'autre sans jamais passer la variable d'environnment **KUBECONFIG**. 

```bash
kubectl krew install konfig
konfig import /tmp/custom-cluster-KuBeConFig
konfig import --save /tmp/custom-cluster-KuBeConFig
kubie ctx custom-cluster
kubectl get po -A
```

#### Kubie

Réunit les commandes `kubectx` + `kubens` dans un seul binaire (et peut permettre par exemple d'exécuter des commandes extérieures au contexte kube actuel).

```bash
curl -sS https://github.com/sbstp/kubie/releases/download/v0.16.0/kubie-linux-amd64 | bash
chmod +x kubie-linux-amd64
mv kubie-linux-amd64 /usr/local/bin/kubie
```

Quelques exemples concernant l'utilisation de Kubie:

Changer de contexte courant:
```bash
kubie ctx kolo@kolorado.us-east-2.eksctl.io
```

Afficher les namespaces du contexte courant:
```bash
kubie ns
```

Exécuter une commande kubectl sur un contexte accessible:
```bash
kubie exec kolo@kolorado.us-east-2.eksctl.io kube-system kubectl get pods
```

#### K9s

Visualiseur en shell console pour Kubernetes:

```bash
curl -sS https://webinstall.dev/k9s | bash

echo "export PATH='/home/pbackz/.local/bin:$PATH'" >> ~/.bashrc
```

```bash
# List all available CLI options
k9s help
# To get info about K9s runtime (logs, configs, etc..)
k9s info
# To run K9s in a given namespace
k9s -n mycoolns
# Start K9s in an existing KubeConfig context
k9s --context coolCtx
# Start K9s in readonly mode - with all cluster modification commands disabled
k9s --readonly
```

#### GUI

Kubernetes fournit nativement des dashboards.

Cependant, l'interface utilisateur du tableau de bord n'est pas déployée par défaut. Pour la déployer, exécutez la commande suivante:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended.yaml
```

Pour protéger vos données dans le cluster, le tableau de bord se déploie avec une configuration RBAC minimale par défaut. Actuellement, le tableau de bord prend uniquement en charge la connexion avec un token de support. 

Vous pouvez accéder au tableau de bord à l'aide de la commande suivante:

```bash
kubectl proxy
```

Kubectl mettra le tableau de bord à disposition à l'adresse suivante: http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/.

Vous ne pouvez accéder à l'interface utilisateur que depuis la machine sur laquelle la commande est exécutée. Voir kubectl `proxy --help` pour plus d'options.

*N.B.* La méthode d'authentification Kubeconfig ne prend pas en charge les fournisseurs d'identité externes ni l'authentification basée sur un certificat x509. 

Nous y préférerons aujourd'hui l'outil Kubernetes Lens (https://k8slens.dev/), disposant de plus de fonctionnalités.

<img src="k8s-lens.png" alt="k8s-lens" width="900" height="720">

#### APIs

De nombreux sdks conçus pour la majorité des langages permettent également d'accéder à un cluster Kubernetes, d'en visualiser, lister ou manipuler les ressources via des apis.

Toutefois, nous ne saurions que trop conseiller l'utilisation du langage Golang dans lequel Kubernetes est écrit pour manipuler ces APIs ou créer des opérateurs.

### Déploiement, publication manuelle et introspection du deploiement

> Un Deployment fournit des mises à jour déclaratives pour Pods et ReplicaSets.
Vous décrivez un état désiré dans un déploiement et le controlleur déploiement change l'état réel à l'état souhaité à un rythme contrôlé. Vous pouvez définir des Deployments pour créer de nouveaux ReplicaSets, ou pour supprimer des déploiements existants et adopter toutes leurs ressources avec de nouveaux déploiements.

Voici un exemple de déploiement. Il crée un ReplicaSet pour faire apparaître trois pods nginx:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
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
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

Dans cet exemple:

Un déploiement nommé nginx-deployment est créé, indiqué par le champ **.metadata.name**.

Le déploiement crée trois pods répliqués, indiqués par le champ replicas.

Le champ selector définit comment le déploiement trouve les pods à gérer. Dans ce cas, vous sélectionnez simplement un label définie dans le template de pod (app:nginx). Cependant, des règles de sélection plus sophistiquées sont possibles, tant que le modèle de pod satisfait lui-même la règle.

> *N.B.* Le champ matchLabels est une table de hash {clé, valeur}. Une seule {clé, valeur} dans la table matchLabels est équivalente à un élément de matchExpressions, dont le champ clé est "clé", l'opérateur est "In" et le tableau de valeurs contient uniquement "valeur". Toutes les exigences, à la fois de matchLabels et de matchExpressions, doivent être satisfaites pour correspondre.

Les Pods reçoivent le label **app:nginx** dans le champ labels.

La spécification du template de pod dans le champ **.template.spec**, indique que les pods exécutent un conteneur, nginx, qui utilise l'image nginx Docker Hub à la version 1.7.9.

Créez un conteneur et nommez-le nginx en utilisant le champ **name**.

Créez le déploiement en exécutant la commande suivante:

> *N.B.* Vous pouvez spécifier l'indicateur **--record** pour écrire la commande exécutée dans l'annotation de ressource kubernetes.io/change-cause. 

C'est utile pour une future introspection. Par exemple, pour voir les commandes exécutées dans chaque révision de déploiement.

```bash
kubectl apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml
```

Exécutez `kubectl get deployments` pour vérifier si le déploiement a été créé. 

Si le déploiement est toujours en cours de création, la sortie est similaire à:

```bash
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   0/3     0            0           1s
```

Lorsque vous inspectez les déploiements de votre cluster, les champs suivants s'affichent:

- **NAME** - répertorie les noms des déploiements dans le cluster.

- **DESIRED** - affiche le nombre souhaité de répliques de l'application, que vous définissez lorsque vous créez le déploiement. C'est l'état désiré.

- **CURRENT** - affiche le nombre de réplicas en cours d'exécution.

- **UP-TO-DATE** - affiche le nombre de réplicas qui ont été mises à jour pour atteindre l'état souhaité.

- **AVAILABLE** - affiche le nombre de réplicas de l'application disponibles pour vos utilisateurs.

- **AGE** - affiche la durée d'exécution de l'application.


Notez que le nombre de réplicas souhaitées est de 3 selon le champ **.spec.replicas**

Pour voir l'état du déploiement, exécutez:

```bash
kubectl rollout status deployment.v1.apps/nginx-deployment

Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
deployment "nginx-deployment" successfully rolled out
```

Exécutez à nouveau kubectl get deployments quelques secondes plus tard. La sortie est similaire à ceci:

```bash
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           18s
```

Notez que le déploiement a créé les trois répliques et que toutes les répliques sont à jour (elles contiennent le dernier modèle de pod) et disponibles.

Pour voir le ReplicaSet (**rs**) créé par le déploiement, exécutez `kubectl get rs`. 

La sortie est similaire à ceci:

```bash
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-75675f5897   3         3         3       18s
```

**Notez que le nom du ReplicaSet est toujours formaté comme ceci: "[DEPLOYMENT-NAME]-[RANDOM-STRING]".** 

La chaîne aléatoire est générée aléatoirement et utilise le pod-template-hash.

Pour voir les labels générées automatiquement pour chaque Pod, exécutez: 

```bash
kubectl get pods --show-labels

NAME                                READY     STATUS    RESTARTS   AGE       LABELS
nginx-deployment-75675f5897-7ci7o   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
nginx-deployment-75675f5897-kzszj   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
nginx-deployment-75675f5897-qqcnn   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
```

Le ReplicaSet créé garantit qu'il y a trois pods nginx.

> *N.B.* Vous devez spécifier un sélecteur approprié et des labels de template de pod dans un déploiement (dans ce cas, app: nginx). Ne superposez pas les étiquettes ou les sélecteurs avec d'autres contrôleurs (y compris d'autres déploiements et StatefulSets). Kubernetes n'empêche pas les chevauchements de noms, et si plusieurs contrôleurs ont des sélecteurs qui se chevauchent, ces contrôleurs peuvent entrer en conflit et se comporter de façon inattendue. 

### Déploiement d'une plateforme de test (TP)

Dans ce TP vous devrez mettre en place une plateforme de test locale via l'une des solutions suivantes: Minikube, Kind, MicroK8S ou encore Docker Desktop.

Vous créerez des pods disposant d'un label 'test'.

Vous devrez pouvoir lister les pods ainsi que leurs labels et accéder au fichier de configuration du cluster (kubeconfig).