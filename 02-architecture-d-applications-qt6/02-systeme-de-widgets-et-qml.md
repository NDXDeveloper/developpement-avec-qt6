# 2.2 Système de widgets et QML

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

Qt6 propose deux approches principales pour créer des interfaces utilisateur : le système de **widgets** traditionnel et le langage **QML** moderne. Chaque approche a ses avantages et peut même être combinée dans une même application. Voyons comment fonctionnent ces deux systèmes et comment choisir celui qui convient le mieux à votre projet.

## Les widgets : l'approche traditionnelle

### Qu'est-ce qu'un widget ?

Un widget est un élément d'interface graphique : bouton, étiquette, champ de texte, case à cocher, etc. Dans Qt, les widgets sont des objets C++ qui peuvent être affichés à l'écran et avec lesquels l'utilisateur peut interagir.

![Exemples de widgets Qt](illustration_widgets.png)

Les widgets Qt sont :
- **Portables** : ils s'adaptent automatiquement au style du système d'exploitation
- **Personnalisables** : leur apparence peut être modifiée avec des styles et des feuilles de style
- **Organisés hiérarchiquement** : un widget peut contenir d'autres widgets

### Création d'une interface avec des widgets

Voici un exemple simple pour créer une fenêtre avec quelques widgets :

```cpp
#include <QApplication>
#include <QWidget>
#include <QVBoxLayout>
#include <QLabel>
#include <QLineEdit>
#include <QPushButton>

int main(int argc, char *argv[])
{
    QApplication app(argc, argv);

    // Création de la fenêtre principale
    QWidget fenetre;
    fenetre.setWindowTitle("Ma première interface");
    fenetre.resize(300, 150);

    // Création d'un layout vertical pour organiser les widgets
    QVBoxLayout *layout = new QVBoxLayout(&fenetre);

    // Ajout d'une étiquette
    QLabel *label = new QLabel("Entrez votre nom :", &fenetre);
    layout->addWidget(label);

    // Ajout d'un champ de texte
    QLineEdit *champTexte = new QLineEdit(&fenetre);
    layout->addWidget(champTexte);

    // Ajout d'un bouton
    QPushButton *bouton = new QPushButton("Dire Bonjour", &fenetre);
    layout->addWidget(bouton);

    // Connexion du signal clicked() du bouton à une lambda
    QObject::connect(bouton, &QPushButton::clicked, [champTexte]() {
        QString nom = champTexte->text();
        if(nom.isEmpty())
            nom = "inconnu";
        QMessageBox::information(nullptr, "Salutation", "Bonjour, " + nom + " !");
    });

    // Affichage de la fenêtre
    fenetre.show();

    return app.exec();
}
```

Ce code crée une interface simple avec une étiquette, un champ de texte et un bouton. Lorsque l'utilisateur clique sur le bouton, une boîte de dialogue s'affiche avec un message de salutation.

### Système de layouts

Les layouts sont des gestionnaires de disposition qui permettent d'organiser les widgets de manière flexible :

- **QVBoxLayout** : organise les widgets verticalement
- **QHBoxLayout** : organise les widgets horizontalement
- **QGridLayout** : organise les widgets dans une grille
- **QFormLayout** : organise les widgets sous forme d'étiquettes et de champs

```cpp
// Exemple avec QGridLayout
QGridLayout *grille = new QGridLayout();
grille->addWidget(new QLabel("Nom :"), 0, 0);
grille->addWidget(new QLineEdit(), 0, 1);
grille->addWidget(new QLabel("Prénom :"), 1, 0);
grille->addWidget(new QLineEdit(), 1, 1);
```

Les layouts s'adaptent automatiquement lorsque la fenêtre est redimensionnée, ce qui facilite la création d'interfaces réactives.

### Qt Designer : création visuelle d'interfaces

Qt Creator intègre Qt Designer, un outil qui permet de créer des interfaces graphiques par glisser-déposer :

![Qt Designer](illustration_qt_designer.png)

1. Dessinez votre interface en glissant des widgets depuis la palette
2. Organisez-les avec des layouts
3. Définissez les propriétés des widgets dans l'éditeur de propriétés
4. Le résultat est un fichier `.ui` qui peut être chargé dans votre application

```cpp
// Chargement d'une interface créée avec Qt Designer
#include "ui_mainwindow.h"

MainWindow::MainWindow(QWidget *parent) :
    QMainWindow(parent),
    ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    // Accès aux widgets par leur nom
    connect(ui->pushButton, &QPushButton::clicked, this, &MainWindow::onButtonClicked);
}
```

Qt Designer accélère considérablement le développement d'interfaces graphiques sans nécessiter de coder manuellement chaque widget.

## QML : l'approche moderne

### Qu'est-ce que QML ?

QML (Qt Modeling Language) est un langage déclaratif conçu spécifiquement pour créer des interfaces utilisateur modernes. Il fait partie de Qt Quick, le framework d'interface utilisateur nouvelle génération de Qt.

```qml
// Exemple simple en QML
import QtQuick 2.15
import QtQuick.Controls 2.15

ApplicationWindow {
    visible: true
    width: 300
    height: 150
    title: "Ma première interface QML"

    Column {
        anchors.centerIn: parent
        spacing: 10

        Text {
            text: "Entrez votre nom :"
        }

        TextField {
            id: champNom
            width: 200
        }

        Button {
            text: "Dire Bonjour"
            onClicked: {
                var nom = champNom.text || "inconnu"
                messageDialog.text = "Bonjour, " + nom + " !"
                messageDialog.open()
            }
        }
    }

    Dialog {
        id: messageDialog
        title: "Salutation"
        property alias text: messageLabel.text

        Label {
            id: messageLabel
        }

        standardButtons: Dialog.Ok
    }
}
```

### Caractéristiques de QML

QML offre plusieurs avantages :

- **Syntaxe déclarative** : plus intuitive pour décrire des interfaces utilisateur
- **Séparation claire** entre la présentation (QML) et la logique (JavaScript ou C++)
- **Animations fluides** intégrées et faciles à mettre en œuvre
- **Liaison de données** bidirectionnelle entre propriétés
- **Idéal pour les interfaces tactiles** et les applications mobiles

### Structure d'un document QML

Un document QML est composé d'éléments imbriqués, chacun avec des propriétés :

```qml
Element {                     // Type d'élément
    propriete1: valeur        // Propriété simple
    propriete2: "texte"       // Propriété de type chaîne

    Property propriete3: 100  // Propriété définie par l'utilisateur

    signal monSignal          // Définition d'un signal

    Element2 {                // Élément enfant
        id: monIdentifiant    // Identifiant pour référencer cet élément
        anchors.fill: parent  // Système de positionnement (anchors)
    }

    onMonSignal: {            // Gestionnaire de signal
        console.log("Signal reçu")
    }
}
```

### Types d'éléments QML courants

QML propose de nombreux éléments prêts à l'emploi :

- **Rectangle, Image, Text** : éléments visuels de base
- **MouseArea** : zone sensible aux interactions de la souris
- **Row, Column, Grid** : conteneurs pour organiser les éléments
- **Animations** : PropertyAnimation, NumberAnimation, etc.
- **Contrôles** : Button, TextField, ComboBox, etc. (QtQuick.Controls)

### Intégration avec C++

QML peut facilement être intégré avec du code C++ :

```cpp
// Enregistrement d'une classe C++ pour l'utiliser en QML
#include <QGuiApplication>
#include <QQmlApplicationEngine>
#include <QQmlContext>
#include "monmodele.h"

int main(int argc, char *argv[])
{
    QGuiApplication app(argc, argv);

    // Création d'un modèle C++
    MonModele modele;

    // Moteur QML
    QQmlApplicationEngine engine;

    // Exposition du modèle au contexte QML
    engine.rootContext()->setContextProperty("monModele", &modele);

    // Chargement du fichier QML principal
    engine.load(QUrl(QStringLiteral("qrc:/main.qml")));

    return app.exec();
}
```

En QML, on peut ensuite accéder au modèle :

```qml
Button {
    text: "Mettre à jour"
    onClicked: {
        monModele.mettreAJour()
        texteResultat.text = monModele.resultat
    }
}
```

## Comment choisir entre widgets et QML ?

### Choisir les widgets si :

- Vous développez une application de bureau traditionnelle
- Vous avez besoin d'une intégration parfaite avec le système d'exploitation
- Vous utilisez beaucoup de contrôles complexes (tableaux, arbres, etc.)
- Vous préférez coder entièrement en C++
- Votre équipe a déjà une forte expertise en widgets Qt

### Choisir QML si :

- Vous créez une interface moderne avec des animations fluides
- Vous développez pour mobile (Android, iOS) ou des écrans tactiles
- Votre design est personnalisé et ne suit pas les conventions du système
- Vous souhaitez séparer clairement l'interface de la logique
- Vous préférez une approche déclarative pour concevoir l'interface

### Approche hybride

Qt6 permet également de combiner les deux approches :

- Utiliser QWidget comme conteneur principal et intégrer des éléments QML
- Utiliser QML pour l'interface et C++ pour la logique métier
- Créer des widgets personnalisés avec QML

```cpp
// Exemple d'intégration de QML dans une application widgets
#include <QApplication>
#include <QWidget>
#include <QVBoxLayout>
#include <QQuickWidget>

int main(int argc, char *argv[])
{
    QApplication app(argc, argv);

    // Création d'une fenêtre principale avec widgets
    QWidget fenetre;
    QVBoxLayout *layout = new QVBoxLayout(&fenetre);

    // Ajout d'un widget standard
    QPushButton *bouton = new QPushButton("Bouton standard");
    layout->addWidget(bouton);

    // Ajout d'un widget QML
    QQuickWidget *qmlWidget = new QQuickWidget();
    qmlWidget->setSource(QUrl("qrc:/monInterface.qml"));
    qmlWidget->setResizeMode(QQuickWidget::SizeRootObjectToView);
    layout->addWidget(qmlWidget);

    fenetre.show();
    return app.exec();
}
```

## Conseils pour les débutants

### Pour commencer avec les widgets :

1. Explorez les widgets disponibles dans la documentation Qt
2. Apprenez à utiliser Qt Designer pour créer rapidement des interfaces
3. Comprenez le système de layouts pour des interfaces responsives
4. Expérimentez avec les signaux et slots pour rendre l'interface interactive

### Pour commencer avec QML :

1. Familiarisez-vous avec la syntaxe déclarative de QML
2. Apprenez les éléments de base (Rectangle, Text, Image, etc.)
3. Explorez le système d'ancrage et de positionnement
4. Expérimentez avec les animations et les transitions

## Conclusion

Qt6 offre deux approches puissantes pour créer des interfaces utilisateur : les widgets traditionnels et QML moderne. Chacune a ses forces et ses cas d'utilisation idéaux. Les widgets brillent dans les applications de bureau traditionnelles, tandis que QML excelle dans les interfaces modernes et animées.

En tant que débutant, n'hésitez pas à explorer les deux approches pour comprendre leurs spécificités. Avec l'expérience, vous saurez intuitivement quelle approche convient le mieux à chaque projet. Et rappelez-vous : vous pouvez toujours combiner les deux pour tirer parti du meilleur des deux mondes !

⏭️ [Architecture Model-View-Controller (MVC)](/02-architecture-d-applications-qt6/03-architecture-model-view-controller-mvc.md)
