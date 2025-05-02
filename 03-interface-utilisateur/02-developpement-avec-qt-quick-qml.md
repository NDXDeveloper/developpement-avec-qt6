# 3.2 Développement avec Qt Quick/QML

Qt Quick est le framework moderne d'interface utilisateur de Qt6, tandis que QML (Qt Modeling Language) est le langage déclaratif utilisé pour créer ces interfaces. Ensemble, ils permettent de développer rapidement des interfaces graphiques fluides, animées et adaptées aux écrans tactiles. Dans cette section, nous découvrirons comment créer des interfaces attrayantes avec Qt Quick et QML.

## Introduction à QML

### Qu'est-ce que QML ?

QML est un langage déclaratif qui permet de décrire l'interface utilisateur et son comportement de manière intuitive. Contrairement à l'approche impérative du C++, QML vous permet de décrire "ce qui" doit être affiché plutôt que "comment" l'afficher.

Voici un exemple simple de code QML :

```qml
import QtQuick 2.15
import QtQuick.Controls 2.15

Rectangle {
    width: 300
    height: 200
    color: "lightblue"

    Text {
        anchors.centerIn: parent
        text: "Bonjour QML !"
        font.pixelSize: 24
        color: "navy"
    }

    Button {
        anchors.bottom: parent.bottom
        anchors.horizontalCenter: parent.horizontalCenter
        anchors.bottomMargin: 20
        text: "Cliquez-moi"

        onClicked: {
            console.log("Bouton cliqué")
        }
    }
}
```

Ce code crée une interface simple avec :
- Un rectangle bleu clair comme fond
- Un texte centré "Bonjour QML !"
- Un bouton en bas qui affiche un message dans la console quand on clique dessus

### Structure d'un fichier QML

Un fichier QML contient généralement :

1. **Imports** - Déclarations qui indiquent quels modules QML utiliser
2. **Élément racine** - L'élément principal qui contient tous les autres
3. **Éléments enfants** - Les éléments contenus dans l'élément racine
4. **Propriétés** - Caractéristiques des éléments (taille, couleur, etc.)
5. **Gestionnaires de signaux** - Code exécuté en réponse aux événements

## Éléments de base de Qt Quick

### Éléments visuels fondamentaux

Qt Quick propose plusieurs éléments visuels de base :

#### Rectangle

Élément de base pour dessiner des rectangles, avec ou sans bordure :

```qml
Rectangle {
    width: 100
    height: 100
    color: "red"
    border.color: "black"
    border.width: 2
    radius: 10  // Pour des coins arrondis
}
```

#### Text

Affiche du texte formaté :

```qml
Text {
    text: "Bonjour QML"
    font.family: "Arial"
    font.pixelSize: 24
    font.bold: true
    color: "blue"
}
```

#### Image

Affiche une image :

```qml
Image {
    source: "qrc:/images/logo.png"  // Depuis les ressources Qt
    width: 100
    height: 100
    fillMode: Image.PreserveAspectFit  // Garde les proportions
}
```

#### MouseArea

Zone sensible aux interactions de la souris :

```qml
Rectangle {
    width: 100
    height: 100
    color: mouseArea.pressed ? "darkred" : "red"  // Change de couleur quand pressé

    MouseArea {
        id: mouseArea
        anchors.fill: parent  // Couvre tout le rectangle parent
        onClicked: {
            console.log("Rectangle cliqué")
        }
    }
}
```

### Positionnement et mise en page

#### Le système d'ancrage

Qt Quick utilise un système d'ancrage puissant pour positionner les éléments les uns par rapport aux autres :

```qml
Rectangle {
    id: conteneur
    width: 300
    height: 200

    Rectangle {
        id: rouge
        width: 100
        height: 50
        color: "red"
        anchors.left: parent.left
        anchors.top: parent.top
        anchors.margins: 10  // Marge pour tous les ancrages
    }

    Rectangle {
        width: 100
        height: 50
        color: "blue"
        anchors.right: parent.right
        anchors.bottom: parent.bottom
        anchors.margins: 10
    }

    Rectangle {
        width: 50
        height: 50
        color: "green"
        anchors.centerIn: parent  // Centré dans le parent
    }
}
```

#### Conteneurs de positionnement

Pour organiser plusieurs éléments, Qt Quick propose des conteneurs similaires aux layouts de Qt Widgets :

##### Column

Place les éléments verticalement :

```qml
Column {
    spacing: 10  // Espace entre les éléments

    Rectangle { width: 100; height: 50; color: "red" }
    Rectangle { width: 100; height: 50; color: "green" }
    Rectangle { width: 100; height: 50; color: "blue" }
}
```

##### Row

Place les éléments horizontalement :

```qml
Row {
    spacing: 10

    Rectangle { width: 50; height: 100; color: "red" }
    Rectangle { width: 50; height: 100; color: "green" }
    Rectangle { width: 50; height: 100; color: "blue" }
}
```

##### Grid

Place les éléments dans une grille :

```qml
Grid {
    columns: 2  // Nombre de colonnes
    spacing: 10

    Rectangle { width: 50; height: 50; color: "red" }
    Rectangle { width: 50; height: 50; color: "green" }
    Rectangle { width: 50; height: 50; color: "blue" }
    Rectangle { width: 50; height: 50; color: "yellow" }
}
```

##### Flow

Place les éléments de gauche à droite, puis de haut en bas (comme du texte) :

```qml
Flow {
    width: 300
    spacing: 10

    Rectangle { width: 80; height: 50; color: "red" }
    Rectangle { width: 120; height: 50; color: "green" }
    Rectangle { width: 150; height: 50; color: "blue" }
    Rectangle { width: 100; height: 50; color: "yellow" }
}
```

## Contrôles Qt Quick

Qt Quick propose une bibliothèque de contrôles prêts à l'emploi similaires aux widgets, mais avec un aspect plus moderne :

### Boutons et contrôles d'entrée

```qml
import QtQuick 2.15
import QtQuick.Controls 2.15

Column {
    spacing: 10
    padding: 20

    Button {
        text: "Cliquez-moi"
        onClicked: console.log("Bouton cliqué")
    }

    CheckBox {
        text: "Option 1"
        checked: true
    }

    RadioButton {
        text: "Choix 1"
        checked: true
    }

    RadioButton {
        text: "Choix 2"
    }

    TextField {
        placeholderText: "Entrez du texte..."
        onTextChanged: console.log("Texte : " + text)
    }

    Slider {
        from: 0
        to: 100
        value: 50
        onValueChanged: console.log("Valeur : " + value)
    }
}
```

### Vues et modèles

Qt Quick permet d'afficher des collections de données avec différentes vues :

#### ListView

Affiche une liste d'éléments :

```qml
ListView {
    width: 200
    height: 300

    model: ["Pomme", "Banane", "Orange", "Fraise", "Cerise"]

    delegate: Rectangle {
        width: ListView.view.width
        height: 50
        color: index % 2 === 0 ? "#f0f0f0" : "#e0e0e0"

        Text {
            anchors.centerIn: parent
            text: modelData  // "modelData" contient la valeur du modèle pour cet élément
        }

        MouseArea {
            anchors.fill: parent
            onClicked: console.log("Élément sélectionné : " + modelData)
        }
    }
}
```

#### GridView

Affiche une grille d'éléments :

```qml
GridView {
    width: 300
    height: 300
    cellWidth: 100
    cellHeight: 100

    model: ["Rouge", "Vert", "Bleu", "Jaune", "Violet", "Orange"]

    delegate: Rectangle {
        width: 90
        height: 90
        color: modelData.toLowerCase()  // Utilise le nom de couleur comme couleur

        Text {
            anchors.centerIn: parent
            text: modelData
            color: "white"
        }
    }
}
```

## Propriétés et liaison de données

### Propriétés personnalisées

Vous pouvez définir vos propres propriétés pour stocker et manipuler des données :

```qml
Rectangle {
    id: compteur
    width: 200
    height: 200

    property int valeur: 0  // Propriété personnalisée

    Text {
        anchors.centerIn: parent
        text: compteur.valeur  // Utilise la propriété du rectangle parent
        font.pixelSize: 36
    }

    MouseArea {
        anchors.fill: parent
        onClicked: compteur.valeur += 1  // Incrémente la propriété
    }
}
```

### Liaison de données (binding)

QML offre un puissant système de liaison de données qui met automatiquement à jour les propriétés liées :

```qml
Rectangle {
    id: curseur
    width: 300
    height: 100

    Rectangle {
        id: poignee
        width: 50
        height: 50
        color: "blue"
        x: mouseArea.pressed ? mouseArea.mouseX - width/2 : x  // Liaison à la position de la souris

        MouseArea {
            id: mouseArea
            anchors.fill: parent
            drag.target: parent  // Permet de faire glisser le rectangle
            drag.axis: Drag.XAxis  // Limite le glissement à l'axe X
            drag.minimumX: 0
            drag.maximumX: curseur.width - poignee.width
        }
    }

    Text {
        anchors.top: curseur.bottom
        anchors.topMargin: 10
        anchors.horizontalCenter: curseur.horizontalCenter
        text: "Valeur : " + Math.round(poignee.x / (curseur.width - poignee.width) * 100)
        // Cette liaison calcule automatiquement une valeur de 0 à 100 basée sur la position du poignée
    }
}
```

## Animations et transitions

QML excelle dans la création d'interfaces animées fluides :

### Animations de base

```qml
Rectangle {
    width: 200
    height: 200

    Rectangle {
        id: carre
        width: 50
        height: 50
        color: "red"

        // Animation qui modifie la position X du carré
        NumberAnimation on x {
            from: 0
            to: 150
            duration: 1000  // Durée en millisecondes
            easing.type: Easing.InOutQuad  // Type d'accélération/décélération
            loops: Animation.Infinite  // Répète indéfiniment
        }
    }
}
```

### Animations basées sur les états

```qml
Rectangle {
    width: 200
    height: 200

    Rectangle {
        id: carre
        width: 50
        height: 50
        color: "red"

        // Définit les différents états du carré
        states: [
            State {
                name: "haut_gauche"  // État par défaut
                PropertyChanges { target: carre; x: 0; y: 0 }
            },
            State {
                name: "haut_droite"
                PropertyChanges { target: carre; x: 150; y: 0 }
            },
            State {
                name: "bas_droite"
                PropertyChanges { target: carre; x: 150; y: 150 }
            },
            State {
                name: "bas_gauche"
                PropertyChanges { target: carre; x: 0; y: 150 }
            }
        ]

        // Définit les transitions entre les états
        transitions: [
            Transition {
                from: "*"  // Depuis n'importe quel état
                to: "*"    // Vers n'importe quel état

                // Anime les propriétés x et y
                NumberAnimation {
                    properties: "x,y"
                    duration: 500
                    easing.type: Easing.InOutQuad
                }
            }
        ]

        // Timer qui change l'état toutes les secondes
        Timer {
            interval: 1000
            running: true
            repeat: true

            property int etatActuel: 0

            onTriggered: {
                var etats = ["haut_gauche", "haut_droite", "bas_droite", "bas_gauche"]
                carre.state = etats[etatActuel]
                etatActuel = (etatActuel + 1) % 4
            }
        }
    }
}
```

## Intégration de QML avec C++

Pour des applications complexes, vous pouvez combiner la puissance de QML pour l'interface avec la robustesse du C++ pour la logique métier :

### Exposer des objets C++ à QML

#### Côté C++ :

```cpp
// myclasse.h
class MaClasse : public QObject
{
    Q_OBJECT
    Q_PROPERTY(QString message READ message WRITE setMessage NOTIFY messageChanged)

public:
    MaClasse(QObject *parent = nullptr);

    QString message() const;
    void setMessage(const QString &message);

signals:
    void messageChanged();

public slots:
    void traiterDonnees(const QString &donnees);

private:
    QString m_message;
};

// main.cpp
int main(int argc, char *argv[])
{
    QGuiApplication app(argc, argv);

    // Création de l'objet C++
    MaClasse *objet = new MaClasse(&app);

    // Configuration du moteur QML
    QQmlApplicationEngine engine;

    // Exposition de l'objet au contexte QML
    engine.rootContext()->setContextProperty("maClasse", objet);

    // Chargement du fichier QML principal
    engine.load(QUrl(QStringLiteral("qrc:/main.qml")));

    return app.exec();
}
```

#### Côté QML :

```qml
import QtQuick 2.15
import QtQuick.Controls 2.15

Rectangle {
    width: 300
    height: 200
    color: "lightblue"

    Column {
        anchors.centerIn: parent
        spacing: 20

        Text {
            text: maClasse.message  // Utilise la propriété de l'objet C++
            font.pixelSize: 18
        }

        TextField {
            placeholderText: "Entrez un message"
            onTextChanged: maClasse.message = text  // Modifie la propriété C++
        }

        Button {
            text: "Traiter"
            onClicked: maClasse.traiterDonnees(maClasse.message)  // Appelle le slot C++
        }
    }
}
```

### Enregistrer des types C++ pour QML

Pour utiliser vos propres types C++ directement dans QML :

```cpp
// Dans main.cpp ou ailleurs avant de charger QML
#include <QQmlApplicationEngine>
#include "montype.h"

int main(int argc, char *argv[])
{
    // ...

    // Enregistrement d'un type pour QML
    qmlRegisterType<MonType>("MesTypes", 1, 0, "MonType");

    // ...
}
```

Ensuite, dans QML :

```qml
import QtQuick 2.15
import MesTypes 1.0  // Import du module personnalisé

Rectangle {
    width: 300
    height: 200

    MonType {  // Utilisation directe du type C++ enregistré
        id: monObjet
        propriete1: "valeur"

        onSignal1: console.log("Signal reçu")
    }
}
```

## Application QML complète : un compteur animé

Assemblons ces concepts dans une petite application :

```qml
import QtQuick 2.15
import QtQuick.Controls 2.15
import QtQuick.Layouts 1.15

ApplicationWindow {
    id: fenetre
    visible: true
    width: 300
    height: 400
    title: "Compteur animé"

    property int compteur: 0

    Rectangle {
        anchors.fill: parent
        gradient: Gradient {
            GradientStop { position: 0.0; color: "#f6f6f6" }
            GradientStop { position: 1.0; color: "#d7d7d7" }
        }

        ColumnLayout {
            anchors.centerIn: parent
            spacing: 20

            Text {
                id: affichage
                text: fenetre.compteur.toString()
                font.pixelSize: 72
                color: "#333333"
                Layout.alignment: Qt.AlignHCenter

                // Animation lorsque le compteur change
                Behavior on text {
                    NumberAnimation {
                        target: affichage
                        property: "scale"
                        from: 1.5
                        to: 1.0
                        duration: 300
                        easing.type: Easing.OutBack
                    }
                }
            }

            Row {
                spacing: 20
                Layout.alignment: Qt.AlignHCenter

                RoundButton {
                    text: "-"
                    font.pixelSize: 20
                    width: 60
                    height: 60

                    onClicked: {
                        if (fenetre.compteur > 0) {
                            fenetre.compteur--
                        }
                    }
                }

                RoundButton {
                    text: "+"
                    font.pixelSize: 20
                    width: 60
                    height: 60

                    onClicked: {
                        fenetre.compteur++
                    }
                }
            }

            Button {
                text: "Réinitialiser"
                Layout.alignment: Qt.AlignHCenter

                onClicked: fenetre.compteur = 0
            }
        }
    }
}
```

## Bonnes pratiques QML

1. **Séparez la logique de la présentation**
   - Utilisez QML pour l'interface
   - Utilisez C++ pour la logique complexe
   - Utilisez JavaScript en QML pour la logique simple d'interface

2. **Organisez votre code QML**
   - Créez des composants réutilisables
   - Utilisez des fichiers QML séparés pour les éléments complexes
   - Utilisez des propriétés pour configurer les composants

3. **Optimisez les performances**
   - Limitez le nombre d'éléments QML visibles à la fois
   - Utilisez `Loader` pour charger dynamiquement des composants
   - Évitez les liaisons de données (bindings) complexes ou en cascade

4. **Utilisez les outils Qt**
   - Qt Design Studio pour la conception visuelle
   - Qt Creator pour l'édition et le débogage
   - QML Profiler pour identifier les goulots d'étranglement

## Conclusion

Qt Quick et QML offrent une approche moderne et puissante pour créer des interfaces utilisateur attrayantes et animées. Leur style déclaratif facilite la création d'interfaces complexes avec moins de code, tandis que l'intégration avec C++ permet de ne faire aucun compromis sur les performances ou les fonctionnalités.

En maîtrisant les concepts de base comme les éléments visuels, le positionnement, les propriétés et les animations, vous pouvez créer des applications modernes qui fonctionnent aussi bien sur bureau que sur mobile.

Dans la prochaine section, nous explorerons comment personnaliser l'apparence de vos applications Qt avec les styles et les thèmes.
