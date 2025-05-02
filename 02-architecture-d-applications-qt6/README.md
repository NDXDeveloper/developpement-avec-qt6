# 2. Architecture d'applications Qt6

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

Lorsque vous débutez avec Qt6, comprendre comment structurer votre application est aussi important que de maîtriser la syntaxe du code. Une bonne architecture vous permettra de créer des applications robustes, maintenables et évolutives. Cette section vous présente les concepts fondamentaux de l'architecture d'applications Qt6.

## Qu'est-ce que l'architecture d'une application ?

L'architecture d'une application définit comment ses différentes parties sont organisées et comment elles interagissent entre elles. C'est comme le plan d'une maison : une structure solide est essentielle pour construire quelque chose qui dure.

![Schéma d'architecture Qt](illustration_architecture_generale.png)

## Principes architecturaux de Qt6

Qt6 repose sur plusieurs principes architecturaux fondamentaux :

### Programmation orientée objet

Qt6 exploite pleinement la programmation orientée objet du C++. La classe de base `QObject` fournit des fonctionnalités essentielles comme :
- Le mécanisme de signaux et slots
- La gestion de propriétés
- La hiérarchie parent-enfant pour la gestion mémoire

### Architecture événementielle

Les applications Qt fonctionnent sur un modèle événementiel :
- L'application attend que des événements se produisent (clic, pression d'une touche, etc.)
- Les événements sont placés dans une file d'attente
- La boucle d'événements traite ces événements un par un

Ce modèle permet de créer des interfaces utilisateur réactives sans bloquer l'application.

### Séparation des préoccupations

Qt6 encourage la séparation claire des différentes parties de votre application :
- **Interface utilisateur** : ce que l'utilisateur voit et manipule
- **Logique métier** : les règles et processus de votre application
- **Accès aux données** : communication avec les fichiers, bases de données, etc.

Cette séparation facilite la maintenance et les tests de votre code.

## Types d'applications Qt6

Qt6 vous permet de créer différents types d'applications selon vos besoins :

### Applications de bureau traditionnelles

Utilisant Qt Widgets, ces applications offrent une interface utilisateur classique avec fenêtres, menus, boutons, etc. Elles sont idéales pour les applications professionnelles complexes.

```cpp
int main(int argc, char *argv[])
{
    QApplication app(argc, argv);

    QMainWindow fenetre;
    fenetre.setWindowTitle("Mon Application Qt Widgets");
    fenetre.resize(800, 600);
    fenetre.show();

    return app.exec();
}
```

### Applications modernes avec Qt Quick

Utilisant QML, un langage déclaratif pour décrire des interfaces utilisateur, ces applications offrent des animations fluides et un style moderne, idéal pour les applications mobiles ou nécessitant une expérience utilisateur riche.

```cpp
int main(int argc, char *argv[])
{
    QGuiApplication app(argc, argv);

    QQmlApplicationEngine engine;
    engine.load(QUrl(QStringLiteral("qrc:/main.qml")));

    return app.exec();
}
```

```qml
// main.qml
import QtQuick 2.15
import QtQuick.Controls 2.15

ApplicationWindow {
    visible: true
    width: 800
    height: 600
    title: "Mon Application Qt Quick"

    Text {
        anchors.centerIn: parent
        text: "Bonjour Qt Quick !"
        font.pixelSize: 24
    }
}
```

### Applications hybrides

Qt6 permet également de combiner Qt Widgets et Qt Quick pour tirer parti des avantages des deux approches.

## Structure typique d'un projet Qt6

Un projet Qt6 bien organisé suit généralement cette structure :

```
MonApplication/
├── CMakeLists.txt              // Fichier de configuration du projet
├── main.cpp                    // Point d'entrée de l'application
├── include/                    // Fichiers d'en-tête (.h)
│   ├── mainwindow.h
│   └── ...
├── src/                        // Fichiers d'implémentation (.cpp)
│   ├── mainwindow.cpp
│   └── ...
├── ui/                         // Fichiers d'interface utilisateur
│   ├── mainwindow.ui
│   └── ...
├── qml/                        // Fichiers QML (pour Qt Quick)
│   ├── main.qml
│   └── ...
└── resources/                  // Ressources (images, icônes, etc.)
    └── resources.qrc
```

## Pourquoi l'architecture est importante

Une bonne architecture apporte de nombreux avantages :

1. **Maintenabilité** : Le code est plus facile à comprendre et à modifier
2. **Évolutivité** : Vous pouvez facilement ajouter de nouvelles fonctionnalités
3. **Réutilisabilité** : Les composants peuvent être réutilisés dans d'autres projets
4. **Testabilité** : Les différentes parties peuvent être testées indépendamment
5. **Collaboration** : Plusieurs développeurs peuvent travailler ensemble efficacement

## Approches architecturales communes

Qt6 est flexible et supporte plusieurs approches architecturales populaires :

### Architecture MVC (Modèle-Vue-Contrôleur)

Sépare les données (Modèle), l'interface utilisateur (Vue) et la logique de l'application (Contrôleur).

### Architecture MVVM (Modèle-Vue-VueModèle)

Variante moderne du MVC, particulièrement adaptée à QML avec son système de liaisons de données.

### Architecture modulaire

Divise l'application en modules indépendants qui communiquent via des interfaces bien définies.

---

Dans les sections suivantes, nous explorerons en détail les différentes composantes de l'architecture d'une application Qt6, en commençant par le modèle d'application de base.

⏭️ [Modèle d'application Qt (QApplication)](/02-architecture-d-applications-qt6/01-modele-d-application-qt-qapplication.md)
