# 🚀 Automated CI/CD Pipeline with Jenkins, Docker Hub, AWS ECR & Docker Swarm

Ce projet met en œuvre une chaîne **CI/CD automatisée** permettant de construire, tester, conteneuriser et déployer une application multi-services sur un cluster **Docker Swarm** exécuté sur **AWS EC2**.

Le pipeline Jenkins automatise l'ensemble du processus :

```text
GitHub
   │
   │ Webhook
   ▼
Jenkins
   │
   ├── Checkout
   ├── Build & Test
   ├── Docker Build
   ├── Push → Docker Hub
   ├── Push → AWS ECR
   │
   ▼
Docker Swarm
   │
   ├── Manager
   │
   ├── Worker 1
   ├── Worker 2
   └── Worker N
```

L'objectif est de démontrer une architecture DevOps combinant **CI/CD, Docker, Registry, AWS et orchestration de conteneurs**.

---

# 📐 Architecture du système

## Workflow CI/CD

```text
                       Developer
                           │
                           │ git push
                           ▼
                    ┌───────────────┐
                    │    GitHub     │
                    └───────┬───────┘
                            │
                         Webhook
                            │
                            ▼
                    ┌───────────────┐
                    │    Jenkins    │
                    │               │
                    │ CI/CD Pipeline│
                    └───────┬───────┘
                            │
                 ┌──────────┼──────────┐
                 │          │          │
                 ▼          ▼          ▼
              Build       Test    Docker Build
                                      │
                                      ▼
                             ┌──────────────────┐
                             │  Docker Image    │
                             └────────┬─────────┘
                                      │
                         ┌────────────┴────────────┐
                         │                         │
                         ▼                         ▼
                  ┌─────────────┐          ┌─────────────┐
                  │ Docker Hub  │          │    AWS ECR  │
                  │  Registry   │          │  Registry   │
                  └──────┬──────┘          └──────┬──────┘
                         │                         │
                         └────────────┬────────────┘
                                      │
                                      ▼
                             ┌─────────────────┐
                             │  Docker Swarm   │
                             │     Manager     │
                             └────────┬────────┘
                                      │
                       ┌──────────────┼──────────────┐
                       │              │              │
                       ▼              ▼              ▼
                  ┌─────────┐   ┌─────────┐   ┌─────────┐
                  │ Worker 1│   │ Worker 2│   │ Worker N│
                  └─────────┘   └─────────┘   └─────────┘
```

---

# 🔄 Fonctionnement du pipeline

## 1. Developer → GitHub

Le développeur pousse une modification du code :

```bash
git add .
git commit -m "Update application"
git push origin main
```

---

## 2. GitHub Webhook → Jenkins

Un **GitHub Webhook** déclenche automatiquement le pipeline Jenkins lorsqu'une modification est détectée.

```text
Git Push
   │
   ▼
GitHub
   │
   │ Webhook
   ▼
Jenkins
```

Cela permet de mettre en place un processus de **Continuous Integration** automatisé.

---

# 3. Checkout

Jenkins récupère le code source depuis GitHub.

```text
GitHub
   │
   ▼
Jenkins Workspace
```

---

# 4. Build & Test

Le pipeline compile l'application et exécute les tests disponibles.

Exemple :

```bash
mvn clean test
```

ou selon la technologie utilisée :

```bash
npm test
```

Cette étape permet de détecter les erreurs avant la création de l'image Docker.

---

# 5. Docker Build

Jenkins construit ensuite l'image Docker.

```bash
docker build -t app:${BUILD_NUMBER} .
```

Le numéro du build Jenkins est utilisé comme **tag de version**.

Exemple :

```text
app:15
app:16
app:17
```

Cela permet de conserver une traçabilité entre :

```text
Jenkins Build
      ↓
Docker Image
      ↓
Deployment
```

---

# 6. Publication sur Docker Hub

L'image est publiée sur Docker Hub.

Exemple :

```text
docker.io/<username>/app:17
```

Deux tags peuvent être utilisés :

```text
17
latest
```

Le tag versionné permet de conserver une version précise de l'application.

---

# 7. Publication sur AWS ECR

L'image est également publiée dans **Amazon Elastic Container Registry (ECR)**.

Exemple :

```text
<ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/app:17
```

ECR fournit un registre privé intégré à l'écosystème AWS.

---

# 8. Déploiement Docker Swarm

Après publication de l'image, Jenkins déclenche le déploiement :

```bash
docker stack deploy -c docker-compose.yml my_app_stack
```

Docker Swarm distribue ensuite les services sur les nœuds disponibles.

```text
              Docker Swarm
                   │
              ┌────┴────┐
              │ Manager │
              └────┬────┘
                   │
          ┌────────┼────────┐
          │        │        │
          ▼        ▼        ▼
       Worker 1 Worker 2 Worker 3
```

---

# 🐳 Docker Swarm

Docker Swarm fournit plusieurs fonctionnalités importantes :

* Service orchestration
* Service replication
* Rolling updates
* Service discovery
* Load balancing interne
* Self-healing
* Scaling horizontal
* Gestion des nœuds

---

# 📈 Réplication des services

Un service peut être configuré avec plusieurs replicas.

Exemple :

```yaml
services:

  banking:
    image: <ECR_REPOSITORY>/banking:latest

    deploy:
      replicas: 3
```

Le résultat peut être :

```text
Banking Service
      │
      ├── Replica 1
      ├── Replica 2
      └── Replica 3
```

Si un conteneur échoue, Swarm peut recréer une nouvelle instance afin de maintenir le nombre de replicas configuré.

---

# 🛠️ Stack technique

| Catégorie        | Technologie       | Rôle                       |
| ---------------- | ----------------- | -------------------------- |
| SCM              | Git / GitHub      | Gestion du code            |
| CI/CD            | Jenkins           | Automatisation du pipeline |
| Containerization | Docker            | Création des images        |
| Registry         | Docker Hub        | Registry d'images          |
| Registry Cloud   | AWS ECR           | Registry privé AWS         |
| Orchestration    | Docker Swarm      | Déploiement des services   |
| Cloud            | AWS EC2           | Infrastructure             |
| Network          | AWS VPC           | Réseau cloud               |
| OS               | Amazon Linux 2023 | Système des instances      |
| Deployment       | Docker Stack      | Déploiement des services   |

---

# 📁 Structure du projet

```text
Docker-Projec-With_jenkins/
│
├── architecture.png
│
├── Dockerfile
│
├── docker-compose.yml
│
├── Jenkinsfile
│
├── PROCESS.txt
│
└── Screenshot/
    │
    ├── Ec2s.png
    ├── Jenkins-Consol.png
    ├── Repo-on-DockerHub.png
    ├── Docker.png
    ├── InternetBanking.png
    ├── Loan.png
    ├── mobilebanking.png
    └── insurance.png
```

---

# ⚙️ Jenkins Pipeline

Le pipeline Jenkins peut être organisé selon les étapes suivantes :

```text
pipeline
   │
   ├── Checkout Source Code
   │
   ├── Build & Test
   │
   ├── Docker Build
   │
   ├── Login Docker Hub
   │
   ├── Push Docker Hub
   │
   ├── Login AWS ECR
   │
   ├── Tag Image
   │
   ├── Push AWS ECR
   │
   └── Deploy Docker Swarm
```

---

# 🧩 Exemple de Jenkinsfile

```groovy
pipeline {

    agent any

    environment {
        DOCKER_HUB_REPO = 'username/app'
        AWS_REGION = 'us-east-1'
        ECR_REGISTRY = '123456789012.dkr.ecr.us-east-1.amazonaws.com'
        ECR_REPO = 'app'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/username/repository.git'
            }
        }

        stage('Build & Test') {
            steps {
                echo 'Building and testing application...'
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                    docker build \
                    -t ${DOCKER_HUB_REPO}:${BUILD_NUMBER} .
                """
            }
        }

        stage('Push Docker Hub') {
            steps {
                script {

                    docker.withRegistry(
                        'https://index.docker.io/v1/',
                        'docker-hub-credentials-id'
                    ) {

                        def image =
                            docker.image(
                                "${DOCKER_HUB_REPO}:${BUILD_NUMBER}"
                            )

                        image.push()
                        image.push('latest')
                    }
                }
            }
        }

        stage('Push AWS ECR') {
            steps {
                sh """
                    aws ecr get-login-password \
                    --region ${AWS_REGION} |
                    docker login \
                    --username AWS \
                    --password-stdin ${ECR_REGISTRY}

                    docker tag \
                    ${DOCKER_HUB_REPO}:${BUILD_NUMBER} \
                    ${ECR_REGISTRY}/${ECR_REPO}:${BUILD_NUMBER}

                    docker push \
                    ${ECR_REGISTRY}/${ECR_REPO}:${BUILD_NUMBER}
                """
            }
        }

        stage('Deploy Docker Swarm') {
            steps {
                sh """
                    docker stack deploy \
                    -c docker-compose.yml \
                    my_app_stack
                """
            }
        }
    }
}
```

---

# ☁️ Infrastructure AWS

L'environnement AWS peut être organisé ainsi :

```text
                         AWS VPC
                            │
            ┌───────────────┴───────────────┐
            │                               │
         Jenkins                       Docker Swarm
                                             │
                              ┌──────────────┼──────────────┐
                              │              │              │
                              ▼              ▼              ▼
                           Manager        Worker 1       Worker 2
```

Les instances EC2 utilisent **Amazon Linux 2023**.

---

# 🔐 Sécurité réseau

Le Security Group doit autoriser uniquement les ports nécessaires.

Exemples :

| Port         | Utilisation             |
| ------------ | ----------------------- |
| 22           | SSH                     |
| 80           | HTTP                    |
| 443          | HTTPS                   |
| 2377         | Docker Swarm management |
| 7946 TCP/UDP | Node communication      |
| 4789 UDP     | Overlay network         |

Les ports Docker Swarm doivent idéalement être accessibles uniquement entre les instances du cluster via leurs règles réseau internes.

---

# 📦 Docker Compose / Stack

Le fichier :

```text
docker-compose.yml
```

définit les services déployés dans Swarm.

Exemple :

```yaml
version: "3.8"

services:

  banking:
    image: <ECR_REGISTRY>/banking:latest

    deploy:
      replicas: 2

      restart_policy:
        condition: on-failure

  loan:
    image: <ECR_REGISTRY>/loan:latest

    deploy:
      replicas: 2

  mobilebanking:
    image: <ECR_REGISTRY>/mobilebanking:latest

    deploy:
      replicas: 2

  insurance:
    image: <ECR_REGISTRY>/insurance:latest

    deploy:
      replicas: 2
```

---

# 🚀 Initialisation du cluster Docker Swarm

Sur le Manager :

```bash
docker swarm init --advertise-addr <MANAGER-IP>
```

Docker fournit ensuite une commande permettant aux Workers de rejoindre le cluster.

Sur chaque Worker :

```bash
docker swarm join \
    --token <SWARM-TOKEN> \
    <MANAGER-IP>:2377
```

Vérifier les nœuds :

```bash
docker node ls
```

Exemple :

```text
ID          HOSTNAME    STATUS    AVAILABILITY    MANAGER STATUS
xxxxx       manager     Ready     Active          Leader
xxxxx       worker1     Ready     Active
xxxxx       worker2     Ready     Active
```

---

# 🚀 Déploiement manuel

Une fois le cluster configuré :

```bash
docker stack deploy \
    -c docker-compose.yml \
    my_app_stack
```

Vérifier les services :

```bash
docker service ls
```

Vérifier les tâches :

```bash
docker service ps my_app_stack_banking
```

---

# 📊 Scaling

Docker Swarm permet d'augmenter le nombre de replicas :

```bash
docker service scale \
    my_app_stack_banking=5
```

Vérifier :

```bash
docker service ls
```

---

# 🔄 Mise à jour de l'application

Lorsqu'une nouvelle image est publiée, le pipeline peut déclencher une nouvelle version du service.

```text
Code Change
     │
     ▼
GitHub
     │
     ▼
Jenkins
     │
     ▼
Docker Build
     │
     ▼
ECR
     │
     ▼
Docker Swarm
     │
     ▼
Rolling Update
```

Cela permet d'automatiser le cycle :

**Code → Image → Registry → Deployment**

---

# 📸 Captures et démonstration

## 1. Infrastructure AWS EC2

```text
Screenshot/Ec2s.png
```

Cette capture présente les instances EC2 utilisées pour l'environnement Docker Swarm.

---

## 2. Pipeline Jenkins

```text
Screenshot/Jenkins-Consol.png
```

Cette capture montre l'exécution du pipeline Jenkins.

---

## 3. Docker Hub

```text
Screenshot/Repo-on-DockerHub.png
```

Cette capture montre l'image Docker publiée dans Docker Hub.

---

## 4. Docker Swarm

```text
Screenshot/Docker.png
```

Cette capture présente l'état des services et conteneurs Docker.

---

# 🌐 Applications déployées

Le cluster héberge plusieurs services applicatifs :

### Internet Banking

```text
Screenshot/InternetBanking.png
```

### Loan Service

```text
Screenshot/Loan.png
```

### Mobile Banking

```text
Screenshot/mobilebanking.png
```

### Insurance

```text
Screenshot/insurance.png
```

---

# 🧪 Tests

Vérifier les services :

```bash
docker service ls
```

Vérifier les replicas :

```bash
docker service ps my_app_stack_banking
```

Vérifier les nœuds :

```bash
docker node ls
```

Tester l'application :

```bash
curl http://<PUBLIC-IP>
```

---

# 🛑 Gestion du Stack

Supprimer le stack :

```bash
docker stack rm my_app_stack
```

Lister les stacks :

```bash
docker stack ls
```

Lister les services :

```bash
docker service ls
```

---

# 🎯 Concepts DevOps démontrés

Ce projet permet de démontrer les compétences suivantes :

### CI/CD

* Jenkins Pipeline
* GitHub Webhook
* Continuous Integration
* Automated Deployment
* Pipeline as Code

### Docker

* Dockerfile
* Docker Images
* Docker Containers
* Image Tagging
* Docker Registry

### Container Registry

* Docker Hub
* AWS ECR
* Versioned Images

### Docker Swarm

* Manager / Worker architecture
* Service deployment
* Replicas
* Scaling
* Service discovery
* Rolling updates
* Self-healing

### AWS

* EC2
* VPC
* Security Groups
* ECR
* Linux administration

---

# 🔐 Gestion des credentials

Les informations sensibles doivent être stockées dans **Jenkins Credentials** et non dans le repository GitHub.

Exemples :

```text
Jenkins Credentials
│
├── GitHub credentials
├── Docker Hub credentials
├── AWS credentials
└── SSH credentials
```

Les secrets suivants ne doivent jamais être commités :

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
DOCKER_PASSWORD
SSH_PRIVATE_KEY
```

---

# 📚 Commandes utiles

### Docker

```bash
docker ps
docker images
docker network ls
docker volume ls
```

### Swarm

```bash
docker swarm join-token worker
docker node ls
docker service ls
docker service ps <SERVICE>
docker stack ls
docker stack services <STACK>
```

### Stack

```bash
docker stack deploy -c docker-compose.yml my_app_stack
docker stack services my_app_stack
docker stack ps my_app_stack
docker stack rm my_app_stack
```

---

# 🔮 Évolution possible

Cette architecture peut évoluer vers une plateforme Kubernetes.

### Architecture actuelle

```text
GitHub
   │
   ▼
Jenkins
   │
   ▼
Docker
   │
   ▼
Docker Hub / ECR
   │
   ▼
Docker Swarm
```

### Évolution Kubernetes

```text
GitHub
   │
   ▼
Jenkins
   │
   ▼
Docker Build
   │
   ▼
AWS ECR
   │
   ▼
Kubernetes / EKS
   │
   ├── Deployment
   ├── Service
   ├── Ingress
   └── ConfigMap / Secret
```

Une évolution supplémentaire pourrait intégrer :

* Kubernetes
* Amazon EKS
* Helm
* Argo CD
* Prometheus
* Grafana
* Terraform
* GitOps

---

# 👨‍💻 Author

**Boubacar Djibrilla**

Software Engineering | AWS | DevOps | Cloud

GitHub: `boubacardjibrilla`
