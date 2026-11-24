# IA ANALYSE

Dépôt central de collecte des données de diagnostic pour les dépôts GitHub de **mco2vie**.

## Objectif

- Recevoir des archives de logs, rapports et dumps générés par les autres projets.
- Centraliser les fichiers dans la Release GitHub `analysis-data`.
- Garder les données de diagnostic séparées du code source.
- Fournir un workflow réutilisable par les dépôts actuels et futurs.

## Sécurité — à faire avant toute collecte

**Ce dépôt doit être privé avant de recevoir des fichiers.** Les logs et dumps peuvent contenir des chemins personnels, noms de compte, jetons ou autres informations sensibles.

Le workflow vérifie automatiquement que ce dépôt est privé et s'arrête si ce n'est pas le cas. Passe le dépôt en privé dans **Settings → General → Danger Zone → Change repository visibility**.

## Configuration initiale

Dans chaque dépôt source :

1. Ouvre **Settings → Secrets and variables → Actions**.
2. Ajoute un secret nommé `IA_ANALYSE_TOKEN`.
3. Sa valeur doit être un Personal Access Token (fine-grained) autorisé sur le dépôt `mco2vie/ia-analyse`, avec la permission **Contents: Read and write**.
4. Ajoute le workflow d'exemple ci-dessous dans `.github/workflows/send-analysis.yml`.

Le secret ne doit jamais être écrit en clair dans un fichier ou un commit.

## Utilisation depuis un autre dépôt

Exemple de workflow à adapter aux chemins réellement produits par le projet :

```yaml
name: Envoyer les données à IA ANALYSE

on:
  workflow_dispatch:
  # Décommente et adapte si l'envoi doit être automatique :
  # push:
  #   branches: [ main ]

jobs:
  collect:
    uses: mco2vie/ia-analyse/.github/workflows/collect.yml@main
    with:
      file_paths: |
        logs/**
        dumps/**
        reports/**
    secrets:
      IA_ANALYSE_TOKEN: ${{ secrets.IA_ANALYSE_TOKEN }}
```

Le workflow est lancé depuis le dépôt source. Il regroupe les fichiers correspondants dans une archive ZIP, puis la téléverse dans la Release `analysis-data` de ce dépôt central. Le nom de l'archive contient le dépôt et l'identifiant du run pour éviter d'écraser les envois précédents.

## Limites et bonnes pratiques

- Les chemins sont relatifs à la racine du dépôt source.
- N'ajoute que les dossiers nécessaires : les dumps peuvent être volumineux.
- Vérifie et expurge les secrets, mots de passe, clés API et données personnelles avant l'envoi.
- Les fichiers envoyés dans une Release ne sont pas automatiquement supprimés. Supprime les assets devenus inutiles après analyse.
- Ce système collecte les fichiers ; il ne lance pas à lui seul une analyse IA automatique.
- Ne commit jamais les dumps ou fichiers dérivés du jeu dans les dépôts de code.

## Structure

- `.github/workflows/collect.yml` : workflow réutilisable par les autres dépôts.
- `examples/send-analysis.yml` : modèle à copier dans un dépôt source.
