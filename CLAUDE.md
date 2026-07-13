# Instructions du dépôt

Dépôt de configuration de clusters Kubernetes Talos Linux gérés via `talhelper`, secrets chiffrés avec SOPS/age.

## Structure

- `docs/` — documentation générique (avec placeholders) : `INSTALLATION.md`, `BOOTSTRAP.md`, `COMMANDES.md`.
- `<cluster>/` — un répertoire par cluster (actuellement `kalecgos/`), avec `talconfig.yaml`, `talsecret.sops.yaml`, `clusterconfig/` (généré, gitignored) et un `README.md`.
- `archive/` — anciens clusters (neltharion, ysera, onyxia).

## Règle : README de cluster

Le README de chaque cluster doit contenir **toutes les commandes détaillées de [`docs/BOOTSTRAP.md`](docs/BOOTSTRAP.md) et [`docs/COMMANDES.md`](docs/COMMANDES.md)**, avec les placeholders **déjà substitués** par les valeurs réelles du cluster :

| Placeholder | Source (dans `talconfig.yaml` du cluster) |
|-------------|-------------------------------------------|
| `<IP>` | `nodes[].ipAddress` |
| `<cluster>` | nom du répertoire du cluster |
| `<node-file>` | `<clusterName>-<hostname>.yaml` (fichier généré dans `clusterconfig/`) |
| `<context>` | `admin@<clusterName>` |

Structure attendue du README :
1. Tableaux **Informations**, **Stockage**, **Paramètres** (valeurs du cluster).
2. Section **Bootstrap** — copie de `BOOTSTRAP.md`, placeholders substitués.
3. Section **Commandes courantes** — copie de `COMMANDES.md`, placeholders substitués.

Quand `docs/BOOTSTRAP.md` ou `docs/COMMANDES.md` change, répercuter la modification dans le README de chaque cluster.

## Secrets

- Générer avec `talhelper gensecret > talsecret.sops.yaml` puis **toujours** `sops -e -i talsecret.sops.yaml`.
- Ne jamais commiter de secret déchiffré.
- Modifier `talconfig.yaml`, jamais les fichiers `clusterconfig/` (générés).
