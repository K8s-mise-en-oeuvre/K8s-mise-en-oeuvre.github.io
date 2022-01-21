+++
title = "Les fichiers descriptifs"
description = ""
date = 2022-01-01T08:00:00+00:00
updated = 2022-01-01T08:00:00+00:00
draft = false
weight = 10
sort_by = "weight"
template = "docs/page.html"

[extra]
toc = true
top = false
+++

## Les fichiers descriptifs

### Syntaxe YAML

Le YAML, qu'est-ce que c'est ?

> YAML, acronyme de Yet Another Markup Language dans sa version 1.01, devient l'acronyme récursif de YAML Ain't Markup Language (« YAML n’est pas un langage de balisage ») dans sa version 1.12, est un format de représentation de données par sérialisation Unicode. Il reprend des concepts d'autres langages comme XML, ou encore du format de message électronique tel que documenté par RFC 2822. YAML a été proposé par Clark Evans en 20013, et implémenté par ses soins ainsi que par Brian Ingerson et Oren Ben-Kiki.
Son objet est de représenter des informations plus élaborées que le simple CSV en gardant cependant une lisibilité presque comparable, et bien plus grande en tout cas que du XML.

*Source* [YAML definition](https://fr.wikipedia.org/wiki/YAML)

L'essentiel de la syntaxe YAML:

```yaml
---  # document start

# Comments in YAML look like this.

################
# SCALAR TYPES #
################

# Our root object (which continues for the entire document) will be a map,
# which is equivalent to a dictionary, hash or object in other languages.
key: value
another_key: Another value goes here.
a_number_value: 100
scientific_notation: 1e+12
# The number 1 will be interpreted as a number, not a boolean. if you want
# it to be interpreted as a boolean, use true
boolean: true
null_value: null
key with spaces: value
# Notice that strings don't need to be quoted. However, they can be.
however: 'A string, enclosed in quotes.'
'Keys can be quoted too.': "Useful if you want to put a ':' in your key."
single quotes: 'have ''one'' escape pattern'
double quotes: "have many: \", \0, \t, \u263A, \x0d\x0a == \r\n, and more."
# UTF-8/16/32 characters need to be encoded
Superscript two: \u00B2

# Multiple-line strings can be written either as a 'literal block' (using |),
# or a 'folded block' (using '>').
literal_block: |
    This entire block of text will be the value of the 'literal_block' key,
    with line breaks being preserved.

    The literal continues until de-dented, and the leading indentation is
    stripped.

        Any lines that are 'more-indented' keep the rest of their indentation -
        these lines will be indented by 4 spaces.
folded_style: >
    This entire block of text will be the value of 'folded_style', but this
    time, all newlines will be replaced with a single space.

    Blank lines, like above, are converted to a newline character.

        'More-indented' lines keep their newlines, too -
        this text will appear over two lines.

####################
# COLLECTION TYPES #
####################

# Nesting uses indentation. 2 space indent is preferred (but not required).
a_nested_map:
  key: value
  another_key: Another Value
  another_nested_map:
    hello: hello

# Maps don't have to have string keys.
0.25: a float key

# Keys can also be complex, like multi-line objects
# We use ? followed by a space to indicate the start of a complex key.
? |
  This is a key
  that has multiple lines
: and this is its value

# YAML also allows mapping between sequences with the complex key syntax
# Some language parsers might complain
# An example
? - Manchester United
  - Real Madrid
: [2001-01-01, 2002-02-02]

# Sequences (equivalent to lists or arrays) look like this
# (note that the '-' counts as indentation):
a_sequence:
  - Item 1
  - Item 2
  - 0.5  # sequences can contain disparate types.
  - Item 4
  - key: value
    another_key: another_value
  -
    - This is a sequence
    - inside another sequence
  - - - Nested sequence indicators
      - can be collapsed

# Since YAML is a superset of JSON, you can also write JSON-style maps and
# sequences:
json_map: {"key": "value"}
json_seq: [3, 2, 1, "takeoff"]
and quotes are optional: {key: [3, 2, 1, takeoff]}

#######################
# EXTRA YAML FEATURES #
#######################

# YAML also has a handy feature called 'anchors', which let you easily duplicate
# content across your document. Both of these keys will have the same value:
anchored_content: &anchor_name This string will appear as the value of two keys.
other_anchor: *anchor_name

# Anchors can be used to duplicate/inherit properties
base: &base
  name: Everyone has same name

# The regexp << is called Merge Key Language-Independent Type. It is used to
# indicate that all the keys of one or more specified maps should be inserted
# into the current map.

foo:
  <<: *base
  age: 10

bar:
  <<: *base
  age: 20

# foo and bar would also have name: Everyone has same name

# YAML also has tags, which you can use to explicitly declare types.
explicit_string: !!str 0.5
# Some parsers implement language specific tags, like this one for Python's
# complex number type.
python_complex_number: !!python/complex 1+2j

# We can also use yaml complex keys with language specific tags
? !!python/tuple [5, 7]
: Fifty Seven
# Would be {(5, 7): 'Fifty Seven'} in Python

####################
# EXTRA YAML TYPES #
####################

# Strings and numbers aren't the only scalars that YAML can understand.
# ISO-formatted date and datetime literals are also parsed.
datetime: 2001-12-15T02:59:43.1Z
datetime_with_spaces: 2001-12-14 21:59:43.10 -5
date: 2002-12-14

# The !!binary tag indicates that a string is actually a base64-encoded
# representation of a binary blob.
gif_file: !!binary |
  R0lGODlhDAAMAIQAAP//9/X17unp5WZmZgAAAOfn515eXvPz7Y6OjuDg4J+fn5
  OTk6enp56enmlpaWNjY6Ojo4SEhP/++f/++f/++f/++f/++f/++f/++f/++f/+
  +f/++f/++f/++f/++f/++SH+Dk1hZGUgd2l0aCBHSU1QACwAAAAADAAMAAAFLC
  AgjoEwnuNAFOhpEMTRiggcz4BNJHrv/zCFcLiwMWYNG84BwwEeECcgggoBADs=

# YAML also has a set type, which looks like this:
set:
  ? item1
  ? item2
  ? item3
or: {item1, item2, item3}

# Sets are just maps with null values; the above is equivalent to:
set2:
  item1: null
  item2: null
  item3: null

...  # document end
```

*Source* [Learn YAML in 10 minutes](https://learnxinyminutes.com/docs/yaml/)

### Scalabilité d'un déploiement

Vous pouvez mettre à l'échelle un déploiement en utilisant la commande suivante:

```bash
kubectl scale deployment/nginx-deployment --replicas=10

deployment.apps/nginx-deployment scaled
```

En supposant que l'autoscaling horizontal des pods est activé dans votre cluster, **vous pouvez configurer un autoscaler pour votre déploiement** et choisir le nombre minimum et maximum de pods que vous souhaitez exécuter en fonction de l'utilisation du CPU de vos pods existants.

```bash
kubectl autoscale deployment/nginx-deployment --min=10 --max=15 --cpu-percent=80

deployment.apps/nginx-deployment scaled
```

Les déploiements RollingUpdate permettent d'exécuter plusieurs versions d'une application en même temps. 
Lorsque vous ou un autoscaler mettez à l'échelle un déploiement RollingUpdate qui est au milieu d'un déploiement (en cours ou en pause), le contrôleur de déploiement équilibre les replicas supplémentaires dans les ReplicaSets actifs existants (ReplicaSets avec Pods) afin d'atténuer le risque. 

C'est ce qu'on appelle la mise à l'échelle proportionnelle.

Par exemple, vous exécutez un déploiement avec 10 replicas, maxSurge=3 et maxUnavailable=2.

Assurez-vous que les 10 replicas de votre déploiement sont running.

```bash
kubectl get deploy

NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment     10        10        10           10          50s
```

Vous effectuez une mise à jour vers une nouvelle image qui se trouve être non résoluble depuis l'intérieur du cluster.

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:sometag

deployment.apps/nginx-deployment image updated
```

La mise à jour de l'image lance un nouveau déploiement avec ReplicaSet nginx-deployment-1989198191, mais il est bloqué en raison de l'exigence maxUnavailable que vous avez mentionnée ci-dessus. 

Vérifiez l'état du déploiement.

```bash
kubectl get rs

NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-1989198191   5         5         0         9s
nginx-deployment-618515232    8         8         8         1m
```

Puis une nouvelle demande de mise à l'échelle pour que le déploiement arrive. 
L'autoscaler incrémente les replicas du déploiement à 15. Le contrôleur de déploiement doit décider où ajouter ces 5 nouveaux replicas. 

Si vous n'utilisiez pas la mise à l'échelle proportionnelle, les 5 replicas seraient ajoutés dans le nouveau ReplicaSet. Avec la mise à l'échelle proportionnelle, vous répartissez les replicas supplémentaires sur tous les ReplicaSets. 
Les proportions les plus importantes vont aux ReplicaSets ayant le plus de replicas et les proportions les plus faibles vont aux ReplicaSets ayant moins de replicas. Les restes sont ajoutés au ReplicaSet ayant le plus de replicas. 

Les ReplicaSets avec zéro replica ne sont pas mis à l'échelle.

Dans notre exemple ci-dessus, 3 replicas sont ajoutées à l'ancien ReplicaSet et 2 replicas sont ajoutées au nouveau ReplicaSet. Le processus de déploiement devrait finalement déplacer toutes les replicas vers le nouveau ReplicaSet, en supposant que les nouveaux replicas soient valides. 

Pour confirmer cela, exécutez:

```bash
kubectl get deploy

NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment     15        18        7            8           7m
```

Le statut de déploiement confirme comment les replicas ont été ajoutés à chaque ReplicaSet.

```bash
kubectl get rs

NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-1989198191   7         7         0         7m
nginx-deployment-618515232    11        11        11        7m
```

### Stratégie de mise à jour sans interruption

> *N.B.* Le re-déploiement d'un déploiement est déclenché si et seulement si le modèle de pod du déploiement (c'est-à-dire **.spec.template**) est modifié, par exemple si les labels ou les images de conteneur du template sont mis à jour. D'autres mises à jour, telles que la mise à l'échelle du déploiement, ne déclenchent pas de rollout.

Suivez les étapes ci-dessous pour mettre à jour votre déploiement:

Mettons à jour les pods nginx pour utiliser l'image nginx: 1.9.1 au lieu de l'image nginx: 1.7.9.

kubectl --record deployment.apps/nginx-deployment set image deployment.v1.apps/nginx-deployment nginx=nginx:1.9.1

ou utilisez la commande suivante:

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1 --record

deployment.apps/nginx-deployment image updated
```

Alternativement, vous pouvez éditer le déploiement et changer `.spec.template.spec.containers[0].image` de 'nginx: 1.7.9' à 'nginx: 1.9.1':

```bash
kubectl edit deployment.v1.apps/nginx-deployment

deployment.apps/nginx-deployment edited
```

Pour voir l'état du déploiement, exécutez:

```bash
kubectl rollout status deployment.v1.apps/nginx-deployment

Waiting for rollout to finish: 2 out of 3 new replicas have been updated...

# OR

deployment "nginx-deployment" successfully rolled out
```

Obtenez plus de détails sur votre déploiement mis à jour:

Une fois le déploiement réussi, vous pouvez afficher le déploiement en exécutant `kubectl get deployments.` La sortie est similaire à ceci:

```bash
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           36s
```

Exécutez kubectl get rs pour voir que le déploiement a mis à jour les pods en créant un nouveau ReplicaSet et en le redimensionnant jusqu'à 3 replicas, ainsi qu'en réduisant l'ancien ReplicaSet à 0 réplicas.

```bash
kubectl get rs

NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-1564180365   3         3         3       6s
nginx-deployment-2035384211   0         0         0       36s
```

L'exécution de `kubectl get pods` ne devrait désormais afficher que les nouveaux pods:

```bash
NAME                                READY     STATUS    RESTARTS   AGE
nginx-deployment-1564180365-khku8   1/1       Running   0          14s
nginx-deployment-1564180365-nacti   1/1       Running   0          14s
nginx-deployment-1564180365-z9gth   1/1       Running   0          14s
```

La prochaine fois que vous souhaitez mettre à jour ces pods, il vous suffit de mettre à jour le modèle de pod de déploiement à nouveau.

**Le déploiement garantit que seul un certain nombre de pods sont en panne pendant leur mise à jour. Par défaut, il garantit qu'au moins 75% du nombre souhaité de pods sont en place (25% max indisponible).**

Le déploiement garantit également que seul un certain nombre de pods sont créés au-dessus du nombre souhaité de pods. **Par défaut, il garantit qu'au plus 125% du nombre de pods souhaité sont en hausse (surtension maximale de 25%).**

Par exemple, si vous regardez attentivement le déploiement ci-dessus, vous verrez qu'il a d'abord créé un nouveau pod, puis supprimé certains anciens pods et en a créé de nouveaux. Il ne tue pas les anciens Pods tant qu'un nombre suffisant de nouveaux Pods n'est pas apparu, et ne crée pas de nouveaux Pods tant qu'un nombre suffisant de Pods anciens n'a pas été tué. Il s'assure qu'au moins 2 pods sont disponibles et qu'au maximum 4 pods au total sont disponibles.

Obtenez les détails de votre déploiement:

```bash
kubectl describe deployments

Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Thu, 30 Nov 2021 10:56:25 +0000
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision=2
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
    Labels:  app=nginx
    Containers:
    nginx:
        Image:        nginx:1.9.1
        Port:         80/TCP
        Environment:  <none>
        Mounts:       <none>
    Volumes:        <none>
    Conditions:
    Type           Status  Reason
    ----           ------  ------
    Available      True    MinimumReplicasAvailable
    Progressing    True    NewReplicaSetAvailable
    OldReplicaSets:  <none>
    NewReplicaSet:   nginx-deployment-1564180365 (3/3 replicas created)
    Events:
    Type    Reason             Age   From                   Message
    ----    ------             ----  ----                   -------
    Normal  ScalingReplicaSet  2m    deployment-controller  Scaled up replica set nginx-deployment-2035384211 to 3
    Normal  ScalingReplicaSet  24s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 1
    Normal  ScalingReplicaSet  22s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 2
    Normal  ScalingReplicaSet  22s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 2
    Normal  ScalingReplicaSet  19s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 1
    Normal  ScalingReplicaSet  19s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 3
    Normal  ScalingReplicaSet  14s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 0
```

Ici, vous voyez que lorsque vous avez créé le déploiement pour la première fois, il a créé un ReplicaSet (nginx-deployment-2035384211) et l'a mis à l'échelle directement jusqu'à 3 replicas. 

Lorsque vous avez mis à jour le déploiement, il a créé un nouveau ReplicaSet (nginx-deployment-1564180365) et l'a mis à l'échelle jusqu'à 1, puis a réduit l'ancien ReplicaSet à 2, de sorte qu'au moins 2 pods étaient disponibles et au plus 4 pods ont été créés à chaque fois. 

Il a ensuite poursuivi la montée en charge du nouveau et de l'ancien ReplicaSet, avec la même stratégie de mise à jour continue. 

Enfin, vous aurez 3 réplicas disponibles dans le nouveau ReplicaSet, et l'ancien ReplicaSet est réduit à 0.

### Revenir à une révision précédente

Suivez les étapes ci-dessous pour restaurer le déploiement de la version actuelle à la version précédente, qui est la version 2.

Vous avez maintenant décidé d'annuler le déploiement actuel et le retour à la révision précédente:

```bash
kubectl rollout undo deployment.v1.apps/nginx-deployment

deployment.apps/nginx-deployment
```

Alternativement, vous pouvez revenir à une révision spécifique en la spécifiant avec **--to-revision**:

```bash
kubectl rollout undo deployment.v1.apps/nginx-deployment --to-revision=2

deployment.apps/nginx-deployment
```

Pour plus de détails sur les commandes liées au déploiement, lisez `kubectl rollout`.

Le déploiement est maintenant rétabli à une précédente révision stable. Comme vous pouvez le voir, **un événement DeploymentRollback pour revenir à la révision 2 est généré à partir du contrôleur de déploiement**.

Vérifiez si la restauration a réussi et que le déploiement s'exécute comme prévu, exécutez:

```bash
kubectl get deployment nginx-deployment

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           30m
```

Obtenez la description du déploiement:

```bash
kubectl describe deployment nginx-deployment

Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Sun, 02 Sep 2021 18:17:55 -0500
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision=4
                        kubernetes.io/change-cause=kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.9.1 --record=true
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.9.1
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-c4747d96c (3/3 replicas created)
Events:
  Type    Reason              Age   From                   Message
  ----    ------              ----  ----                   -------
  Normal  ScalingReplicaSet   12m   deployment-controller  Scaled up replica set nginx-deployment-75675f5897 to 3
  Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-c4747d96c to 1
  Normal  ScalingReplicaSet   11m   deployment-controller  Scaled down replica set nginx-deployment-75675f5897 to 2
  Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-c4747d96c to 2
  Normal  ScalingReplicaSet   11m   deployment-controller  Scaled down replica set nginx-deployment-75675f5897 to 1
  Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-c4747d96c to 3
  Normal  ScalingReplicaSet   11m   deployment-controller  Scaled down replica set nginx-deployment-75675f5897 to 0
  Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-595696685f to 1
  Normal  DeploymentRollback  15s   deployment-controller  Rolled back deployment "nginx-deployment" to revision 2
  Normal  ScalingReplicaSet   15s   deployment-controller  Scaled down replica set nginx-deployment
```

### Suppression d'un déploiement

```bash
kubectl delete deploy deployment.apps/nginx-deployment

# or

kubectl delete deploy nginx-deployment
```

### Publication et analyse d'un déploiement (TP)

#### Partie 1

#### Partie 2

#### Partie 3
