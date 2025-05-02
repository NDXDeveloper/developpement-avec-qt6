# 2.4 Organisation modulaire du code

Une application Qt bien conçue est généralement organisée en modules indépendants qui interagissent entre eux. Cette approche modulaire facilite le développement, la maintenance et l'évolution de votre application. Dans cette section, nous allons explorer les différentes façons d'organiser votre code Qt6 de manière modulaire.

## Pourquoi adopter une organisation modulaire ?

Lorsque votre application grandit, il devient essentiel d'organiser votre code de manière claire et structurée. Voici les principaux avantages d'une approche modulaire :

- **Lisibilité améliorée** : Le code est plus facile à comprendre quand il est organisé en modules ayant chacun une responsabilité claire
- **Maintenance simplifiée** : Les modifications peuvent être apportées à un module sans affecter les autres
- **Réutilisabilité** : Les modules bien conçus peuvent être réutilisés dans d'autres projets
- **Travail en équipe facilité** : Plusieurs développeurs peuvent travailler sur différents modules simultanément
- **Tests unitaires simplifiés** : Les modules indépendants sont plus faciles à tester

## Structure de base d'un projet Qt

Avant de parler de modularité, examinons la structure typique d'un projet Qt :

```
MonApplication/
├── CMakeLists.txt            # Fichier de configuration CMake principal
├── src/                      # Dossier contenant le code source
│   ├── main.cpp              # Point d'entrée de l'application
│   ├── mainwindow.h          # Déclaration de la classe MainWindow
│   ├── mainwindow.cpp        # Implémentation de la classe MainWindow
│   └── CMakeLists.txt        # Fichier CMake pour le dossier src
├── include/                  # En-têtes publics (si nécessaire)
├── resources/                # Ressources (images, fichiers de traduction, etc.)
│   └── resources.qrc         # Fichier de ressources Qt
├── forms/                    # Fichiers UI créés avec Qt Designer
│   └── mainwindow.ui         # Interface utilisateur de la fenêtre principale
└── tests/                    # Tests unitaires et d'intégration
    └── CMakeLists.txt        # Fichier CMake pour les tests
```

Cette structure de base est un bon point de départ, mais pour des applications plus complexes, nous devrons aller plus loin dans l'organisation modulaire.

## Approches de modularisation dans Qt6

### 1. Séparation par couches fonctionnelles

Une approche classique consiste à diviser votre application en couches fonctionnelles :

```
MonApplication/
├── src/
│   ├── ui/                   # Couche interface utilisateur
│   │   ├── mainwindow.h/cpp
│   │   ├── dialogs/          # Boîtes de dialogue
│   │   └── widgets/          # Widgets personnalisés
│   ├── models/               # Couche modèles de données
│   │   ├── usermodel.h/cpp
│   │   └── itemmodel.h/cpp
│   ├── services/             # Couche services (logique métier)
│   │   ├── userservice.h/cpp
│   │   └── dataservice.h/cpp
│   └── data/                 # Couche accès aux données
│       ├── database.h/cpp
│       └── restapi.h/cpp
```

Cette organisation permet de séparer clairement les responsabilités entre les couches :
- La couche UI s'occupe uniquement de l'affichage
- Les modèles représentent les données
- Les services contiennent la logique métier
- La couche data gère l'accès aux sources de données externes

### 2. Organisation par fonctionnalités

Une autre approche consiste à organiser votre code par fonctionnalités ou domaines métier :

```
MonApplication/
├── src/
│   ├── core/                 # Fonctionnalités de base
│   │   ├── application.h/cpp
│   │   └── settings.h/cpp
│   ├── users/                # Gestion des utilisateurs
│   │   ├── userwidget.h/cpp
│   │   ├── usermodel.h/cpp
│   │   └── userservice.h/cpp
│   ├── products/             # Gestion des produits
│   │   ├── productwidget.h/cpp
│   │   ├── productmodel.h/cpp
│   │   └── productservice.h/cpp
│   └── reporting/            # Fonctionnalité de rapports
│       ├── reportwidget.h/cpp
│       ├── reportmodel.h/cpp
│       └── reportgenerator.h/cpp
```

Cette organisation regroupe tous les éléments (UI, modèles, services) liés à une même fonctionnalité, ce qui facilite le développement de fonctionnalités complètes.

## Création de bibliothèques Qt

Pour des projets plus importants, vous pouvez diviser votre application en plusieurs bibliothèques :

### Bibliothèques statiques

Une bibliothèque statique est liée à votre application lors de la compilation. Voici comment en créer une avec CMake :

```cmake
# Dans le CMakeLists.txt de votre bibliothèque
set(LIB_SOURCES
    mylib.cpp
    utility.cpp
)

add_library(mylib STATIC ${LIB_SOURCES})
target_link_libraries(mylib PRIVATE Qt6::Core Qt6::Widgets)
```

Pour utiliser cette bibliothèque dans votre application :

```cmake
# Dans le CMakeLists.txt de votre application
add_executable(myapp main.cpp)
target_link_libraries(myapp PRIVATE mylib)
```

### Bibliothèques dynamiques (partagées)

Les bibliothèques dynamiques sont chargées lors de l'exécution, ce qui permet de les mettre à jour sans recompiler l'application :

```cmake
# Dans le CMakeLists.txt de votre bibliothèque
set(LIB_SOURCES
    mylib.cpp
    utility.cpp
)

add_library(mylib SHARED ${LIB_SOURCES})
target_link_libraries(mylib PRIVATE Qt6::Core Qt6::Widgets)
```

Pour exporter correctement les symboles dans une bibliothèque partagée sous Windows, vous devrez définir des macros d'exportation :

```cpp
// mylib_global.h
#pragma once

#include <QtCore/qglobal.h>

#if defined(MYLIB_LIBRARY)
#  define MYLIB_EXPORT Q_DECL_EXPORT
#else
#  define MYLIB_EXPORT Q_DECL_IMPORT
#endif

// myclass.h
#include "mylib_global.h"

class MYLIB_EXPORT MyClass {
public:
    MyClass();
    void doSomething();
};
```

## Système de plugins Qt

Qt offre un puissant système de plugins qui permet d'étendre votre application de manière modulaire :

### Création d'une interface de plugin

```cpp
// interface.h
#pragma once
#include <QObject>
#include <QtPlugin>

class PluginInterface {
public:
    virtual ~PluginInterface() {}
    virtual QString name() const = 0;
    virtual void initialize() = 0;
};

#define PluginInterface_iid "com.mycompany.MyApp.PluginInterface"
Q_DECLARE_INTERFACE(PluginInterface, PluginInterface_iid)
```

### Implémentation d'un plugin

```cpp
// myplugin.h
#pragma once
#include "interface.h"

class MyPlugin : public QObject, public PluginInterface {
    Q_OBJECT
    Q_PLUGIN_METADATA(IID PluginInterface_iid)
    Q_INTERFACES(PluginInterface)

public:
    QString name() const override;
    void initialize() override;
};

// myplugin.cpp
#include "myplugin.h"

QString MyPlugin::name() const {
    return "Mon Plugin";
}

void MyPlugin::initialize() {
    qDebug() << "Le plugin est initialisé !";
}
```

### Chargement des plugins dans l'application

```cpp
#include <QPluginLoader>
#include "interface.h"

void loadPlugins() {
    QDir pluginsDir(QCoreApplication::applicationDirPath() + "/plugins");

    foreach (QString fileName, pluginsDir.entryList(QDir::Files)) {
        QPluginLoader loader(pluginsDir.absoluteFilePath(fileName));
        QObject *plugin = loader.instance();

        if (plugin) {
            PluginInterface *interface = qobject_cast<PluginInterface *>(plugin);
            if (interface) {
                qDebug() << "Plugin chargé :" << interface->name();
                interface->initialize();
            }
        } else {
            qDebug() << "Erreur de chargement du plugin :" << loader.errorString();
        }
    }
}
```

## Espaces de noms (namespaces)

Les espaces de noms sont un moyen efficace d'organiser votre code et d'éviter les conflits de noms :

```cpp
// Dans userservice.h
namespace MonApp {
namespace Services {

class UserService {
public:
    UserService();
    void createUser(const QString &username);
};

} // namespace Services
} // namespace MonApp

// Utilisation
#include "userservice.h"

void maFonction() {
    MonApp::Services::UserService service;
    service.createUser("alice");
}

// Ou avec using
using namespace MonApp::Services;

void maFonction() {
    UserService service;
    service.createUser("alice");
}
```

## Bonnes pratiques d'organisation modulaire

### 1. Principe de responsabilité unique

Chaque classe ou module ne devrait avoir qu'une seule raison de changer. Par exemple, séparez la logique métier de l'interface utilisateur.

### 2. Dépendances explicites

Rendez les dépendances entre vos modules explicites, généralement via les constructeurs :

```cpp
// UserService dépend explicitement de UserRepository
class UserService {
public:
    // La dépendance est explicite dans le constructeur
    UserService(UserRepository *repository) : m_repository(repository) {}

private:
    UserRepository *m_repository;
};

// Usage
UserRepository *repository = new UserRepository();
UserService *service = new UserService(repository);
```

### 3. Interfaces vs. implémentations

Utilisez des interfaces pour découpler les modules :

```cpp
// Interface
class DataStorageInterface {
public:
    virtual ~DataStorageInterface() {}
    virtual bool saveData(const QString &key, const QVariant &value) = 0;
    virtual QVariant loadData(const QString &key) = 0;
};

// Implémentations
class FileStorage : public DataStorageInterface {
public:
    bool saveData(const QString &key, const QVariant &value) override;
    QVariant loadData(const QString &key) override;
};

class DatabaseStorage : public DataStorageInterface {
public:
    bool saveData(const QString &key, const QVariant &value) override;
    QVariant loadData(const QString &key) override;
};

// Classe qui utilise l'interface sans connaître l'implémentation
class ConfigManager {
public:
    ConfigManager(DataStorageInterface *storage) : m_storage(storage) {}

    void setSetting(const QString &key, const QVariant &value) {
        m_storage->saveData(key, value);
    }

private:
    DataStorageInterface *m_storage;
};
```

### 4. Signaux et slots pour la communication entre modules

Les signaux et slots sont parfaits pour découpler les modules :

```cpp
// Module 1
class UserManager : public QObject {
    Q_OBJECT
public:
    void createUser(const QString &username) {
        // Logique de création d'utilisateur
        emit userCreated(username);
    }

signals:
    void userCreated(const QString &username);
};

// Module 2
class NotificationManager : public QObject {
    Q_OBJECT
public:
    NotificationManager() {
        // Aucune connaissance directe de UserManager
    }

public slots:
    void onUserCreated(const QString &username) {
        qDebug() << "Notification : Nouvel utilisateur créé :" << username;
    }
};

// Connexion des modules
UserManager *userManager = new UserManager();
NotificationManager *notifManager = new NotificationManager();

// Communication entre modules via signaux et slots
connect(userManager, &UserManager::userCreated,
        notifManager, &NotificationManager::onUserCreated);
```

## Exemple concret d'organisation modulaire

Voici un exemple d'organisation pour une application de gestion de contacts :

```
ContactManager/
├── src/
│   ├── main.cpp
│   ├── core/
│   │   ├── application.h/cpp         # Classe Application principale
│   │   └── settings.h/cpp            # Gestion des paramètres
│   ├── models/
│   │   ├── contactmodel.h/cpp        # Modèle de données pour les contacts
│   │   └── groupmodel.h/cpp          # Modèle de données pour les groupes
│   ├── services/
│   │   ├── contactservice.h/cpp      # Logique métier pour les contacts
│   │   └── importexportservice.h/cpp # Import/export de contacts
│   ├── data/
│   │   ├── database.h/cpp            # Accès à la base de données
│   │   └── jsonparser.h/cpp          # Analyse de fichiers JSON
│   ├── ui/
│   │   ├── mainwindow.h/cpp          # Fenêtre principale
│   │   ├── contactdialog.h/cpp       # Dialogue d'édition de contact
│   │   └── widgets/
│   │       ├── contactlist.h/cpp     # Widget de liste de contacts
│   │       └── contactdetails.h/cpp  # Widget de détails de contact
│   └── utils/
│       ├── logger.h/cpp              # Système de journalisation
│       └── stringutils.h/cpp         # Utilitaires de chaînes
```

## Conclusion

L'organisation modulaire du code est essentielle pour le développement d'applications Qt6 de qualité, surtout lorsque leur taille augmente. En divisant votre code en modules bien définis avec des responsabilités claires, vous faciliterez la maintenance, améliorerez la réutilisabilité et permettrez à plusieurs développeurs de travailler efficacement sur le même projet.

Les techniques que nous avons vues (séparation par couches, organisation par fonctionnalités, bibliothèques, plugins, espaces de noms) peuvent être combinées selon les besoins de votre projet. L'important est de garder une structure cohérente et de maintenir un couplage faible entre les modules.

Dans les prochaines sections, nous explorerons plus en détail le développement d'interfaces utilisateur avec Qt Widgets et Qt Quick/QML, en gardant à l'esprit ces principes d'organisation modulaire.
