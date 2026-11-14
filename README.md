# Forge-Apk

Workflows GitHub Actions réutilisables pour mes applis Android (MyRadioPlayer, MyWhiskyBeer…).

| Workflow | Rôle |
|---|---|
| `android-build.yml` | APK de debug à chaque push et PR : artefact, commentaire sur la PR, **pré-release « debug »** avec un lien fixe |
| `android-release.yml` | APK de release signé, publié dans une release GitHub (tag `v*` ou bouton) |

## Utilisation dans un projet

`.github/workflows/build.yml` :

```yaml
name: Build APK

on:
  push:
    branches: [main]
  pull_request:
  workflow_dispatch:

jobs:
  build:
    uses: florianfish/Forge-Apk/.github/workflows/android-build.yml@main
    permissions:
      contents: write        # pré-release « debug »
      pull-requests: write   # commentaire sur les PR
    with:
      apk-name: MonAppli
      # gradle-version: "9.9.1"                       # sans wrapper dans le repo
      # gradle-tasks: assembleDebug testDebugUnitTest
```

`.github/workflows/release.yml` :

```yaml
name: Release APK

on:
  push:
    tags: ["v*"]
  workflow_dispatch:
    inputs:
      type:
        description: "Type de version (1.4.0 → mineure 1.5.0, correctif 1.4.1, majeure 2.0.0)"
        type: choice
        options: [mineure, correctif, majeure]
        default: mineure

jobs:
  release:
    uses: florianfish/Forge-Apk/.github/workflows/android-release.yml@main
    permissions:
      contents: write
    with:
      apk-name: MonAppli
      type: ${{ inputs.type }}
      # gradle-version: "9.9.1"
    secrets: inherit
```

## Paramètres

### `android-build.yml`

| Entrée | Défaut | Rôle |
|---|---|---|
| `apk-name` | — (obligatoire) | Artefact `<apk-name>-apk`, fichier `<apk-name>-debug.apk` |
| `gradle-version` | vide | Version de Gradle à installer ; vide = `./gradlew` du repo |
| `gradle-tasks` | `assembleDebug` | Tâches Gradle (ajouter `testDebugUnitTest` pour les tests) |
| `java-version` | `17` | JDK Temurin |
| `apk-path` | `app/build/outputs/apk/debug/app-debug.apk` | APK produit |
| `debug-release` | `true` | Publier la pré-release « debug » sur la branche par défaut |
| `stable-debug-key` | `true` | Signer avec la clé de debug fixe (`keys/debug.keystore`) : chaque APK de debug s'installe par-dessus le précédent |

APK de debug toujours à jour : `https://github.com/<owner>/<repo>/releases/download/debug/<apk-name>-debug.apk`
(connexion GitHub requise si le repo est privé).

### `android-release.yml`

| Entrée | Défaut | Rôle |
|---|---|---|
| `apk-name` | — (obligatoire) | Fichier publié `<apk-name>-vX.Y.Z.apk` |
| `type` | `mineure` | Lancement manuel : `mineure`, `correctif` ou `majeure` |
| `gradle-version`, `java-version`, `apk-path` | | Comme ci-dessus (`apk-path` : APK de release) |

Secrets du projet, transmis par `secrets: inherit` : `KEYSTORE_BASE64`, `KEYSTORE_PASSWORD`, `KEY_ALIAS`, `KEY_PASSWORD`.

Le projet lit dans `app/build.gradle.kts` les variables d'environnement `KEYSTORE_FILE`, `KEYSTORE_PASSWORD`, `KEY_ALIAS`, `KEY_PASSWORD` (signature) et `VERSION_CODE` (numéro du run), `VERSION_NAME` (tag sans le `v`).

## Visibilité

Un workflow réutilisable d'un repo **privé** n'est utilisable que par des repos privés, et seulement si l'accès est ouvert (Settings → Actions → General → Access). Un repo **public** comme MyRadioPlayer ne peut appeler que les workflows d'un repo public : Forge-Apk doit donc être public pour servir aux deux.

## Versions

Les projets appellent `@main` : toute modification ici s'applique à leur prochain build. Pour figer une version, poser un tag (`v1`) et appeler `@v1`.
