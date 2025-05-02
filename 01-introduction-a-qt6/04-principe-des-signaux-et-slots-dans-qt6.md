# 1.4 Principe des signaux et slots dans Qt6

Le mécanisme de signaux et slots est l'une des caractéristiques les plus puissantes et distinctives de Qt. C'est le cœur de la programmation d'interfaces graphiques avec Qt. Dans cette section, nous allons découvrir ce concept fondamental de manière simple et accessible.

## Qu'est-ce que le mécanisme de signaux et slots ?

### Le problème à résoudre

Dans une application graphique, nous avons besoin d'un moyen pour que les différents objets puissent communiquer entre eux. Par exemple :

- Lorsqu'un utilisateur clique sur un bouton, comment informer le reste de l'application ?
- Comment mettre à jour plusieurs éléments d'interface quand une donnée change ?

Les approches traditionnelles, comme les fonctions de rappel (callbacks), sont souvent complexes et sources d'erreurs. Qt propose une solution élégante : les signaux et slots.

### La solution de Qt

![Principe des signaux et slots](illustration_signaux_slots.png)

Le principe est simple :

- Un **signal** est émis par un objet lorsqu'un événement particulier se produit
- Un **slot** est une fonction qui est appelée en réponse à un signal particulier
- Qt se charge de connecter les signaux aux slots de manière sécurisée

C'est un peu comme un système d'émetteur-récepteur radio, mais pour les objets de votre application !

## Comprendre les signaux

### Qu'est-ce qu'un signal ?

Un signal est une notification émise par un objet pour indiquer qu'un événement s'est produit. Par exemple :

- Un bouton émet un signal `clicked()` lorsqu'il est cliqué
- Un curseur émet un signal `valueChanged(int)` lorsque sa valeur change
- Une zone de texte émet un signal `textChanged(QString)` lorsque son contenu est modifié

### Caractéristiques des signaux

- Les signaux ne font rien par eux-mêmes, ils se contentent d'annoncer un événement
- Les signaux peuvent transporter des données (paramètres)
- Un signal peut être connecté à plusieurs slots
- Un signal peut même être connecté à un autre signal

## Comprendre les slots

### Qu'est-ce qu'un slot ?

Un slot est une fonction normale qui peut être connectée à un signal. Lorsque le signal est émis, le slot est exécuté. Par exemple :

- Un slot `sauvegarderDocument()` peut être appelé quand un bouton "Sauvegarder" est cliqué
- Un slot `mettreAJourAffichage()` peut être appelé quand la valeur d'un curseur change
- Un slot `validerFormulaire()` peut être appelé quand le texte d'un champ est modifié

### Caractéristiques des slots

- Les slots sont des fonctions C++ normales
- Ils peuvent être appelés normalement comme n'importe quelle fonction
- Un slot peut recevoir des paramètres transmis par le signal
- Un slot peut être connecté à plusieurs signaux différents

## Connexion de signaux et slots

### Syntaxe de base

Dans Qt6, la syntaxe recommandée pour connecter un signal à un slot utilise des pointeurs de fonction :

```cpp
// Crée un bouton
QPushButton *bouton = new QPushButton("Cliquez-moi");

// Connecte le signal clicked() du bouton au slot quitter() de l'application
connect(bouton, &QPushButton::clicked, &QApplication::quit);
```

### Connexion avec paramètres

Si le signal transmet des données, le slot doit accepter ces données :

```cpp
// Crée un curseur qui va de 0 à 100
QSlider *curseur = new QSlider(Qt::Horizontal);
curseur->setRange(0, 100);

// Crée un affichage numérique
QLCDNumber *affichage = new QLCDNumber();

// Connecte le signal valueChanged(int) du curseur au slot display(int) de l'affichage
connect(curseur, &QSlider::valueChanged, affichage, &QLCDNumber::display);
```

Dans cet exemple, quand la valeur du curseur change, le signal `valueChanged(int)` transmet la nouvelle valeur au slot `display(int)` qui met à jour l'affichage.

## Exemple complet

Voici un exemple simple d'une petite application utilisant les signaux et slots :

```cpp
#include <QApplication>
#include <QWidget>
#include <QVBoxLayout>
#include <QPushButton>
#include <QLabel>

int main(int argc, char *argv[])
{
    QApplication app(argc, argv);

    // Crée une fenêtre
    QWidget fenetre;
    fenetre.setWindowTitle("Exemple Signaux et Slots");

    // Crée un label et un bouton
    QLabel *label = new QLabel("Compteur : 0");
    QPushButton *bouton = new QPushButton("Incrémenter");

    // Crée un layout vertical et y ajoute les widgets
    QVBoxLayout *layout = new QVBoxLayout(&fenetre);
    layout->addWidget(label);
    layout->addWidget(bouton);

    // Crée une variable pour compter les clics
    int compteur = 0;

    // Connecte le signal clicked() du bouton à une lambda qui incrémente le compteur
    QObject::connect(bouton, &QPushButton::clicked, [&](){
        compteur++;
        label->setText(QString("Compteur : %1").arg(compteur));
    });

    // Affiche la fenêtre et démarre l'application
    fenetre.show();
    return app.exec();
}
```

Cet exemple crée une petite application avec un label et un bouton. Chaque fois que le bouton est cliqué, le compteur est incrémenté et le texte du label est mis à jour.

## Signaux et slots dans QML

Dans QML, la syntaxe est légèrement différente mais le concept reste le même :

```qml
import QtQuick 2.15
import QtQuick.Controls 2.15

ApplicationWindow {
    visible: true
    width: 300
    height: 200
    title: "Exemple QML"

    Column {
        anchors.centerIn: parent
        spacing: 20

        Label {
            id: compteurLabel
            text: "Compteur : 0"
        }

        Button {
            text: "Incrémenter"
            onClicked: {
                // Le code ici est exécuté quand le bouton est cliqué
                var compteur = parseInt(compteurLabel.text.split(" ")[1])
                compteur++
                compteurLabel.text = "Compteur : " + compteur
            }
        }
    }
}
```

## Avantages des signaux et slots

Ce mécanisme offre de nombreux avantages :

1. **Couplage faible** - Les objets ne savent rien les uns des autres, ce qui facilite la maintenance
2. **Type-safe** - Le compilateur vérifie que les types des paramètres sont compatibles
3. **Flexibilité** - On peut facilement changer les connexions sans modifier le code des classes
4. **Multiplateforme** - Fonctionne de la même manière sur tous les systèmes

## Cas d'utilisation courants

Les signaux et slots sont utiles dans de nombreuses situations :

- **Interfaces utilisateur** - Réagir aux actions de l'utilisateur
- **Validation de formulaires** - Vérifier les entrées lorsqu'elles changent
- **Synchronisation de vues** - Mettre à jour plusieurs vues quand un modèle change
- **Communication entre threads** - Échange sécurisé de données entre threads

## Conclusion

Le mécanisme de signaux et slots est l'un des piliers de la programmation avec Qt. Il peut sembler un peu complexe au début, mais une fois compris, il devient un outil extrêmement puissant qui simplifie grandement le développement d'applications interactives.

Dans les prochaines sections, nous verrons comment utiliser ces concepts dans des applications plus complexes et comment tirer parti de toute la puissance de Qt6.
