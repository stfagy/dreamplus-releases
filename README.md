# Procédure de publication d’une nouvelle version de Dream+

Ce document décrit la procédure complète pour publier une nouvelle version de l’application Android TV **Dream+** et permettre sa détection automatique par les téléviseurs déjà équipés de l’application.

---

## 1. Principe général

Le système de mise à jour fonctionne ainsi :

1. Dream+ démarre sur la TV.
2. L’application interroge l’API du backend :

   ```text
   https://dreamiptv-1ba73d421171.herokuapp.com/api/app-version/
   ```

3. Le backend retourne la dernière version disponible.
4. Dream+ compare le `versionCode` distant avec son `versionCode` local.
5. Si une version plus récente existe, une popup propose la mise à jour.
6. L’APK est téléchargé depuis :

   ```text
   https://bit.ly/dream-apk
   ```

7. Android demande confirmation pour installer la mise à jour.
8. Au prochain démarrage, Dream+ supprime automatiquement l’APK temporaire téléchargé.

Le lien Bitly pointe vers :

```text
https://github.com/stfagy/dream/releases/latest/download/dreamplus.apk
```

Il est donc important que **chaque Release GitHub contienne un fichier nommé exactement `dreamplus.apk`**.

---

# 2. Préparer la nouvelle version Android

Dans :

```text
app-tv/build.gradle.kts
```

modifier les valeurs :

```kotlin
defaultConfig {
    versionCode = 3
    versionName = "1.2"
}
```

## Règle importante

Le `versionCode` doit **toujours augmenter**.

Exemple :

| Version | versionCode | versionName |
|---|---:|---|
| Première version | 1 | 1.0 |
| Mise à jour 1 | 2 | 1.1 |
| Mise à jour 2 | 3 | 1.2 |
| Mise à jour 3 | 4 | 1.3 |

Android utilise principalement :

```text
versionCode
```

pour déterminer si un APK est plus récent.

Le `versionName` est la version affichée à l’utilisateur.

---

# 3. Vérifier la signature de l’application

Toutes les versions publiées doivent être signées avec **la même clé Android**.

Une mise à jour ne pourra pas remplacer l’application installée si les signatures sont différentes.

Exemple incorrect :

```text
Version installée :
clé debug Android Studio

Nouvelle version :
clé release différente

=> INSTALL_FAILED_UPDATE_INCOMPATIBLE
```

Il faut donc toujours générer les APK publics avec la même clé de signature.

---

# 4. Générer l’APK Release

Depuis Android Studio :

```text
Build
  > Generate Signed App Bundle / APK
  > APK
```

Choisir :

```text
release
```

et utiliser la clé de signature habituelle de Dream+.

Après compilation, récupérer l’APK généré.

Il se trouve généralement dans :

```text
app-tv/build/outputs/apk/release/
```

Exemple :

```text
app-tv-release.apk
```

---

# 5. Renommer l’APK

Avant de l’envoyer sur GitHub, renommer obligatoirement le fichier :

```text
dreamplus.apk
```

Sous PowerShell :

```powershell
Rename-Item ".\app-tv-release.apk" "dreamplus.apk"
```

Le nom doit rester identique à chaque Release car le lien permanent GitHub est :

```text
https://github.com/stfagy/dream/releases/latest/download/dreamplus.apk
```

---

# 6. Tester l’APK avant publication

Il est conseillé de tester l’APK sur une TV avant de créer la Release.

Installation / mise à jour par ADB :

```powershell
adb install -r C:\chemin\vers\dreamplus.apk
```

Exemple :

```powershell
adb install -r C:\Users\pc\Downloads\dreamplus.apk
```

Une installation correcte doit se terminer par :

```text
Success
```

---

# 7. Vérifier l’espace disponible sur la TV

Dream+ vérifie l’espace avant téléchargement, mais un contrôle manuel peut être utile :

```powershell
adb shell df -h /data
```

L’APK peut nécessiter beaucoup plus que sa propre taille pendant l’installation car Android conserve temporairement :

- l’ancienne version ;
- la nouvelle version ;
- les fichiers temporaires d’installation.

---

# 8. Créer la nouvelle GitHub Release

Ouvrir le dépôt GitHub :

```text
https://github.com/stfagy/dream
```

Puis :

```text
Releases
> Draft a new release
```

## Créer un nouveau tag

Exemple :

```text
v1.2
```

Le tag doit idéalement correspondre au `versionName`.

Exemples :

```text
versionName = 1.2
tag GitHub = v1.2
```

ou :

```text
versionName = 1.2.1
tag GitHub = v1.2.1
```

---

# 9. Renseigner la Release

Exemple :

```text
Tag :
v1.2

Titre :
Dream+ 1.2
```

Dans la description, indiquer les principales évolutions.

Exemple :

```markdown
## Nouveautés

- Amélioration du lecteur Live TV
- Correction de la navigation télécommande
- Amélioration du système de mise à jour
- Corrections diverses
```

---

# 10. Ajouter l’APK à la Release

Dans :

```text
Attach binaries by dropping them here or selecting them
```

ajouter :

```text
dreamplus.apk
```

Vérifier impérativement le nom du fichier.

Il doit être exactement :

```text
dreamplus.apk
```

et non :

```text
app-tv-release.apk
DreamPlus.apk
dreamplus-1.2.apk
```

---

# 11. Publier la Release

Cliquer sur :

```text
Publish release
```

Une fois publiée, vérifier que cette URL télécharge bien l’APK :

```text
https://github.com/stfagy/dream/releases/latest/download/dreamplus.apk
```

Puis vérifier également :

```text
https://bit.ly/dream-apk
```

Le téléchargement du nouvel APK doit démarrer.

---

# 12. Mettre à jour la version côté backend

Une fois la Release GitHub publiée et testée, mettre à jour les variables Heroku.

Exemple pour Dream+ 1.2 :

```powershell
heroku config:set DREAMPLUS_LATEST_VERSION_CODE=3 DREAMPLUS_LATEST_VERSION_NAME=1.2 --app dreamiptv
```

Ajouter éventuellement un message :

```powershell
heroku config:set DREAMPLUS_UPDATE_MESSAGE="Nouvelle version de Dream+ disponible." --app dreamiptv
```

---

# 13. Mise à jour facultative ou obligatoire

## Mise à jour facultative

L’utilisateur peut choisir :

```text
Plus tard
```

Commande :

```powershell
heroku config:set DREAMPLUS_UPDATE_MANDATORY=false --app dreamiptv
```

La popup propose :

```text
[ Plus tard ]    [ Mettre à jour ]
```

---

## Mise à jour obligatoire

À utiliser uniquement si une ancienne version n’est plus compatible avec le backend.

Commande :

```powershell
heroku config:set DREAMPLUS_UPDATE_MANDATORY=true --app dreamiptv
```

La popup ne permet plus de continuer sans mettre à jour.

Après une mise à jour critique, penser à remettre :

```powershell
heroku config:set DREAMPLUS_UPDATE_MANDATORY=false --app dreamiptv
```

si les prochaines mises à jour doivent redevenir facultatives.

---

# 14. Vérifier l’API de version

Dans un navigateur, ouvrir :

```text
https://dreamiptv-1ba73d421171.herokuapp.com/api/app-version/
```

Le résultat doit ressembler à :

```json
{
  "version_code": 3,
  "version_name": "1.2",
  "download_url": "https://bit.ly/dream-apk",
  "mandatory": false,
  "message": "Nouvelle version de Dream+ disponible."
}
```

Vérifier en particulier :

```text
version_code
version_name
download_url
mandatory
```

---

# 15. Tester la détection automatique

Sur une TV possédant une version plus ancienne de Dream+ :

1. fermer complètement Dream+ ;
2. relancer l’application ;
3. attendre la vérification de version ;
4. la popup de mise à jour doit apparaître.

Exemple :

```text
Mise à jour disponible

Dream+ 1.2 est disponible
Version actuelle : 1.1

[ Plus tard ]    [ Mettre à jour ]
```

---

# 16. Tester le téléchargement

Choisir :

```text
Mettre à jour
```

Dream+ doit :

1. vérifier l’espace disponible ;
2. télécharger l’APK ;
3. afficher la progression ;
4. ouvrir l’installateur Android.

Lors de la toute première mise à jour automatique, Android peut demander :

```text
Autoriser Dream+ à installer des applications inconnues
```

Autoriser cette option.

Revenir ensuite à Dream+ pour poursuivre l’installation.

---

# 17. Installer la mise à jour

Android affiche :

```text
Voulez-vous mettre à jour cette application ?
```

Choisir :

```text
Installer
```

Puis :

```text
Ouvrir
```

Dream+ doit démarrer avec la nouvelle version.

---

# 18. Suppression automatique de l’APK

L’APK est téléchargé temporairement dans le cache privé de Dream+ :

```text
cache/app_updates/dreamplus.apk
```

Le fichier n’est pas supprimé immédiatement lorsque l’installateur Android démarre, car Android peut encore en avoir besoin.

Au prochain démarrage de Dream+ :

```text
nouvelle version démarrée
        ↓
APK temporaire détecté
        ↓
suppression automatique
```

Il n’est donc normalement pas nécessaire de supprimer manuellement l’APK.

---

# 19. Vérifier la version installée avec ADB

Pour vérifier le package Dream+ :

```powershell
adb shell dumpsys package com.stf.dreamplus.tv | findstr version
```

On doit retrouver notamment :

```text
versionCode=3
versionName=1.2
```

---

# 20. Ordre recommandé de publication

Toujours respecter cet ordre :

```text
1. Modifier versionCode/versionName
2. Compiler l’APK Release signé
3. Tester l’APK
4. Renommer en dreamplus.apk
5. Créer la GitHub Release
6. Ajouter dreamplus.apk
7. Publier la Release
8. Tester https://bit.ly/dream-apk
9. Modifier DREAMPLUS_LATEST_VERSION_CODE sur Heroku
10. Modifier DREAMPLUS_LATEST_VERSION_NAME sur Heroku
11. Vérifier /api/app-version/
12. Tester sur une TV avec une ancienne version
```

## Pourquoi cet ordre ?

Il ne faut pas annoncer au backend une nouvelle version **avant que l’APK soit réellement disponible**.

Sinon les TV pourraient détecter la mise à jour alors que GitHub sert encore l’ancienne Release ou qu’aucun nouvel APK n’est disponible.

---

# 21. Exemple complet : passage de 1.2 à 1.3

## Android

Dans :

```text
app-tv/build.gradle.kts
```

mettre :

```kotlin
versionCode = 4
versionName = "1.3"
```

Compiler et signer.

Renommer :

```text
dreamplus.apk
```

---

## GitHub

Créer :

```text
Tag : v1.3
Titre : Dream+ 1.3
```

Ajouter :

```text
dreamplus.apk
```

Publier.

Tester :

```text
https://bit.ly/dream-apk
```

---

## Backend Heroku

```powershell
heroku config:set DREAMPLUS_LATEST_VERSION_CODE=4 DREAMPLUS_LATEST_VERSION_NAME=1.3 DREAMPLUS_UPDATE_MESSAGE="Dream+ 1.3 est disponible." DREAMPLUS_UPDATE_MANDATORY=false --app dreamiptv
```

---

## Vérification

Ouvrir :

```text
https://dreamiptv-1ba73d421171.herokuapp.com/api/app-version/
```

Attendu :

```json
{
  "version_code": 4,
  "version_name": "1.3",
  "download_url": "https://bit.ly/dream-apk",
  "mandatory": false,
  "message": "Dream+ 1.3 est disponible."
}
```

Relancer ensuite Dream+ 1.2 sur une TV.

La mise à jour 1.3 doit être proposée.

---

# 22. Résumé rapide

Pour chaque nouvelle version :

```text
VERSION ANDROID
versionCode + 1
versionName = nouvelle version

        ↓

BUILD APK RELEASE SIGNÉ

        ↓

RENOMMER
dreamplus.apk

        ↓

GITHUB RELEASE
vX.Y

        ↓

PUBLIER dreamplus.apk

        ↓

TESTER
https://bit.ly/dream-apk

        ↓

HEROKU
DREAMPLUS_LATEST_VERSION_CODE
DREAMPLUS_LATEST_VERSION_NAME
DREAMPLUS_UPDATE_MESSAGE
DREAMPLUS_UPDATE_MANDATORY

        ↓

TESTER
/api/app-version/

        ↓

RELANCER DREAM+ SUR TV
```

---

# 23. Liens utiles

## APK dernière version

```text
https://bit.ly/dream-apk
```

## GitHub Release

```text
https://github.com/stfagy/dream/releases/latest
```

## APK GitHub direct

```text
https://github.com/stfagy/dream/releases/latest/download/dreamplus.apk
```

## API de version Dream+

```text
https://dreamiptv-1ba73d421171.herokuapp.com/api/app-version/
```

## Page d’aide utilisateur

```text
https://dreamiptv-1ba73d421171.herokuapp.com/aide-tv/
```
