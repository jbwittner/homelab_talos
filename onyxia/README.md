# Cluster onyxia

Cluster Kubernetes Talos Linux mono-nœud, bare metal auto-hébergé (à la maison).

## Informations

| Champ | Valeur |
|-------|--------|
| Nom | onyxia |
| Endpoint | https://192.168.1.113:6443 |
| Talos | v1.13.4 |
| Kubernetes | v1.36.2 |
| Nœud | onyxia-host — control plane |
| IP | 192.168.1.113 |

## Stockage

| Disque | Série | Modèle | Taille | Transport | Rôle |
|--------|-------|--------|--------|-----------|------|
| nvme0n1 | 2102357DWVN0RA105082 | EXT X200E SSD | 1.0 TB | nvme | Système (installDisk) |
| sda | — | UDisk | 16 GB | usb | Média USB (boot) |

## Paramètres

Valeurs à substituer dans la [procédure de bootstrap](../docs/BOOTSTRAP.md) et les [commandes courantes](../docs/COMMANDES.md) :

| Placeholder | Valeur |
|-------------|--------|
| `<IP>` | 192.168.1.113 |
| `<cluster>` | onyxia |
| `<node-file>` | onyxia-onyxia-host.yaml |
| `<context>` | admin@onyxia |
