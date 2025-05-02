# 9.3 Développement mobile (Android, iOS)

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

## Introduction

Qt6 offre des capacités puissantes pour développer des applications mobiles sur Android et iOS à partir d'une base de code commune. Cette section vous guidera à travers les étapes essentielles pour créer, configurer et déployer des applications Qt6 sur ces plateformes mobiles.

## Prérequis pour le développement mobile

Avant de commencer le développement mobile avec Qt6, vous devez installer plusieurs outils supplémentaires :

### Pour Android :
- **Android Studio** ou au minimum les outils Android SDK et NDK
- **Java Development Kit (JDK)** version 11 ou supérieure
- **Gradle** (généralement inclus avec Android Studio)

### Pour iOS :
- **Xcode** (disponible uniquement sur macOS)
- **iOS SDK**
- Un compte développeur Apple (pour déployer sur des appareils physiques)

## Configuration de Qt6 pour le développement mobile

### Configuration de l'environnement Android

1. **Installer Android Studio** depuis le site officiel (recommandé pour les débutants)

2. **Configurer Qt Creator pour Android** :
   - Ouvrez Qt Creator
   - Allez dans `Édition > Préférences > Appareils > Android`
   - Configurez les chemins vers :
     - Java JDK
     - Android SDK
     - Android NDK

```
# Exemple de chemins sur Windows
JDK : C:\Program Files\Java\jdk-11
Android SDK : C:\Users\username\AppData\Local\Android\Sdk
Android NDK : C:\Users\username\AppData\Local\Android\Sdk\ndk\23.1.7779620
```

3. **Configurer un appareil Android** :
   - Soit un émulateur Android (créé via Android Studio)
   - Soit un appareil physique en mode développeur avec le débogage USB activé

### Configuration de l'environnement iOS

1. **Installer Xcode** depuis l'App Store (uniquement sur macOS)

2. **Configurer Qt Creator pour iOS** :
   - Ouvrez Qt Creator
   - Allez dans `Édition > Préférences > Kits`
   - Assurez-vous qu'un kit iOS est correctement configuré
   - Vérifiez que Xcode est détecté

3. **Configurer un appareil iOS** :
   - Soit le simulateur iOS
   - Soit un appareil physique enregistré dans votre compte développeur

## Création d'un projet mobile avec Qt6

### Approche recommandée : Qt Quick/QML

Pour les applications mobiles, Qt Quick (avec QML) est généralement préféré aux widgets traditionnels car :
- Il est optimisé pour les écrans tactiles
- Il offre des animations fluides
- Il utilise un style déclaratif plus adapté aux interfaces mobiles

### Créer un nouveau projet Qt Quick

1. Ouvrez Qt Creator
2. Sélectionnez `Fichier > Nouveau fichier ou projet`
3. Choisissez `Application > Application Qt Quick`
4. Suivez l'assistant, sélectionnez les plateformes cibles (Android, iOS)
5. Choisissez CMake comme système de build

Voici à quoi peut ressembler un fichier QML simple pour mobile :

```qml
// Main.qml
import QtQuick 2.15
import QtQuick.Controls 2.15
import QtQuick.Layouts 1.15

ApplicationWindow {
    visible: true
    width: 360
    height: 640
    title: "Mon Application Mobile"

    ColumnLayout {
        anchors.fill: parent
        anchors.margins: 20
        spacing: 15

        Label {
            Layout.alignment: Qt.AlignHCenter
            text: "Bonjour Mobile !"
            font.pixelSize: 24
        }

        Button {
            Layout.alignment: Qt.AlignHCenter
            text: "Appuyez ici"
            onClicked: messageLabel.visible = true
        }

        Label {
            id: messageLabel
            Layout.alignment: Qt.AlignHCenter
            text: "Vous avez appuyé sur le bouton !"
            visible: false
            font.pixelSize: 18
            color: "green"
        }
    }
}
```

## Considérations spécifiques à Android

### Structure du projet Android

Un projet Qt pour Android inclut plusieurs éléments spécifiques :

- **AndroidManifest.xml** : Fichier de configuration principal
- **build.gradle** : Script de construction utilisé par Gradle
- **res/** : Dossier contenant les ressources (icônes, chaînes, etc.)

### Configurer le fichier AndroidManifest.xml

Ce fichier est crucial car il définit les permissions et métadonnées de votre application :

```xml
<?xml version="1.0"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="org.qtproject.example"
          android:versionName="1.0"
          android:versionCode="1"
          android:installLocation="auto">

    <!-- Permissions nécessaires -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

    <application android:hardwareAccelerated="true"
                 android:name="org.qtproject.qt.android.bindings.QtApplication"
                 android:label="Mon App Qt"
                 android:icon="@drawable/icon">

        <activity android:configChanges="orientation|uiMode|screenLayout|screenSize|smallestScreenSize|layoutDirection|locale|fontScale|keyboard|keyboardHidden|navigation|mcc|mnc|density"
                  android:name="org.qtproject.qt.android.bindings.QtActivity"
                  android:label="Mon App Qt"
                  android:screenOrientation="unspecified"
                  android:launchMode="singleTop">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>

            <meta-data android:name="android.app.lib_name" android:value="-- %%INSERT_APP_LIB_NAME%% --" />
            <meta-data android:name="android.app.arguments" android:value="-- %%INSERT_APP_ARGUMENTS%% --" />
            <meta-data android:name="android.app.extract_android_style" android:value="minimal" />
        </activity>
    </application>
</manifest>
```

### Personnaliser les icônes Android

Les icônes Android doivent être fournies en plusieurs résolutions :

```
android/res/drawable-ldpi/icon.png    (36x36)
android/res/drawable-mdpi/icon.png    (48x48)
android/res/drawable-hdpi/icon.png    (72x72)
android/res/drawable-xhdpi/icon.png   (96x96)
android/res/drawable-xxhdpi/icon.png  (144x144)
android/res/drawable-xxxhdpi/icon.png (192x192)
```

Vous pouvez les définir dans votre projet CMake :

```cmake
if(ANDROID)
    set(ANDROID_PACKAGE_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/android")
    set(ANDROID_EXTRA_LIBS
        ${CMAKE_CURRENT_SOURCE_DIR}/path/to/additional/libs/library.so
    )
endif()
```

### API natives Android

Pour accéder aux fonctionnalités spécifiques d'Android, utilisez le module `QtAndroidExtras` :

```cpp
#include <QGuiApplication>
#include <QtAndroid>

// Exemple : demander une permission à l'exécution
bool requestPermission()
{
    QtAndroid::PermissionResult result = QtAndroid::checkPermission("android.permission.CAMERA");
    if (result == QtAndroid::PermissionResult::Denied) {
        QtAndroid::requestPermissionsSync(QStringList() << "android.permission.CAMERA");
        result = QtAndroid::checkPermission("android.permission.CAMERA");
        if (result == QtAndroid::PermissionResult::Denied)
            return false;
    }
    return true;
}
```

### Compilation et déploiement sur Android

Pour compiler et déployer votre application Qt sur Android :

1. **Sélectionnez le kit Android** dans Qt Creator

2. **Configurez votre appareil** :
   - Sélectionnez un émulateur ou un appareil physique connecté

3. **Lancez la compilation et le déploiement** :
   - Cliquez sur le bouton "Exécuter" ou utilisez Ctrl+R
   - Qt Creator compilera l'application, créera un APK et l'installera

## Considérations spécifiques à iOS

### Structure du projet iOS

Un projet Qt pour iOS comprend :

- **Info.plist** : Fichier de configuration principal
- **LaunchScreen.storyboard** : Écran de lancement
- **Images.xcassets** : Ressources graphiques (icônes, images)

### Configurer le fichier Info.plist

Ce fichier définit les métadonnées de votre application iOS :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>CFBundleDisplayName</key>
    <string>Mon App Qt</string>
    <key>CFBundleExecutable</key>
    <string>${EXECUTABLE_NAME}</string>
    <key>CFBundleIdentifier</key>
    <string>com.yourcompany.appname</string>
    <key>CFBundleName</key>
    <string>${PRODUCT_NAME}</string>
    <key>CFBundlePackageType</key>
    <string>APPL</string>
    <key>CFBundleShortVersionString</key>
    <string>1.0</string>
    <key>CFBundleVersion</key>
    <string>1.0</string>
    <key>LSRequiresIPhoneOS</key>
    <true/>
    <key>UILaunchStoryboardName</key>
    <string>LaunchScreen</string>
    <key>UISupportedInterfaceOrientations</key>
    <array>
        <string>UIInterfaceOrientationPortrait</string>
        <string>UIInterfaceOrientationLandscapeLeft</string>
        <string>UIInterfaceOrientationLandscapeRight</string>
    </array>
</dict>
</plist>
```

Dans votre projet CMake :

```cmake
if(IOS)
    set_target_properties(${PROJECT_NAME} PROPERTIES
        MACOSX_BUNDLE_INFO_PLIST "${CMAKE_CURRENT_SOURCE_DIR}/ios/Info.plist"
    )
endif()
```

### Personnaliser les icônes iOS

iOS nécessite différentes tailles d'icônes pour différents appareils :

```
Icon-60@2x.png (120x120)
Icon-60@3x.png (180x180)
Icon-76.png (76x76)
Icon-76@2x.png (152x152)
Icon-83.5@2x.png (167x167)
Icon-1024.png (1024x1024)
```

### API natives iOS

Pour accéder aux fonctionnalités spécifiques d'iOS, vous pouvez utiliser l'Objective-C ou Swift via le mécanisme de pont de Qt :

```cpp
#include <QGuiApplication>
#include <QDebug>

#ifdef Q_OS_IOS
#import <UIKit/UIKit.h>
#endif

void requestCameraPermission()
{
#ifdef Q_OS_IOS
    // Utilisation des API iOS natives
    AVAuthorizationStatus status = [AVCaptureDevice authorizationStatusForMediaType:AVMediaTypeVideo];
    switch (status) {
    case AVAuthorizationStatusAuthorized:
        qDebug() << "Caméra déjà autorisée";
        break;
    case AVAuthorizationStatusDenied:
        qDebug() << "L'utilisateur a refusé l'accès à la caméra";
        break;
    case AVAuthorizationStatusRestricted:
        qDebug() << "Accès à la caméra restreint";
        break;
    case AVAuthorizationStatusNotDetermined:
        [AVCaptureDevice requestAccessForMediaType:AVMediaTypeVideo completionHandler:^(BOOL granted) {
            if (granted) {
                qDebug() << "Accès à la caméra autorisé";
            } else {
                qDebug() << "Accès à la caméra refusé";
            }
        }];
        break;
    }
#endif
}
```

### Compilation et déploiement sur iOS

Pour compiler et déployer votre application Qt sur iOS :

1. **Sélectionnez le kit iOS** dans Qt Creator

2. **Configurez votre appareil** :
   - Sélectionnez un simulateur iOS ou un appareil physique enregistré

3. **Lancez la compilation et le déploiement** :
   - Cliquez sur le bouton "Exécuter" ou utilisez Ctrl+R
   - Qt Creator compilera l'application et la déploiera sur le simulateur ou l'appareil

## Conception d'interfaces adaptatives

### Densité d'écran et tailles d'écran

Les appareils mobiles ont des tailles et densités d'écran variées. Utilisez ces techniques pour créer des interfaces adaptatives :

#### Unités indépendantes de la densité

Dans QML, utilisez les unités `dp` (density-independent pixels) :

```qml
Rectangle {
    width: 50 * Screen.devicePixelRatio  // S'adapte à la densité de l'écran
    height: 50 * Screen.devicePixelRatio
    color: "blue"
}
```

#### Adaptez l'interface selon l'orientation

```qml
Item {
    width: parent.width
    height: parent.height

    // Détection de l'orientation
    property bool isPortrait: height > width

    ColumnLayout {
        anchors.fill: parent
        visible: isPortrait
        // Interface en mode portrait
    }

    RowLayout {
        anchors.fill: parent
        visible: !isPortrait
        // Interface en mode paysage
    }
}
```

### Bonnes pratiques pour les interfaces tactiles

1. **Taille des éléments interactifs** : Au moins 9-10mm (environ 48dp) pour être facilement touchés

2. **Feedback visuel** : Ajoutez des animations subtiles pour confirmer les interactions

3. **Gestes tactiles** : Utilisez le module Qt Quick pour gérer les gestes comme le balayage et le pincement

```qml
MouseArea {
    anchors.fill: parent

    onClicked: {
        // Action au toucher simple
    }

    PinchArea {
        anchors.fill: parent
        onPinchUpdated: {
            // Gérer le pincement pour zoomer
            var scaleFactor = pinch.scale - pinch.previousScale
            image.scale *= 1 + scaleFactor
        }
    }
}
```

## Optimisations pour les appareils mobiles

### Performances et gestion de la mémoire

Les appareils mobiles ont des ressources limitées par rapport aux ordinateurs de bureau :

1. **Chargement paresseux (lazy loading)** : Chargez les composants uniquement quand ils sont nécessaires

```qml
Loader {
    id: heavyComponentLoader
    active: false  // Ne sera pas chargé immédiatement
    source: "HeavyComponent.qml"
}

Button {
    text: "Charger le composant"
    onClicked: heavyComponentLoader.active = true
}
```

2. **Limiter les animations** : Réduisez la complexité des animations sur les appareils moins puissants

3. **Tester sur des appareils d'entrée de gamme** pour s'assurer de bonnes performances partout

### Gestion de l'énergie

Économisez la batterie en suivant ces principes :

1. **Réduisez les opérations en arrière-plan** lorsque l'application n'est pas au premier plan

2. **Optimisez les opérations réseau** : Regroupez les requêtes et utilisez la mise en cache

3. **Suspension des timers** quand ils ne sont pas nécessaires

```cpp
// En C++
void AppController::applicationStateChanged(Qt::ApplicationState state)
{
    if (state == Qt::ApplicationActive) {
        // Application au premier plan
        startUpdateTimer();
    } else {
        // Application en arrière-plan
        stopUpdateTimer();
    }
}

// Connexion du signal
connect(qApp, &QGuiApplication::applicationStateChanged,
        this, &AppController::applicationStateChanged);
```

### Gestion hors-ligne

Les applications mobiles peuvent perdre la connectivité :

1. **Détection de la connectivité** :

```cpp
#include <QNetworkInformation>

// Vérifier l'état actuel de la connectivité
bool isOnline = QNetworkInformation::instance()->reachability() == QNetworkInformation::Reachability::Online;

// Réagir aux changements de connectivité
connect(QNetworkInformation::instance(), &QNetworkInformation::reachabilityChanged,
        this, [](QNetworkInformation::Reachability reachability) {
    if (reachability == QNetworkInformation::Reachability::Online) {
        // L'appareil est de nouveau en ligne
        syncWithServer();
    } else {
        // L'appareil est hors ligne
        enableOfflineMode();
    }
});
```

2. **Stockage local** : Utilisez une base de données SQLite locale pour stocker les données

```cpp
#include <QSqlDatabase>
#include <QSqlQuery>

QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE");
db.setDatabaseName("local_storage.db");
if (db.open()) {
    QSqlQuery query;
    query.exec("CREATE TABLE IF NOT EXISTS offline_data "
               "(id INTEGER PRIMARY KEY, data TEXT, synced INTEGER)");

    // Stocker des données
    query.prepare("INSERT INTO offline_data (data, synced) VALUES (?, 0)");
    query.addBindValue(jsonData);
    query.exec();
}
```

## Publication sur les stores

### Google Play Store (Android)

Pour publier sur le Google Play Store :

1. **Créer un APK signé ou un App Bundle** :
   - Dans Qt Creator, allez dans `Projets > Build > Android build > Create Signed APK`
   - Ou utilisez la ligne de commande avec Gradle

2. **Créer un compte développeur** sur Google Play Console

3. **Soumettre votre application** :
   - Remplir les informations du store (description, captures d'écran)
   - Téléverser l'APK
   - Configurer les tests et le déploiement

### Apple App Store (iOS)

Pour publier sur l'App Store :

1. **Générer une archive** :
   - Ouvrez le projet Xcode généré
   - Sélectionnez `Product > Archive`

2. **Créer un compte développeur Apple** et configurer App Store Connect

3. **Soumettre votre application** via l'organizer d'Xcode :
   - Remplir les informations du store (description, captures d'écran)
   - Téléverser l'archive
   - Soumettre pour examen

## Conseils pour les débutants

1. **Commencez petit** : Créez d'abord une application simple qui fonctionne sur les deux plateformes

2. **Utilisez principalement QML/Qt Quick** pour les interfaces mobiles, pas les widgets

3. **Testez régulièrement** sur des appareils réels, pas seulement sur les émulateurs

4. **Apprenez les cycles de vie des applications** sur chaque plateforme mobile

5. **Commencez avec des permissions minimales** et ajoutez-en uniquement quand nécessaire

## Ressources d'apprentissage

- Documentation officielle Qt pour Android et iOS
- Exemples fournis avec Qt (dossier `examples`)
- Tutoriels en ligne sur le développement mobile Qt

## Conclusion

Le développement mobile avec Qt6 vous permet de créer des applications pour Android et iOS à partir d'une base de code commune, tout en offrant la possibilité d'accéder aux fonctionnalités spécifiques à chaque plateforme. Bien que la configuration initiale puisse sembler complexe, Qt Creator simplifie grandement le processus de développement et de déploiement.

Dans la section suivante, nous explorerons la gestion des ressources multi-plateformes, une compétence essentielle pour créer des applications efficaces sur toutes les plateformes.

⏭️ [Gestion des ressources multi-plateformes](/09-developpement-multiplateforme-avec-qt6/04-gestion-des-ressources-multi-plateformes.md)
