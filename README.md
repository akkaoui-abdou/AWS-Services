# Guide d’architecture AWS moderne — De la conception au déploiement d’une application Cloud

En tant qu’Architecte Solutions AWS, je vais organiser les services selon **l’ordre réel de construction d’une application de production sur AWS**.

Une architecture moderne suit généralement ce flux :

```
Utilisateur
    |
    v
CloudFront (CDN)
    |
    v
Load Balancer / API Gateway
    |
    v
EC2 / ECS / Lambda
    |
    +----------------+
    |                |
    v                v
DynamoDB          S3
(Base NoSQL)      (Fichiers)
    |
    v
SQS (Messages asynchrones)

Sécurité transverse :
IAM + KMS + VPC
```

---

# 1. IAM (Identity and Access Management)

## Rôle principal

IAM est le service AWS qui permet de gérer :

* Les utilisateurs
* Les rôles
* Les permissions
* Les accès aux ressources AWS

Il répond à la question :

> "Qui peut faire quoi sur quelles ressources AWS ?"

---

## Problème résolu

Dans un environnement Cloud, plusieurs acteurs utilisent AWS :

* Développeurs
* Applications
* Services AWS
* Administrateurs

IAM permet d’éviter :

* Les accès non autorisés
* L’utilisation de comptes root
* Les permissions excessives

---

## Quand l'utiliser ?

Toujours.

IAM est le premier service à configurer dans un nouveau compte AWS.

Exemple :

Une API Django déployée sur EC2 doit accéder à S3.

On ne met pas une clé AWS dans le code :

```python
AWS_ACCESS_KEY="xxxx"
AWS_SECRET_KEY="xxxx"
```

On utilise un rôle IAM :

```
EC2 Instance
      |
      |
 IAM Role
      |
      |
 Permission S3 Read/Write
```

---

## Intégration avec les autres services

IAM est utilisé par :

* EC2
* Lambda
* ECS
* S3
* DynamoDB
* Bedrock
* API Gateway

Tous les services AWS utilisent IAM.

---

## Bonnes pratiques

✅ Utiliser des rôles IAM

❌ Ne jamais utiliser la clé root

✅ Principe du moindre privilège :

Exemple :

Mauvais :

```
Allow *
```

Bon :

```
Allow s3:GetObject
Resource:
bucket-production/images/*
```

---

# 2. VPC (Virtual Private Cloud)

## Rôle principal

VPC permet de créer un réseau privé dans AWS.

C’est l’équivalent d’un réseau d’entreprise.

Il contient :

* Subnets
* Routes
* Firewall
* IP privées
* Security Groups

---

## Problème résolu

Comment isoler une application critique ?

Exemple :

```
Internet

   |
   |
Public Subnet

Load Balancer


Private Subnet

Application Backend


Private Subnet

Database
```

---

## Architecture réseau classique

```
VPC
|
+-- Public Subnet
|       |
|       Load Balancer
|
+-- Private Subnet
        |
        ECS / EC2
        |
        DynamoDB
```

---

## Bonnes pratiques

Créer plusieurs zones :

```
Region Europe
 |
 +-- Availability Zone A
 |
 +-- Availability Zone B
```

Pour garantir la haute disponibilité.

---

# 3. EC2 (Elastic Compute Cloud)

## Rôle principal

EC2 fournit des serveurs virtuels.

C’est comme louer un serveur Linux dans AWS.

---

## Problème résolu

Avant Cloud :

```
Acheter serveur physique
Installer OS
Configurer réseau
Maintenir hardware
```

Avec EC2 :

```
Créer VM en quelques minutes
```

---

## Exemple

Application Django :

```
EC2
 |
 +-- Ubuntu
 |
 +-- Python
 |
 +-- Django
 |
 +-- Gunicorn
 |
 +-- Nginx
```

---

## Quand utiliser EC2 ?

Utiliser EC2 quand :

* Besoin de contrôle complet
* Applications legacy
* Configuration spécifique OS

---

## Bonnes pratiques

✅ Auto Scaling Group

✅ AMI personnalisées

✅ Security Groups restrictifs

---

# 4. Load Balancer (ALB / NLB)

## Rôle

Distribuer le trafic entre plusieurs serveurs.

---

## Sans Load Balancer

```
Users

 |
 |
EC2 unique
```

Problème :

* Si EC2 tombe → application indisponible
* Difficulté à supporter la charge

---

## Avec Load Balancer

```
             Users

               |
               |
        Application Load Balancer

          /          \
         /            \

      EC2             EC2
```

---

## ALB vs NLB

| ALB         | NLB                    |
| ----------- | ---------------------- |
| HTTP/HTTPS  | TCP/UDP                |
| API Web     | Très haute performance |
| Routing URL | Faible latence         |

---

# 5. S3 (Simple Storage Service)

## Rôle

Stockage objet.

Utilisé pour :

* Images
* Documents
* Vidéos
* Backup
* Fichiers statiques

---

## Exemple application React + Django

React :

```
S3 Bucket

index.html
main.js
css
images
```

Django :

```
S3

/media
    user_photo.jpg
```

---

## Intégration

```
React
 |
CloudFront
 |
S3
```

---

## Bonnes pratiques

✅ Activer versioning

✅ Chiffrer avec KMS

✅ Bloquer accès public

---

# 6. CloudFront

## Rôle

CDN AWS.

Il rapproche les contenus des utilisateurs.

---

## Problème

Utilisateur France :

```
Paris

   |
   |
S3 USA
```

Latence élevée.

Avec CloudFront :

```
Paris User

 |
Edge Location Paris

 |
S3
```

---

## Utilisation

Parfait pour :

* Applications React
* Images
* Vidéos
* Assets statiques

---

# 7. KMS (Key Management Service)

## Rôle

Gestion des clés de chiffrement.

---

## Utilisation

Chiffrer :

* S3
* DynamoDB
* EBS
* Secrets

Exemple :

```
S3 Bucket

Encrypted Data

       +

KMS Key
```

---

# 8. ECS (Elastic Container Service)

## Rôle

Exécuter des containers Docker.

---

## Architecture

```
Docker Image

      |
      v

ECS Cluster

      |
      |
 Docker Containers
```

---

## Exemple

Microservices :

```
ECS

 |
 +-- User Service
 |
 +-- Payment Service
 |
 +-- Notification Service
```

---

## ECS vs EC2

| EC2             | ECS                |
| --------------- | ------------------ |
| Gestion serveur | Gestion containers |
| Plus manuel     | Plus automatisé    |
| VM              | Docker             |

---

# 9. API Gateway

## Rôle

Créer une API publique.

---

## Architecture Serverless

```
Client

 |
 |
API Gateway

 |
 |
Lambda

 |
 |
DynamoDB
```

---

## Fonctionnalités

* Authentification
* Rate limiting
* Monitoring
* Version API

---

# 10. Lambda

## Rôle

Exécuter du code sans gérer serveur.

---

## Exemple Python

```python
def lambda_handler(event, context):

    return {
        "statusCode":200,
        "body":"Hello AWS"
    }
```

---

## Quand utiliser Lambda ?

Très adapté pour :

* APIs simples
* Jobs automatiques
* Traitement événements

---

# 11. DynamoDB

## Rôle

Base NoSQL managée.

---

## Exemple

Table Users :

```
UserId
Name
Email
```

---

## Avantages

* Très scalable
* Faible latence
* Pas de serveur DB

---

## Utilisation

Avec Lambda :

```
Lambda

 |
 |

DynamoDB
```

---

# 12. SQS (Simple Queue Service)

## Rôle

File de messages.

---

## Problème

Application lente :

```
Upload fichier

|
Traitement long
|
Réponse lente
```

Solution :

```
API

 |
 |
SQS Queue

 |
 |
Worker
```

---

## Exemple

Commande e-commerce :

```
Order API

 |
SQS

 |
Payment Service
```

---

# 13. Amazon Bedrock

## Rôle

Service AWS pour utiliser des modèles d’IA générative.

Modèles disponibles :

* Claude
* Llama
* Titan

---

## Architecture IA moderne

```
Application

 |
API Gateway

 |
Lambda

 |
Bedrock

 |
LLM
```

---

## Exemple

Assistant IA :

```
Utilisateur

Question

 |
Backend

 |
Bedrock

 |
Réponse IA
```

---

# Architecture AWS complète exemple

Application Full Stack React + Python Django/FastAPI :

```
                 Users
                   |
                   |
              CloudFront
                   |
                   |
             ALB / API Gateway
                   |
        +----------+----------+
        |                     |
       ECS                 Lambda
        |                     |
        |                     |
     Django API          AI Service
        |
        |
    DynamoDB

        |
        |
       S3
        |
        |
      KMS


        |
        |
       SQS
```

Sécurité :

```
IAM
 |
VPC
 |
Security Groups
 |
KMS Encryption
```

---

# Flux utilisateur complet

## 1. Utilisateur ouvre le site

```
Browser
 |
CloudFront
 |
S3
```

React est chargé.

---

## 2. Appel API

```
React

 |
HTTPS

 |
API Gateway

 |
Lambda / ECS

 |
DynamoDB
```

---

## 3. Upload fichier

```
React

 |
S3 Upload

 |
Event

 |
SQS

 |
Worker Lambda
```

---

# EC2/ECS vs Serverless

| Critère              | EC2/ECS                  | Lambda/API Gateway |
| -------------------- | ------------------------ | ------------------ |
| Serveur              | Oui                      | Non                |
| Contrôle             | Fort                     | Faible             |
| Scalabilité          | Configuration nécessaire | Automatique        |
| Coût faible charge   | Moyen                    | Excellent          |
| Applications longues | Oui                      | Limité             |
| Microservices        | Excellent                | Excellent          |

---

# Services AWS prioritaires pour un développeur Full Stack Python/React

Ordre d’apprentissage recommandé :

## Niveau 1 — Fondamentaux

1. IAM
2. VPC
3. EC2
4. S3
5. RDS/DynamoDB

## Niveau 2 — Architecture Production

6. Load Balancer
7. Auto Scaling
8. ECS
9. CloudFront

## Niveau 3 — Serverless

10. Lambda
11. API Gateway
12. SQS

## Niveau 4 — Cloud avancé

13. KMS
14. Bedrock
15. EventBridge
16. CloudWatch
17. Terraform / CDK

Pour un profil **Full Stack Python + React + DevOps**, la maîtrise de **IAM, VPC, ECS, Lambda, API Gateway, S3, CloudFront, DynamoDB et SQS** couvre déjà une grande partie des architectures AWS utilisées en entreprise.
