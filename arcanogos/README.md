# Cluster kalecgos

Cluster Kubernetes Talos Linux mono-nœud, homelab (réseau local).

## Informations

| Champ | Valeur |
|-------|--------|
| Nom | kalecgos (clusterName `bleu-kalecgos`) |
| Endpoint | https://192.168.1.12:6443 |
| Talos | v1.13.4 |
| Kubernetes | v1.36.2 |
| Nœud | vert-eranikus — control plane |
| IP | 192.168.1.12 |

> Mono-nœud : `allowSchedulingOnControlPlanes: true` — workloads autorisés sur le control plane.

## Stockage

| Disque | Série | Rôle |
|--------|-------|------|
| nvme0n1 | 202512100355 | Système (installDisk) |

## Paramètres

Valeurs de ce cluster (déjà substituées dans les commandes ci-dessous) :

| Placeholder | Valeur |
|-------------|--------|
| `<IP>` | 192.168.1.12 |
| `<cluster>` | kalecgos |
| `<node-file>` | bleu-kalecgos-vert-eranikus.yaml |
| `<context>` | admin@bleu-kalecgos |

> Toujours exécuter les commandes depuis ce répertoire (`cd kalecgos/`). Les chemins `./clusterconfig/...` en dépendent.
> Docs génériques : [BOOTSTRAP.md](../docs/BOOTSTRAP.md), [COMMANDES.md](../docs/COMMANDES.md).

---

# Bootstrap (première installation)

## 0. Vérifier le stockage (avant config)

Mode `maintenance` (pas encore de `talosconfig`) → `--insecure`. Étape pour déterminer les séries de disques à mettre dans `talconfig.yaml` :

```bash
# Disques physiques (modèle, série, taille, transport)
talosctl get disks --insecure --nodes 192.168.1.12

# Volumes détectés (partitions, systèmes de fichiers, labels)
talosctl get discoveredvolumes --insecure --nodes 192.168.1.12
```

> `--insecure` car aucun certificat TLS avant l'application de la config.
> `volumestatus` / `uservolumeconfig` indisponibles en mode maintenance — voir après config.

## Prérequis

Le nœud doit démarrer sur l'ISO Talos (mode `maintenance`). Vérifier qu'il est joignable :
```bash
talosctl version --insecure --nodes 192.168.1.12
```

## 1. Générer les secrets (avant la config)

`talhelper genconfig` a besoin de `talsecret.sops.yaml`. Le générer puis le chiffrer **avant** l'étape suivante :

```bash
cd kalecgos/
talhelper gensecret > talsecret.sops.yaml
sops -e -i talsecret.sops.yaml
```

> `gensecret` écrit les secrets en clair — toujours enchaîner avec `sops -e -i`. Un `.sops.yaml` valide (règle + clé age) doit exister à la racine du dépôt.
> Si le cluster existe déjà, réutiliser le `talsecret.sops.yaml` existant et sauter cette étape.

## 2. Générer les configurations

```bash
cd kalecgos/
talhelper genconfig
```

Produit dans `clusterconfig/` :
- `talosconfig` — configuration client
- `bleu-kalecgos-vert-eranikus.yaml` — configuration du nœud

## 3. Appliquer la configuration au nœud

```bash
talosctl apply-config --insecure \
  --nodes 192.168.1.12 \
  --file ./clusterconfig/bleu-kalecgos-vert-eranikus.yaml
```

> `--insecure` obligatoire au premier démarrage (pas encore de certificat TLS). Le nœud redémarre automatiquement après l'application.

## 4. Attendre que le nœud soit prêt

```bash
talosctl -n 192.168.1.12 -e 192.168.1.12 \
  --talosconfig=./clusterconfig/talosconfig \
  health --wait-timeout 10m
```

Ou via le dashboard :
```bash
talosctl -n 192.168.1.12 -e 192.168.1.12 dashboard \
  --talosconfig=./clusterconfig/talosconfig
```

## 5. Bootstrapper etcd (une seule fois)

À exécuter **une seule fois** sur le control plane :
```bash
talosctl bootstrap \
  --nodes 192.168.1.12 \
  --endpoints 192.168.1.12 \
  --talosconfig=./clusterconfig/talosconfig
```

> Ne jamais relancer cette commande une fois le cluster démarré.

## 6. Récupérer le kubeconfig

```bash
talosctl kubeconfig \
  --nodes 192.168.1.12 \
  --endpoints 192.168.1.12 \
  --talosconfig=./clusterconfig/talosconfig \
  --force
```

Le kubeconfig est fusionné dans `~/.kube/config`. Vérifier l'accès :
```bash
kubectl --context=admin@bleu-kalecgos get nodes
```

## Vérifier le stockage (après config)

Une fois le cluster bootstrappé, utiliser `talosconfig` :

```bash
# Disques physiques
talosctl -n 192.168.1.12 -e 192.168.1.12 get disks --talosconfig=./clusterconfig/talosconfig

# Volumes détectés
talosctl -n 192.168.1.12 -e 192.168.1.12 get discoveredvolumes --talosconfig=./clusterconfig/talosconfig

# Statut des volumes gérés (phase, taille, point de montage)
talosctl -n 192.168.1.12 -e 192.168.1.12 get volumestatus --talosconfig=./clusterconfig/talosconfig

# Volume utilisateur (config déclarée)
talosctl -n 192.168.1.12 -e 192.168.1.12 get uservolumeconfig --talosconfig=./clusterconfig/talosconfig

# Points de montage et espace utilisé
talosctl -n 192.168.1.12 -e 192.168.1.12 mounts --talosconfig=./clusterconfig/talosconfig
```

---

# Commandes courantes

## Démarrage rapide

### Générer les configurations
```bash
cd kalecgos/
talhelper genconfig
```

### Monitorer le nœud
```bash
talosctl -n 192.168.1.12 -e 192.168.1.12 dashboard --talosconfig=./clusterconfig/talosconfig
```

### Récupérer des informations
```bash
# État des disques
talosctl -n 192.168.1.12 -e 192.168.1.12 get disks --talosconfig=./clusterconfig/talosconfig

# Version du cluster
talosctl -n 192.168.1.12 -e 192.168.1.12 version --talosconfig=./clusterconfig/talosconfig
```

## Mettre à jour le cluster

```bash
cd kalecgos/

# 1. Modifier talconfig.yaml
# 2. Régénérer les configurations
talhelper genconfig

# 3. Appliquer les changements
talhelper gencommand apply
# ou directement :
talosctl apply-config --talosconfig=./clusterconfig/talosconfig \
  --nodes=192.168.1.12 \
  --file=./clusterconfig/bleu-kalecgos-vert-eranikus.yaml
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

### Générer le talsecret.sops.yaml
```bash
# Depuis zéro : générer les secrets puis les chiffrer en place
talhelper gensecret > talsecret.sops.yaml
sops -e -i talsecret.sops.yaml

# À partir d'une machine config existante
talhelper gensecret -f /tmp/machineconfig.yaml > talsecret.sops.yaml
sops -e -i talsecret.sops.yaml
```

> `talhelper gensecret` écrit les secrets en clair. Toujours enchaîner avec `sops -e -i` avant de commiter. Un `.sops.yaml` valide doit exister à la racine du dépôt.

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

⚠️ **Ne jamais commiter le `talsecret.yaml` déchiffré** (gitignored).

## Débogage

```bash
# Dashboard en temps réel
talosctl -n 192.168.1.12 -e 192.168.1.12 dashboard --talosconfig=./clusterconfig/talosconfig

# Logs système
talosctl -n 192.168.1.12 -e 192.168.1.12 logs --talosconfig=./clusterconfig/talosconfig

# État des disques (avant talosconfig : ajouter --insecure)
talosctl -n 192.168.1.12 -e 192.168.1.12 get disks --talosconfig=./clusterconfig/talosconfig

# Version et status
talosctl -n 192.168.1.12 -e 192.168.1.12 version --talosconfig=./clusterconfig/talosconfig
```

## Pièges courants

| Piège | Solution |
|-------|----------|
| SOPS ne peut décrypter | Clé privée age non disponible localement |
| `--insecure` rejeté | Utiliser seulement au démarrage initial (mode maintenance) |
| Disques non reconnus | Vérifier avec `talosctl get disks` |
| Modifications ignorées | Éditer `talconfig.yaml`, pas les fichiers `clusterconfig/` |
| Commande exécutée hors dossier cluster | `cd kalecgos/` avant toute commande |

## Fichiers du cluster

| Fichier | Description |
|---------|-------------|
| `talconfig.yaml` | Configuration déclarative du cluster |
| `talsecret.sops.yaml` | Secrets chiffrés (certificats, tokens) |
| `clusterconfig/talosconfig` | Configuration client talosctl (généré) |
| `clusterconfig/bleu-kalecgos-vert-eranikus.yaml` | Config du nœud (généré) |
