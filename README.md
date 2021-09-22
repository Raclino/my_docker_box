# Box
## _Gestionnaire d'environnement_

 Box un gestionnaire d'environnements permettant d'effectuer les actions suivantes :

- Création d'un environnement Debian dédié pour une application
- Installation de l'application et de ses dépendances en vous basant sur APT
- Exécution de l'application dans un environnement isolé

## Prérequis

Installer ```Python3``` et ```pip``` sur votre Debian.

```sh
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install python3 python3-pip
```
Installer ```PyYAML```
```sh
pip install pyyaml
```
Créer le dossier d'installation des environnements:
```sh
sudo mkdir -p /var/lib/box/base && sudo mkdir /var/lib/box/env
```
Télécharger et décompresser l'archive Debian l'archive Debian
```sh
sudo wget 'https://github.com/debuerreotype/docker-debian-artifacts/raw/3503997cf522377bc4e4967c7f0fcbcb18c69fc8/buster/slim/rootfs.tar.xz' --directory /var/lib/box/
sudo tar -xf /var/lib/box/rootfs.tar.xz -C /var/lib/box/base
```
## Téléchargement de Box

```sh
git clone git@rendu-git.etna-alternance.net:module-8034/activity-43945/group-870460.git /the/path/you/want
```
Vous trouverez dans ce dossier

- Le script python ```box```.
- Les fichiers de configuration d'environnement test:
    - ```mongo.yml``` pour un environnement de MongoDB4.4
    - ```share.yml``` pour un environnement sh
    - ```user_tester.yml``` pour un environnement avec un utilisateur

## Utilisation de box

Placez vous dans le dossier souhaité
```sh
cd /the/path/you/want
```
**Commandes disponibles**

Installer un environnement:
```sh
sudo ./box build file.yml
```
- Votre environnement sera installé dans ```/var/lib/box/env```

Lister les environnements installés:
```sh
sudo ./box list
```
Lancer un environnement:
```sh
sudo ./box run environment_name command_you_want
```
**Partage de fichier**

Vous pouvez partager un fichier entre l'hôte et votre environnement avec ```--share``` de la façon suivante:
```sh
sudo ./box run --share '/path/on/host:/path/in/env' environment_name command_you_want
```
