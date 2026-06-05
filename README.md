# Talos Cluster Configuration Repository

Gestionnaire de configuration pour clusters Kubernetes Talos Linux, utilisant `talhelper` pour la gestion déclarative et SOPS pour le chiffrement des secrets.

## 📋 Vue d'ensemble

Ce dépôt centralise les configurations de clusters Talos Linux, permettant une gestion reproductible, versionée et sécurisée des clusters Kubernetes. Chaque cluster est isolé dans son propre répertoire avec ses configurations, secrets chiffrés et résultats générés.

## 📁 Structure du répertoire

```
talos_cluster/
├── README.md                    # Ce fichier
├── AGENTS.md                    # Documentation détaillée (toolchain et patterns)
├── archive/                     # Clusters archivés/inactifs
│   ├── README.md
│   ├── talconfig.yaml
│   └── neltharion/
│       ├── talconfig.yaml
│       ├── talsecret.sops.yaml
│       └── clusterconfig/
├── nozdormu/                    # Cluster actif
│   ├── README.md
│   ├── talconfig.yaml           # Configuration principale du cluster
│   ├── talsecret.sops.yaml      # Secrets chiffrés avec SOPS
│   ├── .sops.yaml              # Règles et clés de chiffrement age
│   └── clusterconfig/          # Généré par talhelper
│       ├── talosconfig          # Configuration client Talos
│       └── *.yaml               # Configurations spécifiques aux nœuds
```

## 🛠 Prérequis

### Outils requis
- **talhelper** - Gestionnaire de configurations Talos
- **talosctl** - CLI Talos pour la gestion des nœuds
- **sops** - Outil de chiffrement/déchiffrement
- **age** - Chiffrement symétrique (back-end de SOPS)

### Installation macOS
```bash
brew install talhelper talosctl sops age
```

### Clés de chiffrement
- **Clé privée age** - Requise pour déchiffrer les secrets (non versionée)
- **Clé publique age** - Stockée dans `.sops.yaml` pour le chiffrement

## 🚀 Démarrage rapide

### 1. Visualiser un cluster existant
```bash
cd nozdormu/
talhelper genconfig      # Génère les configurations
```

### 2. Monitorer un nœud
```bash
cd nozdormu/
talosctl -n 91.134.140.158 -e 91.134.140.158 dashboard --talosconfig=./clusterconfig/talosconfig
```

### 3. Récupérer des informations
```bash
cd nozdormu/
# État des disques
talosctl -n 91.134.140.158 -e 91.134.140.158 get disks --talosconfig=./clusterconfig/talosconfig

# Status du cluster
talosctl -n 91.134.140.158 -e 91.134.140.158 version --talosconfig=./clusterconfig/talosconfig
```

## 📝 Fichiers clés

### talconfig.yaml
Configuration déclarative du cluster définissant :
- Nom du cluster
- Version Kubernetes/Talos
- Nœuds (hostname, rôle, IP, disque d'installation)
- Extensions système (iSCSI, outils, etc.)
- Volumes utilisateur pour stockage supplémentaire

**Exemple :**
```yaml
clusterName: cluster-talos
endpoint: https://91.134.140.158:6443
kubernetesVersion: v1.35.0
talosVersion: v1.12.2
allowSchedulingOnControlPlanes: true
nodes:
  - hostname: cp-node
    controlPlane: true
    ipAddress: 91.134.140.158
    installDisk: /dev/sda
    schematic:
      customization:
        systemExtensions:
          officialExtensions:
            - siderolabs/iscsi-tools
```

### talsecret.sops.yaml
Secrets du cluster **chiffrés** avec age :
- Certificats TLS
- Clés de machine
- Secrets Kubernetes
- Tokens d'API

⚠️ **Ne jamais commiter le fichier `talsecret.yaml` déchiffré** (gitignored)

Pour éditer :
```bash
cd nozdormu/
sops talsecret.sops.yaml
```

### .sops.yaml
Règles de chiffrement age du cluster :
```yaml
creation_rules:
  - path_regex: talsecret.sops.yaml
    age: <CLÉ_PUBLIQUE_AGE>
```

## 🔄 Workflows courants

### Démarrer un nouveau cluster
```bash
mkdir clusters/nouveau-cluster
cd clusters/nouveau-cluster

# 1. Créer talconfig.yaml avec la définition du cluster
# 2. Créer .sops.yaml avec la clé publique age
# 3. Créer talsecret.sops.yaml avec les secrets chiffrés
# 4. Générer les configurations
talhelper genconfig

# 5. Appliquer aux nœuds (--insecure sauf si talosconfig existe)
talhelper gencommand apply --extra-flags --insecure
# Copier et exécuter la commande affichée

# 6. Bootstrapper le cluster
talhelper gencommand bootstrap
# Exécuter sur le nœud control plane

# 7. Récupérer le kubeconfig
talhelper gencommand kubeconfig
```

### Mettre à jour un cluster existant
```bash
cd nozdormu/

# 1. Modifier talconfig.yaml
# 2. Régénérer les configurations
talhelper genconfig

# 3. Appliquer les changements
talhelper gencommand apply
# ou appliquer directement :
talosctl apply-config --talosconfig=./clusterconfig/talosconfig \
  --nodes=91.134.140.158 \
  --file=./clusterconfig/nozdormu-talos-vps-e920ec2a.yaml
```

### Ajouter du stockage (Longhorn)
```bash
cd nozdormu/

# 1. Vérifier les disques disponibles
talosctl -n 91.134.140.158 -e 91.134.140.158 get disks --talosconfig=./clusterconfig/talosconfig

# 2. Ajouter userVolumes à talconfig.yaml pour le nœud
# 3. Régénérer et appliquer
talhelper genconfig
talhelper gencommand apply
```

### Archiver un cluster
```bash
# Déplacer le répertoire du cluster dans archive/
mv nozdormu/ archive/
git add archive/nozdormu
git commit -m "Archive cluster nozdormu"
```

## 📊 Gestion des secrets avec SOPS

### Décrypter un secret
```bash
sops -d talsecret.sops.yaml
```

### Chiffrer un secret
```bash
sops talsecret.sops.yaml  # Lance l'éditeur en mode chiffrement
```

### Créer une nouvelle clé age
```bash
age-keygen -o age-key.txt
# Ajouter la clé publique dans .sops.yaml
# Sauvegarder la clé privée en lieu sûr
```

## ⚙️ Commands talhelper courants

| Commande | Effet |
|----------|-------|
| `talhelper genconfig` | Génère configs nœuds + talosconfig |
| `talhelper gencommand apply` | Affiche commande apply |
| `talhelper gencommand bootstrap` | Affiche commande bootstrap |
| `talhelper gencommand kubeconfig` | Affiche commande récupération kubeconfig |
| `talhelper gencommand upgrade` | Affiche commande upgrade |
| `talhelper gencommand reset` | Affiche commande reset cluster |

**⚠️ Important :** Toujours exécuter depuis le répertoire du cluster (ex: `cd nozdormu/`)

## ⚠️ Pièges courants

| Piège | Solution |
|-------|----------|
| `talhelper: command not found` | Exécuter depuis le répertoire du cluster |
| SOPS ne peut décrypter | Clé privée age non disponible localement |
| `--insecure` rejeté | Utiliser seulement au démarrage initial |
| Disques non reconnus | Vérifier avec `talosctl get disks` |
| Modifications ignorées | Éditer `talconfig.yaml`, pas les fichiers `clusterconfig/` |

## 🔍 Débogage

### Dashboard en temps réel
```bash
cd cluster/
talosctl -n <IP> -e <IP> dashboard --talosconfig=./clusterconfig/talosconfig
```

### Logs système
```bash
talosctl -n <IP> -e <IP> logs --talosconfig=./clusterconfig/talosconfig
```

### État des disques
```bash
talosctl -n <IP> -e <IP> get systemdisks --insecure  # Avant talosconfig
talosctl -n <IP> -e <IP> get disks --talosconfig=./clusterconfig/talosconfig
```

### Version et status
```bash
talosctl -n <IP> -e <IP> version --talosconfig=./clusterconfig/talosconfig
talosctl -n <IP> -e <IP> status --talosconfig=./clusterconfig/talosconfig
```

## 📚 Documentation supplémentaire

- **[AGENTS.md](./AGENTS.md)** - Guide technique complet (toolchain, patterns, workflows avancés)
- [Documentation officielle Talos](https://www.talos.dev/)
- [talhelper GitHub](https://github.com/budimanjaya/talhelper)
- [SOPS GitHub](https://github.com/mozilla/sops)

## 🤝 Contribution

Avant de commiter :
1. Valider avec `talhelper genconfig`
2. Ne jamais commiter `talsecret.yaml` déchiffré
3. Utiliser des messages de commit clairs
4. Documenter les changements significatifs

## 📄 License

[Ajouter votre license ici]

---

**Dernière mise à jour :** février 2026
