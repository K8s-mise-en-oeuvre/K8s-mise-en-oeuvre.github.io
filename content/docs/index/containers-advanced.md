+++
title = "Gestion avancée des containers"
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

## Gestion avancée des conteneurs 

### Qu'est-ce qu'un container ?

<img src="https://K8s-mise-en-oeuvre.github.io/docs/containers.jpg" alt="containers" width="900" height="720">

> "Caisson métallique parallélépipédique conçu pour le transport de marchandise par différents mode de transports.
> Ses dimensions ont été standardisées au niveau international et il est muni a tous les angles de pièces de préemption
> permettant de l'arrimer ou de le transborder d'un véhicule a l'autre." 
*Définition d'un container maritime*

Nous allons voir que ces propriétés se retrouvent dans la conception informatique du container.

C'est tout simplement faire tourner un ensemble de processus systèmes ou métier au sein d'un même environnement, isolé.

Une des problématique de la virtualisation par hyperviseur est celle de la boîte noire. Difficile de recenser l'ensemble des composants, ce qui peut être critique en cas de panne de la VM, impossible donc d'en déterminer l'état.

Comment faire également lorsque les ressources ne sont pas partagées, comme le cache des applications par exemple ? 

Physiquement, un container est un système de fichiers contenant l'ensemble des binaires dont on a besoin pour faire tourner
notre application.

L'image apporte également des métadonnées rattachées aux layers dont on peut ensuite déterminer la composition exacte.

A la fin j'ai une brique atomique, je n'ai pas besoin de savoir quelles sont les couches successivement empilées du container.

Cela permet de résoudre la problématique du packaging OS (.deb, .rpm, etc.), non agnostique.

*Docker on Linux*
<img src="https://K8s-mise-en-oeuvre.github.io/docs/docker-on-linux.png" alt="docker-on-linux" width="900" height="720">

Les containers sont immutables, ils ne laissent rien derrière eux, ont un format autosufisant.

Les développeurs peuvent passer d'un langage a l'autre pour les applications sans jamais impacter l'infrastructure de PROD.

C'est toujours un 'runner' de container tel containerd qui execute le processus sur l'hôte mais on se fiche désormais de savoir quel est le launcher sous le container (npm, shell, cargo, etc.).

Les containers amènent de l'auditabilité en production.

Autour de cette infrastructure, d'autres composants, tel un orchestrateur peuvent être intégrés.

> Le container runtime est le logiciel responsable de l'exécution des conteneurs.

Kubernetes est compatible avec plusieurs containers runtime: **Docker, containerd, cri-o, rktlet ainsi que toute implémentation de Kubernetes CRI (Container Runtime Interface)**

*Docker and Kubernetes use containerd*
<img src="https://K8s-mise-en-oeuvre.github.io/docs/docker-and-kubernetes-use-containerd.png" alt="docker-and-kubernetes-use-containerd" width="900" height="720">


<img src="https://K8s-mise-en-oeuvre.github.io/docs/containerd-platform.png" alt="containerd-plateform" width="900" height="720">

*Ressources supplémentaires*

Image disposant d'une binary toolbox minimaliste: [Busybox](https://www.busybox.net)

Outil d'introspection des layers des images: [Dive](https://github.com/wagoodman/dive)

*Demo CI dive*
<img src="https://K8s-mise-en-oeuvre.github.io/docs/demo-ci-dive.png" alt="dive-demo" width="900" height="720">


Container runtime de docker: [Containerd](https://containerd.io)

<img src="https://K8s-mise-en-oeuvre.github.io/docs/containerd.webp" alt="containerd" width="900" height="720">


<img src="https://K8s-mise-en-oeuvre.github.io/docs/docker_containerd.png" alt="docker_containerd" width="900" height="720">


<img src="https://K8s-mise-en-oeuvre.github.io/docs/cri.png" alt="CRI" width="900" height="720">


<img src="https://K8s-mise-en-oeuvre.github.io/docs/ctr.png" alt="CTR" width="720" height="600">

### Dockerfile

> Les Dockerfiles contiennent des instructions pour créer une image Docker.Chaque instruction est écrite sur une ligne et est donnée sous la forme <INSTRUCTION><argument(s)>. Les fichiers Docker sont utilisés pour créer des images Docker à l'aide de la commande `docker build `.

Les principales directives pouvant être utilisées dans un Dockerfile sont:

- **FROM** - Il s’agit de la première instruction. Elle est indispensable, car elle définit l’image de base avec le système d’exploitation et l’application portant le reste des couches. **Vous pouvez également spécifier une image de base minimale en utilisant l’instruction `FROM scratch`**.

- **COPY** - Cette commande ajoute dans votre image des fichiers locaux ou distants tels le code source d’une application ou un fichier de configuration. Il existe aussi une commande ADD qui peut être très utile lorsque l’on veut extraire à la volée un tar.gz. ADD permet de faire ceci en une instruction alors que COPY ne permet que de copier la source vers une destination.

- **RUN** - Exécuter avec RUN toute commande souhaitée pour créer une nouvelle couche sur l’image.

- **CMD** - Pour spécifier une commande à exécuter seulement lors du lancement du conteneur. 

#### HelloWorld Dockerfile

Un fichier Dockerfile minimal pourrait ressembler à ceci:

```bash
FROM alpine
CMD ["echo", "Hello Classroom!"]
```

Cela indiquera à Docker de créer une image basée sur Alpine (FROM), une distribution minimale pour les conteneurs et d'exécuter une commande spécifique (CMD) lors de l'exécution de l'image résultante.

Construisez et exécutez-le:

```bash
docker build -t hello .
docker run --rm hello

# Output
Hello Classroom!
```

### Création et automatisation d'images personnalisées 

Exexmple de code applicatif go:

```go
package main

import (
	"net/http"
	"os"

	"github.com/labstack/echo/v4"
	"github.com/labstack/echo/v4/middleware"
)

func main() {

	e := echo.New()

	e.Use(middleware.Logger())
	e.Use(middleware.Recover())

	e.GET("/", func(c echo.Context) error {
		return c.HTML(http.StatusOK, "Hello, Docker! <3")
	})

	e.GET("/ping", func(c echo.Context) error {
		return c.JSON(http.StatusOK, struct{ Status string }{Status: "OK"})
	})

	httpPort := os.Getenv("HTTP_PORT")
	if httpPort == "" {
		httpPort = "8080"
	}

	e.Logger.Fatal(e.Start(":" + httpPort))
}
```

*Dockerfile*
```dockerfile
# syntax=docker/dockerfile:1

FROM golang:1.16-alpine

WORKDIR /app

COPY go.mod ./
COPY go.sum ./
RUN go mod download

COPY *.go ./

RUN go build -o /docker-gs-ping

EXPOSE 8080

CMD [ "/docker-gs-ping" ]
```

Golang build images: [Golang build images](https://docs.docker.com/language/golang/build-images)

Docker multistage building: [Multistage building](https://docs.docker.com/develop/develop-images/multistage-build)


### Déploiement d'une image personnalisée

Init du package applicatif golang:

```bash
go mod init example.com/echo

go: creating new go.mod: module example.com/echo
go: to add module requirements and sums:
        go mod tidy
```

Installation des dépendances:

```bash
go mod tidy

go: finding module for package github.com/labstack/echo/v4/middleware
go: finding module for package github.com/labstack/echo/v4
go: downloading github.com/labstack/echo/v4 v4.6.3
go: downloading github.com/labstack/echo v3.3.10+incompatible
go: found github.com/labstack/echo/v4 in github.com/labstack/echo/v4 v4.6.3
go: found github.com/labstack/echo/v4/middleware in github.com/labstack/echo/v4 v4.6.3
go: downloading golang.org/x/crypto v0.0.0-20210817164053-32db794688a5
go: downloading golang.org/x/net v0.0.0-20210913180222-943fd674d43e
go: downloading github.com/labstack/gommon v0.3.1
go: downloading github.com/golang-jwt/jwt v3.2.2+incompatible
go: downloading github.com/valyala/fasttemplate v1.2.1
go: downloading golang.org/x/time v0.0.0-20201208040808-7e3f01d25324
go: downloading github.com/mattn/go-isatty v0.0.14
go: downloading github.com/mattn/go-colorable v0.1.11
go: downloading github.com/valyala/bytebufferpool v1.0.0
go: downloading golang.org/x/sys v0.0.0-20211103235746-7861aae1554b
```

Build de l'image personnalisée:

```bash
docker build --tag docker-gs-ping .

[+] Building 19.3s (12/12) FINISHED

=> [internal] load build definition from Dockerfile                                                               0.0s
=> => transferring dockerfile: 220B                                                                               0.0s
=> [internal] load .dockerignore                                                                                  0.0s
=> => transferring context: 2B                                                                                    0.0s
=> [internal] load metadata for docker.io/library/golang:1.16-alpine                                              8.0s
=> [internal] load build context                                                                                  0.0s
=> => transferring context: 5.75kB                                                                                0.0s
=> [1/7] FROM docker.io/library/golang:1.16-alpine@sha256:c4b36cff558405c8368be606325261a21a9a3ae9f79dc1bd3fff9f  6.6s
=> => resolve docker.io/library/golang:1.16-alpine@sha256:c4b36cff558405c8368be606325261a21a9a3ae9f79dc1bd3fff9f  0.0s
=> => sha256:c4b36cff558405c8368be606325261a21a9a3ae9f79dc1bd3fff9f831a0411f2 1.65kB / 1.65kB                     0.0s
=> => sha256:802b7b395fac3d9b0ed552be257a3f19739486f6e712b7f5276f76214ad1efa1 1.36kB / 1.36kB                     0.0s
=> => sha256:4bcb0d501de3d5c9d95c65a8ef94ef2d169244bf06970413780748ac6fc7536c 5.20kB / 5.20kB                     0.0s
=> => sha256:59bf1c3509f33515622619af21ed55bbe26d24913cedbca106468a5fb37a50c3 2.82MB / 2.82MB                     0.4s
=> => sha256:666ba61612fd7c93393f9a5bc1751d8a9929e32d51501dba691da9e8232bc87b 282.16kB / 282.16kB                 0.2s
=> => sha256:8ed8ca4862056a130f714accb3538decfa0663fec84e635d8b5a0a3305353dee 155B / 155B                         0.1s
=> => sha256:fa6e261410ce0cebd518eb5ee56b9bb56fa66883992dc54ab7bd25afdca2b521 105.85MB / 105.85MB                 2.9s
=> => sha256:033ff68f5bd26b7f1a7567e74101649886332f6bffed2928025b2dc88a49a909 155B / 155B                         0.5s
=> => extracting sha256:59bf1c3509f33515622619af21ed55bbe26d24913cedbca106468a5fb37a50c3                          0.3s
=> => extracting sha256:666ba61612fd7c93393f9a5bc1751d8a9929e32d51501dba691da9e8232bc87b                          0.1s
=> => extracting sha256:8ed8ca4862056a130f714accb3538decfa0663fec84e635d8b5a0a3305353dee                          0.0s
=> => extracting sha256:fa6e261410ce0cebd518eb5ee56b9bb56fa66883992dc54ab7bd25afdca2b521                          3.4s
=> => extracting sha256:033ff68f5bd26b7f1a7567e74101649886332f6bffed2928025b2dc88a49a909                          0.0s
=> [2/7] WORKDIR /app                                                                                             0.3s
=> [3/7] COPY go.mod ./                                                                                           0.0s
=> [4/7] COPY go.sum ./                                                                                           0.0s
=> [5/7] RUN go mod download                                                                                      1.8s
=> [6/7] COPY *.go ./                                                                                             0.0s
=> [7/7] RUN go build -o /docker-gs-ping                                                                          2.0s
=> exporting to image                                                                                             0.5s
=> => exporting layers                                                                                            0.5s
=> => writing image sha256:d6282f70c0eed3c1f1f5c705c99c64c0aa1ac703d240630a9a0be1b437dc2025                       0.0s
=> => naming to docker.io/library/docker-gs-ping                                                                  0.0s
```

Run de l'image:

```bash
docker run -it docker.io/library/docker-gs-ping

   ____    __
  / __/___/ /  ___
 / _// __/ _ \/ _ \
/___/\__/_//_/\___/ v4.6.3
High performance, minimalist Go web framework
https://echo.labstack.com
____________________________________O/_______
                                    O\
⇨ http server started on [::]:8080

```

### Un conteneur et plusieurs services

> Le processus principal en cours d'exécution d'un conteneur est le ENTRYPOINT et/ou le CMD à la fin du Dockerfile. Il est généralement recommandé de séparer les zones de préoccupation en utilisant un service par conteneur. Ce service peut être divisé en plusieurs processus (par exemple, le serveur Web Apache lance plusieurs processus de travail). Il est acceptable d'avoir plusieurs processus, mais pour tirer le meilleur parti de Docker, évitez qu'un conteneur soit responsable de plusieurs aspects de votre application globale. Vous pouvez connecter plusieurs conteneurs à l'aide de réseaux définis par l'utilisateur et de volumes partagés.

Le processus principal du conteneur est responsable de la gestion de tous les processus qu'il démarre. Dans certains cas, le processus principal n'est pas bien conçu et ne gère pas la "récolte" (l'arrêt) des processus enfants de manière élégante lorsque le conteneur sort. Si votre processus entre dans cette catégorie, vous pouvez utiliser l'option --init lorsque vous exécutez le conteneur. L'option --init insère un minuscule processus d'initialisation dans le conteneur en tant que processus principal, et gère la récupération de tous les processus à la sortie du conteneur. La gestion de ces processus de cette manière est supérieure à l'utilisation d'un processus init complet tel que sysvinit, upstart, ou systemd pour gérer le cycle de vie des processus dans votre conteneur.

Si vous devez exécuter plus d'un service dans un conteneur, vous pouvez le faire de plusieurs manières différentes.

Placez toutes vos commandes dans un script wrapper, avec des informations de test et de débogage. Exécutez le script wrapper comme votre CMD. Voici un exemple très naïf. 

Tout d'abord, le script wrapper :

```bash
#!/bin/bash

# Start the first process
./my_first_process &
  
# Start the second process
./my_second_process &
  
# Wait for any process to exit
wait -n
  
# Exit with status of process that exited first
exit $?
```

Puis, le dockerfile:

```dockerfile
# syntax=docker/dockerfile:1
FROM ubuntu:latest
COPY my_first_process my_first_process
COPY my_second_process my_second_process
COPY my_wrapper_script.sh my_wrapper_script.sh
CMD ./my_wrapper_script.sh
```

Si vous avez un processus principal qui doit démarrer en premier et rester en cours d'exécution, mais que vous avez temporairement besoin de lancer d'autres processus (peut-être pour interagir avec le processus principal), vous pouvez utiliser le contrôle des tâches de bash pour faciliter cela.

```bash
#!/bin/bash
  
# turn on bash's job control
set -m
  
# Start the primary process and put it in the background
./my_main_process &
  
# Start the helper process
./my_helper_process
  
# the my_helper_process might need to know how to wait on the
# primary process to start before it does its work and returns
  
# now we bring the primary process back into the foreground
# and leave it there
fg %1
```

```dockerfile
# syntax=docker/dockerfile:1
FROM ubuntu:latest
COPY my_main_process my_main_process
COPY my_helper_process my_helper_process
COPY my_wrapper_script.sh my_wrapper_script.sh
CMD ./my_wrapper_script.sh
```

Notons toutefois que cette façon de procéder n'est que peu adaptée à une utilisation dans Kubernetes où dans une approche orientée micro-services l'on préferera 1 container pour 1 service exposé.

*Sources*

[Multi Service container](https://docs.docker.com/config/containers/multi-service_container/)

### Création et automatisation d'images personnalisées (TP)

#### Partie 1

Dans ce TP vous aurez à charge de réaliser une image personnalisée sur la base d'un autre code applicatif (Golang/Java/Python,etc.) que vous souhaitez containeriser.

L'image devra être buildée et run sans erreurs.

#### Partie 2 (Facultatif)

Étoffer votre image en rajoutant plusieurs services comme vu dans la section "Un conteneur et plusieurs services".

#### Partie 3 (Facultatif)

Étoffer à nouveau votre image en la construisant cette fois selon les règles du multi-stage building (https://docs.docker.com/develop/develop-images/multistage-build) qui visera a séparer le build du ou des binaires des processus qui seront run par l'image finale.

