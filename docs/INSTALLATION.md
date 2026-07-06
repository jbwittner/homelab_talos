# Prérequis et installation

## Outils requis
- **talhelper** — Gestionnaire de configurations Talos
- **talosctl** — CLI Talos pour la gestion des nœuds
- **sops** — Outil de chiffrement/déchiffrement
- **age** — Chiffrement symétrique (back-end de SOPS)

## Installation macOS
```bash
brew install talhelper talosctl sops age
```

## Clés de chiffrement
- **Clé privée age** — Requise pour déchiffrer les secrets (non versionée)
- **Clé publique age** — Stockée dans `.sops.yaml` pour le chiffrement

### Créer une nouvelle clé age
```bash
age-keygen -o age-key.txt
# Ajouter la clé publique dans .sops.yaml
# Sauvegarder la clé privée en lieu sûr
```
