# 6.3 Graphiques vectoriels avec Qt SVG

## Introduction aux graphiques vectoriels

Les graphiques vectoriels sont différents des images bitmap (comme les JPEG ou PNG) que nous avons vues précédemment. Au lieu de stocker l'information pixel par pixel, les graphiques vectoriels décrivent des formes mathématiques comme des lignes, des courbes et des polygones. Cela offre plusieurs avantages :

- **Mise à l'échelle sans perte de qualité** : Les graphiques vectoriels peuvent être agrandis à n'importe quelle taille sans devenir pixelisés
- **Taille de fichier souvent plus petite** : Particulièrement pour les images simples avec peu de détails
- **Édition facile** : Les éléments individuels peuvent être modifiés sans affecter le reste de l'image
- **Animations fluides** : Parfait pour les interfaces utilisateur interactives

Qt6 fournit un excellent support pour les graphiques vectoriels via le module Qt SVG, qui permet d'afficher, de créer et de manipuler des fichiers au format SVG (Scalable Vector Graphics).

## Configuration du projet pour Qt SVG

### Ajout du module à votre projet

Pour utiliser Qt SVG, vous devez d'abord configurer votre projet. Voici comment faire avec qmake et CMake.

#### Avec qmake

Ajoutez cette ligne à votre fichier .pro :

```
QT += svg
```

#### Avec CMake

Si vous utilisez CMake, ajoutez ces lignes à votre fichier CMakeLists.txt :

```cmake
find_package(Qt6 REQUIRED COMPONENTS Svg)
target_link_libraries(your_app_name PRIVATE Qt6::Svg)
```

### Inclure les en-têtes nécessaires

Dans vos fichiers source, incluez les en-têtes suivants selon vos besoins :

```cpp
#include <QSvgWidget>       // Pour afficher un fichier SVG dans un widget
#include <QSvgRenderer>     // Pour rendre un SVG sur n'importe quel périphérique de peinture (QPainter)
#include <QSvgGenerator>    // Pour créer des fichiers SVG
```

## Affichage de fichiers SVG

Qt offre plusieurs façons d'afficher des fichiers SVG dans votre application.

### Utilisation de QSvgWidget

La façon la plus simple d'afficher un fichier SVG est d'utiliser la classe `QSvgWidget` :

```cpp
#include <QApplication>
#include <QSvgWidget>

int main(int argc, char *argv[])
{
    QApplication app(argc, argv);

    // Création d'un widget SVG
    QSvgWidget svgWidget("chemin/vers/votre/fichier.svg");

    // Définir une taille pour le widget
    svgWidget.resize(400, 300);

    // Afficher le widget
    svgWidget.show();

    return app.exec();
}
```

### Utilisation de QSvgRenderer avec QPainter

Si vous avez besoin de plus de contrôle sur le rendu, ou si vous souhaitez intégrer un SVG dans une interface existante, vous pouvez utiliser `QSvgRenderer` avec `QPainter` :

```cpp
#include <QWidget>
#include <QPainter>
#include <QSvgRenderer>

class SvgView : public QWidget
{
public:
    SvgView(const QString &svgPath, QWidget *parent = nullptr)
        : QWidget(parent)
    {
        // Charger le fichier SVG
        renderer.load(svgPath);
    }

protected:
    void paintEvent(QPaintEvent *event) override
    {
        QPainter painter(this);

        // Fond blanc
        painter.fillRect(rect(), Qt::white);

        // Rendre le SVG dans le widget
        renderer.render(&painter, rect());
    }

private:
    QSvgRenderer renderer;
};
```

### Chargement d'un SVG depuis d'autres sources

Vous pouvez également charger un SVG depuis diverses sources, pas seulement depuis un fichier :

```cpp
// Depuis un fichier
renderer.load("chemin/vers/fichier.svg");

// Depuis une ressource Qt
renderer.load(":/images/icone.svg");

// Depuis une chaîne de caractères contenant du SVG
QString svgContent = "<svg width='100' height='100'><circle cx='50' cy='50' r='40' fill='red'/></svg>";
QByteArray svgData = svgContent.toUtf8();
renderer.load(svgData);
```

## Manipulation des éléments SVG

### Accès aux éléments via leur ID

Si votre fichier SVG contient des éléments avec des ID, vous pouvez les manipuler individuellement :

```cpp
// Vérifier si un élément existe
bool hasElement = renderer.elementExists("circleID");

// Obtenir le rectangle englobant d'un élément
QRectF bounds = renderer.boundsOnElement("circleID");

// Rendre uniquement un élément spécifique
renderer.render(&painter, "circleID", targetRect);
```

Voici un exemple de fichier SVG avec des ID :

```xml
<svg width="200" height="200" xmlns="http://www.w3.org/2000/svg">
    <rect id="rectangle" x="10" y="10" width="100" height="80" fill="blue" />
    <circle id="cercle" cx="150" cy="50" r="30" fill="red" />
    <path id="chemin" d="M10,150 L100,120 L180,150" stroke="green" stroke-width="3" fill="none" />
</svg>
```

## Création de fichiers SVG

Qt vous permet également de créer des fichiers SVG à l'aide de `QSvgGenerator`. Cela fonctionne comme un périphérique de peinture standard avec `QPainter` :

```cpp
#include <QSvgGenerator>
#include <QPainter>

void createSvgFile()
{
    // Configuration du générateur SVG
    QSvgGenerator generator;
    generator.setFileName("dessin.svg");
    generator.setSize(QSize(200, 200));
    generator.setViewBox(QRect(0, 0, 200, 200));
    generator.setTitle("Mon premier SVG avec Qt");
    generator.setDescription("Un exemple de génération SVG avec Qt6");

    // Création d'un peintre pour dessiner sur le générateur
    QPainter painter;
    painter.begin(&generator);

    // Dessiner un fond
    painter.fillRect(0, 0, 200, 200, Qt::white);

    // Dessiner un rectangle
    painter.setPen(QPen(Qt::blue, 2));
    painter.setBrush(QBrush(Qt::green));
    painter.drawRect(20, 20, 160, 80);

    // Dessiner un cercle
    painter.setPen(QPen(Qt::red, 2));
    painter.setBrush(QBrush(Qt::yellow));
    painter.drawEllipse(100, 120, 60, 60);

    // Dessiner du texte
    painter.setPen(Qt::black);
    painter.setFont(QFont("Arial", 12));
    painter.drawText(40, 180, "SVG généré avec Qt!");

    // Terminer la peinture
    painter.end();
}
```

## Animation de SVG

Vous pouvez créer des animations en modifiant dynamiquement les propriétés d'un SVG ou en utilisant les transformations Qt.

### Animation par mise à jour périodique

```cpp
#include <QWidget>
#include <QPainter>
#include <QSvgRenderer>
#include <QTimer>

class AnimatedSvgView : public QWidget
{
    Q_OBJECT

public:
    AnimatedSvgView(const QString &svgPath, QWidget *parent = nullptr)
        : QWidget(parent), rotationAngle(0)
    {
        // Charger le fichier SVG
        renderer.load(svgPath);

        // Configurer un timer pour l'animation
        QTimer *timer = new QTimer(this);
        connect(timer, &QTimer::timeout, this, &AnimatedSvgView::updateAnimation);
        timer->start(30); // 30ms = environ 33 FPS
    }

protected:
    void paintEvent(QPaintEvent *event) override
    {
        QPainter painter(this);

        // Fond blanc
        painter.fillRect(rect(), Qt::white);

        // Sauvegarder l'état du peintre
        painter.save();

        // Déplacer l'origine au centre du widget
        painter.translate(width() / 2, height() / 2);

        // Appliquer la rotation
        painter.rotate(rotationAngle);

        // Déplacer l'origine de retour pour centrer le SVG
        painter.translate(-width() / 4, -height() / 4);

        // Rendre le SVG dans une zone de taille réduite
        renderer.render(&painter, QRectF(0, 0, width() / 2, height() / 2));

        // Restaurer l'état du peintre
        painter.restore();
    }

private slots:
    void updateAnimation()
    {
        // Incrémenter l'angle de rotation
        rotationAngle = (rotationAngle + 2) % 360;

        // Déclencher un repeint
        update();
    }

private:
    QSvgRenderer renderer;
    int rotationAngle;
};
```

### Animation avec QPropertyAnimation

Une autre approche consiste à utiliser `QPropertyAnimation` pour animer des propriétés :

```cpp
#include <QWidget>
#include <QPainter>
#include <QSvgRenderer>
#include <QPropertyAnimation>

class AnimatedSvgView : public QWidget
{
    Q_OBJECT
    Q_PROPERTY(qreal scale READ scale WRITE setScale)

public:
    AnimatedSvgView(const QString &svgPath, QWidget *parent = nullptr)
        : QWidget(parent), m_scale(1.0)
    {
        // Charger le fichier SVG
        renderer.load(svgPath);

        // Créer une animation de mise à l'échelle
        QPropertyAnimation *animation = new QPropertyAnimation(this, "scale");
        animation->setDuration(2000);         // 2 secondes
        animation->setStartValue(0.5);        // Commencer à 50%
        animation->setEndValue(2.0);          // Terminer à 200%
        animation->setEasingCurve(QEasingCurve::InOutQuad);
        animation->setLoopCount(-1);          // Répéter infiniment
        animation->start();
    }

    qreal scale() const { return m_scale; }

    void setScale(qreal scale)
    {
        m_scale = scale;
        update(); // Déclencher un repeint
    }

protected:
    void paintEvent(QPaintEvent *event) override
    {
        QPainter painter(this);

        // Fond blanc
        painter.fillRect(rect(), Qt::white);

        // Sauvegarder l'état du peintre
        painter.save();

        // Déplacer l'origine au centre du widget
        painter.translate(width() / 2, height() / 2);

        // Appliquer la mise à l'échelle
        painter.scale(m_scale, m_scale);

        // Déplacer l'origine de retour pour centrer le SVG
        painter.translate(-width() / 2, -height() / 2);

        // Rendre le SVG
        renderer.render(&painter, rect());

        // Restaurer l'état du peintre
        painter.restore();
    }

private:
    QSvgRenderer renderer;
    qreal m_scale;
};
```

## Créer des interfaces avec des icônes SVG

Les SVG sont parfaits pour les icônes dans vos applications, car ils s'adaptent bien à différentes tailles d'écran et résolutions.

### Utilisation de SVG dans QIcon

```cpp
// Créer une icône à partir d'un fichier SVG
QIcon icon(":/icons/document.svg");

// Utiliser l'icône dans un bouton
QPushButton *button = new QPushButton(icon, "Nouveau document");
```

### Thèmes et coloration des SVG

Vous pouvez changer dynamiquement les couleurs des SVG en utilisant les propriétés CSS :

```xml
<!-- icon.svg avec une classe pour la couleur -->
<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg">
    <circle class="fill-color" cx="50" cy="50" r="40" fill="currentColor"/>
</svg>
```

```cpp
// Charger le SVG depuis une chaîne de texte et modifier la couleur
QString svgTemplate = "<svg width='100' height='100' xmlns='http://www.w3.org/2000/svg'>"
                      "<circle cx='50' cy='50' r='40' fill='%1'/>"
                      "</svg>";

// Créer des variantes avec différentes couleurs
QString redSvg = svgTemplate.arg("red");
QString blueSvg = svgTemplate.arg("blue");

// Charger les variantes
QSvgRenderer redRenderer(redSvg.toUtf8());
QSvgRenderer blueRenderer(blueSvg.toUtf8());
```

## Exemple complet : Un visualiseur SVG simple

Voici un exemple complet d'une application qui permet de charger et d'afficher des fichiers SVG, avec des options de mise à l'échelle et de rotation :

```cpp
#include <QApplication>
#include <QMainWindow>
#include <QSvgRenderer>
#include <QPainter>
#include <QFileDialog>
#include <QSlider>
#include <QLabel>
#include <QToolBar>
#include <QVBoxLayout>
#include <QHBoxLayout>
#include <QAction>

class SvgViewer : public QWidget
{
    Q_OBJECT
public:
    SvgViewer(QWidget *parent = nullptr) : QWidget(parent), scale(1.0), rotation(0)
    {
        // Définir une taille minimale
        setMinimumSize(300, 300);
    }

    bool loadSvg(const QString &path)
    {
        bool success = renderer.load(path);
        if (success) {
            update(); // Déclencher un repeint
        }
        return success;
    }

    void setScale(double newScale)
    {
        scale = newScale;
        update();
    }

    void setRotation(int degrees)
    {
        rotation = degrees;
        update();
    }

protected:
    void paintEvent(QPaintEvent *event) override
    {
        QPainter painter(this);

        // Fond blanc
        painter.fillRect(rect(), Qt::white);

        if (renderer.isValid()) {
            // Sauvegarder l'état du peintre
            painter.save();

            // Déplacer l'origine au centre
            painter.translate(width() / 2, height() / 2);

            // Appliquer la rotation
            painter.rotate(rotation);

            // Appliquer la mise à l'échelle
            painter.scale(scale, scale);

            // Calculer la taille du SVG à sa vraie échelle
            QSize svgSize = renderer.defaultSize();
            QRect svgRect(-svgSize.width() / 2, -svgSize.height() / 2,
                           svgSize.width(), svgSize.height());

            // Rendre le SVG
            renderer.render(&painter, svgRect);

            // Restaurer l'état du peintre
            painter.restore();
        } else {
            // Aucun SVG chargé, afficher un message
            painter.setPen(Qt::black);
            painter.setFont(QFont("Arial", 12));
            painter.drawText(rect(), Qt::AlignCenter, "Aucun fichier SVG chargé.\nUtilisez Fichier > Ouvrir pour charger un SVG.");
        }
    }

private:
    QSvgRenderer renderer;
    double scale;
    int rotation;
};

class MainWindow : public QMainWindow
{
    Q_OBJECT
public:
    MainWindow(QWidget *parent = nullptr) : QMainWindow(parent)
    {
        // Créer le visualiseur SVG
        svgViewer = new SvgViewer(this);
        setCentralWidget(svgViewer);

        // Créer les contrôles
        createControls();

        // Créer le menu
        createMenu();

        setWindowTitle("Visualiseur SVG Qt");
        resize(800, 600);
    }

private slots:
    void openSvgFile()
    {
        QString fileName = QFileDialog::getOpenFileName(this,
                                                       "Ouvrir un fichier SVG",
                                                       QString(),
                                                       "Fichiers SVG (*.svg)");
        if (!fileName.isEmpty()) {
            if (!svgViewer->loadSvg(fileName)) {
                // Erreur lors du chargement
                QMessageBox::warning(this, "Erreur", "Impossible de charger le fichier SVG.");
            }
        }
    }

private:
    void createControls()
    {
        // Créer une barre d'outils pour les contrôles
        QToolBar *controlBar = new QToolBar("Contrôles", this);
        addToolBar(Qt::BottomToolBarArea, controlBar);

        // Widget pour contenir les sliders
        QWidget *sliderWidget = new QWidget(this);
        QVBoxLayout *sliderLayout = new QVBoxLayout(sliderWidget);

        // Contrôle de mise à l'échelle
        QHBoxLayout *scaleLayout = new QHBoxLayout();
        QLabel *scaleLabel = new QLabel("Échelle:", sliderWidget);
        QSlider *scaleSlider = new QSlider(Qt::Horizontal, sliderWidget);
        scaleSlider->setRange(10, 300);  // 10% à 300%
        scaleSlider->setValue(100);      // 100% par défaut
        scaleLayout->addWidget(scaleLabel);
        scaleLayout->addWidget(scaleSlider);

        // Contrôle de rotation
        QHBoxLayout *rotateLayout = new QHBoxLayout();
        QLabel *rotateLabel = new QLabel("Rotation:", sliderWidget);
        QSlider *rotateSlider = new QSlider(Qt::Horizontal, sliderWidget);
        rotateSlider->setRange(0, 359);  // 0 à 359 degrés
        rotateSlider->setValue(0);       // 0 degré par défaut
        rotateLayout->addWidget(rotateLabel);
        rotateLayout->addWidget(rotateSlider);

        // Ajouter les layouts au layout principal
        sliderLayout->addLayout(scaleLayout);
        sliderLayout->addLayout(rotateLayout);

        // Ajouter le widget à la barre d'outils
        controlBar->addWidget(sliderWidget);

        // Connecter les signaux/slots
        connect(scaleSlider, &QSlider::valueChanged, [this](int value) {
            svgViewer->setScale(value / 100.0);
        });

        connect(rotateSlider, &QSlider::valueChanged, svgViewer, &SvgViewer::setRotation);
    }

    void createMenu()
    {
        QMenu *fileMenu = menuBar()->addMenu("&Fichier");

        QAction *openAction = new QAction("&Ouvrir", this);
        openAction->setShortcut(QKeySequence::Open);
        connect(openAction, &QAction::triggered, this, &MainWindow::openSvgFile);
        fileMenu->addAction(openAction);

        fileMenu->addSeparator();

        QAction *exitAction = new QAction("&Quitter", this);
        exitAction->setShortcut(QKeySequence::Quit);
        connect(exitAction, &QAction::triggered, this, &QMainWindow::close);
        fileMenu->addAction(exitAction);
    }

    SvgViewer *svgViewer;
};

int main(int argc, char *argv[])
{
    QApplication app(argc, argv);

    MainWindow window;
    window.show();

    return app.exec();
}

#include "main.moc"
```

## Utilisation pratique : Création d'un diagramme simple

Voici comment créer un diagramme simple en utilisant Qt SVG :

```cpp
void createFlowchartSvg()
{
    // Configuration du générateur SVG
    QSvgGenerator generator;
    generator.setFileName("diagramme.svg");
    generator.setSize(QSize(500, 300));
    generator.setViewBox(QRect(0, 0, 500, 300));
    generator.setTitle("Diagramme de flux");

    // Création d'un peintre pour dessiner sur le générateur
    QPainter painter;
    painter.begin(&generator);

    // Fond blanc
    painter.fillRect(0, 0, 500, 300, Qt::white);

    // Configuration du stylo et de la brosse
    QPen pen(Qt::black, 2);
    painter.setPen(pen);

    // Dessiner des boîtes
    QRectF startBox(50, 50, 100, 50);
    QRectF processBox(200, 50, 100, 50);
    QRectF decisionBox(200, 150, 100, 80);
    QRectF endBox(350, 50, 100, 50);

    // Remplir les boîtes avec différentes couleurs
    painter.setBrush(QColor(200, 255, 200));  // Vert clair
    painter.drawRect(startBox);

    painter.setBrush(QColor(200, 200, 255));  // Bleu clair
    painter.drawRect(processBox);

    painter.setBrush(QColor(255, 255, 200));  // Jaune clair
    painter.drawRect(endBox);

    painter.setBrush(QColor(255, 200, 200));  // Rouge clair
    painter.drawPolygon(QPolygonF({
        QPointF(200, 190),                 // Gauche
        QPointF(250, 150),                 // Haut
        QPointF(300, 190),                 // Droite
        QPointF(250, 230)                  // Bas
    }));

    // Dessiner des flèches
    painter.drawLine(150, 75, 200, 75);        // Start -> Process
    painter.drawLine(300, 75, 350, 75);        // Process -> End
    painter.drawLine(250, 100, 250, 150);      // Process -> Decision

    // Ajouter des pointes de flèche
    QPolygonF arrowHead1;
    arrowHead1 << QPointF(195, 75) << QPointF(185, 70) << QPointF(185, 80);
    painter.setBrush(Qt::black);
    painter.drawPolygon(arrowHead1);

    QPolygonF arrowHead2;
    arrowHead2 << QPointF(345, 75) << QPointF(335, 70) << QPointF(335, 80);
    painter.drawPolygon(arrowHead2);

    QPolygonF arrowHead3;
    arrowHead3 << QPointF(250, 145) << QPointF(245, 135) << QPointF(255, 135);
    painter.drawPolygon(arrowHead3);

    // Ajouter du texte
    painter.setFont(QFont("Arial", 10));
    painter.drawText(startBox, Qt::AlignCenter, "Début");
    painter.drawText(processBox, Qt::AlignCenter, "Processus");
    painter.drawText(QRectF(200, 170, 100, 40), Qt::AlignCenter, "Décision");
    painter.drawText(endBox, Qt::AlignCenter, "Fin");

    // Terminer la peinture
    painter.end();
}
```

## Bonnes pratiques pour l'utilisation de SVG

### Optimisation des SVG

Pour des performances optimales :

1. **Simplifiez vos SVG** : Réduisez le nombre de nœuds et de points
2. **Utilisez des ID** : Nommez les éléments que vous souhaitez manipuler
3. **Évitez les filtres complexes** : Certains effets SVG peuvent être coûteux à rendre
4. **Préférez les formes simples** : Utilisez des rectangles, cercles et polygones plutôt que des chemins complexes quand c'est possible

### Intégration avec les styles Qt

Vous pouvez adapter vos SVG au thème de votre application :

```cpp
// Obtenir une couleur du thème actuel
QColor themeColor = palette().color(QPalette::Highlight);

// Convertir en chaîne hexadécimale
QString colorString = themeColor.name();

// Remplacer la couleur dans le SVG
QString svgContent = originalSvgContent;
svgContent.replace("fill=\"#0000FF\"", "fill=\"" + colorString + "\"");

// Charger le SVG modifié
QSvgRenderer renderer(svgContent.toUtf8());
```

## Conclusion

Qt SVG est un module puissant qui vous permet d'intégrer facilement des graphiques vectoriels dans vos applications Qt. Les avantages des SVG, comme la mise à l'échelle sans perte de qualité et la petite taille des fichiers, en font un excellent choix pour les icônes, les diagrammes et les interfaces utilisateur modernes.

Dans ce chapitre, nous avons exploré les bases de l'affichage et de la création de SVG avec Qt, ainsi que des techniques plus avancées comme l'animation et la manipulation d'éléments individuels.

À mesure que vous progressez dans votre apprentissage, n'hésitez pas à explorer les nombreuses possibilités offertes par SVG pour améliorer l'apparence et l'expérience utilisateur de vos applications Qt.

## Exercices pratiques

1. **Exercice de base** : Créez une application qui affiche un fichier SVG et permet à l'utilisateur de zoomer et dézoomer.

2. **Exercice intermédiaire** : Modifiez l'exemple du visualiseur SVG pour ajouter la possibilité de changer les couleurs des éléments du SVG.

3. **Exercice avancé** : Créez une petite application de diagramme où l'utilisateur peut ajouter et connecter des formes simples, puis exporter le résultat en SVG.

## Ressources supplémentaires

- [Documentation Qt SVG](https://doc.qt.io/qt-6/qsvg-overview.html)
- [Tutoriels SVG sur MDN](https://developer.mozilla.org/fr/docs/Web/SVG/Tutorial)
- [Éditeur SVG en ligne Inkscape](https://inkscape.org/) (pour créer des SVG)
- [SVG Optimiser](https://github.com/svg/svgo) (pour optimiser la taille des fichiers SVG)
