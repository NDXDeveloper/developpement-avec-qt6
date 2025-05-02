# 6. Multimédia et graphiques

## Introduction

Bienvenue dans ce chapitre consacré aux fonctionnalités multimédia et graphiques de Qt6 !

Qt6 offre un ensemble complet d'outils pour manipuler des éléments graphiques et multimédia dans vos applications. Que vous souhaitiez créer une interface utilisateur riche, intégrer des contenus multimédias ou créer des visualisations dynamiques, Qt6 propose des solutions adaptées à tous les besoins.

Dans ce chapitre, nous allons explorer les différentes possibilités offertes par Qt6 dans le domaine du multimédia et des graphiques. Nous commencerons par une vue d'ensemble avant d'entrer dans les détails de chaque module.

## Pourquoi utiliser les fonctionnalités multimédia et graphiques de Qt6 ?

L'intégration d'éléments multimédias et graphiques peut considérablement améliorer l'expérience utilisateur de votre application. Voici quelques raisons d'utiliser les fonctionnalités de Qt6 pour cela :

- **Multiplateforme** : Les fonctionnalités graphiques et multimédias de Qt6 fonctionnent de manière cohérente sur différentes plateformes.
- **API unifiée** : Qt6 propose une API cohérente et facile à utiliser pour toutes les fonctionnalités multimédias.
- **Performance** : Les modules graphiques de Qt6 sont optimisés pour offrir d'excellentes performances.
- **Intégration facile** : Les éléments multimédias s'intègrent harmonieusement avec les autres composants de Qt.

## Vue d'ensemble des modules multimédia et graphiques

Avant d'explorer chaque module en détail, voici un aperçu des principales fonctionnalités disponibles :

### Modules graphiques

| Module | Description |
|--------|-------------|
| Qt GUI | Module de base pour les éléments graphiques |
| Qt Widgets | Composants d'interface utilisateur traditionnels |
| Qt Quick | Framework déclaratif pour interfaces modernes |
| Qt SVG | Support des graphiques vectoriels |
| Qt Charts | Création de graphiques et diagrammes interactifs |
| Qt 3D | Rendu et animations 3D |

### Modules multimédia

| Module | Description |
|--------|-------------|
| Qt Multimedia | Lecture et capture audio/vidéo |
| Qt WebEngine | Intégration de contenu web |
| Qt Print Support | Impression de documents |

## Configuration requise

Avant de commencer à utiliser les fonctionnalités multimédia et graphiques de Qt6, assurez-vous que votre environnement de développement est correctement configuré :

1. **Qt6 installé** : Vérifiez que vous avez bien installé Qt6 avec les modules multimédia et graphiques.
2. **Dépendances** : Certains modules peuvent nécessiter des bibliothèques supplémentaires selon votre système d'exploitation.
3. **Qt Creator** : L'IDE Qt Creator simplifie grandement le développement avec ces modules.

```bash
# Sur Ubuntu/Debian, installation des dépendances nécessaires
sudo apt-get install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev

# Sur macOS avec Homebrew
brew install qt6
```

## Premier pas : Création d'une fenêtre graphique simple

Commençons par créer une fenêtre simple qui servira de base pour nos exemples multimédias :

```cpp
#include <QApplication>
#include <QMainWindow>
#include <QLabel>

int main(int argc, char *argv[])
{
    QApplication app(argc, argv);

    // Création de la fenêtre principale
    QMainWindow mainWindow;
    mainWindow.setWindowTitle("Ma première application multimédia");
    mainWindow.resize(800, 600);

    // Ajout d'un label
    QLabel *label = new QLabel("Bienvenue dans Qt6 Multimédia !");
    label->setAlignment(Qt::AlignCenter);
    mainWindow.setCentralWidget(label);

    // Affichage de la fenêtre
    mainWindow.show();

    return app.exec();
}
```

Ce code simple crée une fenêtre avec un texte centré. C'est le point de départ pour intégrer des éléments multimédias.

## Utilisation des ressources graphiques

Qt6 propose un système de ressources qui permet d'intégrer facilement des images et autres fichiers dans votre application.

### Étape 1 : Créer un fichier de ressources

Créez un fichier `.qrc` (par exemple `resources.qrc`) :

```xml
<!DOCTYPE RCC>
<RCC>
    <qresource prefix="/">
        <file>images/logo.png</file>
        <file>images/background.jpg</file>
    </qresource>
</RCC>
```

### Étape 2 : Ajouter le fichier au projet

Dans votre fichier `.pro` :

```
RESOURCES += resources.qrc
```

Ou dans CMakeLists.txt :

```cmake
qt_add_resources(MY_APP_RESOURCES resources.qrc)
```

### Étape 3 : Utiliser les ressources

```cpp
QPixmap pixmap(":/images/logo.png");
QLabel *imageLabel = new QLabel();
imageLabel->setPixmap(pixmap);
```

## Manipulation d'images basique

Qt6 offre plusieurs classes pour manipuler des images :

### Charger et afficher une image

```cpp
#include <QPixmap>
#include <QLabel>

// Charger une image depuis un fichier
QPixmap image("chemin/vers/image.jpg");

// Créer un label pour afficher l'image
QLabel *imageLabel = new QLabel();
imageLabel->setPixmap(image);
imageLabel->setScalingContents(true);  // Redimensionner l'image pour s'adapter au label
```

### Redimensionner une image

```cpp
// Redimensionner l'image à 200x150 pixels en conservant le ratio
QPixmap resizedImage = image.scaled(200, 150, Qt::KeepAspectRatio);
```

### Appliquer des effets simples

Pour appliquer des effets plus avancés, vous pouvez convertir votre QPixmap en QImage :

```cpp
#include <QImage>

QImage imgOriginal = image.toImage();

// Inverser les couleurs
QImage imgInverted = imgOriginal.invertPixels();

// Convertir en niveaux de gris
QImage imgGrayscale = imgOriginal.convertToFormat(QImage::Format_Grayscale8);
```

## Dessiner avec QPainter

La classe `QPainter` est le cœur du système de dessin 2D de Qt :

```cpp
#include <QPainter>
#include <QWidget>

class DrawingWidget : public QWidget
{
protected:
    void paintEvent(QPaintEvent *event) override
    {
        QPainter painter(this);

        // Définir un stylo pour les contours
        QPen pen(Qt::blue);
        pen.setWidth(2);
        painter.setPen(pen);

        // Définir une brosse pour le remplissage
        QBrush brush(Qt::green);
        painter.setBrush(brush);

        // Dessiner un rectangle
        painter.drawRect(50, 50, 200, 150);

        // Dessiner un cercle
        painter.drawEllipse(300, 100, 100, 100);

        // Dessiner du texte
        painter.setFont(QFont("Arial", 24));
        painter.drawText(100, 250, "Hello Qt Graphics!");
    }
};
```

## Animations simples

Qt6 permet de créer facilement des animations pour vos interfaces :

```cpp
#include <QPropertyAnimation>
#include <QLabel>

// Créer un label à animer
QLabel *animatedLabel = new QLabel("Animation");
animatedLabel->setStyleSheet("background-color: yellow; border-radius: 10px;");
animatedLabel->resize(100, 50);
layout->addWidget(animatedLabel);

// Créer une animation pour déplacer le label
QPropertyAnimation *animation = new QPropertyAnimation(animatedLabel, "geometry");
animation->setDuration(2000);  // 2 secondes
animation->setStartValue(QRect(0, 0, 100, 50));
animation->setEndValue(QRect(200, 100, 100, 50));
animation->setEasingCurve(QEasingCurve::OutBounce);  // effet de rebond

// Démarrer l'animation
animation->start();
```

## Conclusion et prochaines étapes

Dans cette introduction aux fonctionnalités multimédia et graphiques de Qt6, nous avons exploré les bases de la manipulation d'images et du dessin 2D. Ces connaissances constituent une base solide pour aborder les sujets plus avancés comme le traitement d'images, la lecture audio/vidéo, les graphiques vectoriels et les animations complexes.

Dans les prochaines sections, nous approfondirons chacun de ces sujets en détail et explorerons comment les intégrer efficacement dans vos applications.

## Exercices pratiques

Pour vous familiariser avec ces concepts, essayez ces petits exercices :

1. Créez une application qui affiche une image et permet de la redimensionner avec un curseur (slider).
2. Implémenter un simple éditeur de dessin où l'utilisateur peut tracer des formes basiques.
3. Créez une animation qui fait rebondir un objet à l'écran.

Bon développement avec Qt6 !
