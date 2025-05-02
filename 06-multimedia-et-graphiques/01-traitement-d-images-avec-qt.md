# 6.1 Traitement d'images avec Qt

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

## Introduction au traitement d'images

Le traitement d'images est une fonctionnalité essentielle pour de nombreuses applications modernes, qu'il s'agisse de retoucher des photos, d'analyser des données visuelles ou simplement d'afficher des images dans votre interface. Qt6 offre un ensemble d'outils puissants mais accessibles pour manipuler les images.

Dans cette section, nous allons découvrir comment:
- Charger et sauvegarder des images dans différents formats
- Manipuler les pixels et appliquer des transformations
- Créer des filtres simples
- Gérer efficacement la mémoire lors du traitement d'images

## Les classes principales pour le traitement d'images

Qt6 propose trois classes principales pour travailler avec les images:

| Classe   | Description                                                     | Utilisation typique                               |
|----------|-----------------------------------------------------------------|---------------------------------------------------|
| QImage   | Manipulation des pixels et formats d'image indépendants         | Traitement d'image, accès aux pixels              |
| QPixmap  | Optimisée pour l'affichage et le rendu à l'écran               | Affichage d'images dans l'interface utilisateur   |
| QPicture | Format vectoriel pour enregistrer et rejouer des commandes QPainter | Sauvegarde d'opérations de dessin complexes      |

Commençons par comprendre quand utiliser chaque classe:

```cpp
// Quand utiliser QImage:
QImage image(1024, 768, QImage::Format_RGB32);  // Pour créer une nouvelle image
image.setPixelColor(10, 20, QColor(255, 0, 0));  // Pour accéder aux pixels

// Quand utiliser QPixmap:
QPixmap pixmap(":/images/logo.png");  // Pour charger et afficher une image
label->setPixmap(pixmap);  // Parfait pour l'affichage dans un QLabel

// Quand utiliser QPicture:
QPicture picture;
QPainter painter(&picture);
painter.drawRect(10, 20, 80, 60);  // Enregistre les commandes de dessin
painter.end();
picture.save("dessin.pic");  // Format spécifique à Qt
```

## Chargement et sauvegarde d'images

### Formats supportés

Qt6 prend en charge de nombreux formats d'image par défaut:
- PNG (*.png)
- JPEG (*.jpg, *.jpeg)
- BMP (*.bmp)
- GIF (*.gif)
- PBM/PGM/PPM (*.pbm, *.pgm, *.ppm)
- XBM (*.xbm)
- XPM (*.xpm)

Pour voir tous les formats supportés sur votre système:

```cpp
QStringList formats = QImageReader::supportedImageFormats();
for (const auto &format : formats) {
    qDebug() << "Format supporté:" << format;
}
```

### Chargement d'une image

```cpp
// Méthode 1: Directement dans le constructeur
QImage image("chemin/vers/image.jpg");
if (image.isNull()) {
    qDebug() << "Erreur: Impossible de charger l'image";
}

// Méthode 2: Avec load()
QImage image;
bool success = image.load("chemin/vers/image.jpg");
if (!success) {
    qDebug() << "Erreur: Impossible de charger l'image";
}

// Méthode 3: Depuis une ressource Qt
QImage image(":/images/logo.png");
```

### Sauvegarde d'une image

```cpp
QImage image(300, 200, QImage::Format_RGB32);
image.fill(Qt::white);  // Remplir avec du blanc

// Dessiner quelque chose sur l'image
QPainter painter(&image);
painter.setPen(Qt::blue);
painter.drawText(20, 30, "Hello Qt Image!");
painter.end();

// Sauvegarder l'image
bool success = image.save("nouvelle_image.png", "PNG");
if (!success) {
    qDebug() << "Erreur: Impossible de sauvegarder l'image";
}
```

## Formats d'image et profondeur de couleur

Qt6 supporte différents formats pour représenter les pixels d'une image:

```cpp
// Créer une image en niveaux de gris (8 bits)
QImage grayImage(width, height, QImage::Format_Grayscale8);

// Créer une image RGB (24 bits)
QImage rgbImage(width, height, QImage::Format_RGB888);

// Créer une image RGBA avec canal alpha (32 bits)
QImage rgbaImage(width, height, QImage::Format_RGBA8888);
```

### Conversion entre formats

```cpp
// Convertir vers un autre format
QImage rgbaImage = rgbImage.convertToFormat(QImage::Format_RGBA8888);

// Conversion en niveaux de gris
QImage grayImage = rgbImage.convertToFormat(QImage::Format_Grayscale8);
```

## Manipulation des pixels

L'accès aux pixels est fondamental pour le traitement d'images. Qt6 offre plusieurs méthodes pour lire et modifier les pixels.

### Accès aux pixels individuels

```cpp
QImage image(100, 100, QImage::Format_RGB32);

// Obtenir la couleur d'un pixel
QColor pixelColor = image.pixelColor(10, 20);
int red = pixelColor.red();
int green = pixelColor.green();
int blue = pixelColor.blue();

// Modifier la couleur d'un pixel
image.setPixelColor(10, 20, QColor(255, 0, 0));  // Rouge
```

### Parcourir tous les pixels

```cpp
QImage image("photo.jpg");

// Méthode 1: Avec setPixelColor (plus simple mais moins efficace)
for (int y = 0; y < image.height(); ++y) {
    for (int x = 0; x < image.width(); ++x) {
        QColor color = image.pixelColor(x, y);
        // Modifier la couleur (exemple: inverser)
        image.setPixelColor(x, y, QColor(255 - color.red(),
                                         255 - color.green(),
                                         255 - color.blue()));
    }
}

// Méthode 2: Avec scanLine (plus efficace mais plus complexe)
if (image.format() == QImage::Format_RGB32) {
    for (int y = 0; y < image.height(); ++y) {
        QRgb *line = reinterpret_cast<QRgb*>(image.scanLine(y));
        for (int x = 0; x < image.width(); ++x) {
            // Obtenir les composantes de couleur
            int red = qRed(line[x]);
            int green = qGreen(line[x]);
            int blue = qBlue(line[x]);

            // Modifier la couleur (exemple: inverser)
            line[x] = qRgb(255 - red, 255 - green, 255 - blue);
        }
    }
}
```

## Transformations d'images

### Redimensionnement

```cpp
QImage originalImage("grande_image.jpg");

// Redimensionner à une taille spécifique
QImage resized = originalImage.scaled(800, 600);

// Redimensionner en conservant le ratio
QImage resizedRatio = originalImage.scaled(800, 600, Qt::KeepAspectRatio);

// Redimensionner en ignorant le ratio
QImage resizedIgnoreRatio = originalImage.scaled(800, 600, Qt::IgnoreAspectRatio);
```

### Rotation et retournement

```cpp
QImage originalImage("image.jpg");

// Rotation de 90 degrés
QImage rotated90 = originalImage.transformed(QTransform().rotate(90));

// Rotation de 180 degrés
QImage rotated180 = originalImage.transformed(QTransform().rotate(180));

// Miroir horizontal
QImage mirrored = originalImage.mirrored(true, false);

// Miroir vertical
QImage flipped = originalImage.mirrored(false, true);
```

### Rognage (crop)

```cpp
QImage originalImage("image.jpg");

// Rogner une partie de l'image (x, y, largeur, hauteur)
QImage cropped = originalImage.copy(100, 50, 400, 300);
```

## Filtres et effets simples

Voici quelques exemples de filtres simples que vous pouvez appliquer à vos images:

### Effet niveau de gris

```cpp
QImage applyGrayscale(const QImage &image)
{
    QImage result = image.copy();
    for (int y = 0; y < result.height(); ++y) {
        for (int x = 0; x < result.width(); ++x) {
            QColor color = result.pixelColor(x, y);
            // Formule standard pour convertir en niveaux de gris
            int gray = qRound(0.299 * color.red() +
                              0.587 * color.green() +
                              0.114 * color.blue());
            result.setPixelColor(x, y, QColor(gray, gray, gray));
        }
    }
    return result;
}
```

### Effet sépia

```cpp
QImage applySepiaEffect(const QImage &image)
{
    QImage result = image.copy();
    for (int y = 0; y < result.height(); ++y) {
        for (int x = 0; x < result.width(); ++x) {
            QColor oldColor = result.pixelColor(x, y);

            // Convertir en sépia
            int r = oldColor.red();
            int g = oldColor.green();
            int b = oldColor.blue();

            int newRed = qMin(255, qRound(r * 0.393 + g * 0.769 + b * 0.189));
            int newGreen = qMin(255, qRound(r * 0.349 + g * 0.686 + b * 0.168));
            int newBlue = qMin(255, qRound(r * 0.272 + g * 0.534 + b * 0.131));

            result.setPixelColor(x, y, QColor(newRed, newGreen, newBlue));
        }
    }
    return result;
}
```

### Réglage de la luminosité

```cpp
QImage adjustBrightness(const QImage &image, int factor)
{
    // factor: -255 (noir) à 255 (blanc)
    QImage result = image.copy();
    for (int y = 0; y < result.height(); ++y) {
        for (int x = 0; x < result.width(); ++x) {
            QColor color = result.pixelColor(x, y);

            int r = qBound(0, color.red() + factor, 255);
            int g = qBound(0, color.green() + factor, 255);
            int b = qBound(0, color.blue() + factor, 255);

            result.setPixelColor(x, y, QColor(r, g, b));
        }
    }
    return result;
}
```

## Optimisation et performance

Le traitement d'images peut être gourmand en ressources. Voici quelques astuces pour optimiser vos applications:

### Utiliser la bonne classe

- `QImage` pour la manipulation de pixels
- `QPixmap` pour l'affichage à l'écran
- Convertir entre les formats seulement quand nécessaire

### Traitement par lots

```cpp
// Déconseillé: Verrouiller/déverrouiller pour chaque pixel
for (int y = 0; y < image.height(); ++y) {
    for (int x = 0; x < image.width(); ++x) {
        image.setPixelColor(x, y, newColor);  // Lent
    }
}

// Recommandé: Utiliser scanLine pour un accès direct
if (image.format() == QImage::Format_RGB32) {
    for (int y = 0; y < image.height(); ++y) {
        QRgb *line = reinterpret_cast<QRgb*>(image.scanLine(y));
        for (int x = 0; x < image.width(); ++x) {
            line[x] = newColorValue;  // Beaucoup plus rapide
        }
    }
}
```

### Traitement multithreading

Pour les opérations lourdes, vous pouvez diviser l'image en sections et les traiter en parallèle avec QtConcurrent:

```cpp
#include <QtConcurrent>

void processImageRegion(QImage &image, QRect region, std::function<QColor(QColor)> processor)
{
    for (int y = region.top(); y <= region.bottom(); ++y) {
        for (int x = region.left(); x <= region.right(); ++x) {
            QColor oldColor = image.pixelColor(x, y);
            image.setPixelColor(x, y, processor(oldColor));
        }
    }
}

QImage processImageMultithreaded(const QImage &sourceImage, std::function<QColor(QColor)> processor)
{
    QImage result = sourceImage.copy();

    // Diviser l'image en 4 régions (ou plus selon votre CPU)
    int halfHeight = result.height() / 2;
    int halfWidth = result.width() / 2;

    QRect topLeft(0, 0, halfWidth, halfHeight);
    QRect topRight(halfWidth, 0, halfWidth, halfHeight);
    QRect bottomLeft(0, halfHeight, halfWidth, halfHeight);
    QRect bottomRight(halfWidth, halfHeight, halfWidth, halfHeight);

    // Traiter chaque région dans un thread séparé
    QFuture<void> f1 = QtConcurrent::run([&result, topLeft, processor]() {
        processImageRegion(result, topLeft, processor);
    });
    QFuture<void> f2 = QtConcurrent::run([&result, topRight, processor]() {
        processImageRegion(result, topRight, processor);
    });
    QFuture<void> f3 = QtConcurrent::run([&result, bottomLeft, processor]() {
        processImageRegion(result, bottomLeft, processor);
    });
    QFuture<void> f4 = QtConcurrent::run([&result, bottomRight, processor]() {
        processImageRegion(result, bottomRight, processor);
    });

    // Attendre que tous les threads terminent
    f1.waitForFinished();
    f2.waitForFinished();
    f3.waitForFinished();
    f4.waitForFinished();

    return result;
}

// Utilisation:
QImage result = processImageMultithreaded(sourceImage, [](QColor color) {
    // Exemple: convertir en niveau de gris
    int gray = qRound(0.299 * color.red() + 0.587 * color.green() + 0.114 * color.blue());
    return QColor(gray, gray, gray);
});
```

## Exemple complet: Mini-éditeur d'images

Voici un exemple simple d'une classe qui pourrait servir de base à un mini-éditeur d'images:

```cpp
#ifndef IMAGEEDITOR_H
#define IMAGEEDITOR_H

#include <QWidget>
#include <QImage>
#include <QLabel>
#include <QPushButton>
#include <QSlider>
#include <QVBoxLayout>
#include <QHBoxLayout>
#include <QFileDialog>

class ImageEditor : public QWidget
{
    Q_OBJECT

public:
    ImageEditor(QWidget *parent = nullptr) : QWidget(parent)
    {
        // Initialisation de l'interface
        imageLabel = new QLabel("Chargez une image");
        imageLabel->setAlignment(Qt::AlignCenter);
        imageLabel->setMinimumSize(400, 300);

        // Boutons pour les opérations
        loadButton = new QPushButton("Charger");
        saveButton = new QPushButton("Sauvegarder");
        grayscaleButton = new QPushButton("Niveaux de gris");
        sepiaButton = new QPushButton("Sépia");

        // Slider pour la luminosité
        brightnessSlider = new QSlider(Qt::Horizontal);
        brightnessSlider->setRange(-100, 100);
        brightnessSlider->setValue(0);

        // Disposition des widgets
        QVBoxLayout *mainLayout = new QVBoxLayout;
        QHBoxLayout *buttonLayout = new QHBoxLayout;

        buttonLayout->addWidget(loadButton);
        buttonLayout->addWidget(saveButton);
        buttonLayout->addWidget(grayscaleButton);
        buttonLayout->addWidget(sepiaButton);

        mainLayout->addWidget(imageLabel);
        mainLayout->addLayout(buttonLayout);
        mainLayout->addWidget(new QLabel("Luminosité:"));
        mainLayout->addWidget(brightnessSlider);

        setLayout(mainLayout);

        // Connexion des signaux
        connect(loadButton, &QPushButton::clicked, this, &ImageEditor::loadImage);
        connect(saveButton, &QPushButton::clicked, this, &ImageEditor::saveImage);
        connect(grayscaleButton, &QPushButton::clicked, this, &ImageEditor::applyGrayscale);
        connect(sepiaButton, &QPushButton::clicked, this, &ImageEditor::applySepia);
        connect(brightnessSlider, &QSlider::valueChanged, this, &ImageEditor::adjustBrightness);

        // Titre de la fenêtre
        setWindowTitle("Mini Éditeur d'Images Qt");
    }

private slots:
    void loadImage()
    {
        QString fileName = QFileDialog::getOpenFileName(this, "Ouvrir une image",
                                                        QDir::homePath(),
                                                        "Images (*.png *.jpg *.bmp)");
        if (!fileName.isEmpty()) {
            originalImage.load(fileName);
            currentImage = originalImage;
            updateDisplay();
            // Activer les boutons
            saveButton->setEnabled(true);
            grayscaleButton->setEnabled(true);
            sepiaButton->setEnabled(true);
            brightnessSlider->setEnabled(true);
        }
    }

    void saveImage()
    {
        if (currentImage.isNull())
            return;

        QString fileName = QFileDialog::getSaveFileName(this, "Sauvegarder l'image",
                                                        QDir::homePath(),
                                                        "PNG (*.png);;JPEG (*.jpg);;BMP (*.bmp)");
        if (!fileName.isEmpty()) {
            currentImage.save(fileName);
        }
    }

    void applyGrayscale()
    {
        if (currentImage.isNull())
            return;

        for (int y = 0; y < currentImage.height(); ++y) {
            for (int x = 0; x < currentImage.width(); ++x) {
                QColor color = currentImage.pixelColor(x, y);
                int gray = qRound(0.299 * color.red() +
                                 0.587 * color.green() +
                                 0.114 * color.blue());
                currentImage.setPixelColor(x, y, QColor(gray, gray, gray));
            }
        }
        updateDisplay();
    }

    void applySepia()
    {
        if (currentImage.isNull())
            return;

        for (int y = 0; y < currentImage.height(); ++y) {
            for (int x = 0; x < currentImage.width(); ++x) {
                QColor oldColor = currentImage.pixelColor(x, y);

                int r = oldColor.red();
                int g = oldColor.green();
                int b = oldColor.blue();

                int newRed = qMin(255, qRound(r * 0.393 + g * 0.769 + b * 0.189));
                int newGreen = qMin(255, qRound(r * 0.349 + g * 0.686 + b * 0.168));
                int newBlue = qMin(255, qRound(r * 0.272 + g * 0.534 + b * 0.131));

                currentImage.setPixelColor(x, y, QColor(newRed, newGreen, newBlue));
            }
        }
        updateDisplay();
    }

    void adjustBrightness(int value)
    {
        if (originalImage.isNull())
            return;

        // Repartir de l'image originale pour éviter les artefacts cumulatifs
        currentImage = originalImage.copy();

        for (int y = 0; y < currentImage.height(); ++y) {
            for (int x = 0; x < currentImage.width(); ++x) {
                QColor color = currentImage.pixelColor(x, y);

                int r = qBound(0, color.red() + value, 255);
                int g = qBound(0, color.green() + value, 255);
                int b = qBound(0, color.blue() + value, 255);

                currentImage.setPixelColor(x, y, QColor(r, g, b));
            }
        }
        updateDisplay();
    }

    void updateDisplay()
    {
        if (currentImage.isNull())
            return;

        // Adapter l'image à la taille du label
        QPixmap pixmap = QPixmap::fromImage(currentImage);
        pixmap = pixmap.scaled(imageLabel->size(), Qt::KeepAspectRatio, Qt::SmoothTransformation);
        imageLabel->setPixmap(pixmap);
    }

    void resizeEvent(QResizeEvent *event) override
    {
        QWidget::resizeEvent(event);
        // Mettre à jour l'affichage quand la fenêtre est redimensionnée
        updateDisplay();
    }

private:
    QLabel *imageLabel;
    QPushButton *loadButton;
    QPushButton *saveButton;
    QPushButton *grayscaleButton;
    QPushButton *sepiaButton;
    QSlider *brightnessSlider;

    QImage originalImage;  // Image originale non modifiée
    QImage currentImage;   // Image après modifications
};

#endif // IMAGEEDITOR_H
```

Utilisation dans un programme principal:

```cpp
#include <QApplication>
#include "ImageEditor.h"

int main(int argc, char *argv[])
{
    QApplication app(argc, argv);

    ImageEditor editor;
    editor.resize(800, 600);
    editor.show();

    return app.exec();
}
```

## Conclusion

Dans cette section, nous avons exploré les bases du traitement d'images avec Qt6. Vous avez maintenant les connaissances nécessaires pour:
- Charger et sauvegarder des images dans différents formats
- Manipuler des pixels individuels et appliquer des transformations
- Créer des filtres simples comme le niveau de gris ou sépia
- Optimiser vos traitements pour de meilleures performances

Le traitement d'images est un domaine vaste, et Qt6 offre de nombreuses fonctionnalités avancées que vous pourrez explorer à mesure que vous progressez. N'hésitez pas à expérimenter avec vos propres filtres et effets!

## Exercices pratiques

1. **Exercice basique**: Créez une application qui charge une image et permet de la convertir en niveaux de gris.

2. **Exercice intermédiaire**: Ajoutez des fonctionnalités à notre mini-éditeur d'images (rotation, miroir, contraste).

3. **Exercice avancé**: Implémentez un filtre de détection de contours (comme le filtre de Sobel) et optimisez-le avec le traitement multithreading.

## Ressources supplémentaires

- [Documentation Qt sur QImage](https://doc.qt.io/qt-6/qimage.html)
- [Documentation Qt sur QPixmap](https://doc.qt.io/qt-6/qpixmap.html)
- [Tutoriels sur le traitement d'images avec Qt](https://doc.qt.io/qt-6/qtimageprocessing-index.html)

⏭️ [Audio et vidéo avec Qt Multimedia](/06-multimedia-et-graphiques/02-audio-et-video-avec-qt-multimedia.md)
