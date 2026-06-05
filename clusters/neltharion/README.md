# Cluster neltharion

Cluster Kubernetes Talos Linux mono-nœud hébergé chez OVH.

## Informations

| Champ | Valeur |
|-------|--------|
| Nom | neltharion |
| Endpoint | https://5.135.136.115:6443 |
| Talos | v1.13.2 |
| Kubernetes | v1.36.1 |
| Nœud | ns3058844 — control plane |
| IP | 5.135.136.115 |
| Interface | eno1 — gateway 5.135.136.254 |

## Stockage

| Disque | Série | Taille | Rôle |
|--------|-------|--------|------|
| nvme0n1 | CVPF6325009K450RGN | 450 GB | Système (installDisk) |
| nvme1n1 | CVPF71620076450RGN | 450 GB | Data (UserVolumeConfig `data`, XFS) |

## Commandes courantes

```bash
# Générer les configurations
talhelper genconfig

# Dashboard en temps réel
talosctl -n 5.135.136.115 -e 5.135.136.115 dashboard \
  --talosconfig=./clusterconfig/talosconfig

# État des disques
talosctl -n 5.135.136.115 -e 5.135.136.115 get disks \
  --talosconfig=./clusterconfig/talosconfig

# Appliquer la configuration
talosctl apply-config \
  --talosconfig=./clusterconfig/talosconfig \
  --nodes=5.135.136.115 \
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
