# Cluster ysera

Cluster Kubernetes Talos Linux mono-nœud hébergé chez OVH.

## Informations

| Champ | Valeur |
|-------|--------|
| Nom | ysera |
| Endpoint | https://51.255.205.150:6443 |
| Talos | v1.13.5 |
| Kubernetes | v1.36.1 |
| Nœud | vps-7a5b6065 — control plane |
| IP | 51.255.205.150/24 |
| Interface | eno1 — gateway 51.255.205.254 |

## Bootstrap (première installation)

Procédure complète pour installer Talos et bootstrapper le cluster depuis zéro.

### Prérequis

Le nœud doit démarrer sur l'ISO Talos (mode `maintenance`). Vérifier qu'il est joignable :
```bash
talosctl version --insecure --nodes 51.255.205.150
```

### 1. Générer les configurations

```bash
cd ysera/
talhelper genconfig
```

Produit dans `clusterconfig/` :
- `talosconfig` — configuration client
- `ysera-vps-7a5b6065.yaml` — configuration du nœud

### 2. Appliquer la configuration au nœud

```bash
talosctl apply-config --insecure \
  --nodes 51.255.205.150 \
  --file ./clusterconfig/ysera-vps-7a5b6065.yaml
```

> `--insecure` est obligatoire au premier démarrage car il n'y a pas encore de certificat TLS en place. Le nœud redémarre automatiquement après l'application.

### 3. Attendre que le nœud soit prêt

Surveiller le démarrage (attendre que `TYPE` passe à `running`) :
```bash
talosctl -n 51.255.205.150 -e 51.255.205.150 \
  --talosconfig=./clusterconfig/talosconfig \
  health --wait-timeout 10m
```

Ou via le dashboard :
```bash
talosctl -n 51.255.205.150 -e 51.255.205.150 dashboard \
  --talosconfig=./clusterconfig/talosconfig
```

### 4. Bootstrapper etcd (une seule fois)

À exécuter **une seule fois** sur le premier nœud control plane :
```bash
talosctl bootstrap \
  --nodes 51.255.205.150 \
  --endpoints 51.255.205.150 \
  --talosconfig=./clusterconfig/talosconfig
```

> Ne jamais relancer cette commande une fois le cluster démarré.

### 5. Récupérer le kubeconfig

```bash
talosctl kubeconfig \
  --nodes 51.255.205.150 \
  --endpoints 51.255.205.150 \
  --talosconfig=./clusterconfig/talosconfig \
  --force
```

Le kubeconfig est fusionné dans `~/.kube/config`. Vérifier l'accès :
```bash
kubectl --context=admin@ysera get nodes
```

---

## Commandes courantes

```bash
# Générer les configurations
talhelper genconfig

# Dashboard en temps réel
talosctl -n 51.255.205.150 -e 51.255.205.150 dashboard \
  --talosconfig=./clusterconfig/talosconfig

# État des disques
talosctl -n 51.255.205.150 -e 51.255.205.150 get disks \
  --talosconfig=./clusterconfig/talosconfig

# Appliquer la configuration
talosctl apply-config \
  --talosconfig=./clusterconfig/talosconfig \
  --nodes=51.255.205.150 \
  --file=./clusterconfig/ysera-vps-7a5b6065.yaml
```

## Fichiers

| Fichier | Description |
|---------|-------------|
| `talconfig.yaml` | Configuration déclarative du cluster |
| `talsecret.sops.yaml` | Secrets chiffrés (certificats, tokens) |
| `clusterconfig/talosconfig` | Configuration client talosctl (généré) |
| `clusterconfig/ysera-vps-7a5b6065.yaml` | Config du nœud (généré) |

Pour éditer les secrets :
```bash
sops talsecret.sops.yaml
```
