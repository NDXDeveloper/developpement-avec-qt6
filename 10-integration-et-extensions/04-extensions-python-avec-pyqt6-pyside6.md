# 10.4 Extensions Python avec PyQt6/PySide6

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

## Introduction

Python est l'un des langages de programmation les plus populaires au monde, apprécié pour sa simplicité et sa lisibilité. Qt, de son côté, est un framework C++ puissant pour créer des applications multiplateforme. Bonne nouvelle : vous pouvez combiner le meilleur des deux mondes grâce à PyQt6 et PySide6, deux modules qui permettent d'utiliser Qt depuis Python !

Cette section vous expliquera comment intégrer Python dans vos projets Qt6 ou, inversement, comment utiliser Qt6 depuis vos projets Python. Que vous soyez un développeur C++ cherchant à simplifier certaines parties de votre code avec Python, ou un développeur Python voulant créer des interfaces graphiques professionnelles, cette section est faite pour vous.

## PyQt6 vs PySide6 : Comprendre les différences

Avant de commencer, il est important de comprendre qu'il existe deux bibliothèques principales pour utiliser Qt avec Python :

### PyQt6
- Développé par Riverbank Computing
- License GPL ou commerciale
- Existe depuis longtemps, communauté établie
- Documentation riche et nombreux exemples

### PySide6 (Qt for Python)
- Développé officiellement par The Qt Company
- License LGPL (plus permissive que la GPL)
- Support officiel de Qt
- Partie du projet Qt

Les deux bibliothèques sont très similaires en termes d'API, et le code écrit pour l'une peut généralement être adapté pour l'autre avec des modifications mineures. La principale différence réside dans la licence : si vous développez une application commerciale et que vous ne souhaitez pas la publier sous GPL, PySide6 peut être le meilleur choix.

Pour ce tutoriel, nous présenterons des exemples compatibles avec les deux bibliothèques lorsque c'est possible, en indiquant les différences le cas échéant.

## Installation

### Installation de PySide6

```bash
pip install PySide6
```

### Installation de PyQt6

```bash
pip install PyQt6
```

Si vous travaillez avec des environnements virtuels (ce qui est recommandé), n'oubliez pas d'activer votre environnement avant d'installer ces paquets.

## Votre première application Qt en Python

Commençons par créer une application simple "Hello World" avec une interface graphique :

```python
# Exemple compatible PySide6 et PyQt6
import sys

# Essayer d'importer PySide6, sinon utiliser PyQt6
try:
    from PySide6.QtWidgets import QApplication, QMainWindow, QPushButton, QVBoxLayout, QWidget, QLabel
    from PySide6.QtCore import Slot
except ImportError:
    from PyQt6.QtWidgets import QApplication, QMainWindow, QPushButton, QVBoxLayout, QWidget, QLabel
    from PyQt6.QtCore import pyqtSlot as Slot

class MainWindow(QMainWindow):
    def __init__(self):
        super().__init__()

        # Configuration de la fenêtre principale
        self.setWindowTitle("Ma première application Qt en Python")
        self.setGeometry(100, 100, 400, 200)

        # Création du widget central et de son layout
        central_widget = QWidget()
        self.setCentralWidget(central_widget)

        layout = QVBoxLayout(central_widget)

        # Ajout d'un label
        self.label = QLabel("Bienvenue dans Qt avec Python !")
        layout.addWidget(self.label)

        # Ajout d'un bouton
        self.button = QPushButton("Cliquez-moi")
        layout.addWidget(self.button)

        # Connexion du signal clicked du bouton au slot
        self.button.clicked.connect(self.on_button_clicked)

    @Slot()
    def on_button_clicked(self):
        self.label.setText("Bravo ! Vous avez cliqué sur le bouton.")

if __name__ == "__main__":
    app = QApplication(sys.argv)
    window = MainWindow()
    window.show()
    sys.exit(app.exec())  # Dans PyQt6 : app.exec() (sans parenthèses)
```

Ce code crée une fenêtre simple avec un label et un bouton. Lorsque vous cliquez sur le bouton, le texte du label change.

## Concepts clés de Qt en Python

### Widgets et layouts

Comme en C++, la création d'interfaces utilisateur en Python avec Qt repose sur les widgets et les layouts :

```python
# Création de widgets
label = QLabel("Texte")
button = QPushButton("Cliquer")
line_edit = QLineEdit()

# Organisation avec des layouts
layout = QVBoxLayout()
layout.addWidget(label)
layout.addWidget(button)
layout.addWidget(line_edit)

# Création d'un widget qui contiendra le layout
container = QWidget()
container.setLayout(layout)
```

### Signaux et slots

Le système de signaux et slots de Qt est également disponible en Python, avec quelques différences syntaxiques :

```python
# Connexion d'un signal à un slot
button.clicked.connect(self.on_button_clicked)

# Définition d'un slot
@Slot()  # Décorateur optionnel mais recommandé
def on_button_clicked(self):
    print("Bouton cliqué !")
```

### Utilisation de Qt Designer avec Python

Qt Designer est un outil puissant pour créer des interfaces graphiques visuellement. Vous pouvez l'utiliser avec Python :

1. Créez votre interface avec Qt Designer et sauvegardez-la en tant que fichier `.ui`
2. Convertissez le fichier `.ui` en code Python :

Pour PySide6 :
```bash
pyside6-uic mainwindow.ui -o ui_mainwindow.py
```

Pour PyQt6 :
```bash
pyuic6 mainwindow.ui -o ui_mainwindow.py
```

3. Utilisez le fichier généré dans votre code :

```python
from ui_mainwindow import Ui_MainWindow

class MainWindow(QMainWindow):
    def __init__(self):
        super().__init__()

        # Créer l'UI
        self.ui = Ui_MainWindow()
        self.ui.setupUi(self)

        # Connecter les signaux
        self.ui.pushButton.clicked.connect(self.on_button_clicked)

    @Slot()
    def on_button_clicked(self):
        self.ui.label.setText("Bouton cliqué !")
```

## Créer un module d'extension C++ pour Python

Maintenant que nous avons vu comment utiliser Qt depuis Python, examinons l'inverse : comment créer une extension C++ que vous pourrez utiliser dans vos scripts Python.

### Pourquoi créer une extension C++ ?

- **Performance** : Pour les opérations intensives en calcul
- **Accès aux bibliothèques C/C++** : Utiliser du code existant
- **Fonctionnalités de bas niveau** : Accéder à des fonctionnalités système

### Introduction à SIP et Shiboken

Pour créer des extensions C++ pour Python, vous pouvez utiliser les outils suivants :

- **SIP** : Utilisé par PyQt
- **Shiboken** : Utilisé par PySide

Nous nous concentrerons sur Shiboken, l'outil officiel de Qt pour Python.

### Installation des outils de développement

```bash
pip install shiboken6-generator
pip install shiboken6
```

### Exemple simple : Créer une classe C++ et l'exposer à Python

Voici un exemple simple qui montre comment créer une classe C++ et l'exposer à Python :

1. Définir la classe C++ (fichier `mathutils.h`) :

```cpp
#ifndef MATHUTILS_H
#define MATHUTILS_H

class MathUtils {
public:
    MathUtils();
    ~MathUtils();

    // Méthodes que nous voulons exposer à Python
    int add(int a, int b);
    int multiply(int a, int b);
    double power(double base, double exponent);
};

#endif // MATHUTILS_H
```

2. Implémenter la classe (fichier `mathutils.cpp`) :

```cpp
#include "mathutils.h"
#include <cmath>

MathUtils::MathUtils() {}

MathUtils::~MathUtils() {}

int MathUtils::add(int a, int b) {
    return a + b;
}

int MathUtils::multiply(int a, int b) {
    return a * b;
}

double MathUtils::power(double base, double exponent) {
    return std::pow(base, exponent);
}
```

3. Créer un fichier d'interface typologique (fichier `mathutils.xml`) :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<typesystem package="mathutils">
    <primitive-type name="int"/>
    <primitive-type name="double"/>

    <object-type name="MathUtils">
        <include file-name="mathutils.h" location="local"/>
    </object-type>
</typesystem>
```

4. Créer un fichier de projet CMake (fichier `CMakeLists.txt`) :

```cmake
cmake_minimum_required(VERSION 3.16)
project(mathutils_extension)

find_package(Qt6 REQUIRED COMPONENTS Core)
find_package(Shiboken6 REQUIRED)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Sources C++
set(sources
    mathutils.cpp
    mathutils.h
)

# Génération du module Python avec Shiboken
shiboken_add_python_module(mathutils
    ${sources}
    "mathutils.xml"
)
```

5. Compiler le module :

```bash
mkdir build
cd build
cmake ..
make
```

6. Utiliser le module dans Python :

```python
import mathutils

# Créer une instance de MathUtils
math = mathutils.MathUtils()

# Utiliser les méthodes
result1 = math.add(5, 3)
result2 = math.multiply(4, 2)
result3 = math.power(2, 8)

print(f"Addition: 5 + 3 = {result1}")
print(f"Multiplication: 4 * 2 = {result2}")
print(f"Puissance: 2^8 = {result3}")
```

### Intégration avec Qt

Maintenant, voyons comment exposer une classe C++ basée sur Qt à Python :

1. Définir une classe dérivée de QObject (fichier `qtwidget.h`) :

```cpp
#ifndef MYQTWIDGET_H
#define MYQTWIDGET_H

#include <QWidget>
#include <QPushButton>
#include <QVBoxLayout>
#include <QLabel>

class MyQtWidget : public QWidget {
    Q_OBJECT

public:
    MyQtWidget(QWidget *parent = nullptr);
    ~MyQtWidget();

    void setText(const QString &text);
    QString text() const;

public slots:
    void handleButtonClick();

signals:
    void textChanged(const QString &text);

private:
    QLabel *m_label;
    QPushButton *m_button;
    int m_clickCount;
};

#endif // MYQTWIDGET_H
```

2. Implémenter la classe (fichier `qtwidget.cpp`) :

```cpp
#include "qtwidget.h"

MyQtWidget::MyQtWidget(QWidget *parent) : QWidget(parent), m_clickCount(0) {
    m_label = new QLabel("Cliquez sur le bouton", this);
    m_button = new QPushButton("Cliquer", this);

    QVBoxLayout *layout = new QVBoxLayout(this);
    layout->addWidget(m_label);
    layout->addWidget(m_button);

    connect(m_button, &QPushButton::clicked, this, &MyQtWidget::handleButtonClick);

    setLayout(layout);
}

MyQtWidget::~MyQtWidget() {
}

void MyQtWidget::setText(const QString &text) {
    m_label->setText(text);
    emit textChanged(text);
}

QString MyQtWidget::text() const {
    return m_label->text();
}

void MyQtWidget::handleButtonClick() {
    m_clickCount++;
    setText(QString("Nombre de clics : %1").arg(m_clickCount));
}
```

3. Adapter le fichier typologique (fichier `qtwidget.xml`) :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<typesystem package="qtwidget">
    <primitive-type name="int"/>
    <primitive-type name="QString"/>

    <object-type name="MyQtWidget">
        <include file-name="qtwidget.h" location="local"/>
        <signal name="textChanged" />
    </object-type>
</typesystem>
```

4. Mettre à jour le CMakeLists.txt pour inclure Qt :

```cmake
cmake_minimum_required(VERSION 3.16)
project(qtwidget_extension)

find_package(Qt6 REQUIRED COMPONENTS Core Widgets)
find_package(Shiboken6 REQUIRED)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_AUTOMOC ON)

# Sources C++
set(sources
    qtwidget.cpp
    qtwidget.h
)

# Génération du module Python avec Shiboken
shiboken_add_python_module(qtwidget
    ${sources}
    "qtwidget.xml"
    GLOBAL_HEADER
)

target_link_libraries(qtwidget PRIVATE Qt6::Core Qt6::Widgets)
```

5. Utiliser le widget dans Python :

```python
import sys
import qtwidget
from PySide6.QtWidgets import QApplication

# Fonction de callback pour le signal textChanged
def on_text_changed(text):
    print(f"Le texte a changé : {text}")

if __name__ == "__main__":
    app = QApplication(sys.argv)

    # Créer une instance de notre widget
    widget = qtwidget.MyQtWidget()

    # Connecter le signal textChanged
    widget.textChanged.connect(on_text_changed)

    # Montrer le widget
    widget.resize(300, 150)
    widget.show()

    sys.exit(app.exec())
```

## Utilisation de plugins Qt en Python

Vous pouvez également utiliser les plugins Qt que vous avez créés en C++ dans vos applications Python.

```python
from PySide6.QtCore import QPluginLoader, QDir

# Charger un plugin
loader = QPluginLoader("chemin/vers/mon_plugin.dll")  # ou .so sur Linux
plugin = loader.instance()

if plugin:
    # Utiliser le plugin
    # Vous devrez faire un cast vers l'interface appropriée
    # Cela dépend de votre plugin spécifique
    print("Plugin chargé avec succès")
else:
    print(f"Erreur lors du chargement du plugin : {loader.errorString()}")
```

## Bonnes pratiques pour les extensions Python avec Qt

### 1. Gérer les références cycliques

Python utilise un ramasse-miettes basé sur le comptage de références, qui peut avoir des problèmes avec les références cycliques. Soyez attentif lors de la création de connexions signal-slot qui pourraient créer des cycles.

### 2. Libérer les ressources explicitement

Bien que Python et Qt aient des mécanismes de nettoyage, il est préférable de libérer explicitement les ressources lorsque vous avez terminé avec elles.

```python
# Déconnecter les signaux lorsqu'ils ne sont plus nécessaires
button.clicked.disconnect(self.on_button_clicked)

# Supprimer les widgets manuellement si nécessaire
widget.deleteLater()
```

### 3. Éviter les fuites de mémoire

Lorsque vous créez des extensions C++ pour Python, assurez-vous de gérer correctement la mémoire. Shiboken et SIP aident à cela, mais vous devez être vigilant.

### 4. Documentation

Documentez votre code Python et vos extensions C++ pour faciliter la maintenance et l'utilisation par d'autres développeurs.

```python
class MyClass:
    """
    Cette classe fait quelque chose d'utile.

    Elle fournit des fonctionnalités pour X, Y et Z.
    """

    def my_method(self, param1, param2):
        """
        Fait quelque chose de spécifique.

        Args:
            param1: Description du premier paramètre
            param2: Description du second paramètre

        Returns:
            Description de la valeur de retour
        """
        # Implémentation...
```

## Conclusion

L'intégration de Python avec Qt6 ouvre un monde de possibilités. Vous pouvez :

- Créer rapidement des interfaces graphiques en Python avec PyQt6 ou PySide6
- Développer des extensions C++ pour Python lorsque vous avez besoin de plus de performance
- Combiner du code existant en C++ avec la facilité d'utilisation de Python

Que vous soyez un développeur Python cherchant à créer des GUI professionnelles ou un développeur C++ souhaitant intégrer la flexibilité de Python, ces outils vous offrent le meilleur des deux mondes.

L'écosystème PyQt6/PySide6 continue d'évoluer et de s'améliorer, rendant l'intégration Python-Qt de plus en plus puissante et facile à utiliser.

## Ressources supplémentaires

- [Documentation officielle de PySide6](https://doc.qt.io/qtforpython-6/)
- [Documentation de PyQt6](https://www.riverbankcomputing.com/static/Docs/PyQt6/)
- [Tutoriels Qt for Python](https://doc.qt.io/qtforpython-6/tutorials/index.html)
- [Guide du générateur Shiboken](https://doc.qt.io/qtforpython-6/shiboken6/index.html)

⏭️ [Bonnes pratiques Qt6](/11-bonnes-pratiques-qt6)
