# Cluster neltharion

Cluster Kubernetes Talos Linux mono-nœud, bare metal hébergé chez OVH.

## Informations

| Champ | Valeur |
|-------|--------|
| Nom | neltharion |
| Endpoint | https://5.135.136.115:6443 |
| Talos | v1.13.4 |
| Kubernetes | v1.36.2 |
| Nœud | ns3058844 — control plane |
| IP | 5.135.136.115 |
| Interface | eno1 — gateway 5.135.136.254 |

## Stockage

| Disque | Série | Modèle | Taille | Rôle |
|--------|-------|--------|--------|------|
| nvme0n1 | CVPF71620076450RGN | INTEL SSDPE2MX450G7 | 450 GB | Système (installDisk) |
| nvme1n1 | CVPF6325009K450RGN | INTEL SSDPE2MX450G7 | 450 GB | Data (UserVolumeConfig `data`, XFS) |

## Paramètres

Valeurs à substituer dans la [procédure de bootstrap](../docs/BOOTSTRAP.md) et les [commandes courantes](../docs/COMMANDES.md) :

| Placeholder | Valeur |
|-------------|--------|
| `<IP>` | 5.135.136.115 |
| `<cluster>` | neltharion |
| `<node-file>` | neltharion-ns3058844.yaml |
| `<context>` | admin@neltharion |
