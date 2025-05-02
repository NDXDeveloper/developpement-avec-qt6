# 9.2 Spécificités Windows, Linux, macOS

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

## Introduction

Bien que Qt6 soit conçu pour créer des applications multiplateformes avec une base de code commune, chaque système d'exploitation possède des particularités que vous devez connaître. Cette section vous guidera à travers les spécificités de Windows, Linux et macOS pour développer des applications Qt6 qui s'intègrent parfaitement à chaque plateforme.

## Considérations générales pour le développement multiplateforme

Avant d'examiner chaque plateforme en détail, voici quelques principes généraux à garder à l'esprit :

1. **Tester sur toutes les plateformes cibles** : Ne supposez jamais qu'une application qui fonctionne sur une plateforme fonctionnera identiquement sur une autre.

2. **Éviter le code spécifique à une plateforme** : Utilisez les abstractions Qt autant que possible.

3. **Gérer les différences de chemins** : Utilisez `QDir` et `QFile` pour gérer les différences de séparateurs de chemins (`\` vs `/`).

4. **Adapter l'interface utilisateur** : Respectez les conventions d'interface de chaque plateforme.

## Spécificités Windows

### Installation et configuration

Sur Windows, vous avez plusieurs options pour compiler des applications Qt6 :

- **MSVC (Microsoft Visual C++)** : Recommandé pour les applications commerciales
- **MinGW** : Alternative open-source, basée sur GCC

#### Configuration pour MSVC

```cmake
# Dans CMakeLists.txt
if(MSVC)
    # Désactiver les avertissements de fonctions obsolètes
    add_compile_definitions(_CRT_SECURE_NO_WARNINGS)

    # Utiliser la multithreading
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP")
endif()
```

### Particularités de l'interface utilisateur

Windows utilise plusieurs conventions d'interface spécifiques :

1. **Barre de titre** : Les contrôles de fenêtre sont à droite (minimiser, maximiser, fermer)
2. **Menus** : Les menus Fichier, Édition, etc. sont standards
3. **Icônes et thèmes** : Windows utilise ses propres icônes et styles

#### Adapter le style pour Windows

```cpp
#ifdef Q_OS_WIN
    // Utiliser le style Windows natif
    QApplication::setStyle("windows");

    // Ou utiliser le style Fusion pour une apparence cohérente multiplateforme
    // QApplication::setStyle("fusion");
#endif
```

### Ressources spécifiques à Windows

#### Icône d'application

Pour ajouter une icône à votre application Windows, créez un fichier `.rc` :

```
// app.rc
IDI_ICON1 ICON DISCARDABLE "icon.ico"
```

Puis incluez-le dans votre CMakeLists.txt :

```cmake
if(WIN32)
    set(APP_ICON_RESOURCE_WINDOWS "${CMAKE_CURRENT_SOURCE_DIR}/resources/app.rc")
    target_sources(${PROJECT_NAME} PRIVATE ${APP_ICON_RESOURCE_WINDOWS})
endif()
```

#### Métadonnées de version

Créez un fichier `version.rc` pour inclure les métadonnées de version Windows :

```
1 VERSIONINFO
FILEVERSION     1,0,0,0
PRODUCTVERSION  1,0,0,0
BEGIN
    BLOCK "StringFileInfo"
    BEGIN
        BLOCK "040904E4"
        BEGIN
            VALUE "CompanyName",      "Votre Entreprise"
            VALUE "FileDescription",  "Description de l'application"
            VALUE "FileVersion",      "1.0.0"
            VALUE "ProductName",      "Nom de l'application"
            VALUE "ProductVersion",   "1.0.0"
        END
    END
    BLOCK "VarFileInfo"
    BEGIN
        VALUE "Translation", 0x409, 1252
    END
END
```

### Déploiement sur Windows

Pour déployer votre application sur Windows, vous devez inclure toutes les DLL Qt nécessaires :

1. **Méthode manuelle** : Copiez les DLL nécessaires à côté de votre exécutable
2. **windeployqt** : Utilisez l'outil fourni par Qt

```bash
windeployqt --qmldir path/to/qml/files path/to/your/app.exe
```

3. **Installateurs** : Utilisez NSIS, Inno Setup ou WiX pour créer des installateurs

## Spécificités Linux

### Installation et configuration

Linux offre plusieurs options de compilation :

- **GCC** : Le compilateur standard sur Linux
- **Clang** : Une alternative populaire

Qt6 nécessite plusieurs bibliothèques de développement sur Linux :

```bash
# Ubuntu/Debian
sudo apt-get install build-essential libgl1-mesa-dev
```

#### Configuration pour Linux

```cmake
# Dans CMakeLists.txt
if(UNIX AND NOT APPLE)
    # Activer les threads POSIX
    set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
    find_package(Threads REQUIRED)
    target_link_libraries(${PROJECT_NAME} PRIVATE Threads::Threads)

    # Ajouter les options de compilation
    target_compile_options(${PROJECT_NAME} PRIVATE -Wall -Wextra)
endif()
```

### Particularités de l'interface utilisateur

Linux présente une grande variété d'environnements de bureau (GNOME, KDE, Xfce, etc.), chacun avec ses propres conventions :

1. **Variations de style** : L'apparence peut varier considérablement selon l'environnement de bureau
2. **Thèmes système** : Les applications doivent respecter le thème choisi par l'utilisateur

```cpp
#ifdef Q_OS_LINUX
    // Utiliser le style de l'environnement de bureau
    QApplication::setStyle(QStyleFactory::create("fusion"));

    // Vérifier si une variable d'environnement indique un thème sombre
    if (qgetenv("GTK_THEME").contains("dark") || qgetenv("QT_STYLE_OVERRIDE").contains("dark")) {
        // Appliquer une palette sombre
        QPalette darkPalette;
        darkPalette.setColor(QPalette::Window, QColor(53, 53, 53));
        darkPalette.setColor(QPalette::WindowText, Qt::white);
        // ... autres couleurs ...
        QApplication::setPalette(darkPalette);
    }
#endif
```

### Ressources spécifiques à Linux

#### Icône d'application

Pour les environnements de bureau Linux, créez des fichiers `.desktop` :

```ini
[Desktop Entry]
Type=Application
Name=Mon Application Qt
Exec=/usr/bin/monapplication
Icon=/usr/share/icons/hicolor/256x256/apps/monapplication.png
Comment=Description de l'application
Categories=Utility;
```

#### Intégration avec DBus

DBus est un système de communication inter-processus largement utilisé sur Linux :

```cpp
#ifdef Q_OS_LINUX
    #include <QtDBus/QDBusInterface>
    #include <QtDBus/QDBusReply>

    // Exemple : vérifier si le système est en mode sombre via DBus
    QDBusInterface interface("org.freedesktop.portal.Desktop",
                            "/org/freedesktop/portal/desktop",
                            "org.freedesktop.portal.Settings");

    QDBusReply<QVariant> reply = interface.call("Read", "org.freedesktop.appearance", "color-scheme");
    if (reply.isValid()) {
        int colorScheme = reply.value().toInt();
        bool isDarkMode = (colorScheme == 1);
        // Adapter l'interface utilisateur en conséquence
    }
#endif
```

### Déploiement sur Linux

Pour déployer votre application sur Linux :

1. **Bibliothèques partagées** : Les DLL Qt doivent être installées ou fournies avec l'application

2. **AppImage** : Format portable permettant de distribuer des applications sans installation

```bash
# Créer un AppImage avec linuxdeployqt
linuxdeployqt path/to/your/app -appimage
```

3. **Packages** : Créez des packages .deb, .rpm selon les distributions cibles

## Spécificités macOS

### Installation et configuration

Sur macOS, vous utiliserez généralement :

- **Clang** : Le compilateur par défaut
- **Xcode** : L'IDE d'Apple pour le développement

#### Configuration pour macOS

```cmake
# Dans CMakeLists.txt
if(APPLE)
    # Configurer le bundle macOS
    set(CMAKE_MACOSX_BUNDLE ON)
    set(MACOSX_BUNDLE_BUNDLE_NAME "${PROJECT_NAME}")
    set(MACOSX_BUNDLE_GUI_IDENTIFIER "com.yourdomain.${PROJECT_NAME}")
    set(MACOSX_BUNDLE_BUNDLE_VERSION "${PROJECT_VERSION}")
    set(MACOSX_BUNDLE_SHORT_VERSION_STRING "${PROJECT_VERSION}")

    # Définir la version minimale de macOS
    set(CMAKE_OSX_DEPLOYMENT_TARGET "10.14")
endif()
```

### Particularités de l'interface utilisateur

macOS a des conventions d'interface distinctes :

1. **Barre de titre** : Les contrôles de fenêtre sont à gauche (fermer, réduire, agrandir)
2. **Barre de menu** : La barre de menu est en haut de l'écran, pas dans la fenêtre
3. **Préférences** : Accessibles via le menu de l'application, pas via un menu "Outils"

```cpp
#ifdef Q_OS_MACOS
    // Déplacer le menu "Préférences" au bon endroit pour macOS
    QMenuBar* menuBar = this->menuBar();
    QMenu* editMenu = menuBar->findChild<QMenu*>("menuEdit");

    if (editMenu) {
        // Trouver l'action "Préférences" et la déplacer
        QAction* preferencesAction = findChild<QAction*>("actionPreferences");
        if (preferencesAction) {
            // Retirer du menu précédent si nécessaire
            QMenu* oldMenu = preferencesAction->menu();
            if (oldMenu) oldMenu->removeAction(preferencesAction);

            // Ajouter au menu Édition
            editMenu->addSeparator();
            editMenu->addAction(preferencesAction);
        }
    }

    // Définir le rôle pour les actions standard
    QAction* quitAction = findChild<QAction*>("actionQuit");
    if (quitAction) quitAction->setMenuRole(QAction::QuitRole);

    QAction* aboutAction = findChild<QAction*>("actionAbout");
    if (aboutAction) aboutAction->setMenuRole(QAction::AboutRole);
#endif
```

### Ressources spécifiques à macOS

#### Icône d'application

Les icônes macOS utilisent le format `.icns` :

```cmake
if(APPLE)
    # Ajouter l'icône au bundle
    set(MACOSX_BUNDLE_ICON_FILE "AppIcon.icns")
    set(APP_ICON_MACOSX "${CMAKE_CURRENT_SOURCE_DIR}/resources/AppIcon.icns")
    set_source_files_properties(${APP_ICON_MACOSX} PROPERTIES
        MACOSX_PACKAGE_LOCATION "Resources")
    target_sources(${PROJECT_NAME} PRIVATE ${APP_ICON_MACOSX})
endif()
```

#### Fichier Info.plist

Le fichier `Info.plist` définit les métadonnées de l'application macOS :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>CFBundleDisplayName</key>
    <string>${MACOSX_BUNDLE_BUNDLE_NAME}</string>
    <key>CFBundleExecutable</key>
    <string>${MACOSX_BUNDLE_EXECUTABLE_NAME}</string>
    <key>CFBundleIdentifier</key>
    <string>${MACOSX_BUNDLE_GUI_IDENTIFIER}</string>
    <key>CFBundleInfoDictionaryVersion</key>
    <string>6.0</string>
    <key>CFBundleName</key>
    <string>${MACOSX_BUNDLE_BUNDLE_NAME}</string>
    <key>CFBundlePackageType</key>
    <string>APPL</string>
    <key>CFBundleShortVersionString</key>
    <string>${MACOSX_BUNDLE_SHORT_VERSION_STRING}</string>
    <key>CFBundleVersion</key>
    <string>${MACOSX_BUNDLE_BUNDLE_VERSION}</string>
    <key>NSHumanReadableCopyright</key>
    <string>${MACOSX_BUNDLE_COPYRIGHT}</string>
    <key>NSPrincipalClass</key>
    <string>NSApplication</string>
    <key>NSHighResolutionCapable</key>
    <true/>
</dict>
</plist>
```

Intégrez-le dans CMake :

```cmake
if(APPLE)
    set_target_properties(${PROJECT_NAME} PROPERTIES
        MACOSX_BUNDLE_INFO_PLIST "${CMAKE_CURRENT_SOURCE_DIR}/resources/Info.plist"
    )
endif()
```

### Déploiement sur macOS

Pour déployer votre application sur macOS :

1. **Bundle macOS** : Les applications macOS sont distribuées sous forme de bundles `.app`

2. **macdeployqt** : Utilisez l'outil fourni par Qt

```bash
macdeployqt path/to/your/app.app -dmg
```

Cette commande créera également un fichier DMG pour distribution.

3. **Signature de code** : Pour distribuer en dehors de l'App Store, signez votre application

```bash
codesign --deep --force --verify --verbose --sign "Developer ID Application: Your Name" path/to/your/app.app
```

## Techniques multiplateforme avancées

### Détection de fonctionnalités spécifiques

Plutôt que de vous baser uniquement sur l'identification de la plateforme, détectez les fonctionnalités disponibles :

```cpp
// Exemple : vérifier si le système supporte OpenGL
if (QOpenGLContext::supportsThreadedOpenGL()) {
    // Utiliser OpenGL avec multi-threading
} else {
    // Utiliser une alternative
}
```

### Architecture à plugins

Utilisez une architecture à plugins pour isoler le code spécifique à chaque plateforme :

```cpp
// Interface commune
class PlatformIntegration : public QObject {
public:
    virtual void setApplicationIcon() = 0;
    virtual void handleSystemEvents() = 0;
    // ...
};

// Implémentations spécifiques chargées dynamiquement
#ifdef Q_OS_WIN
    WindowsIntegration* platform = new WindowsIntegration();
#elif defined(Q_OS_MACOS)
    MacOSIntegration* platform = new MacOSIntegration();
#else
    LinuxIntegration* platform = new LinuxIntegration();
#endif
```

### Tester les différences

Utilisez des tests automatisés pour vérifier la compatibilité multiplateforme :

```cpp
// Dans un test unitaire Qt
void TestPlatformSpecifics::testFilePathHandling()
{
    QString testPath = QDir::tempPath() + QDir::separator() + "test.txt";
    QFile file(testPath);
    QVERIFY(file.open(QIODevice::WriteOnly));
    file.close();
    QVERIFY(QFile::exists(testPath));
    QFile::remove(testPath);
}
```

## Conseils pratiques

1. **Développez sur votre plateforme principale**, mais testez régulièrement sur les autres

2. **Utilisez des machines virtuelles** ou des services CI pour tester sur différentes plateformes

3. **Consultez les directives d'interface utilisateur** de chaque plateforme

4. **Utilisez le style Fusion** de Qt si vous souhaitez une apparence cohérente sur toutes les plateformes

5. **Demandez à des utilisateurs de chaque plateforme** de tester votre application

## Conclusion

Le développement multiplateforme avec Qt6 nécessite une bonne compréhension des spécificités de chaque système d'exploitation. En suivant les bonnes pratiques présentées dans cette section, vous pourrez créer des applications qui s'intègrent naturellement à Windows, Linux et macOS tout en conservant une base de code commune.

Dans la section suivante, nous explorerons le développement pour plateformes mobiles avec Qt6, notamment Android et iOS.

⏭️ [Développement mobile (Android, iOS)](/09-developpement-multiplateforme-avec-qt6/03-developpement-mobile-android-ios.md)
