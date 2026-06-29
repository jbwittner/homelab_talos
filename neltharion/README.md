# Cluster neltharion

Cluster Kubernetes Talos Linux mono-nœud hébergé chez OVH.

## Informations

| Champ | Valeur |
|-------|--------|
| Nom | neltharion |
| Endpoint | https://5.135.136.115:6443 |
| Talos | v1.13.3 |
| Kubernetes | v1.36.1 |
| Nœud | ns3058844 — control plane |
| IP | 5.135.136.115 |
| Interface | eno1 — gateway 5.135.136.254 |

## Stockage

| Disque | Série | Taille | Rôle |
|--------|-------|--------|------|
| nvme1n1 | CVPF71620076450RGN | 450 GB | Système (installDisk) |
| nvme0n1 | CVPF6325009K450RGN | 450 GB | Data (UserVolumeConfig `data`, XFS) |

## Bootstrap (première installation)

Procédure complète pour installer Talos et bootstrapper le cluster depuis zéro.

### Prérequis

Le nœud doit démarrer sur l'ISO Talos (mode `maintenance`). Vérifier qu'il est joignable :
```bash
talosctl version --insecure --nodes 5.135.136.115
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
  --nodes 5.135.136.115 \
  --file ./clusterconfig/neltharion-ns3058844.yaml
```

> `--insecure` est obligatoire au premier démarrage car il n'y a pas encore de certificat TLS en place. Le nœud redémarre automatiquement après l'application.

### 3. Attendre que le nœud soit prêt

Surveiller le démarrage (attendre que `TYPE` passe à `running`) :
```bash
talosctl -n 5.135.136.115 -e 5.135.136.115 \
  --talosconfig=./clusterconfig/talosconfig \
  health --wait-timeout 10m
```

Ou via le dashboard :
```bash
talosctl -n 5.135.136.115 -e 5.135.136.115 dashboard \
  --talosconfig=./clusterconfig/talosconfig
```

### 4. Bootstrapper etcd (une seule fois)

À exécuter **une seule fois** sur le premier nœud control plane :
```bash
talosctl bootstrap \
  --nodes 5.135.136.115 \
  --endpoints 5.135.136.115 \
  --talosconfig=./clusterconfig/talosconfig
```

> Ne jamais relancer cette commande une fois le cluster démarré.

### 5. Récupérer le kubeconfig

```bash
talosctl kubeconfig \
  --nodes 5.135.136.115 \
  --endpoints 5.135.136.115 \
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
