---
published: true
title: "AWS - Installer et parametrer la CLI"
excerpt: "CLI Setup"
author_profile: false
toc: true
toc_sticky: true
toc_label: "CLI Setup"
toc_icon: "terminal"
comments: true
header:
  overlay_image: "assets/images/covers/cover-cloud.jpg"
  overlay_filter: 0.2
  teaser: "assets/images/covers/cover-cloud.jpg"
categories: [cloud, aws, cli]
---

Travailler avec l'interface en ligne de commande d'AWS permet de déployer des services très rapidement, et pousse à bien comprendre les mécanismes de chacun des services. Dans cet article nous verrons comment installer cet outil et le paramétrer.

# Pré-requis

Pour commencer à utiliser la CLI, il faudra uniquement disposer d'un **compte AWS**.

# Installation

La <a href="https://docs.aws.amazon.com/fr_fr/cli/latest/userguide/cli-chap-install.html" target="_blank">documentation</a> est bien faite pour l'installation selon l'OS.

Si on dispose déjà de Python installé. L'outil AWS CLI s'installe facilement via l'outil **pip** de python :

> `pip install awscli --upgrade --user`

# Configuration

Pour commencer à utiliser la CLI, on n'utilise pas nos identifiants AWS classiques, on va créer un **role** via l'outil **IAM**. Pourquoi ?
- Lorsqu'on créé un nouveau role dans IAM, on peut lui attribuer des droits spécifiques, et limiter les pouvoirs qui lui sont attribués. Par exemple, on n'attribue pas les mêmes droits à un developpeur, ou à l'admin système.
- Les identifiants que l'on va utiliser resteront en mémoire dans l'outil, AWS recommande de ne pas utiliser nos identifiant "maitres". En cas de compromission du poste, on pourra toujours révoquer les identifiants et en créer des nouveaux.

## Création d'un utilisateur

1. On se rend dans le service **IAM**, dans la rubrique **Utilisateurs**.
2. On choisit un **nom** et on spécifie qu'on veut un accès par **programmation**.
3. A l'étape suivante on inclue notre utilisateur dans un groupe. Il faut que ce groupe possède les **droits administrateur AdministratorAccess** pour pouvoir lancer tout les services sans limitations (Il est possible de filtrer très spécifiquement les droits par services).
4. *L'étape d'ajout de balises est optionnelle. Elle sert à ajouter des étiquettes pour s'y retrouver entre les différents utilisateurs existants.*
5. On vérifie nos choix et on valide. **Il est important de télécharger les identifiants sur la dernière page**.

On a créé un utilisateur avec les droits nécessaires, et on dispose d'un fichier CSV **credentials.csv** avec les champs suivants :

| **Username** | Nom d'utilisateur |
| **Password** | Le mot de passe pour acceder à la console web amazon (ici normalement vide car on a choisi l'accès par programmation) |
| **Access Key ID** | C'est notre identifiant pour se connecter à AWS CLI |
| **Secret Access Key** | C'est la clé secrète pour se connecter. |
| **Console login link** | Lien d'accès à la console web, non valable dans ce cas (accès programmation) |

## Mise en route de AWS CLI

Pour commencer à utiliser AWS CLI :

> `aws configure`

On indique Access Key / Secret Key pour s'identifier.

# Apprentissage

- Exploration du manuel des commandes

> `aws <command> help`

- Exploration des pages de référence. Par exemple <a href="https://docs.aws.amazon.com/cli/latest/reference/s3api/" target="_blank">cette page</a> pour manipuler des buckets S3.

## Sources

- <a href="https://docs.aws.amazon.com/cli/latest/index.html" target="_blank">Documentation AWS pour CLI</a>
