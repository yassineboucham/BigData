# Atelier HDFS avec Docker — Guide Complet Étape par Étape

Ce README détaille **toutes les étapes** de l'atelier HDFS avec Docker, avec une **explication de chaque commande** pour bien comprendre ce que l'on fait et pourquoi.

---

## Table des matières
1. [Objectifs](#1-objectifs)
2. [Prérequis](#2-prérequis)
3. [Étape 1 — Lancer HDFS avec Docker](#étape-1--lancer-hdfs-avec-docker)
4. [Étape 2 — Accéder à l'interface web](#étape-2--accéder-à-linterface-web)
5. [Étape 3 — Accéder au terminal du conteneur](#étape-3--accéder-au-terminal-du-conteneur)
6. [Étape 4 — Commandes HDFS de base](#étape-4--commandes-hdfs-de-base)
7. [Étape 5 — Visualiser le découpage en blocs](#étape-5--visualiser-le-découpage-en-blocs)
8. [Étape 6 — Comprendre le découpage (exemple théorique)](#étape-6--comprendre-le-découpage-exemple-théorique)
9. [Étape 7 — Schéma conceptuel](#étape-7--schéma-conceptuel)
10. [Lien avec le Big Data](#lien-avec-le-big-data)
11. [Exercices](#exercices)
12. [Questions de compréhension (avec réponses)](#questions-de-compréhension-avec-réponses)
13. [Conclusion](#conclusion)

---

## 1. Objectifs

- Installer et exécuter un environnement Hadoop via Docker
- Accéder à l'interface web HDFS
- Manipuler les commandes de base du système de fichiers distribué
- Comprendre le principe de découpage en blocs
- Visualiser le fonctionnement d'un système distribué
- Comprendre le lien entre HDFS et le Big Data

## 2. Prérequis

- **Docker Desktop** installé sur votre machine
- Accès à un terminal (CMD, PowerShell ou Bash)
- Une connexion Internet (pour télécharger l'image Docker)

---

## Étape 1 — Lancer HDFS avec Docker

### Commande
```bash
docker run -d -p 9870:9870 -p 9000:9000 -e CLUSTER_NAME=test bde2020/hadoop-namenode:2.0.0-hadoop3.2.1-java8
```

### Explication
- `docker run` : crée et démarre un nouveau conteneur à partir d'une image Docker.
- `-d` : lance le conteneur en mode **détaché** (en arrière-plan), pour continuer à utiliser le terminal.
- `-p 9870:9870` : **redirige** le port 9870 du conteneur vers le port 9870 de votre machine. Ce port sert à l'**interface web du NameNode** (monitoring).
- `-p 9000:9000` : redirige le port 9000, utilisé pour la **communication interne** RPC entre le NameNode et les clients HDFS.
- `-e CLUSTER_NAME=test` : définit une **variable d'environnement** qui nomme le cluster Hadoop "test".
- `bde2020/hadoop-namenode:2.0.0-hadoop3.2.1-java8` : c'est **l'image Docker** utilisée — une image préconfigurée contenant Hadoop (NameNode) basée sur Hadoop 3.2.1 et Java 8.

**Résultat attendu :** un conteneur Docker démarre en arrière-plan, exécutant le service NameNode de Hadoop.

### Vérification
```bash
docker ps
```

**Explication :** cette commande liste tous les conteneurs Docker **actuellement en cours d'exécution**. Vous devez y voir le conteneur `bde2020/hadoop-namenode` avec un statut "Up", ainsi que son `CONTAINER ID` (que vous réutiliserez plus tard).

---

## Étape 2 — Accéder à l'interface web

### Action
Ouvrir un navigateur et se rendre à l'adresse :
```
http://localhost:9870
```

### Explication
Cette interface web est fournie par le **NameNode** de Hadoop. Elle permet de visualiser en temps réel :
- **Le NameNode** : l'état général du service (actif, capacité, etc.)
- **L'état du système** : santé du cluster, nombre de DataNodes connectés
- **Les fichiers stockés** : navigation dans l'arborescence HDFS (menu "Utilities" → "Browse the file system")
- **L'espace disque utilisé** : capacité totale, espace utilisé, espace restant

C'est l'équivalent d'un tableau de bord de supervision pour votre cluster HDFS.

---

## Étape 3 — Accéder au terminal du conteneur

### Commande
```bash
docker exec -it <container_id> /bin/bash
```

### Explication
- `docker exec` : exécute une commande **à l'intérieur d'un conteneur déjà lancé**.
- `-it` : combine deux options :
  - `-i` (interactive) : garde l'entrée standard ouverte
  - `-t` (tty) : alloue un pseudo-terminal, pour avoir une session interactive comme un vrai terminal
- `<container_id>` : à remplacer par l'ID (ou le nom) du conteneur obtenu avec `docker ps`
- `/bin/bash` : lance un shell **Bash** à l'intérieur du conteneur

**Résultat :** vous obtenez un terminal **directement dans le conteneur Hadoop**, où les commandes `hdfs` sont disponibles.

---

## Étape 4 — Commandes HDFS de base

> ⚠️ Toutes les commandes ci-dessous s'exécutent **à l'intérieur du conteneur** (après l'étape 3).

### 4.1 Lister les fichiers
```bash
hdfs dfs -ls /
```
**Explication :** affiche le contenu (fichiers et dossiers) à la **racine** du système de fichiers HDFS — équivalent de `ls` sous Linux, mais appliqué à HDFS et non au disque local.

### 4.2 Créer un dossier
```bash
hdfs dfs -mkdir /tp
```
**Explication :** crée un nouveau répertoire nommé `/tp` **dans HDFS** (et non sur le disque local). Ce dossier existera dans l'arborescence distribuée du cluster.

### 4.3 Créer un fichier local
```bash
echo "Bonjour HDFS" > test.txt
```
**Explication :** crée un fichier texte **local** (sur le système de fichiers du conteneur, pas encore dans HDFS) contenant la phrase "Bonjour HDFS". La commande `echo` affiche le texte, et `>` redirige cette sortie vers un fichier.

### 4.4 Copier un fichier vers HDFS
```bash
hdfs dfs -put test.txt /tp
```
**Explication :** `put` **transfère** un fichier du système de fichiers **local** vers **HDFS**. Ici, `test.txt` est copié dans le dossier `/tp` créé précédemment. C'est l'étape où le fichier devient réellement "distribué".

### 4.5 Lire un fichier
```bash
hdfs dfs -cat /tp/test.txt
```
**Explication :** affiche le **contenu** d'un fichier stocké dans HDFS directement dans le terminal — équivalent de `cat` sous Linux, mais lit depuis HDFS.

### 4.6 Supprimer un fichier
```bash
hdfs dfs -rm /tp/test.txt
```
**Explication :** supprime le fichier `test.txt` du dossier `/tp` dans HDFS.

### 4.7 Supprimer un dossier
```bash
hdfs dfs -rm -r /tp
```
**Explication :** l'option `-r` (récursif) supprime le dossier `/tp` **et tout son contenu**. Sans `-r`, HDFS refuserait de supprimer un dossier non vide.

### 4.8 Copier un fichier de HDFS vers le local
```bash
hdfs dfs -get /tp/test.txt .
```
**Explication :** `get` fait l'inverse de `put` : il **télécharge** un fichier depuis HDFS vers le système de fichiers local. Le `.` signifie "dans le répertoire courant".

---

## Étape 5 — Visualiser le découpage en blocs

### 5.1 Créer un fichier volumineux
```bash
dd if=/dev/zero of=bigfile.txt bs=1M count=20
```
**Explication :**
- `dd` : utilitaire permettant de copier/générer des données brutes
- `if=/dev/zero` : source d'entrée = flux infini de zéros (souvent utilisé pour générer des fichiers de test)
- `of=bigfile.txt` : fichier de sortie généré
- `bs=1M` : taille de bloc = 1 mégaoctet par écriture
- `count=20` : nombre de blocs écrits → **20 × 1 MB = environ 20 MB**

**Résultat :** un fichier factice de 20 MB, utile pour tester le comportement de HDFS avec des fichiers plus volumineux.

### 5.2 Copier le fichier vers HDFS
```bash
hdfs dfs -put bigfile.txt /data
```
**Explication :** transfère `bigfile.txt` vers le dossier `/data` dans HDFS (le dossier doit exister au préalable, sinon il faut le créer avec `mkdir` comme à l'étape 4.2).

### 5.3 Visualiser les blocs
```bash
hdfs fsck /data/bigfile.txt -files -blocks -locations
```
**Explication :**
- `hdfs fsck` : (**f**ile **s**ystem **c**heck) outil de diagnostic du système de fichiers HDFS
- `-files` : affiche les informations sur le fichier
- `-blocks` : affiche la liste des **blocs** qui composent le fichier
- `-locations` : indique **sur quels DataNodes** chaque bloc est physiquement stocké

Cette commande permet concrètement de **voir** que le fichier a été découpé en morceaux (blocs) et répartis (même si, dans cet atelier avec un seul conteneur, la distribution réelle sur plusieurs machines est simulée).

---

## Étape 6 — Comprendre le découpage (exemple théorique)

Si un fichier fait **300 MB** et que la taille standard d'un bloc HDFS est de **128 MB**, il sera découpé ainsi :

| Bloc | Taille |
|---|---|
| Bloc 1 | 128 MB |
| Bloc 2 | 128 MB |
| Bloc 3 | ≈ 44 MB (le reste) |

**Explication :** HDFS découpe systématiquement un fichier en blocs de taille fixe (128 MB par défaut dans Hadoop 3.x). Le **dernier bloc** contient simplement ce qui reste, même s'il est plus petit que la taille standard. Chaque bloc est ensuite stocké — et généralement **répliqué** (souvent 3 copies) — sur des **DataNodes différents** pour assurer la disponibilité en cas de panne.

---

## Étape 7 — Schéma conceptuel

```
Fichier volumineux (ex: 300 MB)
            │
            ▼
     Découpage en blocs
   ┌─────────┬─────────┬─────────┐
   │ Bloc 1  │ Bloc 2  │ Bloc 3  │
   │ 128 MB  │ 128 MB  │ 44 MB   │
   └─────────┴─────────┴─────────┘
        │         │         │
        ▼         ▼         ▼
   DataNode 1  DataNode 2  DataNode 3
        (chaque bloc peut être répliqué
         sur plusieurs DataNodes)

   Le NameNode gère les métadonnées :
   - quel fichier contient quels blocs
   - où se trouve chaque bloc (quel DataNode)
```

**Explication :** le **NameNode** est le "chef d'orchestre" — il ne stocke pas les données elles-mêmes, mais sait exactement où se trouve chaque bloc. Les **DataNodes** sont les "ouvriers" qui stockent physiquement les blocs de données.

---

## Lien avec le Big Data

HDFS est une technologie fondamentale du Big Data. Il permet :

- **Le stockage distribué des données** — répartir les données sur plusieurs machines plutôt que sur un seul disque
- **Le découpage des fichiers en blocs** — traiter des fichiers énormes en les divisant en morceaux gérables
- **La réplication des données** — assurer la tolérance aux pannes (si un nœud tombe, les données restent accessibles ailleurs)
- **La gestion de très gros volumes** — dépasser les limites d'un système de fichiers classique
- **L'organisation en cluster** — coordonner plusieurs machines comme un seul système cohérent

Même si cet atelier est réalisé sur un seul ordinateur via Docker, il **simule** une véritable architecture distribuée telle qu'utilisée en production dans les systèmes Big Data (Hadoop, Spark, etc.).

---

## Exercices

### Exercice 1
Créer un dossier `/data`, ajouter un fichier, puis vérifier son contenu dans HDFS.
```bash
hdfs dfs -mkdir /data
echo "Exercice 1" > ex1.txt
hdfs dfs -put ex1.txt /data
hdfs dfs -cat /data/ex1.txt
```

### Exercice 2
Copier plusieurs fichiers dans HDFS et les lire via `cat`.
```bash
echo "Fichier A" > a.txt
echo "Fichier B" > b.txt
hdfs dfs -put a.txt /data
hdfs dfs -put b.txt /data
hdfs dfs -cat /data/a.txt
hdfs dfs -cat /data/b.txt
```

### Exercice 3
Créer un fichier de taille supérieure à 200 MB et observer son découpage en blocs.
```bash
dd if=/dev/zero of=grandfichier.txt bs=1M count=250
hdfs dfs -put grandfichier.txt /data
hdfs fsck /data/grandfichier.txt -files -blocks -locations
```
**Résultat attendu :** avec une taille de bloc de 128 MB, ce fichier de 250 MB sera découpé en **2 blocs de 128 MB** et **1 bloc d'environ 122 MB**.

---

## Questions de compréhension (avec réponses)

**Qu'est-ce que HDFS ?**
HDFS (*Hadoop Distributed File System*) est un système de fichiers distribué conçu pour stocker de très grands volumes de données réparties sur plusieurs machines (nœuds).

**Quel est le rôle du NameNode ?**
Le NameNode gère les **métadonnées** : il sait quels fichiers existent, comment ils sont découpés en blocs, et sur quels DataNodes ces blocs se trouvent. C'est le "cerveau" du cluster.

**Quel est le rôle des DataNodes ?**
Les DataNodes stockent **physiquement** les blocs de données et exécutent les opérations de lecture/écriture demandées par les clients, sous la supervision du NameNode.

**Pourquoi HDFS découpe-t-il les fichiers en blocs ?**
Pour permettre le stockage distribué (répartir la charge sur plusieurs machines), faciliter le traitement parallèle des données, et permettre la réplication indépendante de chaque bloc pour la tolérance aux pannes.

**Quelle est la relation entre HDFS et le Big Data ?**
HDFS fournit la **couche de stockage** fondamentale des architectures Big Data : il permet de stocker, organiser et rendre accessibles des volumes massifs de données de façon fiable et scalable, ce qui est indispensable pour tout traitement Big Data (via Hadoop MapReduce, Spark, Hive, etc.).

---

## Conclusion

HDFS constitue une base essentielle des architectures Big Data. Il permet de stocker des données de manière distribuée, de gérer des volumes importants et d'assurer la tolérance aux pannes grâce à la réplication des blocs. Cet atelier permet de comprendre concrètement ces principes à travers la manipulation d'un environnement simulé avec Docker.

---

*Atelier HDFS avec Docker — ISGA Marrakech, 2CI-ISI.*