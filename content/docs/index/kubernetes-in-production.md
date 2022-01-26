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

*FEATURE STATE: Kubernetes v1.1 [beta]*
Avant de commencer à utiliser un Ingress, vous devez comprendre qu'un Ingress est une ressource en "version Beta".

Un Ingress peut être configuré pour donner aux services des URLs accessibles de l'extérieur, un équilibrage du trafic de charge externe, la terminaison SSL/TLS et un hébergement virtuel basé sur le nom. Un contrôleur d'Ingress est responsable de l'exécution de l'Ingress, généralement avec un load-balancer (équilibreur de charge), bien qu'il puisse également configurer votre routeur périphérique ou des interfaces supplémentaires pour aider à gérer le trafic.

Un Ingress n'expose pas de ports ni de protocoles arbitraires. **Exposer des services autres que HTTP et HTTPS à Internet généralement utilise un service de type Service.Type=NodePort ou Service.Type=LoadBalancer.**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /testpath
        pathType: Prefix
        backend:
          service:
            name: test
            port:
              number: 80
```

La spécification de la ressource Ingress dispose de toutes les informations nécessaires pour configurer un loadbalancer ou un serveur proxy. Plus important encore, il contient une liste de règles d'appariement de toutes les demandes entrantes. La ressource Ingress ne supporte que les règles pour diriger le trafic HTTP.


Pour 2 vhosts sur un ingress:
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: name-virtual-host-ingress
spec:
  rules:
  - host: service-a.example.com
    http:
      paths:
      - backend:
          serviceName: service-a
          servicePort: 80
  - host: service-b.example.com
    http:
      paths:
      - backend:
          serviceName: service-a
          servicePort: 80
```

### Limitation de ressources

Lorsque Kubernetes planifie un Pod, il est important que les conteneurs disposent de suffisamment de ressources pour fonctionner. Si vous planifiez une grosse application sur un node aux ressources limitées, il est possible que le node soit à court de mémoire ou de ressources CPU et que les pods cessent de fonctionner.

> Les requêtes et les limites sont les mécanismes utilisés par Kubernetes pour contrôler les ressources telles que le CPU et la mémoire. Les requêtes sont ce que le conteneur est assuré d'obtenir.

Si un conteneur demande une ressource, Kubernetes ne le programmera que sur un node qui peut lui fournir cette ressource. Les limites, quant à elles, garantissent qu'un conteneur ne dépasse jamais une certaine valeur. Le conteneur est seulement autorisé à aller jusqu'à la limite, et ensuite il est restreint.

Il est important de se rappeler que la limite ne peut jamais être inférieure à la demande. Si vous essayez de le faire, Kubernetes émettra une erreur et ne vous laissera pas exécuter le conteneur.

Les demandes et les limites sont établies par conteneur. Si les pods ne contiennent généralement qu'un seul conteneur, il est courant de voir des pods avec plusieurs conteneurs. Chaque conteneur du pod reçoit sa propre limite et sa propre demande, mais comme les pods sont toujours planifiés en tant que groupe, vous devez additionner les limites et les demandes de chaque conteneur pour obtenir une valeur globale pour le pod.

Il existe deux types de ressources : 

- Le processeur et la mémoire.

Le scheduler de Kubernetes les utilisent pour déterminer où exécuter vos pods.

Une spécification typique de ressources pour un pod peut ressembler à ceci. Ce pod possède deux conteneurs :

```yaml
containers:
  - name: container1
    image: busybox
    resources:
      requests:
        memory: "32Mi"
        cpu: "200m"
      limits:
      memory: "64Mi"
      cpu: "200m"
  - name: container2
    image: busybox
    resources:
      requests:
        memory: "96Mi"
        cpu: "300m"
      limits: 
        memory: "192Mi"
        cpu: "750m"
```

Chaque conteneur du pod peut définir ses propres demandes et limites, qui sont toutes additives. Ainsi, dans l'exemple ci-dessus, le Pod a une demande totale de 500 m de CPU et 128 MiB de mémoire, et une limite totale de 1 CPU et 256MiB de mémoire.

*CPU*
Les ressources CPU sont définies en millicores. Si votre conteneur a besoin de deux cœurs complets pour fonctionner, vous mettez la valeur "2000m". Si votre conteneur n'a besoin que de ¼ de cœur, vous mettrez une valeur de "250m".

Une chose à garder à l'esprit concernant les demandes de CPU est que si vous mettez une valeur supérieure au nombre de cœurs de votre plus gros node, votre pod ne sera jamais programmé. Disons que vous avez un pod qui a besoin de quatre cœurs, mais que votre cluster Kubernetes est composé de VM à double cœur - votre pod ne sera jamais programmé !

À moins que votre application ne soit spécifiquement conçue pour tirer parti de plusieurs cœurs (l'informatique scientifique et certaines bases de données viennent à l'esprit), il est généralement préférable de maintenir la demande de CPU à '1' ou moins, et d'exécuter plus de répliques pour la faire évoluer. Cela donne au système plus de flexibilité et de fiabilité.

*RAM*
Les ressources mémoire sont définies en octets. Normalement, vous donnez une valeur de mebibytes pour la mémoire (c'est en fait la même chose qu'un mégaoctet), mais vous pouvez donner n'importe quelle valeur en octets.

Comme pour le CPU, si vous introduisez une demande de mémoire supérieure à la quantité de mémoire disponible sur vos nodes, le pod ne sera jamais programmé.

Contrairement aux ressources du CPU, la mémoire ne peut pas être compressée. Comme il n'y a aucun moyen de limiter l'utilisation de la mémoire, si un conteneur dépasse sa limite de mémoire, il sera terminé. Si votre pod est géré par un Deployment, StatefulSet, DaemonSet ou un autre type de contrôleur, le contrôleur lance un remplacement.

*Nodes*
Il est important de se rappeler que vous ne pouvez pas définir des demandes plus importantes que les ressources fournies par vos nodes. Par exemple, si vous avez un cluster de machines à deux cœurs, un Pod avec une demande de 2,5 cœurs ne sera jamais scheduler.

### Service Discovery (env, DNS)

> Dans Kubernetes, la détection de services est mise en œuvre avec des noms de service générés automatiquement correspondant à l'adresse IP du service. Les noms de service suivent la spécification standard suivante: my-svc.my-namespace.svc.cluster-domain.example. Les pods peuvent également accéder à des services externes, tels que example.com, via leur nom. Pour en savoir plus sur le fonctionnement du DNS dans Kubernetes, consultez la page [DNS pour les services et les pods](https://github.com/kubernetes/dns/blob/master/docs/specification.md).

Kubernetes fournit les options DNS de cluster suivantes pour résoudre les noms de service et les noms externes :

kube-dns: module complémentaire de cluster déployé par défaut.
Cloud DNS: infrastructure DNS de cluster gérée dans le cloud exploitant Cloud DNS et ne nécessitant pas la gestion des serveurs DNS dans les clusters, comme kube-dns.
Vous pouvez également enregistrer vos services dans l'annuaire des services pour votre cloud provider.

GKE fournit également le module complémentaire facultatif NodeLocal DNSCache pouvant être utilisé avec kube-dns ou Cloud DNS.

*kube-dns*
> **kube-dns est le fournisseur DNS par défaut des clusters**. Il s'exécute comme un déploiement qui programme les pods kube-dns sur les nodes du cluster.

*Cloud DNS*
> Cloud DNS fournit la résolution DNS des pods et des services, sans nécessiter de fournisseur DNS hébergé par le cluster tel que kube-dns. Le contrôleur Cloud DNS provisionne automatiquement les enregistrements DNS des pods et des services dans Cloud DNS pour les adresses ClusterIP, les services sans adresse IP de cluster et les services de noms externes.

*NodeLocal DNSCache*
> NodeLocal DNSCache s'exécute en tant que DaemonSet qui programme un pod de cache DNS sur chaque node de cluster. Ce cache DNS améliore la latence de la résolution DNS, harmonise les délais des résolutions DNS, et peut réduire le nombre de requêtes DNS adressées à kube-dns ou Cloud DNS.

#### Détection de services en dehors d'un cluster

Vous pouvez configurer la détection de services sur plusieurs clusters à l'aide de l'une des méthodes suivantes.

*VPC Cloud DNS Scope*
Un cluster qui utilise Cloud DNS pour les services DNS de cluster doit s'exécuter dans l'un des deux modes disponibles: champ d'application de cluster ou champ d'application de cloud privé virtuel (VPC).

Lorsque vous configurez un cluster avec un champ d'application de cluster, les enregistrements DNS ne peuvent être résolus que dans le cluster à l'aide du schéma `<svc>.<ns>.svc.cluster.local`. **Ce comportement est le même que celui de kube-dns**.

Lorsque vous configurez un cluster avec un champ d'application de VPC, les enregistrements DNS des services du cluster peuvent être résolus dans l'ensemble du VPC. Cela signifie que les clients du même VPC ou connectés à celui-ci via Cloud VPN ou Cloud Interconnect peuvent résoudre directement les enregistrements DNS des services du cluster.

*Services multiclusters*
> Les services multiclusters fournissent la détection de services et l'équilibrage de charge multiclusters pour GKE qui exploite l'objet Service existant. Les services multiclusters sont visibles et accessibles sur n'importe quel cluster GKE doté d'une seule adresse IP virtuelle. Ce comportement est identique à celui d'un service ClusterIP accessible dans un seul cluster.

Les services multiclusters agrègent les services entre les clusters et les rendent adressables sous la forme d'un seul enregistrement DNS multicluster à l'aide du schéma `<svc>.<ns>.svc.clusterset.local`. 

*Sources*

[GKE Service Discovery](https://cloud.google.com/kubernetes-engine/docs/concepts/service-discovery#:~:text=In%20Kubernetes%2C%20service%20discovery%20is%20implemented%20with%20automatically,external%20services%20through%20their%20names%2C%20such%20as%20example.com.)

### Les namespaces et les quotas

Après avoir créé des namespaces, vous pouvez les verrouiller à l'aide de ResourceQuotas. Les quotas de ressources sont très puissants, mais voyons simplement comment les utiliser pour limiter l'utilisation des ressources CPU et mémoire.

Un quota de ressource peut ressembler à ceci:

```yaml
apiVersion: v1
kind: ResourceQuotas
metadata:
  name: sample
spec:
  hard:
  requests.cpu: 500m
  requests.memory: 100Mib
  limits.cpu: 700m
  limits.memory: 500Mib
```

En regardant cet exemple, vous pouvez voir qu'il y a quatre sections. La configuration de chacune de ces sections est facultative.

*requests.cpu* 
> Nombre maximum de demandes combinées de CPU en millicores pour tous les conteneurs de le namespace. Dans l'exemple ci-dessus, vous pouvez avoir 50 conteneurs avec des demandes de 10m, cinq conteneurs avec des demandes de 100m, ou même un conteneur avec une demande de 500m. Tant que le total des demandes de CPU dans le namespace est inférieur à 500m !

*requests.memory* 
> Demandes maximales combinées de mémoire pour tous les conteneurs dans le namespace. Dans l'exemple ci-dessus, vous pouvez avoir 50 conteneurs avec des demandes de 2MiB, cinq conteneurs avec des demandes de CPU de 20MiB, ou même un seul conteneur avec une demande de 100MiB. Tant que la mémoire totale demandée dans le namespace est inférieure à 100MiB !

*limits.cpu* 
> Limites maximales combinées du CPU pour tous les conteneurs dans le namespace. C'est comme requests.cpu mais pour la limite.

*limits.memory*
> Limites maximales de mémoire combinées pour tous les conteneurs dans le namespace. C'est comme requests.memory mais pour la limite.

Si vous utilisez un Namespace de production et de développement (par opposition à un Namespace par équipe ou service), un modèle commun est de ne pas mettre de quota sur le Namespace de production et des quotas stricts sur le Namespace de développement. Cela permet à la production de prendre toutes les ressources dont elle a besoin en cas de pic de trafic.

*LimitRange*
> Vous pouvez également créer une LimitRange dans votre Namespace. Contrairement à un quota, qui concerne le namespace dans son ensemble, une LimitRange s'applique à un conteneur individuel. Cela peut aider à empêcher les gens de créer des conteneurs super petits ou super grands dans le namespace.

Une **LimitRange** peut ressembler à ceci :

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: sample
spec:
  limits:
- default:
    cpu: 600m
    memory: 100Mib
  defaultRequest:
    cpu: 100Mib
    memory: 50Mib
  max:
   cpu: 1000m
   memory: 200Mib
  min:
   cpu: 10m 
   memory: 100Mib
  type: Container
```

En regardant cet exemple, vous pouvez voir qu'il y a quatre sections. Là encore, la définition de chacune de ces sections est facultative

*Section default* 
> La section default définit les limites par défaut pour un conteneur dans un pod. Si vous définissez ces valeurs dans la section limitRange, tous les conteneurs qui ne les définissent pas explicitement se verront attribuer les valeurs par défaut.

*Section defaultRequest* 
> La section defaultRequest définit les demandes par défaut pour un conteneur dans un pod. Si vous définissez ces valeurs dans l'intervalle limitRange, les valeurs par défaut seront attribuées à tous les conteneurs qui ne les définissent pas explicitement.

*Section max*
> La section max définit les limites maximales qu'un conteneur dans un pod peut fixer. La section par défaut ne peut être supérieure à cette valeur. De même, les limites définies pour un conteneur ne peuvent être supérieures à cette valeur. Il est important de noter que si cette valeur est définie et que la section par défaut ne l'est pas, tous les conteneurs qui ne définissent pas explicitement ces valeurs eux-mêmes se verront attribuer les valeurs max comme limite.

*Section min*
> La section min définit les requêtes minimales qu'un conteneur dans un Pod peut définir. La section defaultRequest ne peut être inférieure à cette valeur. De même, les demandes définies sur un conteneur ne peuvent être inférieures à cette valeur. Il est important de noter que si cette valeur est définie et que la section defaultRequest ne l'est pas, la valeur min devient également la valeur defaultRequest.

### Gestion des accès (RBAC)

> Le contrôle d'accès basé sur les rôles (RBAC) est une méthode de régulation de l'accès aux ressources informatiques ou réseau en fonction des rôles des utilisateurs individuels au sein de votre organisation.

> L'autorisation RBAC utilise le groupe d'API `rbac.authorization.k8s.io` pour piloter les décisions d'autorisation, ce qui vous permet de configurer dynamiquement les politiques via l'API Kubernetes.

Pour activer RBAC, démarrez le serveur API avec l'indicateur `--authorization-mode` défini sur une liste séparée par des virgules qui inclut RBAC; par exemple:

```bash
kube-apiserver --authorization-mode=Example,RBAC --other-options --more-options
```

**L'API RBAC déclare quatre types d'objets Kubernetes: Role, ClusterRole, RoleBinding et ClusterRoleBinding**. Vous pouvez décrire les objets, ou les modifier, à l'aide d'outils tels que kubectl, comme tout autre objet Kubernetes.

*Attention !* Ces objets, par conception, imposent des restrictions d'accès. Si vous apportez des modifications à un cluster au fur et à mesure de votre apprentissage, consultez la section Prévention de l'escalade des privilèges et amorçage pour comprendre comment ces restrictions peuvent vous empêcher d'effectuer certaines modifications.

**Rôle et ClusterRole**
Un rôle RBAC ou un ClusterRole contient des règles qui représentent un ensemble de permissions. Les permissions sont purement additives (il n'existe pas de règles de "refus").

Un rôle définit toujours les permissions dans un espace de nom particulier; lorsque vous créez un rôle, vous devez spécifier le namespace auquel il appartient.

**ClusterRole**, en revanche, est une ressource sans namespaces. Les ressources ont des noms différents (Role et ClusterRole) parce qu'un objet Kubernetes doit toujours être soit namespaced soit non namespaced; il ne peut pas être les deux.

Les ClusterRoles ont plusieurs usages. Vous pouvez utiliser un ClusterRole pour:

- Définir des autorisations sur des ressources à espace de noms et les accorder au sein d'un ou de plusieurs espaces de noms individuels.
- Définir des autorisations sur des ressources d'espace de noms et les accorder dans tous les espaces de noms
- Définir des permissions sur des ressources à l'échelle du cluster

Si vous souhaitez définir un rôle au sein d'un espace de noms, utilisez un Role ; si vous souhaitez définir un rôle à l'échelle du cluster, utilisez un ClusterRole.

*Exemple de rôle*

Voici un exemple de rôle dans l'espace de nom "default" qui peut être utilisé pour accorder un accès en lecture aux pods:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # "namespace" omitted since ClusterRoles are not namespaced
  name: secret-reader
rules:
- apiGroups: [""]
  #
  # at the HTTP level, the name of the resource for accessing Secret
  # objects is "secrets"
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```

*Exemple de ClusterRole*

> Un ClusterRole peut être utilisée pour accorder les mêmes permissions qu'un rôle. Étant donné que les ClusterRoles sont adaptés aux clusters, vous pouvez également les utiliser pour accorder l'accès à:

- des ressources à l'échelle du cluster (comme les nodes)
- des points de terminaison non liés aux ressources (comme /healthz sur un healthcheck)
- des ressources namespaces (comme les pods), dans tous les namespaces.

Par exemple: vous pouvez utiliser une ClusterRole pour autoriser un utilisateur particulier à exécuter `kubectl get pods --all-namespaces`.

Voici un exemple de ClusterRole qui peut être utilisé pour accorder un accès en lecture aux secrets dans un espace de noms particulier ou dans tous les espaces de noms (selon la façon dont il est lié) :

```yaml

```

Le nom d'un objet Role ou ClusterRole doit être un nom de segment de chemin valide.

*RoleBinding et ClusterRoleBinding*

> Un role binding accorde les permissions définies dans un rôle à un utilisateur ou à un ensemble d'utilisateurs. Il contient une liste de sujets (utilisateurs, groupes ou comptes de service) et une référence au rôle accordé. Un RoleBinding accorde des permissions dans un espace de nom spécifique, tandis qu'un ClusterRoleBinding accorde cet accès à l'échelle du cluster.

> Un RoleBinding peut faire référence à n'importe quel rôle dans le même espace de noms. Par ailleurs, un RoleBinding peut faire référence à un ClusterRole et lier ce ClusterRole à l'espace de nom du RoleBinding. Si vous souhaitez lier un ClusterRole à tous les espaces de noms de votre cluster, vous utilisez un ClusterRoleBinding.

Le nom d'un objet RoleBinding ou ClusterRoleBinding doit être un nom de segment de chemin valide.

*Exemples de RoleBinding*

Voici un exemple de RoleBinding qui accorde le rôle "pod-reader" à l'utilisateur "jane" dans l'espace de nom "default". Ceci permet à "jane" de lire les pods dans l'espace de noms "default".

```yaml
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
# You can specify more than one "subject"
- kind: User
  name: jane # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```

**Un RoleBinding peut également faire référence à un ClusterRole pour accorder les permissions définies dans ce ClusterRole aux ressources de le namespace du RoleBinding. Ce type de référence vous permet de définir un ensemble de rôles communs à l'ensemble de votre cluster, puis de les réutiliser dans plusieurs namespaces.**

Par exemple, même si le RoleBinding suivant fait référence à un ClusterRole, "dave" (le sujet, sensible à la casse) ne pourra lire que les Secrets dans l'espace de nom "development", car le namespace du RoleBinding (dans ses métadonnées) est "development".

```yaml
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "dave" to read secrets in the "development" namespace.
# You need to already have a ClusterRole named "secret-reader".
kind: RoleBinding
metadata:
  name: read-secrets
  #
  # The namespace of the RoleBinding determines where the permissions are granted.
  # This only grants permissions within the "development" namespace.
  namespace: development
subjects:
- kind: User
  name: dave # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

*ClusterRoleBinding example*

Pour accorder des permissions à l'ensemble d'un cluster, vous pouvez utiliser un ClusterRoleBinding. Le ClusterRoleBinding suivant permet à tout utilisateur du groupe "manager" de lire les secrets dans n'importe quel namespace.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "dave" to read secrets in the "development" namespace.
# You need to already have a ClusterRole named "secret-reader".
kind: RoleBinding
metadata:
  name: read-secrets
  #
  # The namespace of the RoleBinding determines where the permissions are granted.
  # This only grants permissions within the "development" namespace.
  namespace: development
subjects:
- kind: User
  name: dave # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

Après avoir créé un lien, vous ne pouvez pas modifier le rôle ou le ClusterRole auquel il fait référence. Si vous essayez de modifier le roleRef d'une liaison, vous obtenez une erreur de validation. Si vous souhaitez modifier le roleRef d'un binding, vous devez supprimer l'objet binding et en créer un autre.

Il y a deux raisons pour cette restriction:

- Rendre le roleRef immuable permet d'accorder à quelqu'un la permission de mettre à jour un objet de liaison existant, afin qu'il puisse gérer la liste des sujets, sans pouvoir changer le rôle qui est accordé à ces sujets.

L'utilitaire de ligne de commande kubectl auth reconcile crée ou met à jour un fichier manifeste contenant les objets RBAC, et gère la suppression et la recréation des objets de liaison si nécessaire pour modifier le rôle auquel ils se réfèrent. Voir l'utilisation de la commande et les exemples pour plus d'informations.

### Haute disponibilité et mode maintenance 

Cette page explique deux approches différentes pour configurer un Kubernetes à haute disponibilité. cluster utilisant kubeadm:

Avec des nodes de control plane empilés. Cette approche nécessite moins d'infrastructure. Les membres etcd et les nodes du control plane sont co-localisés.

Avec un cluster etcd externe cette approche nécessite plus d'infrastructure. Les nodes du control plane et les membres etcd sont séparés.

Avant de poursuivre, vous devez déterminer avec soin quelle approche répond le mieux aux besoins de vos applications et de l'environnement. Cette comparaison décrit les avantages et les inconvénients de chacune.

Vos clusters doivent exécuter Kubernetes version 1.12 ou ultérieure. Vous devriez aussi savoir que la mise en place de clusters HA avec kubeadm est toujours expérimentale et sera simplifiée davantage dans les futures versions. Vous pouvez par exemple rencontrer des problèmes lors de la mise à niveau de vos clusters. Nous vous encourageons à essayer l’une ou l’autre approche et à nous faire part de vos commentaires dans Suivi des problèmes Kubeadm.

Avertissement: Cette page ne traite pas de l'exécution de votre cluster sur un fournisseur de cloud. Dans un environnement Cloud, les approches documentées ici ne fonctionnent ni avec des objets de type load balancer, ni avec des volumes persistants dynamiques.

#### Pré-requis

Pour les deux méthodes, vous avez besoin de cette infrastructure:

- Trois machines qui répondent aux pré-requis des exigences de kubeadm pour les maîtres (masters)
- Trois machines qui répondent aux pré-requis des exigences de kubeadm pour les workers
- Connectivité réseau complète entre toutes les machines du cluster (public ou réseau privé)
- Privilèges sudo sur toutes les machines
- Accès SSH d'une machine à tous les nodes du cluster
- kubeadm et une kubelet installés sur toutes les machines. kubectl est optionnel.

Pour le cluster etcd externe uniquement, vous avez besoin également de:

- Trois machines supplémentaires pour les membres etcd

*N.B.* Les exemples suivants utilisent Calico en tant que fournisseur de réseau de Pod. Si vous utilisez un autre CNI, pensez à remplacer les valeurs par défaut si nécessaire.

*N.B.* Toutes les commandes d'un control plane ou d'un noeud etcd doivent être éxecutées en tant que root.
Certains plugins réseau CNI tels que Calico nécessitent un CIDR tel que 192.168.0.0 / 16 et  certains comme Weave n'en ont pas besoin. Voir la Documentation du CNI réseau. 

Pour ajouter un CIDR de pod, définissez le champ podSubnet: 192.168.0.0 / 16 sous   l'objet networking de ClusterConfiguration.

##### Créez un load balancer pour kube-apiserver

Note: Il existe de nombreuses configurations pour les équilibreurs de charge (load balancers). L'exemple suivant n'est qu'un exemple. Vos exigences pour votre cluster peuvent nécessiter une configuration différente.

**Créez un load balancer kube-apiserver avec un nom résolu en DNS.**

Dans un environnement cloud, placez vos nodes du control plane derrière un load balancer TCP. Ce load balancer distribue le trafic à tous les nodes du control plane sains dans sa liste. La vérification de la bonne santé d'un apiserver est une vérification TCP sur le port que kube-apiserver écoute (valeur par défaut: 6443).

Il n'est pas recommandé d'utiliser une adresse IP directement dans un environnement cloud.

Le load balancer doit pouvoir communiquer avec tous les nodes du control plane sur le port apiserver. Il doit également autoriser le trafic entrant sur son réseau de port d'écoute.

HAProxy peut être utilisé comme load balancer.

Assurez-vous que l'adresse du load balancer correspond toujours à l'adresse de **ControlPlaneEndpoint** de kubeadm.

Ajoutez les premiers nodes du control plane au load balancer et testez la connexion:

```bash
nc -v LOAD_BALANCER_IP PORT
```

Une erreur connection refused est attendue car l'apiserver n'est pas encore en fonctionnement. Cependant, un timeout signifie que le load balancer ne peut pas communiquer avec le node du control plane. Si un timeout survient, reconfigurez le load balancer pour communiquer avec le node du control plane.

Ajouter les nodes du control plane restants au groupe cible du load balancer.

##### Configurer SSH

SSH est requis si vous souhaitez contrôler tous les nodes à partir d'une seule machine.

Activer ssh-agent sur votre machine ayant accès à tous les autres nodes du cluster:

`eval $(ssh-agent)`

Ajoutez votre clé SSH à la session:

```bash
ssh-add ~/.ssh/path_to_private_key
SSH entre les nodes pour vérifier que la connexion fonctionne correctement.
```

Lorsque vous faites un SSH sur un node, `assurez-vous d’ajouter l’option -A`:

`ssh -A 10.0.0.7`

Lorsque vous utilisez sudo sur n’importe quel node, veillez à préserver l’environnement afin que le SSH forwarding fonctionne:

`sudo -E -s`

#### Control plane empilé et nodes etcd

Sur le premier node du control plane, créez un fichier de configuration appelé **kubeadm-config.yaml**:

```yaml
apiVersion: kubeadm.k8s.io/v1beta1
kind: ClusterConfiguration
kubernetesVersion: stable
apiServer:
  certSANs:
  - "LOAD_BALANCER_DNS"
controlPlaneEndpoint: "LOAD_BALANCER_DNS:LOAD_BALANCER_PORT"
```

kubernetesVersion doit représenter la version de Kubernetes à utiliser. Cet exemple utilise stable.
**controlPlaneEndpoint** doit correspondre à l'adresse ou au DNS et au port du load balancer.
Il est recommandé que les versions de kubeadm, kubelet, kubectl et kubernetes correspondent.

Assurez-vous que le node est dans un état sain puis exécuter:

`sudo kubeadm init --config=kubeadm-config.yaml`

Vous pouvez à présent joindre n'importe quelle machine au cluster en lancant la commande suivante sur chaque nœeud en tant que root:

```bash
kubeadm join 192.168.0.200:6443 --token j04n3m.octy8zely83cy2ts --discovery-token-ca-cert-hash    sha256:84938d2a22203a8e56a787ec0c6ddad7bc7dbd52ebabc62fd5f4dbea72b14d1f
```

Copiez ce jeton dans un fichier texte. Vous en aurez besoin plus tard pour joindre d’autres nodes du control plane au cluster.

Activez l'extension CNI Weave:

```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

Tapez ce qui suit et observez les pods des composants démarrer:

```bash
kubectl get pod -n kube-system -w
```

Il est recommandé de ne joindre les nouveaux nodes du control plane qu'après l'initialisation du premier node.

Copiez les fichiers de certificat du premier node du control plane dans les autres:

Dans l'exemple suivant, remplacez **CONTROL_PLANE_IPS** par les adresses IP des autres nodes du control plane.

```bash
USER=ubuntu # customizable
CONTROL_PLANE_IPS="10.0.0.7 10.0.0.8"
for host in ${CONTROL_PLANE_IPS}; do
    scp /etc/kubernetes/pki/ca.crt "${USER}"@$host:
    scp /etc/kubernetes/pki/ca.key "${USER}"@$host:
    scp /etc/kubernetes/pki/sa.key "${USER}"@$host:
    scp /etc/kubernetes/pki/sa.pub "${USER}"@$host:
    scp /etc/kubernetes/pki/front-proxy-ca.crt "${USER}"@$host:
    scp /etc/kubernetes/pki/front-proxy-ca.key "${USER}"@$host:
    scp /etc/kubernetes/pki/etcd/ca.crt "${USER}"@$host:etcd-ca.crt
    scp /etc/kubernetes/pki/etcd/ca.key "${USER}"@$host:etcd-ca.key
    scp /etc/kubernetes/admin.conf "${USER}"@$host:
done
```

Avertissement: N'utilisez que les certificats de la liste ci-dessus. kubeadm se chargera de générer le reste des certificats avec les SANs requis pour les instances du control plane qui se joignent. Si vous copiez tous les certificats par erreur, la création de noeuds supplémentaires pourrait échouer en raison d'un manque de SANs requis.

##### Étapes pour le reste des nodes du control plane

Déplacer les fichiers créés à l'étape précédente où scp était utilisé:

```bash
USER=ubuntu # customizable
mkdir -p /etc/kubernetes/pki/etcd
mv /home/${USER}/ca.crt /etc/kubernetes/pki/
mv /home/${USER}/ca.key /etc/kubernetes/pki/
mv /home/${USER}/sa.pub /etc/kubernetes/pki/
mv /home/${USER}/sa.key /etc/kubernetes/pki/
mv /home/${USER}/front-proxy-ca.crt /etc/kubernetes/pki/
mv /home/${USER}/front-proxy-ca.key /etc/kubernetes/pki/
mv /home/${USER}/etcd-ca.crt /etc/kubernetes/pki/etcd/ca.crt
mv /home/${USER}/etcd-ca.key /etc/kubernetes/pki/etcd/ca.key
mv /home/${USER}/admin.conf /etc/kubernetes/admin.conf
```

Ce processus écrit tous les fichiers demandés dans le dossier **/etc/kubernetes**.

Lancez kubeadm join sur ce node en utilisant la commande de join qui vous avait été précédemment donnée par kubeadm init sur le premier noeud. Ça devrait ressembler a quelque chose comme ça:

```bash
sudo kubeadm join 192.168.0.200:6443 --token j04n3m.octy8zely83cy2ts --discovery-token-ca-cert-hash sha256:84938d2a22203a8e56a787ec0c6ddad7bc7dbd52ebabc62fd5f4dbea72b14d1f --experimental-control-plane
```

Remarquez l'ajout de l'option --experimental-control-plane. Ce paramètre automatise l'adhésion au control plane du cluster.
Tapez ce qui suit et observez les pods des composants démarrer:

```bash
kubectl get pod -n kube-system -w
```

Répétez ces étapes pour le reste des nodes du control plane.

##### Noeuds etcd externes

##### Configurer le cluster etcd

Suivez ces instructions pour configurer le cluster etcd.
- Configurer le premier node du control plane
- Copiez les fichiers suivants de n’importe quel node du cluster etcd vers ce node.:

```bash
export CONTROL_PLANE="ubuntu@10.0.0.7"
+scp /etc/kubernetes/pki/etcd/ca.crt "${CONTROL_PLANE}":
+scp /etc/kubernetes/pki/apiserver-etcd-client.crt "${CONTROL_PLANE}":
+scp /etc/kubernetes/pki/apiserver-etcd-client.key "${CONTROL_PLANE}":
```

Remplacez la valeur de CONTROL_PLANE par l'utilisateur@hostname de cette machine.

Créez un fichier YAML appelé kubeadm-config.yaml avec le contenu suivant:

```yaml
apiVersion: kubeadm.k8s.io/v1beta1
kind: ClusterConfiguration
kubernetesVersion: stable
apiServer:
  certSANs:
  - "LOAD_BALANCER_DNS"
controlPlaneEndpoint: "LOAD_BALANCER_DNS:LOAD_BALANCER_PORT"
etcd:
    external:
        endpoints:
        - https://ETCD_0_IP:2379
        - https://ETCD_1_IP:2379
        - https://ETCD_2_IP:2379
        caFile: /etc/kubernetes/pki/etcd/ca.crt
        certFile: /etc/kubernetes/pki/apiserver-etcd-client.crt
        keyFile: /etc/kubernetes/pki/apiserver-etcd-client.key
La différence entre etcd empilé et externe, c’est que nous utilisons le champ external pour etcd dans la configuration de kubeadm. Dans le cas de la topologie etcd empilée, c'est géré automatiquement.
```

Remplacez les variables suivantes dans le modèle (template) par les valeurs appropriées pour votre cluster:

```bash
LOAD_BALANCER_DNS
LOAD_BALANCER_PORT
ETCD_0_IP
ETCD_1_IP
ETCD_2_IP
```

Lancez `kubeadm init --config kubeadm-config.yaml` sur ce node.

Ecrivez le résultat de la commande de join dans un fichier texte pour une utilisation ultérieure.

Appliquer le plugin CNI Weave:

```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
Étapes pour le reste des nodes du control plane
Pour ajouter le reste des nodes du control plane, suivez ces instructions. Les étapes sont les mêmes que pour la configuration etcd empilée, à l’exception du fait qu'un membre etcd local n'est pas créé.
```

Pour résumer:

- Assurez-vous que le premier node du control plane soit complètement initialisé.
- Copier les certificats entre le premier node du control plane et les autres nodes du control plane.
- Joignez chaque node du control plane à l'aide de la commande de join que vous avez enregistrée dans un fichier texte, puis ajoutez l'option --experimental-control-plane.

Tâches courantes après l'amorçage du control plane

- Installer un réseau de pod
Suivez ces instructions afin d'installer le réseau de pod. Assurez-vous que cela correspond au pod CIDR que vous avez fourni dans le fichier de configuration principal.

- Installer les workers
Chaque node worker peut maintenant être joint au cluster avec la commande renvoyée à partir du resultat de n’importe quelle commande kubeadm init. L'option --experimental-control-plane ne doit pas être ajouté aux nodes workers.

### Gestion des droits user, sa et mise en place de services exposés (TP)

Dans ce TP vous aurez à charge de définir un ensemble de services de type clusterIP ou nodePort ainsi que la gestion des accès des pods. 

Les pods derrière les services devront pouvoir être joints et listés dans l'ensemble du cluster pour un service account donné. Vous utiliserez pour cela un clusterRole et clusterRoleBinding.

2 services devront exposés un port sur le node, 2 autres devront permettre aux pods de communiquer entre eux à l'intérieur du cluster.
