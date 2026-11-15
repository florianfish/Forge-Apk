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
    secrets: inherit           # clé de debug fixe (DEBUG_KEYSTORE_BASE64)
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
| `stable-debug-key` | `true` | Signer avec la clé de debug fixe (secret `DEBUG_KEYSTORE_BASE64`) : chaque APK de debug s'installe par-dessus le précédent |

Secrets du projet, transmis par `secrets: inherit` :

| Secret | Défaut | Rôle |
|---|---|---|
| `DEBUG_KEYSTORE_BASE64` | — (obligatoire si `stable-debug-key`) | Keystore de debug encodé : `base64 -w0 debug.keystore` |
| `DEBUG_KEYSTORE_PASSWORD` | `android` | Mot de passe du keystore |
| `DEBUG_KEY_ALIAS` | `androiddebugkey` | Alias de la clé |
| `DEBUG_KEY_PASSWORD` | celui du keystore | Mot de passe de la clé |

La clé est imposée à `signingConfigs.debug` par un script d'initialisation Gradle (`gradle -I …`) : la copier dans `~/.android/` ne suffit pas sur les runners. L'empreinte de l'APK produit est ensuite comparée à celle de la clé, et le build échoue si elle ne correspond pas. Sur une PR sans accès aux secrets (Dependabot, fork), l'APK est signé avec une clé jetable, avec un avertissement. **Ne jamais utiliser cette clé pour une release.**

Pour signer aussi les builds locaux avec la même clé : la copier dans `~/.android/debug.keystore`.

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

## Numéros de version

| Build | `VERSION_NAME` | `VERSION_CODE` | Affiché (avec le suffixe debug du projet) |
|---|---|---|---|
| Release `v1.4.0` | `1.4.0` | n° du run de release | `1.4.0` |
| Debug (push / PR) | dernière release, ex. `1.4.0` (absent s'il n'y en a pas : défaut du projet) | n° du run de build | `1.4.0-debug.57` |

Un APK de debug se lit donc « compilé après la 1.4.0, build n° 57 ». Le `versionCode` qui augmente à chaque run permet aux APK de debug de s'installer les uns par-dessus les autres.

Côté projet :

```kotlin
versionCode = System.getenv("VERSION_CODE")?.toIntOrNull() ?: 1
versionName = System.getenv("VERSION_NAME") ?: "0.1.0"
buildTypes {
    debug { versionNameSuffix = "-debug" + (System.getenv("GITHUB_RUN_NUMBER")?.let { ".$it" } ?: "") }
}
```

## Visibilité

Un workflow réutilisable d'un repo **privé** n'est utilisable que par des repos privés, et seulement si l'accès est ouvert (Settings → Actions → General → Access). Un repo **public** comme MyRadioPlayer ne peut appeler que les workflows d'un repo public : Forge-Apk doit donc être public pour servir aux deux.

## Versions

Les projets appellent `@main` : toute modification ici s'applique à leur prochain build. Pour figer une version, poser un tag (`v1`) et appeler `@v1`.

## Renovate

`default.json` est le preset [Renovate](https://docs.renovatebot.com/) partagé par mes projets. Chaque dépôt n'a qu'un `renovate.json` :

```json
{ "extends": ["local>florianfish/Forge-Apk"] }
```

- Passage le lundi avant 6 h (heure de Paris), 5 PR ouvertes au maximum.
- Mineures et correctifs regroupés dans une PR, **fusionnée automatiquement si la CI passe** ; les majeures restent à valider à la main.
- Kotlin (plugins android, compose, serialization) et Node.js (`.nvmrc` + image Docker) ont chacun leur PR.
- npm : une version n'est proposée qu'après 3 jours.
- La version de Gradle passée en `gradle-version:` aux workflows (projets sans wrapper) est suivie aussi.

Forge-Apk n'a pas de CI sur ses PR : ses mises à jour d'actions ne sont jamais fusionnées automatiquement, à relire puisqu'elles touchent tous les projets.

Installation : l'appli GitHub [Renovate](https://github.com/apps/renovate) (Mend), limitée aux dépôts concernés. Fusionner d'abord la PR de Forge-Apk (le preset doit exister sur `main`), puis celles des projets.
