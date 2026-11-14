# Clé de debug partagée

`debug.keystore` signe les APK de **debug** compilés par `android-build.yml`, pour que chaque build s'installe par-dessus le précédent.

C'est une clé de debug, publique par nature (mêmes identifiants que la clé de debug standard d'Android : mot de passe `android`, alias `androiddebugkey`). **Ne jamais l'utiliser pour une release** : les releases sont signées avec la clé privée de chaque projet (secrets `KEYSTORE_*`).

Empreinte SHA-256 : `EE:5F:87:2C:AD:DF:D9:4E:63:2B:9E:17:AE:0C:CF:2E:07:B7:6A:89:BE:30:4B:13:00:8A:06:43:D4:EB:4A:1D`

Pour signer aussi les builds locaux avec cette clé : la copier dans `~/.android/debug.keystore`.
