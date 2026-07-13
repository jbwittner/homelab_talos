# Bootstrap d'un cluster (première installation)

Procédure générique pour installer Talos et bootstrapper un cluster mono-nœud depuis zéro. Elle s'applique à tous les clusters du dépôt — remplacer les placeholders par les valeurs du bloc **Paramètres** du README du cluster concerné :

| Placeholder | Signification |
|-------------|---------------|
| `<IP>` | IP du nœud control plane |
| `<cluster>` | Nom du répertoire / cluster |
| `<node-file>` | Fichier de config nœud généré dans `clusterconfig/` |
| `<context>` | Context kubectl (`admin@<cluster>`) |

> Toujours exécuter depuis le répertoire du cluster (`cd <cluster>/`).

## 0. Vérifier le stockage (avant config)

En mode `maintenance` (pas encore de `talosconfig`), utiliser `--insecure`. C'est l'étape pour déterminer les séries de disques à mettre dans `talconfig.yaml` :

```bash
# Disques physiques (modèle, série, taille, transport)
talosctl get disks --insecure --nodes <IP>

# Volumes détectés (partitions, systèmes de fichiers, labels)
talosctl get discoveredvolumes --insecure --nodes <IP>
```

> `--insecure` car aucun certificat TLS n'est en place avant l'application de la config.
> `volumestatus` / `uservolumeconfig` ne sont pas disponibles en mode maintenance — voir après config.

## Prérequis

Le nœud doit démarrer sur l'ISO Talos (mode `maintenance`). Vérifier qu'il est joignable :
```bash
talosctl version --insecure --nodes <IP>
```

## 1. Générer les secrets (avant la config)

`talhelper genconfig` a besoin de `talsecret.sops.yaml`. Le générer puis le chiffrer en place **avant** l'étape suivante :

```bash
cd <cluster>/
talhelper gensecret > talsecret.sops.yaml
sops -e -i talsecret.sops.yaml
```

> `gensecret` écrit les secrets en clair — toujours enchaîner avec `sops -e -i`. Un `.sops.yaml` valide (règle + clé age) doit exister à la racine du dépôt.
> Si le cluster existe déjà, réutiliser le `talsecret.sops.yaml` existant et sauter cette étape.

## 2. Générer les configurations

```bash
cd <cluster>/
talhelper genconfig
```

Produit dans `clusterconfig/` :
- `talosconfig` — configuration client
- `<node-file>` — configuration du nœud

## 3. Appliquer la configuration au nœud

```bash
talosctl apply-config --insecure \
  --nodes <IP> \
  --file ./clusterconfig/<node-file>
```

> `--insecure` est obligatoire au premier démarrage (pas encore de certificat TLS). Le nœud redémarre automatiquement après l'application.

## 4. Attendre que le nœud soit prêt

```bash
talosctl -n <IP> -e <IP> \
  --talosconfig=./clusterconfig/talosconfig \
  health --wait-timeout 10m
```

Ou via le dashboard :
```bash
talosctl -n <IP> -e <IP> dashboard \
  --talosconfig=./clusterconfig/talosconfig
```

## 5. Bootstrapper etcd (une seule fois)

À exécuter **une seule fois** sur le premier nœud control plane :
```bash
talosctl bootstrap \
  --nodes <IP> \
  --endpoints <IP> \
  --talosconfig=./clusterconfig/talosconfig
```

> Ne jamais relancer cette commande une fois le cluster démarré.

## 6. Récupérer le kubeconfig

```bash
talosctl kubeconfig \
  --nodes <IP> \
  --endpoints <IP> \
  --talosconfig=./clusterconfig/talosconfig \
  --force
```

Le kubeconfig est fusionné dans `~/.kube/config`. Vérifier l'accès :
```bash
kubectl --context=<context> get nodes
```

## Vérifier le stockage (après config)

Une fois le cluster bootstrappé, utiliser `talosconfig` :

```bash
# Disques physiques
talosctl -n <IP> -e <IP> get disks --talosconfig=./clusterconfig/talosconfig

# Volumes détectés
talosctl -n <IP> -e <IP> get discoveredvolumes --talosconfig=./clusterconfig/talosconfig

# Statut des volumes gérés (phase, taille, point de montage)
talosctl -n <IP> -e <IP> get volumestatus --talosconfig=./clusterconfig/talosconfig

# Volume utilisateur (config déclarée)
talosctl -n <IP> -e <IP> get uservolumeconfig --talosconfig=./clusterconfig/talosconfig

# Points de montage et espace utilisé
talosctl -n <IP> -e <IP> mounts --talosconfig=./clusterconfig/talosconfig
```

## Fichiers d'un cluster

| Fichier | Description |
|---------|-------------|
| `talconfig.yaml` | Configuration déclarative du cluster |
| `talsecret.sops.yaml` | Secrets chiffrés (certificats, tokens) |
| `clusterconfig/talosconfig` | Configuration client talosctl (généré) |
| `clusterconfig/<node-file>` | Config du nœud (généré) |

Pour éditer les secrets : `sops talsecret.sops.yaml`.
Autres commandes courantes : voir [COMMANDES.md](COMMANDES.md).
