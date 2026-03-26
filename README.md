# Projet de Fin de Module : Architecture Réseau 3-Tiers sur OpenShift Virtualization

Ce dépôt contient l'Infrastructure as Code (IaC) d'une architecture réseau sécurisée déployée automatiquement sur Red Hat OpenShift via GitHub Actions.

## 🏗️ Architecture Cible

L'infrastructure reproduit un environnement d'entreprise avec une segmentation stricte :
* **VM1 - Passerelle (pfSense) :** Routeur et Pare-feu. Isole les réseaux et gère le NAT vers Internet.
* **VM2 - Serveur Web (Ubuntu 22.04) :** Zone DMZ (`192.168.100.0/24`). Héberge Nginx en reverse proxy et une application Node.js.
* **VM3 - Serveur BDD (Ubuntu 22.04) :** Zone LAN interne (`192.168.10.0/24`). Héberge MySQL. Sécurisé par une `NetworkPolicy` stricte bloquant tout trafic sauf celui venant de la VM2 sur le port 3306.

## 🚀 Fonctionnement du Déploiement (GitOps)

Toute modification apportée aux fichiers YAML de ce dépôt déclenche automatiquement un workflow GitHub Actions :
1. Connexion sécurisée au cluster OpenShift (via les tokens `OC_TOKEN`).
2. Création des switchs virtuels (`NetworkAttachmentDefinition`) et des règles de sécurité (`NetworkPolicy`).
3. Provisionnement séquentiel des VMs via KubeVirt (DB -> Web -> Firewall).
4. Auto-configuration des OS (Ubuntu) via `cloud-init` intégré dans les manifestes Kubernetes.

## 🛠️ Prérequis

Pour que le pipeline fonctionne, les secrets suivants doivent être configurés dans **Settings > Secrets and variables > Actions** de ce repository :
* `OC_SERVER` : L'URL de l'API de votre cluster OpenShift (ex: `https://api.mon-cluster.com:6443`).
* `OC_TOKEN` : Le token d'authentification d'un compte de service avec les droits d'administration sur le namespace `mon-projet`.