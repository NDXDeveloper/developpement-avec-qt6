# 10.2 Création de plugins Qt

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

## Introduction

Les plugins représentent une approche puissante pour rendre vos applications Qt extensibles et modulaires. Au lieu d'intégrer toutes les fonctionnalités dans une seule application monolithique, les plugins vous permettent de charger dynamiquement des fonctionnalités supplémentaires à l'exécution. Cette approche offre de nombreux avantages, notamment une meilleure organisation du code, des mises à jour plus faciles et la possibilité pour les utilisateurs ou les développeurs tiers d'étendre votre application.

Dans cette section, nous allons explorer comment créer et utiliser des plugins dans Qt6, en partant des concepts de base jusqu'à un exemple complet.

## Comprendre les plugins Qt

### Qu'est-ce qu'un plugin ?

Un plugin Qt est essentiellement une bibliothèque partagée (DLL sous Windows, .so sous Linux, .dylib sous macOS) qui peut être chargée dynamiquement par une application Qt. Ce qui distingue un plugin Qt d'une bibliothèque partagée ordinaire, c'est qu'il implémente une interface spécifique que l'application hôte peut utiliser pour interagir avec lui.

### Pourquoi utiliser des plugins ?

Les plugins offrent plusieurs avantages :

1. **Modularité** : Divisez votre application en composants indépendants
2. **Extensibilité** : Permettez à votre application d'être étendue sans modification du code principal
3. **Déploiement flexible** : Mettez à jour ou ajoutez des fonctionnalités sans recompiler l'application entière
4. **Personnalisation** : Permettez aux utilisateurs d'activer uniquement les fonctionnalités dont ils ont besoin
5. **Développement parallèle** : Plusieurs équipes peuvent travailler sur différents plugins simultanément

## Architecture des plugins Qt

### Composants clés d'un système de plugins

Un système de plugins Qt comprend généralement trois composants principaux :

1. **Interface du plugin** : Une interface C++ (généralement basée sur QObject) que tous les plugins doivent implémenter
2. **Chargeur de plugins** : Code dans l'application hôte qui découvre et charge les plugins
3. **Plugins** : Les implémentations concrètes qui fournissent les fonctionnalités

### Système de méta-objets Qt

Le système de plugins Qt repose fortement sur le système de méta-objets de Qt. Les macros comme `Q_PLUGIN_METADATA` et `Q_INTERFACES` sont utilisées pour déclarer les métadonnées du plugin, permettant à Qt de reconnaître et de charger correctement les plugins.

## Création d'un plugin simple

Voyons maintenant comment créer un plugin simple étape par étape.

### Étape 1 : Définir l'interface du plugin

Commençons par définir l'interface que tous nos plugins devront implémenter :

```cpp
// fichier: moninterfaceplugin.h
#ifndef MONINTERFACEPLUGIN_H
#define MONINTERFACEPLUGIN_H

#include <QtPlugin>
#include <QString>

class MonInterfacePlugin {
public:
    virtual ~MonInterfacePlugin() {}

    // Méthodes que tous les plugins doivent implémenter
    virtual QString nom() const = 0;
    virtual QString description() const = 0;
    virtual void executer() = 0;
};

// Cette macro définit l'interface que les plugins doivent implémenter
#define MonInterfacePlugin_iid "com.monentreprise.MonInterfacePlugin"
Q_DECLARE_INTERFACE(MonInterfacePlugin, MonInterfacePlugin_iid)

#endif // MONINTERFACEPLUGIN_H
```

Cette interface définit trois méthodes simples que tous nos plugins devront implémenter : `nom()`, `description()` et `executer()`.

### Étape 2 : Créer un plugin concret

Maintenant, créons un plugin qui implémente cette interface :

```cpp
// fichier: monplugin.h
#ifndef MONPLUGIN_H
#define MONPLUGIN_H

#include <QObject>
#include "moninterfaceplugin.h"

class MonPlugin : public QObject, public MonInterfacePlugin {
    Q_OBJECT
    Q_PLUGIN_METADATA(IID MonInterfacePlugin_iid)
    Q_INTERFACES(MonInterfacePlugin)

public:
    MonPlugin(QObject *parent = nullptr);
    ~MonPlugin();

    // Implémentation des méthodes de l'interface
    QString nom() const override;
    QString description() const override;
    void executer() override;
};

#endif // MONPLUGIN_H
```

```cpp
// fichier: monplugin.cpp
#include "monplugin.h"
#include <QMessageBox>

MonPlugin::MonPlugin(QObject *parent) : QObject(parent) {
}

MonPlugin::~MonPlugin() {
}

QString MonPlugin::nom() const {
    return "Mon Premier Plugin";
}

QString MonPlugin::description() const {
    return "Un plugin d'exemple pour démontrer la création de plugins Qt";
}

void MonPlugin::executer() {
    QMessageBox::information(nullptr, "Mon Plugin", "Exécution du plugin réussie!");
}
```

Remarquez les macros importantes dans ce code :
- `Q_OBJECT` : Nécessaire pour tous les objets Qt qui utilisent des signaux et des slots
- `Q_PLUGIN_METADATA` : Fournit des métadonnées sur le plugin
- `Q_INTERFACES` : Indique quelles interfaces le plugin implémente

### Étape 3 : Configurer le fichier projet du plugin

Pour compiler notre plugin en tant que bibliothèque partagée, nous avons besoin d'un fichier `.pro` (ou CMakeLists.txt) approprié :

```
# fichier: monplugin.pro
QT += widgets

TEMPLATE = lib
CONFIG += plugin
TARGET = monplugin

HEADERS += \
    moninterfaceplugin.h \
    monplugin.h

SOURCES += \
    monplugin.cpp

# Destination du plugin
DESTDIR = ../plugins
```

Si vous utilisez CMake, voici l'équivalent :

```cmake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.16)
project(MonPlugin LANGUAGES CXX)

find_package(Qt6 REQUIRED COMPONENTS Core Widgets)

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)

add_library(monplugin SHARED
    monplugin.cpp
    monplugin.h
    moninterfaceplugin.h
)

target_link_libraries(monplugin PRIVATE
    Qt6::Core
    Qt6::Widgets
)

# Destination du plugin
set_target_properties(monplugin PROPERTIES
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/plugins"
)
```

### Étape 4 : Créer l'application hôte

Maintenant, créons une application qui peut charger et utiliser notre plugin :

```cpp
// fichier: main.cpp
#include <QApplication>
#include <QMainWindow>
#include <QPushButton>
#include <QVBoxLayout>
#include <QDir>
#include <QPluginLoader>
#include <QMessageBox>
#include <QFileDialog>
#include "moninterfaceplugin.h"

class MainWindow : public QMainWindow {
    Q_OBJECT
public:
    MainWindow() {
        QWidget *centralWidget = new QWidget(this);
        setCentralWidget(centralWidget);

        QVBoxLayout *layout = new QVBoxLayout(centralWidget);

        QPushButton *loadButton = new QPushButton("Charger un plugin", this);
        layout->addWidget(loadButton);

        connect(loadButton, &QPushButton::clicked, this, &MainWindow::chargerPlugin);

        setWindowTitle("Démo de Plugins Qt");
        resize(400, 300);
    }

private slots:
    void chargerPlugin() {
        // Demander à l'utilisateur de sélectionner un fichier de plugin
        QString fileName = QFileDialog::getOpenFileName(this,
            "Charger un plugin", "plugins",
            "Plugins (*.dll *.so *.dylib)");

        if (fileName.isEmpty())
            return;

        // Charger le plugin
        QPluginLoader loader(fileName);
        QObject *plugin = loader.instance();

        if (plugin) {
            // Vérifier si le plugin implémente notre interface
            MonInterfacePlugin *monPlugin = qobject_cast<MonInterfacePlugin*>(plugin);
            if (monPlugin) {
                QMessageBox::information(this, "Plugin chargé",
                    "Nom: " + monPlugin->nom() + "\n" +
                    "Description: " + monPlugin->description());

                // Exécuter le plugin
                monPlugin->executer();
            } else {
                QMessageBox::warning(this, "Erreur",
                    "Le fichier n'est pas un plugin compatible");
                loader.unload();
            }
        } else {
            QMessageBox::critical(this, "Erreur",
                "Impossible de charger le plugin: " + loader.errorString());
        }
    }
};

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);
    MainWindow window;
    window.show();
    return app.exec();
}
```

Et son fichier de projet :

```
# fichier: application.pro
QT += widgets

TEMPLATE = app
TARGET = application

HEADERS += \
    moninterfaceplugin.h

SOURCES += \
    main.cpp
```

Ou avec CMake :

```cmake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.16)
project(ApplicationPlugins LANGUAGES CXX)

find_package(Qt6 REQUIRED COMPONENTS Core Widgets)

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)

add_executable(application
    main.cpp
    moninterfaceplugin.h
)

target_link_libraries(application PRIVATE
    Qt6::Core
    Qt6::Widgets
)
```

## Fonctionnalités avancées des plugins

### Découverte automatique de plugins

Dans l'exemple précédent, nous avons demandé à l'utilisateur de sélectionner manuellement un plugin. Dans une application réelle, vous voudrez probablement découvrir automatiquement tous les plugins disponibles :

```cpp
void chargerTousLesPlugins() {
    QDir pluginsDir(QApplication::applicationDirPath());

    // Sur macOS, les plugins peuvent être dans un répertoire différent
    #if defined(Q_OS_MAC)
    if (pluginsDir.dirName() == "MacOS") {
        pluginsDir.cdUp();
        pluginsDir.cdUp();
        pluginsDir.cdUp();
    }
    #endif

    pluginsDir.cd("plugins");

    foreach (QString fileName, pluginsDir.entryList(QDir::Files)) {
        QPluginLoader loader(pluginsDir.absoluteFilePath(fileName));
        QObject *plugin = loader.instance();

        if (plugin) {
            MonInterfacePlugin *monPlugin = qobject_cast<MonInterfacePlugin*>(plugin);
            if (monPlugin) {
                // Stocker le plugin pour une utilisation ultérieure
                m_plugins.append(monPlugin);

                qDebug() << "Plugin chargé:" << monPlugin->nom();
            } else {
                loader.unload();
            }
        }
    }
}
```

### Métadonnées JSON

Qt6 permet d'intégrer des métadonnées JSON directement dans votre plugin, ce qui est très utile pour fournir des informations supplémentaires sans avoir à charger le plugin complet :

```cpp
class MonPluginAvance : public QObject, public MonInterfacePlugin {
    Q_OBJECT
    Q_PLUGIN_METADATA(IID MonInterfacePlugin_iid FILE "metadata.json")
    Q_INTERFACES(MonInterfacePlugin)

    // ...
};
```

Avec un fichier `metadata.json` :

```json
{
    "Name" : "Plugin Avancé",
    "Version" : "1.0.0",
    "CompatibilityVersion" : "1.0",
    "Vendor" : "Mon Entreprise",
    "Copyright" : "(C) 2025 Mon Entreprise",
    "License" : "GPL v3",
    "Category" : "Utilitaires",
    "Description" : "Un plugin avec des métadonnées JSON"
}
```

Vous pouvez accéder à ces métadonnées sans charger complètement le plugin :

```cpp
QPluginLoader loader(fileName);
QJsonObject metadata = loader.metaData().value("MetaData").toObject();

QString name = metadata.value("Name").toString();
QString version = metadata.value("Version").toString();
```

### Types de plugins spécifiques à Qt

Qt fournit plusieurs interfaces de plugins préétablies pour des cas d'utilisation courants :

- **QDesignerCustomWidgetInterface** : Pour créer des widgets personnalisés pour Qt Designer
- **QImageIOPlugin** : Pour ajouter le support de formats d'image
- **QSqlDriverPlugin** : Pour ajouter des pilotes de base de données
- **QStylePlugin** : Pour ajouter des styles d'interface utilisateur

## Bonnes pratiques pour les plugins

### Organisation du code

1. **Séparez clairement les interfaces des implémentations** : Gardez les interfaces dans des fichiers d'en-tête séparés qui sont partagés entre l'application hôte et les plugins.

2. **Versionnez vos interfaces** : Si vous prévoyez de publier des mises à jour, incluez des informations de version dans vos interfaces pour assurer la compatibilité.

### Gestion des dépendances

1. **Minimisez les dépendances externes** : Chaque plugin devrait être aussi autonome que possible.

2. **Attention aux conflits de symboles** : Différents plugins peuvent utiliser les mêmes bibliothèques mais potentiellement des versions différentes, ce qui peut causer des conflits.

### Déploiement

1. **Structure de répertoires** : Organisez vos plugins dans un répertoire dédié, généralement nommé "plugins".

2. **Chemin de recherche des plugins** : Assurez-vous que votre application définit correctement le chemin de recherche des plugins :

```cpp
QApplication app(argc, argv);
app.setLibraryPaths(QStringList() << app.applicationDirPath() + "/plugins");
```

3. **Déploiement multiplateforme** : Tenez compte des différences entre les systèmes d'exploitation :
   - Windows : .dll
   - Linux : .so
   - macOS : .dylib

## Conclusion

La création de plugins est une technique puissante pour rendre vos applications Qt extensibles et modulaires. En séparant clairement les interfaces des implémentations et en utilisant les outils fournis par Qt, vous pouvez créer un système de plugins robuste qui permet à votre application d'évoluer avec le temps.

Les plugins sont particulièrement utiles pour :
- Les applications nécessitant une extensibilité par des tiers
- Les projets de grande envergure avec plusieurs équipes
- Les logiciels qui bénéficient d'une architecture modulaire

En suivant les principes et exemples présentés dans cette section, vous êtes maintenant prêt à intégrer des plugins dans vos propres applications Qt6.

⏭️ [Intégration de code natif (JNI, Objective-C)](/10-integration-et-extensions/03-integration-de-code-natif-jni-objective-c.md)
