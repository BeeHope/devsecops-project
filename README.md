# 🔐 DevSecOps — Parcours Complet UEMF

> **Auteure :** Asma Daoui — 4ème année Ingénierie Cybersécurité  
> **Module :** DevSecOps | **Année :** 2025–2026
> **Université :** Euro-Méditerranéenne de Fès (UEMF) — EDIA  
> **GitHub :** [@BeeHope](https://github.com/BeeHope)

---

## 📋 Vue d'Ensemble

Ce dépôt regroupe **5 projets progressifs** construits tout au long du module DevSecOps, couvrant l'intégralité du cycle de vie logiciel sécurisé — du code au cloud.

```
Code → Test → Security Scan → Build → Deploy → Monitor
  Dev        Sec                          Ops
```

---

## 🗂️ Structure du Dépôt

```
devsecops-project/
├── pipline1notes-app/      # Pipeline 1 — Notes App (GitHub Actions)
├── pipline2-terraform-aws/ # Pipeline 2 — Infrastructure as Code (Terraform AWS)
├── pipline3-crud-app/      # Pipeline 3 — CRUD App (Trivy + Kubernetes 3 replicas)
├── pipline4-secnotes/      # Pipeline 4 — SecNotes (SonarQube + OWASP ZAP + EKS)
├── pipline5-weatherops/    # Pipeline 5 — WeatherOps (GitLab CI/CD + AWS EC2)
├── screenshots/            # Captures d'écran des pipelines
└── docs/                   # Documentation et rapport LaTeX
```

---

## 🚀 Projets

### Pipeline 1 — Notes App

Application web de gestion de notes. Premier contact avec Docker et CI/CD.

| | |
|---|---|
| **Stack** | Node.js + Express |
| **Tests** | Jest (5 tests unitaires) |
| **CI/CD** | GitHub Actions |
| **Container** | Docker (`node:18-alpine`) |

**Pipeline :**
```
checkout → setup-node → install → test → npm-audit
```

**Lancement rapide :**
```bash
docker build -t tp1:1.0 .
docker run -p 4000:3000 tp1:1.0
# → http://localhost:4000
```

---

### Pipeline 2 — AWS Terraform Infrastructure

Infrastructure AWS complète provisionnée en IaC : EC2, S3, Security Group, et pipeline CI/CD de déploiement.

| | |
|---|---|
| **IaC** | Terraform (`~> 5.0`) |
| **Cloud** | AWS EC2 (`t3.micro`) + S3 Bucket |
| **CI/CD** | GitLab CI/CD (build, test, security, package, deploy) |
| **Registry** | GitLab Container Registry |
| **Env. local** | WSL Ubuntu sur Windows |

**Déploiement Terraform :**
```bash
cd terraform/
terraform init
terraform plan
terraform apply
terraform output  # → IP publique de l'instance
```

**Variables d'environnement requises :**
```bash
aws configure
# AWS Access Key ID     : <iam-user-key>
# AWS Secret Access Key : <iam-user-secret>
# Default region        : us-east-1
# Output format         : json
```

---

### Pipeline 3 — CRUD App

API REST CRUD Node.js/MongoDB avec scan d'image Trivy et déploiement Kubernetes **3 replicas**.

| | |
|---|---|
| **Stack** | Node.js + Express + MongoDB |
| **Tests** | Jest + Supertest |
| **Image Scan** | Trivy (HIGH/CRITICAL CVE) |
| **CI/CD** | GitLab CI/CD — 4 stages |
| **Container** | Docker **multi-stage build** (`node:20-alpine`) |
| **Orchestration** | Docker Desktop K8s (3 replicas + StatefulSet MongoDB) |
| **Cloud** | AWS EC2 via Terraform |

**Endpoints :**
```
GET    /health      → Health check
POST   /items       → Créer un item
GET    /items       → Lister tous les items
GET    /items/:id   → Récupérer un item
PUT    /items/:id   → Modifier un item
DELETE /items/:id   → Supprimer un item
```

**Pipeline :**
```
build → test → security (Trivy) → deploy (kubectl)
```

**Lancement Kubernetes :**
```bash
kubectl apply -f k8s/ -n crud-app
kubectl get pods -n crud-app
# → http://localhost:30080/health
```

**Générer KUBE_CONFIG pour GitLab :**
```powershell
[Convert]::ToBase64String([System.IO.File]::ReadAllBytes("$env:USERPROFILE\.kube\config"))
```

---

### Pipeline 4 — SecNotes Vulnerable App

API REST Express/MongoDB avec **10 vulnérabilités pédagogiques intentionnelles**. Projet le plus complet en termes d'analyse de sécurité.

| | |
|---|---|
| **Stack** | Node.js + Express + MongoDB |
| **Tests** | Jest + Supertest + MongoDB Memory Server |
| **SAST** | SonarQube |
| **DAST** | OWASP ZAP (baseline scan) |
| **CI/CD** | GitLab CI/CD — **7 stages, 12 jobs** |
| **Container** | Docker (volontairement non sécurisé — V7) |
| **Orchestration** | Kubernetes local |
| **IaC** | Terraform (plan cluster EKS AWS) |

**Vulnérabilités :**

| ID | Catégorie | Description |
|---|---|---|
| V1 | NoSQL Injection | Requêtes Mongoose non filtrées |
| V2 | Secrets en dur | JWT secret hardcodé |
| V3 | Validation absente | Aucun schéma de validation |
| V4 | IDOR | Contrôle d'accès insuffisant |
| V5 | Dépendances obsolètes | express 4.17.1, lodash 4.17.19 |
| V6 | Hashage faible | Mots de passe MD5 |
| V7 | Dockerfile non sécurisé | Exécution en root |
| V8 | Logs verbeux | Données sensibles dans les logs |
| V9 | Secrets K8s | Base64 non chiffré |
| V10 | CORS permissif | Toutes origines autorisées |

**Pipeline :**
```
install → test → sast (SonarQube) → build → deploy_k8s → zap (DAST) → terraform_plan
```

> ✅ **Pipeline #2435952558** — 12 jobs en succès en 1h07m40s.

---

### Pipeline 5 — WeatherOps

API météo REST avec données simulées pour 5 villes. Introduit la détection de secrets et Kubernetes.

| | |
|---|---|
| **Stack** | Node.js + Express |
| **Tests** | Jest + Supertest (6 tests) |
| **Sécurité** | Détection de secrets hardcodés (regex) |
| **CI/CD** | GitLab CI/CD — 5 stages |
| **Container** | Docker (`node:20-alpine`) |
| **Orchestration** | Kubernetes (namespace, deployment, service) |
| **Cloud** | AWS EC2 `t3.micro` via Terraform |

**Endpoints :**
```
GET /               → Dashboard (liste des villes)
GET /health         → Statut API
GET /weather        → Toutes les villes
GET /weather/:city  → Météo d'une ville
```

**Pipeline :**
```
test → security → build → deploy_staging (auto) → deploy_production (manuel)
```

> ⚠️ **Vulnérabilité démontrée :** `const API_KEY = "sk-demo-123456789"` détectée par le stage `security` → pipeline bloqué. Correction : utiliser `process.env.API_KEY`.

**Lancement rapide :**
```bash
docker build -t weatherops:latest .
docker run -p 9000:3000 --env-file .env weatherops:latest
# → http://localhost:9000
```

---

## 🛡️ Outils de Sécurité

| Outil | Type | Projet | Description |
|---|---|---|---|
| `grep`/regex | SAST basique | Pipeline 5 — WeatherOps | Détection de patterns de secrets (API keys) |
| **SonarQube** | SAST complet | Pipeline 4 — SecNotes | Analyse statique, détection vulnérabilités |
| **Trivy** | Image Scan | Pipeline 3 — CRUD App | Scan CVE images Docker (OS + dépendances) |
| **OWASP ZAP** | DAST | Pipeline 4 — SecNotes | Tests dynamiques sur l'app en cours d'exécution |
| `npm audit` | Dépendances | Tous | Audit vulnérabilités packages npm |

---

## 📊 Comparaison des Pipelines

| Feature | Pipeline 1 | Pipeline 2 | Pipeline 3 | Pipeline 4 | Pipeline 5 |
|---|:---:|:---:|:---:|:---:|:---:|
| **App** | Notes App | Terraform AWS | CRUD App | SecNotes | WeatherOps |
| **CI/CD Platform** | GitHub | GitLab | GitLab | GitLab | GitLab |
| **Stages** | 3 | 5 | 4 | 7 | 5 |
| **Tests auto.** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Secret detect.** | — | — | — | ✅ | ✅ |
| **SAST** | — | — | — | SonarQube | — |
| **Image scan** | — | — | Trivy | — | — |
| **DAST** | — | — | — | OWASP ZAP | — |
| **Kubernetes** | — | — | ✅ | ✅ | ✅ |
| **AWS** | — | EC2 + S3 | EC2 | EKS (plan) | EC2 |
| **Terraform** | — | ✅ | ✅ | ✅ | ✅ |

---

## ⚙️ Prérequis

```bash
# Node.js
node --version  # >= 18

# Docker
docker --version

# kubectl
kubectl version --client

# Terraform
terraform -version  # >= 1.5

# AWS CLI
aws --version
```

---

## 🔑 Bonnes Pratiques Appliquées

- **`.gitignore` / `.dockerignore`** — Exclusion de `node_modules`, `.env`, fichiers sensibles
- **`npm ci`** — Installation reproductible et déterministe
- **Images Alpine** — Surface d'attaque minimale (`node:20-alpine`)
- **Multi-stage Docker** — Séparation build/runtime (Pipeline 3 — CRUD App)
- **Probes Kubernetes** — `readinessProbe` + `livenessProbe` sur `/health`
- **Variables CI/CD GitLab** — Secrets injectés (protégés, masqués, cachés)
- **Staging → Production** — Déploiement auto en staging, manuel en prod

---

## 📈 Améliorations Recommandées

1. **HashiCorp Vault / AWS Secrets Manager** — Gestion centralisée des secrets
2. **Sealed Secrets Kubernetes** — Chiffrement réel des secrets K8s
3. **NetworkPolicy Kubernetes** — Isolation réseau des pods
4. **OWASP ZAP Full Scan** — Scan DAST complet (vs baseline)
5. **Dependabot / Renovate** — Mise à jour automatique des dépendances
6. **SBOM** — Inventaire des composants (Software Bill of Materials)

---

## 📄 Rapport

Le rapport final de synthèse est disponible dans le dossier `docs/` : [`docs/rapport_final_devsecops.tex`](docs/rapport_final_devsecops.tex)

Compilation :
```bash
pdflatex rapport_final_devsecops.tex
pdflatex rapport_final_devsecops.tex  # 2ème passe pour la TOC
```

---

## 📝 Licence

Projet académique — UEMF 2024–2025. Usage pédagogique uniquement.

---

*"La sécurité n'est pas une fonctionnalité à ajouter à la fin : c'est une discipline à intégrer à chaque commit, chaque build, chaque déploiement."*
