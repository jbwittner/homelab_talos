# Talos Cluster Configuration Repository

Gestionnaire de configuration pour clusters Kubernetes Talos Linux, utilisant `talhelper` pour la gestion déclarative et SOPS pour le chiffrement des secrets.

## Vue d'ensemble

Ce dépôt centralise les configurations de clusters Talos Linux, permettant une gestion reproductible, versionée et sécurisée des clusters Kubernetes. Chaque cluster est isolé dans son propre répertoire à la racine du dépôt.

## Structure du répertoire

```
homelab_talos/
├── README.md                         # Ce fichier
├── .sops.yaml                        # Règles de chiffrement age (global)
├── neltharion/                       # Cluster actif
│   ├── README.md                     # Documentation du cluster
│   ├── talconfig.yaml                # Configuration principale du cluster
│   ├── talsecret.sops.yaml           # Secrets chiffrés avec SOPS
│   └── clusterconfig/                # Généré par talhelper (gitignored)
│       ├── talosconfig               # Configuration client Talos
│       └── *.yaml                    # Configurations spécifiques aux nœuds
└── ysera/                            # Cluster actif
    ├── README.md
    ├── talconfig.yaml
    └── talsecret.sops.yaml
```

## Prérequis

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

## Démarrage rapide

### Générer les configurations
```bash
cd neltharion/
talhelper genconfig
```

### Monitorer le nœud
```bash
cd neltharion/
talosctl -n 5.135.136.115 -e 5.135.136.115 dashboard --talosconfig=./clusterconfig/talosconfig
```

### Récupérer des informations
```bash
cd neltharion/

# État des disques
talosctl -n 5.135.136.115 -e 5.135.136.115 get disks --talosconfig=./clusterconfig/talosconfig

# Version du cluster
talosctl -n 5.135.136.115 -e 5.135.136.115 version --talosconfig=./clusterconfig/talosconfig
```

## Workflows courants

### Mettre à jour un cluster existant
```bash
cd neltharion/

# 1. Modifier talconfig.yaml
# 2. Régénérer les configurations
talhelper genconfig

# 3. Appliquer les changements
talhelper gencommand apply
# ou directement :
talosctl apply-config --talosconfig=./clusterconfig/talosconfig \
  --nodes=5.135.136.115 \
  --file=./clusterconfig/neltharion-ns3058844.yaml
```

### Démarrer un nouveau cluster
```bash
mkdir nouveau-cluster
cd nouveau-cluster

# 1. Créer talconfig.yaml avec la définition du cluster
# 2. Créer talsecret.sops.yaml avec les secrets chiffrés
# 3. Générer les configurations
talhelper genconfig

# 4. Appliquer aux nœuds (--insecure au premier démarrage)
talhelper gencommand apply --extra-flags --insecure

# 5. Bootstrapper le cluster
talhelper gencommand bootstrap

# 6. Récupérer le kubeconfig
talhelper gencommand kubeconfig
```

## Gestion des secrets avec SOPS

### Décrypter un secret
```bash
sops -d talsecret.sops.yaml
```

### Éditer un secret
```bash
sops talsecret.sops.yaml  # Lance l'éditeur en mode chiffrement
```

### Créer une nouvelle clé age
```bash
age-keygen -o age-key.txt
# Ajouter la clé publique dans .sops.yaml
# Sauvegarder la clé privée en lieu sûr
```

⚠️ **Ne jamais commiter le fichier `talsecret.yaml` déchiffré** (gitignored)

## Commandes talhelper courantes

| Commande | Effet |
|----------|-------|
| `talhelper genconfig` | Génère configs nœuds + talosconfig |
| `talhelper gencommand apply` | Affiche commande apply |
| `talhelper gencommand bootstrap` | Affiche commande bootstrap |
| `talhelper gencommand kubeconfig` | Affiche commande récupération kubeconfig |
| `talhelper gencommand upgrade` | Affiche commande upgrade |
| `talhelper gencommand reset` | Affiche commande reset cluster |

**Important :** Toujours exécuter depuis le répertoire du cluster (ex: `cd neltharion/`)

## Pièges courants

| Piège | Solution |
|-------|----------|
| SOPS ne peut décrypter | Clé privée age non disponible localement |
| `--insecure` rejeté | Utiliser seulement au démarrage initial |
| Disques non reconnus | Vérifier avec `talosctl get disks` |
| Modifications ignorées | Éditer `talconfig.yaml`, pas les fichiers `clusterconfig/` |

## Débogage

```bash
# Dashboard en temps réel
talosctl -n <IP> -e <IP> dashboard --talosconfig=./clusterconfig/talosconfig

# Logs système
talosctl -n <IP> -e <IP> logs --talosconfig=./clusterconfig/talosconfig

# État des disques (avant talosconfig : --insecure)
talosctl -n <IP> -e <IP> get disks --talosconfig=./clusterconfig/talosconfig

# Version et status
talosctl -n <IP> -e <IP> version --talosconfig=./clusterconfig/talosconfig
```

## Clusters

| Cluster | Statut | IP | Documentation |
|---------|--------|----|---------------|
| [neltharion](neltharion/) | Actif | 5.135.136.115 | [README](neltharion/README.md) |
| [ysera](ysera/) | Actif | 51.255.205.150 | [README](ysera/README.md) |

## Liens utiles

- [Documentation officielle Talos](https://www.talos.dev/)
- [talhelper GitHub](https://github.com/budimanjaya/talhelper)
- [SOPS GitHub](https://github.com/mozilla/sops)

---

**Dernière mise à jour :** juin 2026
