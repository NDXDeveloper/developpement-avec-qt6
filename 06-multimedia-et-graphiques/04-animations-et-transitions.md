# 6.4 Animations et transitions

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

## Introduction aux animations dans Qt

Les animations sont un moyen efficace de rendre votre interface utilisateur plus dynamique et intuitive. Qt6 offre plusieurs façons d'ajouter des animations et des transitions à vos applications, depuis des solutions simples jusqu'à des systèmes d'animation complexes.

Dans ce chapitre, nous allons explorer :
- Les animations de base avec `QPropertyAnimation`
- Les groupes d'animations pour synchroniser plusieurs animations
- Les machines à états et les transitions animées
- Les animations avec Qt Quick (QML)

Que vous souhaitiez animer la position d'un bouton, la transparence d'une image, ou créer des transitions fluides entre différents états de votre application, Qt propose des outils adaptés à vos besoins.

## Animations de propriétés avec QPropertyAnimation

La classe `QPropertyAnimation` est le moyen le plus simple d'animer les propriétés d'un objet Qt. Elle permet de modifier progressivement une propriété d'un objet entre deux valeurs sur une période donnée.

### Concepts de base

Pour créer une animation de propriété, vous aurez besoin de spécifier :
- L'objet à animer
- La propriété à modifier
- La durée de l'animation
- Les valeurs de début et de fin
- Optionnellement, une courbe d'accélération pour contrôler la vitesse

### Exemple simple : Animation de position

Commençons par un exemple simple : animer la position d'un bouton.

```cpp
#include <QApplication>
#include <QPushButton>
#include <QPropertyAnimation>

int main(int argc, char *argv[])
{
    QApplication app(argc, argv);

    // Créer un widget parent pour contenir notre bouton
    QWidget window;
    window.setFixedSize(300, 300);
    window.setWindowTitle("Animation de position");

    // Créer un bouton
    QPushButton *button = new QPushButton("Cliquez-moi", &window);
    button->setGeometry(10, 10, 100, 30);  // Position et taille initiales

    // Créer une animation
    QPropertyAnimation *animation = new QPropertyAnimation(button, "geometry");
    animation->setDuration(3000);  // 3 secondes
    animation->setStartValue(QRect(10, 10, 100, 30));
    animation->setEndValue(QRect(190, 250, 100, 30));

    // Connecter le clic du bouton pour déclencher l'animation
    QObject::connect(button, &QPushButton::clicked, [animation]() {
        animation->start();
    });

    window.show();
    return app.exec();
}
```

Dans cet exemple, lorsque vous cliquez sur le bouton, il se déplace du coin supérieur gauche vers le coin inférieur droit de la fenêtre sur une durée de 3 secondes.

### Animer d'autres propriétés

Vous pouvez animer de nombreuses propriétés des widgets Qt. Voici quelques exemples :

#### Animation de couleur

```cpp
// Créer un label avec un fond coloré
QLabel *colorLabel = new QLabel(widget);
colorLabel->setGeometry(100, 100, 100, 100);
colorLabel->setStyleSheet("background-color: red;");

// Créer l'animation
QPropertyAnimation *colorAnimation = new QPropertyAnimation(colorLabel, "styleSheet");
colorAnimation->setDuration(2000);  // 2 secondes
colorAnimation->setStartValue("background-color: red;");
colorAnimation->setEndValue("background-color: blue;");
```

#### Animation d'opacité

```cpp
// Créer un label avec une image
QLabel *imageLabel = new QLabel(widget);
imageLabel->setPixmap(QPixmap("image.jpg"));
imageLabel->setGeometry(50, 50, 200, 200);

// Animer l'opacité (nécessite de définir la feuille de style)
QPropertyAnimation *opacityAnimation = new QPropertyAnimation(imageLabel, "windowOpacity");
opacityAnimation->setDuration(1500);  // 1.5 secondes
opacityAnimation->setStartValue(1.0); // Complètement opaque
opacityAnimation->setEndValue(0.2);   // Presque transparent
```

### Courbes d'accélération

Les courbes d'accélération permettent de contrôler la vitesse de l'animation au fil du temps. Qt fournit plusieurs courbes prédéfinies dans la classe `QEasingCurve`.

```cpp
// Créer une animation avec une courbe d'accélération
QPropertyAnimation *animation = new QPropertyAnimation(button, "geometry");
animation->setDuration(3000);
animation->setStartValue(QRect(10, 10, 100, 30));
animation->setEndValue(QRect(190, 250, 100, 30));

// Définir une courbe d'accélération avec rebond
animation->setEasingCurve(QEasingCurve::OutBounce);
```

Voici quelques courbes d'accélération couramment utilisées :
- `QEasingCurve::Linear` : Vitesse constante
- `QEasingCurve::InQuad` : Accélération progressive
- `QEasingCurve::OutQuad` : Décélération progressive
- `QEasingCurve::InOutQuad` : Accélération puis décélération
- `QEasingCurve::OutBounce` : Effet de rebond à la fin
- `QEasingCurve::InElastic` : Effet élastique au début

## Groupes d'animations

Parfois, vous souhaitez exécuter plusieurs animations ensemble, soit en parallèle, soit séquentiellement. Qt fournit deux classes pour cela :
- `QParallelAnimationGroup` : Exécute toutes les animations en même temps
- `QSequentialAnimationGroup` : Exécute les animations les unes après les autres

### Animations parallèles

```cpp
#include <QParallelAnimationGroup>

// Créer deux boutons
QPushButton *button1 = new QPushButton("Bouton 1", widget);
button1->setGeometry(10, 10, 100, 30);

QPushButton *button2 = new QPushButton("Bouton 2", widget);
button2->setGeometry(10, 50, 100, 30);

// Créer deux animations
QPropertyAnimation *anim1 = new QPropertyAnimation(button1, "geometry");
anim1->setDuration(2000);
anim1->setStartValue(QRect(10, 10, 100, 30));
anim1->setEndValue(QRect(190, 10, 100, 30));

QPropertyAnimation *anim2 = new QPropertyAnimation(button2, "geometry");
anim2->setDuration(2000);
anim2->setStartValue(QRect(10, 50, 100, 30));
anim2->setEndValue(QRect(190, 50, 100, 30));

// Groupe d'animations parallèles
QParallelAnimationGroup *parallelGroup = new QParallelAnimationGroup;
parallelGroup->addAnimation(anim1);
parallelGroup->addAnimation(anim2);

// Démarrer toutes les animations en même temps
parallelGroup->start();
```

### Animations séquentielles

```cpp
#include <QSequentialAnimationGroup>

// Créer un bouton
QPushButton *button = new QPushButton("Bouton", widget);
button->setGeometry(10, 10, 100, 30);

// Créer trois animations pour des mouvements différents
QPropertyAnimation *anim1 = new QPropertyAnimation(button, "geometry");
anim1->setDuration(1000);
anim1->setStartValue(QRect(10, 10, 100, 30));
anim1->setEndValue(QRect(190, 10, 100, 30));

QPropertyAnimation *anim2 = new QPropertyAnimation(button, "geometry");
anim2->setDuration(1000);
anim2->setStartValue(QRect(190, 10, 100, 30));
anim2->setEndValue(QRect(190, 250, 100, 30));

QPropertyAnimation *anim3 = new QPropertyAnimation(button, "geometry");
anim3->setDuration(1000);
anim3->setStartValue(QRect(190, 250, 100, 30));
anim3->setEndValue(QRect(10, 250, 100, 30));

// Groupe d'animations séquentielles
QSequentialAnimationGroup *sequentialGroup = new QSequentialAnimationGroup;
sequentialGroup->addAnimation(anim1);
sequentialGroup->addAnimation(anim2);
sequentialGroup->addAnimation(anim3);

// Démarrer les animations l'une après l'autre
sequentialGroup->start();
```

### Combinaison de groupes

Vous pouvez également imbriquer des groupes d'animations pour créer des effets plus complexes :

```cpp
// Créer deux boutons
QPushButton *button1 = new QPushButton("Bouton 1", widget);
button1->setGeometry(10, 10, 100, 30);

QPushButton *button2 = new QPushButton("Bouton 2", widget);
button2->setGeometry(10, 50, 100, 30);

// Animer le bouton 1 horizontalement
QPropertyAnimation *anim1 = new QPropertyAnimation(button1, "geometry");
anim1->setDuration(1000);
anim1->setStartValue(QRect(10, 10, 100, 30));
anim1->setEndValue(QRect(190, 10, 100, 30));

// Animer le bouton 2 en deux étapes (droite puis bas)
QPropertyAnimation *anim2a = new QPropertyAnimation(button2, "geometry");
anim2a->setDuration(1000);
anim2a->setStartValue(QRect(10, 50, 100, 30));
anim2a->setEndValue(QRect(190, 50, 100, 30));

QPropertyAnimation *anim2b = new QPropertyAnimation(button2, "geometry");
anim2b->setDuration(1000);
anim2b->setStartValue(QRect(190, 50, 100, 30));
anim2b->setEndValue(QRect(190, 250, 100, 30));

// Groupe séquentiel pour le bouton 2
QSequentialAnimationGroup *seqGroup = new QSequentialAnimationGroup;
seqGroup->addAnimation(anim2a);
seqGroup->addAnimation(anim2b);

// Groupe parallèle pour exécuter les deux ensembles d'animations
QParallelAnimationGroup *parallelGroup = new QParallelAnimationGroup;
parallelGroup->addAnimation(anim1);
parallelGroup->addAnimation(seqGroup);

// Démarrer l'animation combinée
parallelGroup->start();
```

## Contrôle des animations

Qt fournit plusieurs méthodes pour contrôler le comportement des animations :

### Boucles d'animation

Vous pouvez faire boucler une animation en définissant son nombre de boucles :

```cpp
// Boucler indéfiniment (valeur -1)
animation->setLoopCount(-1);

// Boucler 3 fois
animation->setLoopCount(3);
```

### Direction d'animation

Vous pouvez également définir la direction d'une animation :

```cpp
// Aller-retour
animation->setDirection(QPropertyAnimation::Forward);     // Par défaut (début vers fin)
animation->setDirection(QPropertyAnimation::Backward);    // Fin vers début
animation->setDirection(QPropertyAnimation::Alternate);   // Alterne entre les deux
```

### Contrôle manuel de l'animation

Vous pouvez contrôler manuellement l'animation avec les méthodes suivantes :

```cpp
animation->start();   // Démarrer l'animation
animation->pause();   // Mettre en pause
animation->resume();  // Reprendre après une pause
animation->stop();    // Arrêter l'animation
```

Vous pouvez également connecter les signaux d'animation pour réagir à certains événements :

```cpp
// Réagir lorsque l'animation est terminée
QObject::connect(animation, &QPropertyAnimation::finished, []() {
    qDebug() << "Animation terminée !";
});

// Réagir lors d'un changement d'état
QObject::connect(animation, &QPropertyAnimation::stateChanged,
                [](QAbstractAnimation::State newState, QAbstractAnimation::State oldState) {
    if (newState == QAbstractAnimation::Running) {
        qDebug() << "L'animation a démarré !";
    }
});
```

## Animation de l'état des widgets

Qt propose également une façon élégante d'animer les transitions entre différents états d'un widget avec `QStateMachine` et `QState`.

### Exemple simple de machine à états

```cpp
#include <QApplication>
#include <QPushButton>
#include <QStateMachine>
#include <QState>
#include <QPropertyAnimation>

int main(int argc, char *argv[])
{
    QApplication app(argc, argv);

    // Créer une fenêtre et un bouton
    QWidget window;
    window.setFixedSize(300, 300);
    window.setWindowTitle("Animation avec machine à états");

    QPushButton *button = new QPushButton("Cliquez-moi", &window);
    button->setGeometry(10, 10, 100, 30);

    // Créer une machine à états
    QStateMachine *machine = new QStateMachine;

    // Créer deux états
    QState *state1 = new QState(machine);
    QState *state2 = new QState(machine);

    // Définir les propriétés pour chaque état
    state1->assignProperty(button, "geometry", QRect(10, 10, 100, 30));
    state2->assignProperty(button, "geometry", QRect(190, 250, 100, 30));

    // Définir les transitions entre les états
    state1->addTransition(button, &QPushButton::clicked, state2);
    state2->addTransition(button, &QPushButton::clicked, state1);

    // Configurer l'animation pour les transitions
    QPropertyAnimation *animation = new QPropertyAnimation(button, "geometry");
    animation->setDuration(1000);

    // Créer une transition animée
    QSignalTransition *transition1 = state1->addTransition(button, &QPushButton::clicked, state2);
    transition1->addAnimation(animation);

    QSignalTransition *transition2 = state2->addTransition(button, &QPushButton::clicked, state1);
    transition2->addAnimation(animation);

    // Définir l'état initial
    machine->setInitialState(state1);

    // Démarrer la machine à états
    machine->start();

    window.show();
    return app.exec();
}
```

Dans cet exemple, chaque fois que vous cliquez sur le bouton, il se déplace de sa position actuelle à l'autre position, en utilisant une animation.

### Machines à états plus complexes

Vous pouvez créer des machines à états plus complexes avec plusieurs états et transitions :

```cpp
// Créer un bouton
QPushButton *button = new QPushButton("Bouton", widget);
button->setGeometry(10, 10, 100, 30);

// Créer une machine à états
QStateMachine *machine = new QStateMachine;

// Créer quatre états pour les quatre coins de la fenêtre
QState *topLeft = new QState(machine);
QState *topRight = new QState(machine);
QState *bottomRight = new QState(machine);
QState *bottomLeft = new QState(machine);

// Définir les propriétés pour chaque état
topLeft->assignProperty(button, "geometry", QRect(10, 10, 100, 30));
topRight->assignProperty(button, "geometry", QRect(190, 10, 100, 30));
bottomRight->assignProperty(button, "geometry", QRect(190, 250, 100, 30));
bottomLeft->assignProperty(button, "geometry", QRect(10, 250, 100, 30));

// Définir les transitions
topLeft->addTransition(button, &QPushButton::clicked, topRight);
topRight->addTransition(button, &QPushButton::clicked, bottomRight);
bottomRight->addTransition(button, &QPushButton::clicked, bottomLeft);
bottomLeft->addTransition(button, &QPushButton::clicked, topLeft);

// Créer une animation pour toutes les transitions
QPropertyAnimation *animation = new QPropertyAnimation(button, "geometry");
animation->setDuration(1000);
animation->setEasingCurve(QEasingCurve::OutCubic);

// Ajouter l'animation à chaque transition
for (QState *state : {topLeft, topRight, bottomRight, bottomLeft}) {
    for (QAbstractTransition *transition : state->transitions()) {
        QSignalTransition *signalTransition = qobject_cast<QSignalTransition*>(transition);
        if (signalTransition) {
            signalTransition->addAnimation(animation);
        }
    }
}

// Définir l'état initial et démarrer
machine->setInitialState(topLeft);
machine->start();
```

## Exemple complet : Carrousel d'images animé

Voici un exemple plus complet d'une application qui affiche un carrousel d'images avec des transitions animées :

```cpp
#include <QApplication>
#include <QWidget>
#include <QLabel>
#include <QPushButton>
#include <QHBoxLayout>
#include <QVBoxLayout>
#include <QPropertyAnimation>
#include <QParallelAnimationGroup>
#include <QSequentialAnimationGroup>
#include <QPixmap>
#include <QDir>
#include <QFileInfoList>

class ImageCarousel : public QWidget
{
    Q_OBJECT

public:
    ImageCarousel(const QString &imageDir, QWidget *parent = nullptr)
        : QWidget(parent), currentIndex(0)
    {
        // Charger les images depuis le répertoire spécifié
        QDir directory(imageDir);
        QStringList filters;
        filters << "*.jpg" << "*.png" << "*.jpeg";
        directory.setNameFilters(filters);

        QFileInfoList fileList = directory.entryInfoList();
        for (const QFileInfo &fileInfo : fileList) {
            imagePaths.append(fileInfo.absoluteFilePath());
        }

        if (imagePaths.isEmpty()) {
            qDebug() << "Aucune image trouvée dans le répertoire:" << imageDir;
            return;
        }

        // Créer les widgets
        imageLabel = new QLabel(this);
        imageLabel->setFixedSize(600, 400);
        imageLabel->setAlignment(Qt::AlignCenter);
        imageLabel->setStyleSheet("background-color: black;");

        prevButton = new QPushButton("Précédent", this);
        nextButton = new QPushButton("Suivant", this);

        // Créer les layouts
        QHBoxLayout *buttonLayout = new QHBoxLayout();
        buttonLayout->addWidget(prevButton);
        buttonLayout->addWidget(nextButton);

        QVBoxLayout *mainLayout = new QVBoxLayout(this);
        mainLayout->addWidget(imageLabel);
        mainLayout->addLayout(buttonLayout);

        // Connecter les boutons
        connect(prevButton, &QPushButton::clicked, this, &ImageCarousel::showPreviousImage);
        connect(nextButton, &QPushButton::clicked, this, &ImageCarousel::showNextImage);

        // Afficher la première image
        loadImage(currentIndex);

        setWindowTitle("Carrousel d'images animé");
        resize(650, 500);
    }

private slots:
    void showNextImage()
    {
        if (imagePaths.isEmpty())
            return;

        int nextIndex = (currentIndex + 1) % imagePaths.size();
        animateImageTransition(nextIndex, true);
    }

    void showPreviousImage()
    {
        if (imagePaths.isEmpty())
            return;

        int prevIndex = (currentIndex - 1 + imagePaths.size()) % imagePaths.size();
        animateImageTransition(prevIndex, false);
    }

private:
    void loadImage(int index)
    {
        if (index >= 0 && index < imagePaths.size()) {
            QPixmap pixmap(imagePaths[index]);
            pixmap = pixmap.scaled(imageLabel->size(), Qt::KeepAspectRatio, Qt::SmoothTransformation);
            imageLabel->setPixmap(pixmap);
            currentIndex = index;
        }
    }

    void animateImageTransition(int newIndex, bool forward)
    {
        // Créer un nouveau label pour l'image suivante
        QLabel *newLabel = new QLabel(this);
        newLabel->setFixedSize(imageLabel->size());
        newLabel->setAlignment(Qt::AlignCenter);

        // Charger la nouvelle image
        QPixmap pixmap(imagePaths[newIndex]);
        pixmap = pixmap.scaled(newLabel->size(), Qt::KeepAspectRatio, Qt::SmoothTransformation);
        newLabel->setPixmap(pixmap);

        // Positionner le nouveau label en dehors de la vue
        newLabel->move(forward ? imageLabel->width() : -imageLabel->width(), 0);
        newLabel->show();

        // Créer les animations
        QPropertyAnimation *currentAnim = new QPropertyAnimation(imageLabel, "pos");
        currentAnim->setDuration(500);
        currentAnim->setStartValue(QPoint(0, 0));
        currentAnim->setEndValue(QPoint(forward ? -imageLabel->width() : imageLabel->width(), 0));
        currentAnim->setEasingCurve(QEasingCurve::InOutQuad);

        QPropertyAnimation *newAnim = new QPropertyAnimation(newLabel, "pos");
        newAnim->setDuration(500);
        newAnim->setStartValue(QPoint(forward ? imageLabel->width() : -imageLabel->width(), 0));
        newAnim->setEndValue(QPoint(0, 0));
        newAnim->setEasingCurve(QEasingCurve::InOutQuad);

        // Créer un groupe d'animations parallèles
        QParallelAnimationGroup *group = new QParallelAnimationGroup(this);
        group->addAnimation(currentAnim);
        group->addAnimation(newAnim);

        // Connecter le signal de fin pour nettoyer
        connect(group, &QParallelAnimationGroup::finished, [this, newLabel, newIndex]() {
            // Mettre à jour l'image actuelle
            loadImage(newIndex);

            // Supprimer le label temporaire
            newLabel->deleteLater();

            // Remettre l'ancien label à sa position d'origine
            imageLabel->move(0, 0);
        });

        // Démarrer l'animation
        group->start(QAbstractAnimation::DeleteWhenStopped);
    }

private:
    QLabel *imageLabel;
    QPushButton *prevButton;
    QPushButton *nextButton;
    QStringList imagePaths;
    int currentIndex;
};

int main(int argc, char *argv[])
{
    QApplication app(argc, argv);

    // Créer le carrousel avec le chemin vers le dossier d'images
    ImageCarousel carousel("./images");
    carousel.show();

    return app.exec();
}

#include "main.moc"
```

## Animations avec Qt Quick (QML)

Si vous utilisez Qt Quick (QML) pour construire votre interface utilisateur, vous disposez d'un système d'animation encore plus puissant et facile à utiliser.

Voici un exemple simple d'animation en QML :

```qml
import QtQuick 2.15
import QtQuick.Controls 2.15

ApplicationWindow {
    visible: true
    width: 400
    height: 400
    title: "Animations QML"

    Rectangle {
        id: myRectangle
        width: 100
        height: 100
        color: "red"

        // Animation de la propriété x
        Behavior on x {
            NumberAnimation { duration: 500; easing.type: Easing.OutBounce }
        }

        // Animation de la propriété y
        Behavior on y {
            NumberAnimation { duration: 500; easing.type: Easing.OutBounce }
        }
    }

    Button {
        anchors.bottom: parent.bottom
        anchors.horizontalCenter: parent.horizontalCenter
        text: "Animer"

        onClicked: {
            // Générer des positions aléatoires
            myRectangle.x = Math.random() * (parent.width - myRectangle.width)
            myRectangle.y = Math.random() * (parent.height - myRectangle.height - height - 10)
        }
    }
}
```

Ce code crée un rectangle rouge qui se déplace vers une position aléatoire avec un effet de rebond lorsque vous cliquez sur le bouton.

## Bonnes pratiques pour les animations

Pour créer des animations fluides et efficaces, suivez ces bonnes pratiques :

1. **Gardez les animations courtes** : Les animations ne devraient généralement pas durer plus de 300-500 ms.

2. **Utilisez les courbes d'accélération appropriées** : Les mouvements naturels ne sont pas linéaires. Utilisez des courbes comme `InOutQuad` pour des mouvements plus naturels.

3. **Évitez de surcharger l'interface** : Trop d'animations simultanées peuvent distraire et surcharger l'utilisateur.

4. **Testez les performances** : Les animations peuvent être gourmandes en ressources. Testez vos animations sur des appareils aux performances variées.

5. **Utilisez des groupes d'animations** : Ils facilitent la gestion des animations complexes.

6. **Considérez l'accessibilité** : Certains utilisateurs peuvent préférer réduire ou désactiver les animations. Proposez cette option si possible.

## Conclusion

Les animations et transitions sont un excellent moyen d'améliorer l'expérience utilisateur de vos applications Qt. Elles peuvent guider l'attention de l'utilisateur, fournir un retour visuel et rendre votre interface plus agréable à utiliser.

Qt6 offre un ensemble complet d'outils pour créer des animations simples ou complexes, que ce soit avec les widgets traditionnels ou avec Qt Quick.

Dans ce chapitre, nous avons exploré les bases des animations de propriétés, des groupes d'animations, des machines à états animées, et nous avons vu un exemple complet d'application utilisant des animations.

À mesure que vous progressez dans votre apprentissage de Qt, n'hésitez pas à expérimenter avec différentes combinaisons d'animations pour trouver le bon équilibre entre esthétique et fonctionnalité.

## Exercices pratiques

1. **Exercice de base** : Créez une application qui affiche un bouton qui change de couleur progressivement lorsqu'on passe la souris dessus.

2. **Exercice intermédiaire** : Créez une interface avec plusieurs boutons qui s'animent lorsqu'ils sont cliqués (par exemple, ils pourraient se déplacer, changer de taille ou de couleur).

3. **Exercice avancé** : Implémentez un système de menu animé qui utilise une machine à états pour gérer les transitions entre différentes vues de votre application.

## Ressources supplémentaires

- [Documentation Qt sur les animations](https://doc.qt.io/qt-6/animation-overview.html)
- [Documentation sur QPropertyAnimation](https://doc.qt.io/qt-6/qpropertyanimation.html)
- [Documentation sur QStateMachine](https://doc.qt.io/qt-6/qstatemachine.html)
- [Tutoriels sur les animations en QML](https://doc.qt.io/qt-6/qtquick-animation.html)

⏭️ [Multithreading et concurrence](/07-multithreading-et-concurrence)
