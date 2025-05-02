# 9.5 Déploiement d'applications Qt

## Introduction

Le déploiement est l'étape finale du processus de développement, où vous préparez votre application pour la distribution aux utilisateurs finaux. Dans cette section, nous explorerons les différentes méthodes pour déployer efficacement vos applications Qt sur diverses plateformes. Bien que le déploiement puisse sembler intimidant pour les débutants, Qt fournit plusieurs outils pour simplifier ce processus.

## Principes fondamentaux du déploiement

### Comprendre les dépendances

Toute application Qt dépend de plusieurs bibliothèques :

1. **Bibliothèques Qt** : Les modules Qt que votre application utilise (Core, Gui, Widgets, etc.)
2. **Bibliothèques C++** : Les bibliothèques standard C++ et éventuellement d'autres bibliothèques tierces
3. **Ressources** : Images, fichiers de configuration, etc.

Pour un déploiement réussi, toutes ces dépendances doivent être correctement empaquetées avec votre application ou être disponibles sur le système cible.

### Approches de déploiement

Il existe deux approches principales pour le déploiement d'applications Qt :

1. **Déploiement statique** : Compiler les bibliothèques Qt directement dans votre exécutable
   - **Avantages** : Un seul fichier exécutable, pas de dépendances externes
   - **Inconvénients** : Taille d'exécutable plus grande, problèmes potentiels de licence

2. **Déploiement dynamique** : Distribuer les bibliothèques Qt avec votre application
   - **Avantages** : Exécutable plus petit, mise à jour des bibliothèques séparément
   - **Inconvénients** : Plus complexe à configurer, nécessite plus de fichiers

La méthode la plus courante est le déploiement dynamique, que nous allons explorer en détail.

## Déploiement sur Windows

### Préparation pour le déploiement sur Windows

Avant de déployer votre application, assurez-vous qu'elle fonctionne correctement en mode release :

1. Configurez votre projet en mode Release :
   ```
   cmake -DCMAKE_BUILD_TYPE=Release ..
   ```

2. Construisez l'application :
   ```
   cmake --build . --config Release
   ```

### Utilisation de windeployqt

Qt fournit un outil appelé `windeployqt` qui identifie et copie automatiquement toutes les dépendances nécessaires :

```bash
# Se placer dans le dossier contenant l'exécutable
cd path/to/build/directory/Release

# Exécuter windeployqt
C:/Qt/6.6.0/msvc2019_64/bin/windeployqt.exe MonApplication.exe
```

Cet outil va :
- Identifier les modules Qt utilisés par votre application
- Copier les DLL Qt nécessaires à côté de votre exécutable
- Ajouter les plugins requis (imageformats, platforms, etc.)
- Inclure les fichiers de traduction si nécessaire

### Création d'un installateur

Pour une distribution professionnelle, créez un installateur :

1. **NSIS (Nullsoft Scriptable Install System)** - Gratuit et open-source
2. **Inno Setup** - Outil gratuit populaire et facile à utiliser
3. **WiX Toolset** - Solution plus complexe mais très puissante

Exemple de script Inno Setup basique :

```
[Setup]
AppName=Mon Application Qt
AppVersion=1.0
DefaultDirName={pf}\MonApplication
DefaultGroupName=Mon Application

[Files]
Source: "MonApplication.exe"; DestDir: "{app}"
Source: "*.dll"; DestDir: "{app}"
Source: "platforms\*"; DestDir: "{app}\platforms"
Source: "imageformats\*"; DestDir: "{app}\imageformats"
Source: "styles\*"; DestDir: "{app}\styles"
Source: "translations\*"; DestDir: "{app}\translations"

[Icons]
Name: "{group}\Mon Application"; Filename: "{app}\MonApplication.exe"
Name: "{group}\Désinstaller"; Filename: "{uninstallexe}"
```

## Déploiement sur macOS

### Structure d'une application macOS

Sur macOS, les applications sont distribuées sous forme de bundles `.app` :

```
MonApplication.app/
  └── Contents/
       ├── Info.plist
       ├── MacOS/
       │    └── MonApplication  # Exécutable
       ├── Frameworks/         # Bibliothèques Qt et autres
       ├── PlugIns/            # Plugins Qt
       └── Resources/          # Ressources, icônes, etc.
```

### Utilisation de macdeployqt

Similaire à `windeployqt`, macOS dispose de `macdeployqt` :

```bash
# Se placer dans le dossier contenant l'application
cd path/to/build/directory

# Exécuter macdeployqt
/path/to/Qt/6.6.0/clang_64/bin/macdeployqt MonApplication.app
```

### Options supplémentaires de macdeployqt

```bash
# Créer un fichier DMG pour la distribution
macdeployqt MonApplication.app -dmg

# Signer l'application pour la distribution
macdeployqt MonApplication.app -sign-for-notarization="Developer ID Application: Votre Nom"
```

### Notarisation pour macOS

Pour distribuer votre application en dehors de l'App Store, la notarisation est recommandée :

1. Créez un Apple Developer ID
2. Signez votre application :
   ```bash
   codesign --deep --force --verify --verbose --sign "Developer ID Application: Votre Nom" MonApplication.app
   ```
3. Notarisez-la avec Apple :
   ```bash
   xcrun altool --notarize-app --primary-bundle-id "com.votrecompagnie.monapplication" --username "votre@email.com" --password "motdepasse" --file MonApplication.dmg
   ```

## Déploiement sur Linux

### Complexité du déploiement Linux

Le déploiement sur Linux est plus complexe en raison de la variété des distributions. Trois approches sont courantes :

1. **Paquets spécifiques à la distribution** (.deb, .rpm)
2. **AppImage** - Format portable indépendant de la distribution
3. **Flatpak/Snap** - Formats de conteneur modernes avec sandboxing

### Utilisation de linuxdeployqt

L'outil `linuxdeployqt` fonctionne de manière similaire à ses homologues pour Windows et macOS :

```bash
# Installer linuxdeployqt
# (disponible sur GitHub : https://github.com/probonopd/linuxdeployqt)

# Utilisation basique
linuxdeployqt MonApplication -appimage
```

### Création d'un AppImage

Un AppImage est un format exécutable autonome qui fonctionne sur la plupart des distributions Linux :

```bash
# Structure de base
mkdir -p MonApplication.AppDir/usr/{bin,lib,share}

# Copier l'exécutable
cp build/MonApplication MonApplication.AppDir/usr/bin/

# Créer un fichier .desktop
cat > MonApplication.AppDir/MonApplication.desktop << EOF
[Desktop Entry]
Type=Application
Name=Mon Application
Exec=MonApplication
Icon=MonApplication
Categories=Utility;
EOF

# Ajouter une icône
cp resources/icon.png MonApplication.AppDir/MonApplication.png

# Créer l'AppImage avec linuxdeployqt
linuxdeployqt MonApplication.AppDir/usr/bin/MonApplication -appimage
```

### Création de paquets pour des distributions spécifiques

Pour les distributions Debian/Ubuntu (.deb) :

```bash
# Installer les outils nécessaires
sudo apt-get install debhelper

# Créer une structure de répertoire
mkdir -p MonApplication_1.0-1/DEBIAN
mkdir -p MonApplication_1.0-1/usr/bin
mkdir -p MonApplication_1.0-1/usr/share/applications
mkdir -p MonApplication_1.0-1/usr/share/pixmaps

# Créer un fichier de contrôle
cat > MonApplication_1.0-1/DEBIAN/control << EOF
Package: monapplication
Version: 1.0-1
Section: utils
Priority: optional
Architecture: amd64
Depends: libqt6core6, libqt6gui6, libqt6widgets6
Maintainer: Votre Nom <votre@email.com>
Description: Description de Mon Application
EOF

# Copier l'exécutable et les ressources
cp build/MonApplication MonApplication_1.0-1/usr/bin/
cp MonApplication.desktop MonApplication_1.0-1/usr/share/applications/
cp icon.png MonApplication_1.0-1/usr/share/pixmaps/MonApplication.png

# Créer le paquet
dpkg-deb --build MonApplication_1.0-1
```

## Déploiement sur Android

### Préparation pour le déploiement Android

Le déploiement sur Android nécessite :

1. **Android SDK et NDK** correctement configurés
2. **Kit Android** configuré dans Qt Creator
3. **Fichier AndroidManifest.xml** personnalisé

### Génération d'un APK avec Qt Creator

1. Ouvrez votre projet dans Qt Creator
2. Sélectionnez le kit Android
3. Configurez le mode Release
4. Cliquez sur "Créer un package Android APK"

### Signature de l'APK

Pour distribuer sur Google Play Store, votre APK doit être signé :

```bash
# Créer une clé de signature (une seule fois)
keytool -genkey -v -keystore mon_keystore.keystore -alias alias_name -keyalg RSA -keysize 2048 -validity 10000

# Signer l'APK
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore mon_keystore.keystore MonApplication.apk alias_name
```

Dans Qt Creator, vous pouvez configurer la signature dans :
`Projets > Déploiement > Android > Options de construction Android > Créer un package`

## Déploiement sur iOS

### Préparation pour le déploiement iOS

Le déploiement sur iOS nécessite :

1. **Xcode** installé sur macOS
2. **Compte Apple Developer** (payant pour la distribution publique)
3. **Certificats et profils de provisionnement** configurés

### Compilation pour iOS

1. Ouvrez votre projet dans Qt Creator
2. Sélectionnez le kit iOS
3. Configurez le mode Release
4. Cliquez sur Build

### Déploiement via Xcode

Pour finaliser le déploiement :

1. Ouvrez le projet Xcode généré par Qt Creator
2. Configurez les certificats de signature
3. Sélectionnez `Product > Archive`
4. Utilisez `Window > Organizer` pour soumettre l'application

## Techniques avancées de déploiement

### Mise à jour automatique des applications

Implémentez un système de mise à jour automatique :

1. **Vérification de version**
   ```cpp
   void MiseAJour::verifierVersion()
   {
       QNetworkAccessManager *manager = new QNetworkAccessManager(this);
       connect(manager, &QNetworkAccessManager::finished, this, &MiseAJour::onVersionCheckFinished);

       QNetworkRequest request(QUrl("https://monsite.com/version.json"));
       manager->get(request);
   }
   ```

2. **Téléchargement et installation**
   ```cpp
   void MiseAJour::telechargerMiseAJour(const QUrl &url)
   {
       QNetworkAccessManager *manager = new QNetworkAccessManager(this);
       connect(manager, &QNetworkAccessManager::finished, this, &MiseAJour::onDownloadFinished);

       QNetworkRequest request(url);
       manager->get(request);
   }
   ```

### Télémétrie et rapports d'erreur

Collectez des informations sur les plantages pour améliorer la qualité :

```cpp
void configurerRapportsDErreur()
{
    // Configuration de base d'un système de rapport d'erreur
    QCoreApplication::setOrganizationName("VotreCompagnie");
    QCoreApplication::setOrganizationDomain("votrecompagnie.com");
    QCoreApplication::setApplicationName("MonApplication");

    // Installer un gestionnaire d'exceptions
    std::set_terminate([]() {
        // Enregistrer les informations sur le plantage
        QFile crashLog(QStandardPaths::writableLocation(QStandardPaths::AppDataLocation) + "/crash.log");
        if (crashLog.open(QIODevice::WriteOnly | QIODevice::Text)) {
            QTextStream stream(&crashLog);
            stream << "L'application a planté à " << QDateTime::currentDateTime().toString() << "\n";
            // Ajouter plus d'informations comme la stack trace
            crashLog.close();
        }

        // Envoyer le rapport (dans une vraie implémentation)
        // envoyerRapportErreur();

        std::abort();
    });
}
```

## Bonnes pratiques de déploiement

### Tests préalables au déploiement

Avant de déployer, testez minutieusement votre application :

1. **Tests sur toutes les plateformes cibles**
2. **Tests avec différentes configurations matérielles**
3. **Vérification de toutes les dépendances**
4. **Tests de performance et de mémoire**

### Liste de contrôle pour le déploiement

Utilisez cette liste pour vérifier que vous êtes prêt pour le déploiement :

- [ ] L'application a été compilée en mode Release
- [ ] Toutes les dépendances sont identifiées et incluses
- [ ] Les chemins d'accès aux ressources sont relatifs, pas absolus
- [ ] Les outils de déploiement Qt ont été exécutés
- [ ] L'application a été testée sur un système "propre"
- [ ] La documentation utilisateur est complète
- [ ] L'application est correctement signée (si applicable)

### Problèmes courants et solutions

| Problème | Solution |
|----------|----------|
| Bibliothèque manquante | Utilisez des outils comme Dependency Walker (Windows) ou ldd (Linux) pour identifier les dépendances manquantes |
| Erreurs de chemin d'accès | Utilisez QDir::currentPath() ou QCoreApplication::applicationDirPath() pour construire des chemins relatifs |
| Plugins non chargés | Vérifiez que les plugins sont dans le bon sous-dossier (ex: plugins/platforms) |
| Erreurs de signature | Assurez-vous que vos certificats sont valides et correctement configurés |
| Ressources non trouvées | Vérifiez que les ressources sont correctement empaquetées et accessibles |

## Conclusion

Le déploiement d'applications Qt sur plusieurs plateformes peut sembler complexe, mais les outils fournis par Qt simplifient considérablement ce processus. En suivant les instructions et bonnes pratiques présentées dans ce chapitre, vous serez en mesure de distribuer efficacement vos applications, qu'il s'agisse d'un déploiement sur desktop ou mobile.

N'oubliez pas que le déploiement n'est pas une réflexion après coup, mais une partie intégrante du processus de développement. Pensez aux exigences de déploiement dès les premières étapes de votre projet pour éviter les surprises de dernière minute.

Dans les chapitres suivants, nous explorerons l'intégration et les extensions de votre application Qt6, vous permettant d'étendre ses fonctionnalités au-delà des capacités de base de Qt.
