# 10.3 Intégration de code natif (JNI, Objective-C)

## Introduction

L'un des grands avantages de Qt est sa portabilité multiplateforme, qui vous permet d'écrire du code une seule fois et de le déployer sur Windows, macOS, Linux, Android, iOS et d'autres plateformes. Cependant, il arrive parfois que vous ayez besoin d'accéder à des fonctionnalités spécifiques à une plateforme qui ne sont pas directement exposées par Qt. Dans ces cas, l'intégration de code natif devient nécessaire.

Cette section vous guidera à travers le processus d'intégration de code natif dans vos applications Qt6, en se concentrant sur deux cas particulièrement importants :

1. **JNI (Java Native Interface)** pour Android
2. **Objective-C** pour macOS et iOS

## Pourquoi intégrer du code natif ?

Avant de plonger dans les détails techniques, comprenons pourquoi vous pourriez avoir besoin d'intégrer du code natif :

- **Accès aux API spécifiques à la plateforme** : Certaines fonctionnalités ne sont disponibles que via les API natives de la plateforme
- **Réutilisation de code existant** : Vous pourriez avoir du code Java ou Objective-C déjà écrit que vous souhaitez réutiliser
- **Performance** : Dans certains cas, le code natif peut offrir de meilleures performances
- **Intégration avec les services du système** : Notifications, partage, authentification biométrique, etc.

## Principes généraux de l'intégration de code natif

Quelle que soit la plateforme cible, l'intégration de code natif suit généralement ces étapes :

1. **Créer une interface C++** pour les fonctionnalités natives
2. **Implémenter cette interface** pour chaque plateforme cible
3. **Utiliser la détection de plateforme** de Qt pour charger l'implémentation appropriée

Cette approche vous permet de maintenir une interface cohérente dans votre code Qt tout en gérant les spécificités de chaque plateforme en arrière-plan.

## JNI : Intégration de code Java pour Android

### Comprendre JNI

JNI (Java Native Interface) est un framework qui permet à du code Java s'exécutant dans une machine virtuelle Java (JVM) d'appeler et d'être appelé par des applications natives et des bibliothèques écrites dans d'autres langages comme C, C++ et l'assembleur.

Dans le contexte de Qt pour Android, JNI vous permet d'accéder aux API Android depuis votre code C++.

### Configuration de base pour JNI dans Qt

Pour utiliser JNI dans un projet Qt, vous devez d'abord configurer votre fichier `.pro` ou `CMakeLists.txt` :

```qmake
# Pour .pro (qmake)
android {
    QT += androidextras
    ANDROID_PACKAGE_SOURCE_DIR = $$PWD/android
}
```

Avec CMake (Qt6) :

```cmake
if(ANDROID)
    find_package(Qt6 REQUIRED COMPONENTS CoreAndroidExtras)
    target_link_libraries(MonApplication PRIVATE Qt6::CoreAndroidExtras)
    set(ANDROID_PACKAGE_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/android)
endif()
```

### Exemple : Accéder au vibreur Android

Voici un exemple simple pour accéder au vibreur d'un appareil Android :

```cpp
// Dans votre classe C++
#include <QAndroidJniObject>
#include <QtAndroid>

bool vibrer(int millisecondes) {
    if (QtAndroid::androidSdkVersion() >= 26) {
        // Pour Android 8.0 (API niveau 26) et supérieur
        QAndroidJniObject vibratorService = QtAndroid::androidService().callObjectMethod(
            "getSystemService",
            "(Ljava/lang/String;)Ljava/lang/Object;",
            QAndroidJniObject::fromString("vibrator").object()
        );

        if (vibratorService.isValid()) {
            jboolean result = vibratorService.callMethod<jboolean>(
                "hasVibrator", "()Z");

            if (result) {
                vibratorService.callMethod<void>(
                    "vibrate", "(J)V", static_cast<jlong>(millisecondes));
                return true;
            }
        }

        return false;
    } else {
        // Pour les versions antérieures d'Android
        QAndroidJniObject vibrator = QtAndroid::androidService().callObjectMethod(
            "getSystemService",
            "(Ljava/lang/String;)Ljava/lang/Object;",
            QAndroidJniObject::fromString("vibrator").object()
        );

        if (vibrator.isValid()) {
            vibrator.callMethod<void>("vibrate", "(J)V", millisecondes);
            return true;
        }

        return false;
    }
}
```

Pour utiliser cette fonction dans votre code Qt :

```cpp
// Dans un gestionnaire d'événements ou un slot
void MaClasse::onBoutonClique() {
    #ifdef Q_OS_ANDROID
        vibrer(300); // Vibrer pendant 300 ms
    #else
        qDebug() << "La fonctionnalité de vibration n'est disponible que sur Android";
    #endif
}
```

### Comprendre les signatures de méthodes JNI

Les signatures de méthodes JNI peuvent sembler compliquées au premier abord. Voici un guide rapide :

- `()V` : Méthode sans paramètres qui retourne void
- `(I)V` : Méthode qui prend un int et retourne void
- `(Ljava/lang/String;)V` : Méthode qui prend une String et retourne void
- `()Z` : Méthode sans paramètres qui retourne un boolean
- `(J)V` : Méthode qui prend un long et retourne void

Les types de base sont représentés par des lettres uniques :
- `Z` - boolean
- `B` - byte
- `C` - char
- `S` - short
- `I` - int
- `J` - long
- `F` - float
- `D` - double
- `L` - classe (suivi du nom de classe complet et terminé par `;`)
- `[` - tableau

### Créer une classe d'assistance pour Android

Pour simplifier l'utilisation de JNI, vous pouvez créer une classe d'assistance :

```cpp
// AndroidUtils.h
#ifndef ANDROIDUTILS_H
#define ANDROIDUTILS_H

#include <QString>

class AndroidUtils {
public:
    static bool vibrer(int millisecondes);
    static QString getDeviceId();
    static void partagerTexte(const QString &titre, const QString &texte);
    // Ajoutez d'autres méthodes selon vos besoins
};

#endif // ANDROIDUTILS_H
```

```cpp
// AndroidUtils.cpp
#include "AndroidUtils.h"

#ifdef Q_OS_ANDROID
#include <QAndroidJniObject>
#include <QtAndroid>
#endif

bool AndroidUtils::vibrer(int millisecondes) {
#ifdef Q_OS_ANDROID
    // Code JNI comme dans l'exemple précédent
    return true;
#else
    Q_UNUSED(millisecondes)
    return false;
#endif
}

QString AndroidUtils::getDeviceId() {
#ifdef Q_OS_ANDROID
    QAndroidJniObject activity = QtAndroid::androidActivity();
    QAndroidJniObject context = activity.callObjectMethod(
        "getApplicationContext", "()Landroid/content/Context;");

    QAndroidJniObject contentResolver = context.callObjectMethod(
        "getContentResolver", "()Landroid/content/ContentResolver;");

    QAndroidJniObject secure = QAndroidJniObject::getStaticObjectField(
        "android/provider/Settings$Secure",
        "ANDROID_ID",
        "Ljava/lang/String;");

    QAndroidJniObject androidId = QAndroidJniObject::callStaticObjectMethod(
        "android/provider/Settings$Secure",
        "getString",
        "(Landroid/content/ContentResolver;Ljava/lang/String;)Ljava/lang/String;",
        contentResolver.object(),
        secure.object());

    if (androidId.isValid()) {
        return androidId.toString();
    }
#endif
    return QString();
}

void AndroidUtils::partagerTexte(const QString &titre, const QString &texte) {
#ifdef Q_OS_ANDROID
    QAndroidJniObject jTitre = QAndroidJniObject::fromString(titre);
    QAndroidJniObject jTexte = QAndroidJniObject::fromString(texte);

    QAndroidJniObject intent = QAndroidJniObject::callStaticObjectMethod(
        "android/content/Intent",
        "makeMainSelectorActivity",
        "(Ljava/lang/String;Ljava/lang/String;)Landroid/content/Intent;",
        QAndroidJniObject::fromString("android.intent.action.MAIN").object(),
        QAndroidJniObject::fromString("android.intent.category.APP_MESSAGING").object()
    );

    intent.callObjectMethod(
        "putExtra",
        "(Ljava/lang/String;Ljava/lang/String;)Landroid/content/Intent;",
        QAndroidJniObject::fromString("android.intent.extra.TEXT").object(),
        jTexte.object()
    );

    intent.callObjectMethod(
        "putExtra",
        "(Ljava/lang/String;Ljava/lang/String;)Landroid/content/Intent;",
        QAndroidJniObject::fromString("android.intent.extra.SUBJECT").object(),
        jTitre.object()
    );

    QtAndroid::startActivity(intent, 0);
#else
    Q_UNUSED(titre)
    Q_UNUSED(texte)
#endif
}
```

## Objective-C : Intégration pour macOS et iOS

### Configuration de base pour Objective-C dans Qt

Pour intégrer du code Objective-C dans un projet Qt, vous devez configurer votre fichier de projet :

```qmake
# Pour .pro (qmake)
macx | ios {
    LIBS += -framework Foundation -framework UIKit  # Ajoutez d'autres frameworks si nécessaire
    OBJECTIVE_SOURCES += ios_utils.mm
    HEADERS += ios_utils.h
}
```

Avec CMake (Qt6) :

```cmake
if(APPLE)
    find_library(FOUNDATION_LIBRARY Foundation)
    find_library(UIKIT_LIBRARY UIKit)
    target_link_libraries(MonApplication PRIVATE ${FOUNDATION_LIBRARY} ${UIKIT_LIBRARY})

    set_source_files_properties(ios_utils.mm PROPERTIES COMPILE_FLAGS "-x objective-c++")
    target_sources(MonApplication PRIVATE ios_utils.mm ios_utils.h)
endif()
```

### Fichiers d'en-tête et d'implémentation

Vous devez créer un fichier d'en-tête C++ standard et un fichier d'implémentation `.mm` (Objective-C++).

```cpp
// ios_utils.h
#ifndef IOS_UTILS_H
#define IOS_UTILS_H

#include <QString>

class IOSUtils {
public:
    static void partagerTexte(const QString &titre, const QString &texte);
    static QString getUniqueDeviceId();
    static bool canUseCamera();
};

#endif // IOS_UTILS_H
```

```objc
// ios_utils.mm
#include "ios_utils.h"

#ifdef Q_OS_IOS
#import <Foundation/Foundation.h>
#import <UIKit/UIKit.h>
#endif

void IOSUtils::partagerTexte(const QString &titre, const QString &texte) {
#ifdef Q_OS_IOS
    NSString *nsTitle = titre.toNSString();
    NSString *nsText = texte.toNSString();

    UIActivityViewController *activityViewController =
        [[UIActivityViewController alloc] initWithActivityItems:@[nsText]
                                          applicationActivities:nil];

    // Obtenir la fenêtre principale et le contrôleur principal
    UIWindow *window = [UIApplication sharedApplication].keyWindow;
    UIViewController *rootViewController = window.rootViewController;

    // Présenter le contrôleur de partage
    [rootViewController presentViewController:activityViewController
                                     animated:YES
                                   completion:nil];
#else
    Q_UNUSED(titre)
    Q_UNUSED(texte)
#endif
}

QString IOSUtils::getUniqueDeviceId() {
#ifdef Q_OS_IOS
    NSString *uuid = [[[UIDevice currentDevice] identifierForVendor] UUIDString];
    return QString::fromNSString(uuid);
#else
    return QString();
#endif
}

bool IOSUtils::canUseCamera() {
#ifdef Q_OS_IOS
    AVAuthorizationStatus status = [AVCaptureDevice authorizationStatusForMediaType:AVMediaTypeVideo];
    return status == AVAuthorizationStatusAuthorized;
#else
    return false;
#endif
}
```

### Convertir entre QString et NSString

Pour convertir entre QString et NSString, utilisez les méthodes suivantes :

```cpp
// QString vers NSString
NSString *nsString = QString("Texte").toNSString();

// NSString vers QString
QString qString = QString::fromNSString(nsString);
```

### Appels de méthodes Objective-C depuis C++

Dans Objective-C++, vous pouvez mélanger du code C++ et Objective-C. Cela vous permet d'appeler des méthodes Objective-C depuis votre code C++ :

```objc
// Exemple : Afficher une alerte sur iOS
void afficherAlerte(const QString &titre, const QString &message) {
#ifdef Q_OS_IOS
    UIAlertController *alert = [UIAlertController
                                alertControllerWithTitle:titre.toNSString()
                                message:message.toNSString()
                                preferredStyle:UIAlertControllerStyleAlert];

    UIAlertAction *okAction = [UIAlertAction
                             actionWithTitle:@"OK"
                             style:UIAlertActionStyleDefault
                             handler:nil];

    [alert addAction:okAction];

    UIWindow *window = [UIApplication sharedApplication].keyWindow;
    UIViewController *rootViewController = window.rootViewController;

    [rootViewController presentViewController:alert animated:YES completion:nil];
#else
    Q_UNUSED(titre)
    Q_UNUSED(message)
#endif
}
```

## Créer une interface commune pour plusieurs plateformes

Pour maintenir votre code Qt propre et portable, il est recommandé de créer une interface commune qui abstraira les détails spécifiques à la plateforme.

```cpp
// PlatformUtils.h
#ifndef PLATFORMUTILS_H
#define PLATFORMUTILS_H

#include <QString>

class PlatformUtils {
public:
    // Méthodes communes pour toutes les plateformes
    static QString getDeviceId();
    static void partagerTexte(const QString &titre, const QString &texte);
    static bool peutUtiliserCamera();

    // Méthodes spécifiques aux plateformes
    #ifdef Q_OS_ANDROID
    static bool vibrer(int millisecondes);
    #endif

    #ifdef Q_OS_IOS
    static void ouvriAppStore(const QString &appId);
    #endif
};

#endif // PLATFORMUTILS_H
```

```cpp
// PlatformUtils.cpp
#include "PlatformUtils.h"

#ifdef Q_OS_ANDROID
#include "AndroidUtils.h"
#endif

#if defined(Q_OS_IOS) || defined(Q_OS_MACOS)
#include "IOSUtils.h"
#endif

QString PlatformUtils::getDeviceId() {
#ifdef Q_OS_ANDROID
    return AndroidUtils::getDeviceId();
#elif defined(Q_OS_IOS)
    return IOSUtils::getUniqueDeviceId();
#else
    // Implémentation par défaut pour d'autres plateformes
    return "non-mobile-device";
#endif
}

void PlatformUtils::partagerTexte(const QString &titre, const QString &texte) {
#ifdef Q_OS_ANDROID
    AndroidUtils::partagerTexte(titre, texte);
#elif defined(Q_OS_IOS)
    IOSUtils::partagerTexte(titre, texte);
#else
    // Implémentation alternative pour desktop (peut-être utiliser QClipboard)
    qDebug() << "Partage non disponible sur cette plateforme";
    Q_UNUSED(titre)
    Q_UNUSED(texte)
#endif
}

bool PlatformUtils::peutUtiliserCamera() {
#ifdef Q_OS_ANDROID
    // Implémenter la vérification pour Android
    return true; // Simplifié pour l'exemple
#elif defined(Q_OS_IOS)
    return IOSUtils::canUseCamera();
#else
    return false;
#endif
}

#ifdef Q_OS_ANDROID
bool PlatformUtils::vibrer(int millisecondes) {
    return AndroidUtils::vibrer(millisecondes);
}
#endif

#ifdef Q_OS_IOS
void PlatformUtils::ouvriAppStore(const QString &appId) {
    // Implémentation pour iOS
    // Utiliser UIApplication pour ouvrir l'URL de l'App Store
    Q_UNUSED(appId)
}
#endif
```

## Bonnes pratiques pour l'intégration de code natif

### 1. Isolez le code spécifique à la plateforme

Gardez tout le code spécifique à la plateforme dans des fichiers séparés et utilisez des directives de préprocesseur pour compiler seulement ce qui est nécessaire pour chaque plateforme.

### 2. Créez des abstractions propres

Définissez des interfaces C++ claires qui cachent les détails d'implémentation spécifiques à la plateforme. Cela facilite l'utilisation de ces fonctionnalités dans votre code Qt.

### 3. Gérez les cas où la fonctionnalité n'est pas disponible

Assurez-vous que votre code gère correctement les cas où une fonctionnalité spécifique à une plateforme n'est pas disponible, soit en offrant une alternative, soit en informant l'utilisateur.

### 4. Testez sur toutes les plateformes cibles

N'oubliez pas de tester votre application sur toutes les plateformes cibles pour vous assurer que l'intégration native fonctionne comme prévu.

### 5. Documentez les dépendances externes

Documentez clairement toutes les exigences et dépendances spécifiques à la plateforme pour faciliter la maintenance future.

## Cas d'utilisation courants

### Android (JNI)

- Accès aux capteurs spécifiques
- Intégration avec les services Google
- Notifications push
- In-App Billing

### iOS/macOS (Objective-C)

- Intégration avec iCloud
- Utilisation des API Apple spécifiques
- In-App Purchase
- Apple Push Notifications

## Conclusion

L'intégration de code natif dans vos applications Qt6 vous permet d'accéder à des fonctionnalités spécifiques à la plateforme tout en maintenant la portabilité de votre application. En utilisant JNI pour Android et Objective-C pour iOS/macOS, vous pouvez combiner le meilleur des deux mondes : la puissance et l'expressivité de Qt pour la majeure partie de votre application, et l'accès direct aux API natives lorsque nécessaire.

En suivant les bonnes pratiques présentées dans cette section, vous pouvez créer des applications Qt multiplateforme robustes qui offrent une expérience utilisateur optimale sur chaque plateforme, tout en minimisant la quantité de code spécifique à chaque plateforme que vous devez maintenir.
