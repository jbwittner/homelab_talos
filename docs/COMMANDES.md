# Guide des commandes

Commandes générales pour gérer les clusters Talos de ce dépôt. Pour les IPs, noms de fichiers et procédures spécifiques, voir le README de chaque cluster ([neltharion](../neltharion/README.md), [ysera](../ysera/README.md), [onyxia](../onyxia/README.md)).

> **Important :** toujours exécuter les commandes depuis le répertoire du cluster concerné (ex. `cd neltharion/`). Les chemins `./clusterconfig/...` en dépendent.

## Démarrage rapide

### Générer les configurations
```bash
cd <cluster>/
talhelper genconfig
```

### Monitorer le nœud
```bash
talosctl -n <IP> -e <IP> dashboard --talosconfig=./clusterconfig/talosconfig
```

### Récupérer des informations
```bash
# État des disques
talosctl -n <IP> -e <IP> get disks --talosconfig=./clusterconfig/talosconfig

# Version du cluster
talosctl -n <IP> -e <IP> version --talosconfig=./clusterconfig/talosconfig
```

## Workflows courants

### Mettre à jour un cluster existant
```bash
cd <cluster>/

# 1. Modifier talconfig.yaml
# 2. Régénérer les configurations
talhelper genconfig

# 3. Appliquer les changements
talhelper gencommand apply
# ou directement :
talosctl apply-config --talosconfig=./clusterconfig/talosconfig \
  --nodes=<IP> \
  --file=./clusterconfig/<cluster>-<node>.yaml
```

### Démarrer un nouveau cluster (depuis zéro)
```bash
mkdir nouveau-cluster
cd nouveau-cluster

# 1. Créer talconfig.yaml avec la définition du cluster

# 2. Générer les secrets puis les chiffrer en place
talhelper gensecret > talsecret.sops.yaml
sops -e -i talsecret.sops.yaml

# 3. Générer les configurations (crée aussi clusterconfig/.gitignore)
talhelper genconfig

# 4. Appliquer aux nœuds (--insecure au premier démarrage)
talhelper gencommand apply --extra-flags --insecure

# 5. Bootstrapper le cluster
talhelper gencommand bootstrap

# 6. Récupérer le kubeconfig
talhelper gencommand kubeconfig
```

### Importer un cluster Talos existant
```bash
cd nouveau-cluster

# 1. Extraire la machine config du control plane
talosctl -n <controlplane-ip> get mc v1alpha1 -o jsonpath='{.spec}' > /tmp/machineconfig.yaml

# 2. Générer les secrets à partir de la config existante, puis chiffrer
talhelper gensecret -f /tmp/machineconfig.yaml > talsecret.sops.yaml
sops -e -i talsecret.sops.yaml

# 3. Écrire talconfig.yaml en se basant sur le cluster actuel
# 4. Générer et appliquer
talhelper genconfig
```

> `talhelper genconfig` régénère un `talosconfig` valide 365 jours à chaque exécution — le contenu diffère donc à chaque run.

## Commandes talhelper courantes

| Commande | Effet |
|----------|-------|
| `talhelper gensecret` | Génère les secrets (à chiffrer ensuite avec `sops -e -i`) |
| `talhelper genconfig` | Génère configs nœuds + talosconfig |
| `talhelper gencommand apply` | Affiche commande apply |
| `talhelper gencommand bootstrap` | Affiche commande bootstrap |
| `talhelper gencommand kubeconfig` | Affiche commande récupération kubeconfig |
| `talhelper gencommand upgrade` | Affiche commande upgrade |
| `talhelper gencommand reset` | Affiche commande reset cluster |

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

⚠️ **Ne jamais commiter le fichier `talsecret.yaml` déchiffré** (gitignored).

## Débogage

```bash
# Dashboard en temps réel
talosctl -n <IP> -e <IP> dashboard --talosconfig=./clusterconfig/talosconfig

# Logs système
talosctl -n <IP> -e <IP> logs --talosconfig=./clusterconfig/talosconfig

# État des disques (avant talosconfig : ajouter --insecure)
talosctl -n <IP> -e <IP> get disks --talosconfig=./clusterconfig/talosconfig

# Version et status
talosctl -n <IP> -e <IP> version --talosconfig=./clusterconfig/talosconfig
```

## Pièges courants

| Piège | Solution |
|-------|----------|
| SOPS ne peut décrypter | Clé privée age non disponible localement |
| `--insecure` rejeté | Utiliser seulement au démarrage initial (mode maintenance) |
| Disques non reconnus | Vérifier avec `talosctl get disks` |
| Modifications ignorées | Éditer `talconfig.yaml`, pas les fichiers `clusterconfig/` |
| Commande exécutée hors dossier cluster | `cd <cluster>/` avant toute commande |
