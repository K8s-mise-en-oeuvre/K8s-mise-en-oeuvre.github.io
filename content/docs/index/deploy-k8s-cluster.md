+++
title = "Déploiement d'un cluster Kubernetes "
description = ""
date = 2022-01-01T08:00:00+00:00
updated = 2022-01-01T08:00:00+00:00
draft = false
weight = 16
sort_by = "weight"
template = "docs/page.html"

[extra]
toc = true
top = false
+++

## Déploiement d'un cluster Kubernetes 

### Déploiement d'un master-nodeadm, d'un master-node, d'un worker-node

Choisissez un add-on réseau pour les pods et vérifiez s’il nécessite des arguments à passer à l'initialisation de kubeadm. **Selon le fournisseur tiers que vous choisissez, vous devrez peut-être définir le --pod-network-cidr sur une valeur spécifique au fournisseur**.

Maintenant, lancez:

```bash
kubeadm init <args>
```

Pour plus d'informations sur les arguments de `kubeadm init`, voir le guide de référence kubeadm.

Pour une liste complète des options de configuration, voir la documentation du fichier de configuration.

Pour personnaliser les composants du control plane, y compris l'affectation facultative d'IPv6 à la sonde liveness, pour les composants du control plane et du serveur etcd, fournissez des arguments supplémentaires à chaque composant, comme indiqué dans les arguments personnalisés.

Pour lancer encore une fois `kubeadm init`, vous devez d'abord détruire le cluster.

> *N.B.* Si vous joignez un node avec une architecture différente par rapport à votre cluster, créez un Déploiement ou DaemonSet pour kube-proxy et kube-dns sur le node. C’est nécéssaire car les images Docker pour ces composants ne prennent actuellement pas en charge la multi-architecture.

`kubeadm init` exécute d’abord une série de vérifications préalables pour s’assurer que la machine est prête à exécuter Kubernetes. Ces vérifications préalables exposent des avertissements et se terminent en cas d'erreur. Ensuite kubeadm init télécharge et installe les composants du control plane du cluster. 

Cela peut prendre plusieurs minutes. l'output devrait ressembler à:

```bash
[init] Using Kubernetes version: vX.Y.Z
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Activating the kubelet service
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [kubeadm-master localhost] and IPs [10.138.0.4 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [kubeadm-master localhost] and IPs [10.138.0.4 127.0.0.1 ::1]
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubeadm-master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.138.0.4]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 31.501735 seconds
[uploadconfig] storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-X.Y" in namespace kube-system with the configuration for the kubelets in the cluster
[patchnode] Uploading the CRI Socket information "/var/run/dockershim.sock" to the Node API object "kubeadm-master" as an annotation
[mark-control-plane] Marking the node kubeadm-master as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node kubeadm-master as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: <token>
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstraptoken] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstraptoken] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstraptoken] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstraptoken] creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/fr/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join <master-ip>:<master-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

Pour que kubectl fonctionne pour votre utilisateur non root, exécutez ces commandes, qui font également partie du resultat de la commande `kubeadm init`:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Alternativement, si vous êtes root, vous pouvez exécuter:

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```

Faites un enregistrement du retour de la commande `kubeadm join` que `kubeadm init` génère. Vous avez besoin de cette commande pour joindre des nodes à votre cluster.

Le token est utilisé pour l'authentification mutuelle entre le master et les nœuds qui veulent le rejoindre. Le token est secret. Gardez-le en sécurité, parce que n'importe qui avec ce token peut ajouter des node authentifiés à votre cluster. Ces tokens peuvent être listés, créés et supprimés avec la commande `kubeadm token`.

### Mise en place du dashboard et du réseau

Vous devez installer un add-on réseau pour pod afin que vos pods puissent communiquer les uns avec les autres.

Le réseau doit être déployé avant toute application. De plus, CoreDNS ne démarrera pas avant l’installation d’un réseau. kubeadm ne prend en charge que les réseaux basés sur un CNI (et ne prend pas en charge kubenet).

Plusieurs projets fournissent des réseaux de pod Kubernetes utilisant CNI, dont certains supportent les network policies. Allez voir la page des add-ons pour une liste complète des add-ons réseau disponibles.

> Le support IPv6 a été ajouté dans CNI v0.6.0.
CNI bridge et local-ipam sont les seuls plug-ins de réseau IPv6 pris en charge dans Kubernetes version 1.9.

**Notez que kubeadm configure un cluster sécurisé par défaut et impose l’utilisation de RBAC. Assurez-vous que votre manifeste de réseau prend en charge RBAC.**

Veuillez également à ce que votre réseau Pod ne se superpose à aucun des réseaux hôtes, car cela pourrait entraîner des problèmes. Si vous constatez une collision entre le réseau de pod de votre plug-in de réseau et certains de vos réseaux hôtes, vous devriez penser à un remplacement de CIDR approprié et l'utiliser lors de `kubeadm init` avec `--pod-network-cidr` et en remplacement du YAML de votre plugin réseau. 

Vous ne pouvez installer qu'un seul réseau de pod par cluster.

Choisissez-en un parmi les suivants:

- Calico
- Canal
- Cilium
- Flannel
- Kube-router
- Romana
- Weave Net
- JuniperContrail/TungstenFabric

Pour notre part nous ferons le choix de Calico qui est majoritairement utilisé.

Pour plus d'informations sur l'utilisation de Calico, voir Guide de démarrage rapide de Calico sur Kubernetes, Installation de Calico pour les netpols (network policies) et le réseau, ainsi que d'autres resources liées à ce sujet.

[Quickstart Calico on K8s](https://projectcalico.docs.tigera.io/getting-started/kubernetes/quickstart)

[Calico Home](https://www.calicolabs.com/)

> Pour que Calico fonctionne correctement, vous devez passer **--pod-network-cidr = 192.168.0.0 / 16 à kubeadm init ou mettre à jour le fichier calico.yml pour qu'il corresponde à votre réseau de Pod.** Notez que Calico fonctionne uniquement sur amd64, arm64, ppc64le et s390x.

```bash
kubectl apply -f https://docs.projectcalico.org/v3.8/manifests/calico.yaml
```

**Une fois qu'un réseau de pod a été installé, vous pouvez vérifier qu'il fonctionne en vérifiant que le pod CoreDNS est en cours d’exécution** dans l'output de `kubectl get pods --all-namespaces`. Et une fois que le pod CoreDNS est opérationnel, vous pouvez continuer en joignant vos nœuds.

Si votre réseau ne fonctionne pas ou si CoreDNS n'est pas en cours d'exécution, vérifiez notre documentation de dépannage.

Par défaut, votre cluster ne déploie pas de pods sur le master pour des raisons de sécurité. Si vous souhaitez pouvoir déployer des pods sur le master, par exemple, pour un cluster Kubernetes mono-machine pour le développement, exécutez:

```bash
kubectl taint nodes --all node-role.kubernetes.io/master-

node "test-01" untainted
taint "node-role.kubernetes.io/master:" not found
taint "node-role.kubernetes.io/master:" not found
```

Cela supprimera la marque **node-role.kubernetes.io/master** de tous les nodes qui l'ont, y compris du node master, ce qui signifie que le scheduler sera alors capable de déployer des pods partout.

Les nodes sont ceux sur lesquels vos workloads (conteneurs, pods, etc.) sont exécutés. Pour ajouter de nouveaux nodes à votre cluster, procédez comme suit pour chaque machine:

- SSH vers la machine
- Devenir root (par exemple, `sudo su-`)
- Exécutez la commande qui a été récupérée sur l'output de `kubeadm init`. 

Par exemple:

```bash
kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
```

Si vous n'avez pas le jeton, vous pouvez l'obtenir en exécutant la commande suivante sur le node master:

```bash
kubeadm token list
```

Par défaut, les jetons expirent après 24 heures. Si vous joignez un node au cluster après l’expiration du jeton actuel, vous pouvez créer un nouveau jeton en exécutant la commande suivante sur le node master:


```bash
kubeadm token create

5didvk.d09sbcov8ph2amjw
```

Si vous n'avez pas la valeur **--discovery-token-ca-cert-hash**, vous pouvez l'obtenir en exécutant la suite de commande suivante sur le node master:

```bash
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
   openssl dgst -sha256 -hex | sed 's/^.* //'

8cb2de97839780a412b93877f8507ad6c94f73add17d5d7058e91741c9d5ec78

[preflight] Running pre-flight checks

... (log output of join workflow) ...

Node join complete:
* Certificate signing request sent to master and response
  received.
* Kubelet informed of new secure connection details.

Run 'kubectl get nodes' on the master to see this machine join.
```

Quelques secondes plus tard, vous remarquerez ce node dans l'output de `kubectl get nodes`.

Enfin, nous allons installer la dashboard UI:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.4.0/aio/deploy/recommended.yaml
```

### Déploiement d'un cluster (TP)

Dans ce TP vous aurez à charge de spawner un cluster avec kubeadm/eksctl ou les 2 sur un environnement AWS.

Nous devrons au prélable nous assurer de l'existence de 3 VM accessibles.

#### Installation manuelle avec kubeadm

Suivez les étapes de la documentation ci-dessous [Create cluster from scratch with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)

Puis, valider le bon fonctionnement suite à l'installation.

```bash
controlplane $ kubectl cluster-info

Kubernetes master is running at https://172.17.0.10:6443
KubeDNS is running at https://172.17.0.10:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

# Validate resources visibility after installation
kubectl get pods --all-namespaces -o wide
```

Enfin, détruisez le cluster:

Pour annuler ce que kubeadm a fait, vous devez d’abord drainer le node et assurez-vous que le node est vide avant de l'arrêter. 

En communiquant avec le master en utilisant les informations d'identification appropriées, exécutez:

```bash
kubectl drain <node name> --delete-local-data --force --ignore-daemonsets
kubectl delete node <node name>
```

Ensuite, sur le node en cours de suppression, réinitialisez l'état de tout ce qui concerne kubeadm:

```bash
kubeadm reset
```

Le processus de réinitialisation ne réinitialise pas et ne nettoie pas les règles iptables ni les tables IPVS. Si vous souhaitez réinitialiser iptables, vous devez le faire manuellement:

```bash
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
```

Si vous souhaitez réinitialiser les tables IPVS, vous devez exécuter la commande suivante:

```bash
ipvsadm -C
```

Si vous souhaitez recommencer Il suffit de lancer `kubeadm init` ou `kubeadm join` avec les arguments appropriés.

#### Installation sur AWS EKS avec eksctl

Plusieurs types de configuration pour les nodes qui seront spawn pour l'installation du cluster:

- Fargate - Linux - Sélectionnez ce type de node si vous souhaitez exécuter des applications Linux sur AWS Fargate. Fargate est un moteur de calcul sans serveur qui vous permet de déployer des pods Kubernetes sans gérer les instances Amazon EC2.

- nodes gérés - Linux - sélectionnez ce type de node si vous souhaitez exécuter des applications Amazon Linux sur des instances Amazon EC2. Bien que cela ne soit pas abordé dans ce guide, vous pouvez également ajouter des nodes autogérés et Bottlerocket Windows à votre cluster.

##### Installation d'eksctl

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

sudo mv /tmp/eksctl /usr/local/bin
```

##### Création d'un cluster avec eksctl sur des nodes de type fargate

```bash
eksctl create cluster --name kolorado --region us-east-2 --fargate

2021-12-14 02:35:13 [ℹ]  eksctl version 0.76.0
2021-12-14 02:35:13 [ℹ]  using region us-east-2
2021-12-14 02:35:13 [ℹ]  setting availability zones to [us-east-2a us-east-2b us-east-2c]
2021-12-14 02:35:13 [ℹ]  subnets for us-east-2a - public:192.168.0.0/19 private:192.168.96.0/19
2021-12-14 02:35:13 [ℹ]  subnets for us-east-2b - public:192.168.32.0/19 private:192.168.128.0/19
2021-12-14 02:35:13 [ℹ]  subnets for us-east-2c - public:192.168.64.0/19 private:192.168.160.0/19
2021-12-14 02:35:13 [ℹ]  using Kubernetes version 1.21
2021-12-14 02:35:13 [ℹ]  creating EKS cluster "kolorado" in "us-east-2" region with Fargate profile
2021-12-14 02:35:13 [ℹ]  if you encounter any issues, check CloudFormation console or try 'eksctl utils describe-stacks --region=us-east-2 --cluster=kolorado'
2021-12-14 02:35:13 [ℹ]  CloudWatch logging will not be enabled for cluster "kolorado" in "us-east-2"
2021-12-14 02:35:13 [ℹ]  you can enable it with 'eksctl utils update-cluster-logging --enable-types={SPECIFY-YOUR-LOG-TYPES-HERE (e.g. all)} --region=us-east-2 --cluster=kolorado'
2021-12-14 02:35:13 [ℹ]  Kubernetes API endpoint access will use default of {publicAccess=true, privateAccess=false} for cluster "kolorado" in "us-east-2"
2021-12-14 02:35:13 [ℹ]
2 sequential tasks: { create cluster control plane "kolorado",
    2 sequential sub-tasks: {
        wait for control plane to become ready,
        create fargate profiles,
    }
}
2021-12-14 02:35:13 [ℹ]  building cluster stack "eksctl-kolorado-cluster"
2021-12-14 02:35:14 [ℹ]  deploying stack "eksctl-kolorado-cluster"
2021-12-14 02:35:44 [ℹ]  waiting for CloudFormation stack "eksctl-kolorado-cluster"
2021-12-14 02:36:14 [ℹ]  waiting for CloudFormation stack "eksctl-kolorado-cluster"
2021-12-14 02:37:15 [ℹ]  waiting for CloudFormation stack "eksctl-kolorado-cluster"
2021-12-14 02:38:15 [ℹ]  waiting for CloudFormation stack "eksctl-kolorado-cluster"
2021-12-14 02:39:16 [ℹ]  waiting for CloudFormation stack "eksctl-kolorado-cluster"
2021-12-14 02:40:16 [ℹ]  waiting for CloudFormation stack "eksctl-kolorado-cluster"
2021-12-14 02:41:16 [ℹ]  waiting for CloudFormation stack "eksctl-kolorado-cluster"
2021-12-14 02:42:17 [ℹ]  waiting for CloudFormation stack "eksctl-kolorado-cluster"
2021-12-14 02:43:17 [ℹ]  waiting for CloudFormation stack "eksctl-kolorado-cluster"
2021-12-14 02:44:18 [ℹ]  waiting for CloudFormation stack "eksctl-kolorado-cluster"
2021-12-14 02:45:18 [ℹ]  waiting for CloudFormation stack "eksctl-kolorado-cluster"
2021-12-14 02:46:19 [ℹ]  waiting for CloudFormation stack "eksctl-kolorado-cluster"
2021-12-14 02:47:19 [ℹ]  waiting for CloudFormation stack "eksctl-kolorado-cluster"
2021-12-14 02:48:19 [ℹ]  waiting for CloudFormation stack "eksctl-kolorado-cluster"
2021-12-14 02:50:22 [ℹ]  creating Fargate profile "fp-default" on EKS cluster "kolorado"
2021-12-14 02:54:41 [ℹ]  created Fargate profile "fp-default" on EKS cluster "kolorado"
2021-12-14 02:57:13 [ℹ]  "coredns" is now schedulable onto Fargate
2021-12-14 02:58:17 [ℹ]  "coredns" is now scheduled onto Fargate
2021-12-14 02:58:17 [ℹ]  "coredns" pods are now scheduled onto Fargate
2021-12-14 02:58:17 [ℹ]  waiting for the control plane availability...
2021-12-14 02:58:17 [✔]  saved kubeconfig as "/home/pbackz/.kube/config"
2021-12-14 02:58:17 [ℹ]  no tasks
2021-12-14 02:58:17 [✔]  all EKS cluster resources for "kolorado" have been created
2021-12-14 02:58:19 [ℹ]  kubectl command should work with "/home/pbackz/.kube/config", try 'kubectl get nodes'
2021-12-14 02:58:19 [✔]  EKS cluster "kolorado" in "us-east-2" region is ready
```

```bash
# Validate resources visibility after installation
kubectl get pods --all-namespaces -o wide

# Show config
kubectl config view
```

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://D574D5657EED6B65990285B60D420EBB.gr7.us-east-2.eks.amazonaws.com
  name: kolorado.us-east-2.eksctl.io
contexts:
- context:
    cluster: kolorado.us-east-2.eksctl.io
    user: kolo@kolorado.us-east-2.eksctl.io
  name: kolo@kolorado.us-east-2.eksctl.io
current-context: kolo@kolorado.us-east-2.eksctl.io
kind: Config
preferences: {}
users:
- name: kolo@kolorado.us-east-2.eksctl.io
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1alpha1
      args:
      - eks
      - get-token
      - --cluster-name
      - kolorado
      - --region
      - us-east-2
      command: aws
      env:
      - name: AWS_STS_REGIONAL_ENDPOINTS
        value: regional
      interactiveMode: IfAvailable
      provideClusterInfo: false
```

Puis détruire le cluster:

```bash
# Delete cluster
eksctl delete cluster --name kolorado --region us-east-2
```

[eksctl examples](https://github.com/weaveworks/eksctl/tree/main/examples)

