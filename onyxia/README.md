# Cluster onyxia

Cluster Kubernetes Talos Linux mono-nœud hébergé chez OVH.

## Informations

| Champ | Valeur |
|-------|--------|
| Nom | onyxia |
| Endpoint | https://192.168.1.113:6443 |
| Talos | v1.13.3 |
| Kubernetes | v1.36.1 |
| Nœud | ns3058844 — control plane |
| IP | 192.168.1.113 |

## Stockage

| Disque | Série | Modèle | Taille | Transport | Rôle |
|--------|-------|--------|--------|-----------|------|
| nvme0n1 | 2102357DWVN0RA105082 | EXT X200E SSD | 1.0 TB | nvme | Système (installDisk) |
| sda | — | UDisk | 16 GB | usb | Média USB (boot) |

### Vérifier le stockage

**Avant config (mode `maintenance`, pas encore de `talosconfig`)** — utiliser `--insecure`.
C'est l'étape pour déterminer les séries de disques à mettre dans `talconfig.yaml` :

```bash
# Disques physiques (modèle, série, taille, transport)
talosctl get disks --insecure --nodes 192.168.1.113

# Volumes détectés (partitions, systèmes de fichiers, labels)
talosctl get discoveredvolumes --insecure --nodes 192.168.1.113
```

> `--insecure` car aucun certificat TLS en place avant l'application de la config.
> `volumestatus` / `uservolumeconfig` ne sont pas disponibles en mode maintenance — voir après config.

**Après config (cluster bootstrappé)** — utiliser `talosconfig` :

```bash
cd onyxia/

# Disques physiques
talosctl -n 192.168.1.113 -e 192.168.1.113 get disks \
  --talosconfig=./clusterconfig/talosconfig

# Volumes détectés
talosctl -n 192.168.1.113 -e 192.168.1.113 get discoveredvolumes \
  --talosconfig=./clusterconfig/talosconfig

# Statut des volumes gérés (phase, taille, point de montage)
talosctl -n 192.168.1.113 -e 192.168.1.113 get volumestatus \
  --talosconfig=./clusterconfig/talosconfig

# Volume utilisateur `data` (config déclarée)
talosctl -n 192.168.1.113 -e 192.168.1.113 get uservolumeconfig \
  --talosconfig=./clusterconfig/talosconfig

# Points de montage et espace utilisé
talosctl -n 192.168.1.113 -e 192.168.1.113 mounts \
  --talosconfig=./clusterconfig/talosconfig
```

## Bootstrap (première installation)

Procédure complète pour installer Talos et bootstrapper le cluster depuis zéro.

### Prérequis

Le nœud doit démarrer sur l'ISO Talos (mode `maintenance`). Vérifier qu'il est joignable :
```bash
talosctl version --insecure --nodes 192.168.1.113
```

### 1. Générer les configurations

```bash
cd neltharion/
talhelper genconfig
```

Produit dans `clusterconfig/` :
- `talosconfig` — configuration client
- `neltharion-ns3058844.yaml` — configuration du nœud

### 2. Appliquer la configuration au nœud

```bash
talosctl apply-config --insecure \
  --nodes 192.168.1.113 \
  --file ./clusterconfig/neltharion-ns3058844.yaml
```

> `--insecure` est obligatoire au premier démarrage car il n'y a pas encore de certificat TLS en place. Le nœud redémarre automatiquement après l'application.

### 3. Attendre que le nœud soit prêt

Surveiller le démarrage (attendre que `TYPE` passe à `running`) :
```bash
talosctl -n 192.168.1.113 -e 192.168.1.113 \
  --talosconfig=./clusterconfig/talosconfig \
  health --wait-timeout 10m
```

Ou via le dashboard :
```bash
talosctl -n 192.168.1.113 -e 192.168.1.113 dashboard \
  --talosconfig=./clusterconfig/talosconfig
```

### 4. Bootstrapper etcd (une seule fois)

À exécuter **une seule fois** sur le premier nœud control plane :
```bash
talosctl bootstrap \
  --nodes 192.168.1.113 \
  --endpoints 192.168.1.113 \
  --talosconfig=./clusterconfig/talosconfig
```

> Ne jamais relancer cette commande une fois le cluster démarré.

### 5. Récupérer le kubeconfig

```bash
talosctl kubeconfig \
  --nodes 192.168.1.113 \
  --endpoints 192.168.1.113 \
  --talosconfig=./clusterconfig/talosconfig \
  --force
```

Le kubeconfig est fusionné dans `~/.kube/config`. Vérifier l'accès :
```bash
kubectl --context=admin@neltharion get nodes
```

---

## Commandes courantes

```bash
# Générer les configurations
talhelper genconfig

# Dashboard en temps réel
talosctl -n 192.168.1.113 -e 192.168.1.113 dashboard \
  --talosconfig=./clusterconfig/talosconfig

# État des disques
talosctl -n 192.168.1.113 -e 192.168.1.113 get disks \
  --talosconfig=./clusterconfig/talosconfig

# Appliquer la configuration
talosctl apply-config \
  --talosconfig=./clusterconfig/talosconfig \
  --nodes=192.168.1.113 \
  --file=./clusterconfig/neltharion-ns3058844.yaml
```

## Fichiers

| Fichier | Description |
|---------|-------------|
| `talconfig.yaml` | Configuration déclarative du cluster |
| `talsecret.sops.yaml` | Secrets chiffrés (certificats, tokens) |
| `clusterconfig/talosconfig` | Configuration client talosctl (généré) |
| `clusterconfig/neltharion-ns3058844.yaml` | Config du nœud (généré) |

Pour éditer les secrets :
```bash
sops talsecret.sops.yaml
```
