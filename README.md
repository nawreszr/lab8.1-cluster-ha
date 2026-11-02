# Lab 8.1 : Cluster K3s en Haute Disponibilité (HA)

## 📖 Description du Lab
Ce laboratoire consistait à transformer un cluster K3s à serveur unique en un cluster hautement disponible (HA) composé de **3 serveurs** (Control Plane + etcd). Nous avons testé la résilience du cluster face à la perte d'un serveur, puis face à la perte de quorum.

## 🏗️ Architecture
*   3 VMs Ubuntu Server agissant comme Control Plane et Worker.
*   Base de données **etcd** embarquée et répliquée sur les 3 serveurs.
*   Application de test (Nginx) déployée avec 3 réplicas.

---

## 🧠 Questions de réflexion (Réponses)

**1. Pourquoi le cluster est-il resté utilisable quand un seul serveur était arrêté ?**
> Parce qu'etcd utilise un algorithme de consensus (Raft) qui nécessite un quorum (une majorité). Avec 3 serveurs, la majorité est de 2. Il restait 2 serveurs actifs, donc le cluster a pu continuer à fonctionner normalement.

**2. Pourquoi `kubectl` a-t-il cessé de fonctionner quand deux serveurs sont tombés ?**
> Parce qu'il ne restait plus qu'un seul serveur sur trois. Le quorum n'était plus atteint (1 < 2), donc etcd n'a plus pu valider de lecture ou d'écriture. L'API server dépend d'etcd, il est donc devenu incapable de répondre (erreur `apiserver not ready`).

**3. Pourquoi la boucle de requêtes a-t-elle pu continuer à répondre pendant la perte du quorum ?**
> Le plan de contrôle (Control Plane) et le plan de données (Data Plane) sont séparés. Les conteneurs Nginx déjà en cours d'exécution sur le serveur 3 n'ont pas eu besoin de contacter l'API Kubernetes pour continuer à répondre aux requêtes HTTP. Le cluster ne peut plus être modifié, mais les applications tournent.

**4. Pourquoi quelques requêtes ont-elles échoué quand un serveur est tombé, alors que le cluster est en HA ?**
> Parce que les pods de l'application (`web`) étaient hébergés sur le serveur 1. Quand il a été éteint, les requêtes vers ces pods ont échoué. Il faut environ 5 minutes pour que Kubernetes déclare le nœud mort et recrée les pods sur les serveurs restants.

**5. Quelles limites de cette installation à 3 serveurs resteraient à lever pour une production ?**
> Il faudrait ajouter un **load balancer externe** devant les 3 serveurs pour que `kubectl` puisse toujours joindre l'API même si le serveur 1 tombe. Il faudrait aussi externaliser le stockage persistant.

---

## 📸 Preuves de réalisation (Captures d'écran)

### 1. Le cluster HA (3 serveurs Ready)
![Capture 01 - Cluster HA](captures/Lab8.1-Capture-01-Cluster-HA-3-Serveurs.png)

### 2. Perte d'un serveur (NotReady)
![Capture 03 - Panne Serveur1](captures/Lab8.1-Capture-03-Panne-Serveur1-NotReady.png)

### 3. Perte de Quorum
![Capture 05 - Perte Quorum](captures/Lab8.1-Capture-05-Perte-Quorum-Timeout.png)

### 4. Retour à la normale
![Capture 10 - Retour Quorum](captures/Lab8.1-Capture-10-Retour-Quorum-et-Donnees.png)
