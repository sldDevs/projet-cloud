# 🖥️ TP OpenShift Virtualization
## Architecture Hybride : 2 VMs Fedora + 1 Conteneur Nginx

> **Namespace :** `seckou-dev`  
> **Plateforme :** Red Hat OpenShift Developer Sandbox

---

## 📐 Architecture

```
                        INTERNET
                            │
                   ┌────────▼────────┐
                   │  VM1 - Fedora   │  ← VirtualMachine OpenShift
                   │  pfSense/       │
                   │  iptables       │
                   │  (Firewall/NAT) │
                   └────────┬────────┘
                            │
              ┌─────────────┼──────────────┐
              │             │              │
     ┌────────▼──────┐     │    ┌─────────▼────────┐
     │  VM2 - Fedora │     │    │  Conteneur Nginx  │
     │  MySQL 8.0    │◄────┘    │  + Node.js        │
     │  (Base de DB) │          │  (Serveur Web)    │
     │ VirtualMachine│          │  Pod OpenShift    │
     └───────────────┘          └──────────────────┘
                                       │
                                  Route publique
                                  (HTTPS OpenShift)
```

---

## 🗂️ Structure du Repo

```
tp-openshift-virt/
├── .github/
│   └── workflows/
│       └── deploy.yml                  ← Pipeline CI/CD
├── network/
│   └── networkpolicy.yaml              ← Règles réseau (3 policies)
├── vms/
│   ├── vm1-pfsense/
│   │   └── virtualmachine.yaml         ← VM Fedora + iptables
│   └── vm2-mysql/
│       └── virtualmachine.yaml         ← VM Fedora + MySQL
├── conteneur/
│   └── nginx/
│       ├── deployment.yaml             ← Nginx + Node.js + Route
│       └── mysql-service.yaml          ← Service vers VM2 MySQL
└── README.md
```

---

## 🖥️ Composants

### VM1 — Firewall/Gateway (VirtualMachine Fedora)
| Paramètre | Valeur |
|-----------|--------|
| **Type** | OpenShift VirtualMachine |
| **OS** | Fedora (template OpenShift) |
| **Rôle** | Firewall NAT avec iptables |
| **CPU/RAM** | 1 core / 2 Gi |
| **Disque** | 30 Gi |
| **User/Pass** | `fedora` / `Pfsense123!` |

### VM2 — Base de données MySQL (VirtualMachine Fedora)
| Paramètre | Valeur |
|-----------|--------|
| **Type** | OpenShift VirtualMachine |
| **OS** | Fedora (template OpenShift) |
| **Rôle** | Serveur MySQL 8.0 |
| **CPU/RAM** | 1 core / 2 Gi |
| **Disque** | 30 Gi |
| **User/Pass VM** | `fedora` / `Mysql123!` |
| **Base MySQL** | `appdb` |
| **User MySQL** | `appuser` / `AppPass123!` |

### Conteneur — Serveur Web (Pod OpenShift)
| Paramètre | Valeur |
|-----------|--------|
| **Type** | Deployment OpenShift |
| **Images** | `nginx:alpine` + `node:18-alpine` |
| **Rôle** | Reverse proxy + App Node.js |
| **Port** | 80 (interne) → Route HTTPS (public) |
| **Connexion DB** | `vm2-mysql-service:3306` |

---

## 🚀 Déploiement Manuel (Terminal OpenShift)

```bash
# 1. Cloner le repo
git clone https://github.com/seckoudiao/tp-openshift-virt.git
cd tp-openshift-virt

# 2. Sélectionner le namespace
oc project seckou-dev

# 3. NetworkPolicies
oc apply -f network/networkpolicy.yaml -n seckou-dev

# 4. VM1 pfSense
oc apply -f vms/vm1-pfsense/virtualmachine.yaml -n seckou-dev

# 5. VM2 MySQL
oc apply -f vms/vm2-mysql/virtualmachine.yaml -n seckou-dev

# 6. Conteneur Nginx + Node.js
oc apply -f conteneur/nginx/ -n seckou-dev
```

---

## 🔧 Vérifications

```bash
# Voir les VMs
oc get vm -n seckou-dev

# Voir les pods
oc get pods -n seckou-dev

# Voir la route publique (URL de ton app)
oc get route nginx-web-route -n seckou-dev

# Voir les NetworkPolicies
oc get networkpolicy -n seckou-dev

# Accès console VM1
virtctl console vm1-pfsense -n seckou-dev

# Accès console VM2
virtctl console vm2-mysql -n seckou-dev

# Logs du conteneur Nginx
oc logs -l app=nginx-web -n seckou-dev
```

---

## 🔐 Secrets GitHub

| Secret | Commande pour obtenir |
|--------|----------------------|
| `OC_TOKEN` | `oc whoami -t` |
| `OC_SERVER` | `https://api.rm2.thpm.p1.openshiftapps.com:6443` |

---

*TP Projet Fin de Module — OpenShift Virtualization + GitHub CI/CD*
