# Clé de debug partagée

`debug.keystore` signe les APK de **debug** compilés par `android-build.yml`, pour que chaque build s'installe par-dessus le précédent.

C'est une clé de debug, publique par nature (mêmes identifiants que la clé de debug standard d'Android : mot de passe `android`, alias `androiddebugkey`). **Ne jamais l'utiliser pour une release** : les releases sont signées avec la clé privée de chaque projet (secrets `KEYSTORE_*`).

Empreinte SHA-256 : `EE:5F:87:2C:AD:DF:D9:4E:63:2B:9E:17:AE:0C:CF:2E:07:B7:6A:89:BE:30:4B:13:00:8A:06:43:D4:EB:4A:1D`

Le workflow l'impose avec le script d'initialisation `debug-signing.init.gradle` (`gradle -I …`), puis vérifie l'empreinte de l'APK produit : le build échoue si elle ne correspond pas. Copier la clé dans `~/.android/` ne suffisait pas sur les runners GitHub.

Pour signer aussi les builds locaux avec cette clé : `FORGE_DEBUG_KEYSTORE=/chemin/debug.keystore ./gradlew -I keys/debug-signing.init.gradle assembleDebug`, ou la copier dans `~/.android/debug.keystore`.
