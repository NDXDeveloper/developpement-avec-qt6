# 2.1 Modèle d'application Qt (QApplication)

La classe `QApplication` est le cœur de toute application Qt. Elle gère le cycle de vie de votre application et coordonne tous ses aspects essentiels. Comprendre `QApplication` est donc fondamental pour maîtriser le développement avec Qt6.

## Qu'est-ce que QApplication ?

`QApplication` est une classe qui représente votre application dans son ensemble. Elle s'occupe de :

- Initialiser et finaliser les ressources Qt
- Gérer la boucle d'événements principale
- Gérer les paramètres de l'application (thèmes, polices, etc.)
- Coordonner les différents widgets et fenêtres
- Traiter les arguments de ligne de commande

C'est la première classe que vous instancierez dans votre programme et généralement la dernière à être détruite.

## Création d'une instance QApplication

Voici comment créer une instance de `QApplication` dans votre programme :

```cpp
#include <QApplication>
#include <QLabel>

int main(int argc, char *argv[])
{
    // Création de l'objet QApplication
    QApplication app(argc, argv);

    // Création d'un widget simple
    QLabel label("Bonjour Qt !");
    label.show();

    // Démarrage de la boucle d'événements
    return app.exec();
}
```

### Les paramètres de QApplication

`QApplication` accepte deux paramètres dans son constructeur :
- `argc` : Le nombre d'arguments de ligne de commande
- `argv` : Un tableau contenant les arguments de ligne de commande

Ces paramètres permettent à Qt de traiter automatiquement certains arguments standard tels que :
- `--style=<style>` pour définir le style visuel de l'application
- `--qmljsdebugger=port:XXXXX` pour activer le débogueur QML
- `--platform=<plateforme>` pour spécifier la plateforme à utiliser

## La boucle d'événements

La méthode `app.exec()` démarre la boucle d'événements, qui est un concept fondamental dans les applications Qt :

![Boucle d'événements](illustration_boucle_evenements.png)

1. L'application attend que des événements se produisent (clic de souris, touche pressée, minuterie expirée, etc.)
2. Quand un événement survient, il est placé dans une file d'attente
3. La boucle d'événements traite ces événements un par un
4. Pour chaque événement, Qt trouve le widget concerné et lui transmet l'événement
5. Ce cycle continue jusqu'à la fin de l'application

### Code simplifié de la boucle d'événements

Pour mieux comprendre, voici une version simplifiée de ce que fait `app.exec()` :

```cpp
int QApplication::exec()
{
    // Initialize
    isRunning = true;

    // Event loop
    while (isRunning) {
        // Récupérer le prochain événement
        QEvent* event = waitForEvent();

        if (event) {
            // Trouver l'objet destinataire
            QObject* receiver = findReceiver(event);

            // Envoyer l'événement à son destinataire
            if (receiver)
                receiver->event(event);

            // Nettoyer l'événement
            delete event;
        }
    }

    return exitCode;
}
```

## Variantes de QApplication

Qt propose plusieurs classes d'application adaptées à différents types de projets :

### QApplication

Utilisez `QApplication` pour les applications avec une interface graphique utilisant des **widgets**. C'est la classe la plus communément utilisée.

```cpp
#include <QApplication>
#include <QPushButton>

int main(int argc, char *argv[])
{
    QApplication app(argc, argv);

    QPushButton button("Cliquez-moi");
    button.show();

    return app.exec();
}
```

### QGuiApplication

Utilisez `QGuiApplication` pour les applications avec une interface graphique utilisant **Qt Quick/QML**, sans widgets.

```cpp
#include <QGuiApplication>
#include <QQmlApplicationEngine>

int main(int argc, char *argv[])
{
    QGuiApplication app(argc, argv);

    QQmlApplicationEngine engine;
    engine.load(QUrl(QStringLiteral("qrc:/main.qml")));

    return app.exec();
}
```

### QCoreApplication

Utilisez `QCoreApplication` pour les applications sans interface graphique (applications en ligne de commande, services, etc.)

```cpp
#include <QCoreApplication>
#include <QTimer>
#include <QDebug>

int main(int argc, char *argv[])
{
    QCoreApplication app(argc, argv);

    // Créer une minuterie qui va afficher un message et quitter
    QTimer::singleShot(3000, &app, []() {
        qDebug() << "Bonjour depuis une application console !";
        QCoreApplication::quit();
    });

    return app.exec();
}
```

## Hiérarchie des classes d'application

Les classes d'application forment une hiérarchie :

```
QCoreApplication  (applications sans interface graphique)
    ↑
QGuiApplication   (applications Qt Quick/QML)
    ↑
QApplication      (applications avec widgets)
```

Chaque niveau ajoute des fonctionnalités adaptées à un type spécifique d'application.

## Fonctionnalités importantes de QApplication

### Accès à l'instance globale

Dans tout votre code, vous pouvez accéder à l'instance de `QApplication` via la fonction statique `qApp` :

```cpp
// Dans n'importe quelle partie de votre code
QApplication* app = qApp;
// ou directement
qApp->setStyleSheet("QPushButton { background-color: yellow; }");
```

### Gestion du style

`QApplication` permet de personnaliser l'apparence de votre application :

```cpp
// Changer le style de toute l'application
app.setStyle("Fusion");

// Appliquer une feuille de style (CSS pour Qt)
app.setStyleSheet("QPushButton { background-color: #3498db; color: white; border-radius: 5px; padding: 5px; }");
```

### Gestion de la session

`QApplication` gère les événements de session du système d'exploitation (déconnexion, arrêt) :

```cpp
// Se connecter au signal de fin de session
QObject::connect(&app, &QApplication::commitDataRequest,
    [](QSessionManager &manager) {
        // Sauvegarder les données non enregistrées
        sauvegarderDonnees();
    });
```

### Gestion du presse-papiers

`QApplication` donne accès au presse-papiers du système :

```cpp
// Copier du texte dans le presse-papiers
QApplication::clipboard()->setText("Texte à copier");

// Lire le texte du presse-papiers
QString texte = QApplication::clipboard()->text();
```

## Cycle de vie d'une application Qt

Le cycle de vie d'une application Qt suit généralement ces étapes :

1. **Initialisation** : création de l'instance `QApplication`
2. **Configuration** : définition des paramètres globaux
3. **Création de l'interface** : création des widgets ou chargement des fichiers QML
4. **Exécution** : démarrage de la boucle d'événements avec `app.exec()`
5. **Finalisation** : nettoyage des ressources quand l'application se termine

## Exemples pratiques

### Application minimale

```cpp
#include <QApplication>
#include <QLabel>

int main(int argc, char *argv[])
{
    QApplication app(argc, argv);

    QLabel label("Application minimale");
    label.show();

    return app.exec();
}
```

### Application avec paramètres personnalisés

```cpp
#include <QApplication>
#include <QMainWindow>
#include <QFont>

int main(int argc, char *argv[])
{
    // Définir des attributs avant la création de QApplication
    QApplication::setApplicationName("Mon Application");
    QApplication::setOrganizationName("Ma Société");
    QApplication::setApplicationVersion("1.0.0");

    QApplication app(argc, argv);

    // Paramètres globaux
    app.setFont(QFont("Arial", 12));
    app.setStyle("Fusion");

    QMainWindow mainWindow;
    mainWindow.setWindowTitle(QApplication::applicationName() + " v" + QApplication::applicationVersion());
    mainWindow.resize(800, 600);
    mainWindow.show();

    return app.exec();
}
```

## Conseils pour les débutants

- **Une seule instance** : Ne créez jamais plus d'une instance de `QApplication` dans votre programme
- **Premiers pas** : Commencez par créer une application minimale et ajoutez progressivement des fonctionnalités
- **Comprendre la boucle** : Rappellez-vous que tant que `app.exec()` est en cours d'exécution, votre application reste active et réactive
- **Choix approprié** : Utilisez `QApplication` pour les applications avec widgets, `QGuiApplication` pour Qt Quick, et `QCoreApplication` pour les applications console

## Conclusion

`QApplication` est le point de départ de toute application Qt. Cette classe gère la boucle d'événements, qui est au cœur du fonctionnement réactif de votre application. Elle offre également de nombreuses fonctionnalités utiles pour personnaliser et gérer votre application.

Dans la section suivante, nous explorerons les systèmes de widgets et QML qui vous permettront de créer des interfaces utilisateur riches et interactives.
