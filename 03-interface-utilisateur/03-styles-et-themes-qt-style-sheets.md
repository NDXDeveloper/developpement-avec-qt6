# 3.3 Styles et thèmes (Qt Style Sheets)

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

Une interface utilisateur attrayante est essentielle pour offrir une bonne expérience à vos utilisateurs. Qt6 propose plusieurs mécanismes pour personnaliser l'apparence de vos applications, dont le plus puissant est Qt Style Sheets (QSS). Ce système, inspiré des CSS (Cascading Style Sheets) utilisés pour le web, vous permet de transformer radicalement l'apparence de vos applications.

## Introduction aux Qt Style Sheets

### Qu'est-ce que Qt Style Sheets ?

Qt Style Sheets est un mécanisme qui permet de personnaliser l'apparence des widgets Qt en utilisant une syntaxe similaire aux CSS du web. Avec QSS, vous pouvez modifier :

- Les couleurs de fond et de texte
- Les bordures et leur style
- Les marges et espacements
- Les polices de caractères
- Les images de fond et icônes
- Et bien plus encore

### Syntaxe de base

La syntaxe de QSS est très similaire à celle des CSS :

```css
/* Sélecteur { propriété: valeur; propriété: valeur; ... } */

QPushButton {
    background-color: #3498db;
    color: white;
    border-radius: 5px;
    padding: 5px 10px;
}
```

Ce style s'appliquera à tous les `QPushButton` de votre application, leur donnant un fond bleu, du texte blanc, des coins arrondis et un espacement interne.

## Application des styles

### Application globale

Pour appliquer un style à toute votre application :

```cpp
// Dans votre main.cpp ou au démarrage de l'application
QApplication app(argc, argv);

// Depuis une chaîne de caractères
app.setStyleSheet("QPushButton { background-color: #3498db; color: white; }");

// Ou depuis un fichier
QFile fichier(":/styles/style.qss");
if (fichier.open(QFile::ReadOnly | QFile::Text)) {
    QTextStream flux(&fichier);
    app.setStyleSheet(flux.readAll());
}
```

### Application à un widget spécifique

Pour appliquer un style à un widget particulier et ses enfants :

```cpp
// Juste pour ce bouton
monBouton->setStyleSheet("background-color: red; color: white;");

// Pour tout un conteneur et ses enfants
monConteneur->setStyleSheet("QLabel { color: blue; } QPushButton { color: green; }");
```

## Sélecteurs

Les sélecteurs permettent de cibler précisément les widgets à styliser :

### Sélecteur de type

Style tous les widgets d'un type donné :

```css
QPushButton {
    background-color: #3498db;
}
```

### Sélecteur d'ID

Style un widget spécifique par son nom d'objet :

```css
QPushButton#btnValider {
    background-color: #2ecc71;  /* Vert */
}
```

Dans votre code C++ :

```cpp
QPushButton *btnValider = new QPushButton("Valider");
btnValider->setObjectName("btnValider");  // Définit l'ID pour le CSS
```

### Sélecteur de classe

Style les widgets d'une classe particulière :

```css
.warning-button {
    background-color: #e74c3c;  /* Rouge */
}
```

Dans votre code C++ :

```cpp
QPushButton *btnAnnuler = new QPushButton("Annuler");
btnAnnuler->setProperty("class", "warning-button");  // Assigne la classe CSS
```

### Sélecteurs hiérarchiques

Style les widgets dans un contexte particulier :

```css
QDialog QPushButton {
    /* Style les boutons uniquement dans les boîtes de dialogue */
    background-color: #95a5a6;
}

QGroupBox > QCheckBox {
    /* Style les cases à cocher directement enfants d'un QGroupBox */
    color: #8e44ad;
}
```

### Sélecteurs d'état

Style les widgets dans des états particuliers :

```css
QPushButton:hover {
    /* Style au survol de la souris */
    background-color: #2980b9;
}

QPushButton:pressed {
    /* Style quand le bouton est pressé */
    background-color: #1f6dad;
}

QCheckBox:checked {
    /* Style quand la case est cochée */
    color: green;
}

QLineEdit:focus {
    /* Style quand le champ a le focus */
    border: 2px solid #3498db;
}

QPushButton:disabled {
    /* Style quand le bouton est désactivé */
    background-color: #bdc3c7;
    color: #7f8c8d;
}
```

## Propriétés de style

### Couleurs et fonds

```css
QWidget {
    background-color: #ecf0f1;    /* Couleur de fond unie */
    color: #2c3e50;               /* Couleur du texte */
}

QFrame {
    background-image: url(:/images/fond.png);  /* Image de fond */
    background-repeat: no-repeat;              /* Ne pas répéter l'image */
    background-position: center;               /* Centrer l'image */
}

QProgressBar {
    /* Dégradé de couleur */
    background: qlineargradient(x1:0, y1:0, x2:1, y2:0,
                               stop:0 #f1c40f, stop:1 #e74c3c);
}
```

### Bordures

```css
QLineEdit {
    border: 2px solid #3498db;      /* Largeur, style et couleur */
    border-radius: 5px;             /* Coins arrondis */
}

QGroupBox {
    border-top: 1px solid #bdc3c7;  /* Bordure seulement en haut */
    border-style: outset;           /* Style de la bordure */
    padding-top: 15px;              /* Espace pour le titre */
}
```

### Polices

```css
QLabel {
    font-family: "Arial";           /* Type de police */
    font-size: 14px;                /* Taille */
    font-weight: bold;              /* Poids (normal, bold) */
    font-style: italic;             /* Style (normal, italic) */
}

QPushButton {
    text-decoration: underline;     /* Soulignement */
    text-transform: uppercase;      /* Transformation de texte */
}
```

### Dimensions et positionnement

```css
QPushButton {
    min-height: 30px;               /* Hauteur minimale */
    max-width: 150px;               /* Largeur maximale */
    padding: 5px 10px;              /* Espace interne (haut/bas gauche/droite) */
    margin: 5px;                    /* Espace externe */
}
```

## Création de thèmes complets

### Structure d'un fichier de thème

Un fichier de thème complet est généralement organisé comme ceci :

```css
/* style.qss */

/* Variables de couleur (commentaires seulement, pas de vraies variables en QSS) */
/* Couleur principale: #3498db */
/* Couleur secondaire: #2ecc71 */
/* Couleur de fond: #ecf0f1 */
/* Couleur de texte: #2c3e50 */

/* Style global */
* {
    font-family: "Segoe UI", Arial, sans-serif;
}

/* Fenêtre principale */
QMainWindow, QDialog {
    background-color: #ecf0f1;
}

/* Widgets de base */
QLabel {
    color: #2c3e50;
}

QLineEdit, QTextEdit, QPlainTextEdit, QSpinBox, QDoubleSpinBox, QComboBox {
    border: 1px solid #bdc3c7;
    border-radius: 4px;
    padding: 3px;
    background-color: white;
    selection-background-color: #3498db;
}

QLineEdit:focus, QTextEdit:focus, QPlainTextEdit:focus {
    border: 2px solid #3498db;
}

/* Boutons */
QPushButton {
    background-color: #3498db;
    color: white;
    border-radius: 4px;
    padding: 5px 15px;
    min-height: 20px;
}

QPushButton:hover {
    background-color: #2980b9;
}

QPushButton:pressed {
    background-color: #1a5276;
}

QPushButton:disabled {
    background-color: #bdc3c7;
    color: #7f8c8d;
}

/* ... et ainsi de suite pour tous les widgets ... */
```

### Chargement à partir d'un fichier de ressources

Pour inclure votre fichier de style dans votre application, utilisez le système de ressources Qt :

1. Créez un fichier `resources.qrc` :

```xml
<!DOCTYPE RCC>
<RCC>
    <qresource prefix="/styles">
        <file>style.qss</file>
    </qresource>
    <qresource prefix="/images">
        <file>fond.png</file>
        <file>icones/save.png</file>
        <file>icones/open.png</file>
    </qresource>
</RCC>
```

2. Chargez-le dans votre application :

```cpp
QFile fichierStyle(":/styles/style.qss");
if (fichierStyle.open(QFile::ReadOnly | QFile::Text)) {
    QTextStream flux(&fichierStyle);
    QString styleSheet = flux.readAll();
    qApp->setStyleSheet(styleSheet);
    fichierStyle.close();
}
```

### Changement de thème à la volée

Vous pouvez permettre à l'utilisateur de changer de thème pendant l'exécution :

```cpp
void MainWindow::changerTheme(const QString &nomTheme)
{
    QString cheminTheme = ":/styles/" + nomTheme + ".qss";
    QFile fichierStyle(cheminTheme);

    if (fichierStyle.open(QFile::ReadOnly | QFile::Text)) {
        QTextStream flux(&fichierStyle);
        qApp->setStyleSheet(flux.readAll());
        fichierStyle.close();
    }
}

// Utilisation
ui->comboBoxThemes->addItem("Clair", "theme_clair");
ui->comboBoxThemes->addItem("Sombre", "theme_sombre");
ui->comboBoxThemes->addItem("Coloré", "theme_colore");

connect(ui->comboBoxThemes, QOverload<int>::of(&QComboBox::currentIndexChanged),
        [this](int index) {
    QString nomTheme = ui->comboBoxThemes->itemData(index).toString();
    changerTheme(nomTheme);
});
```

## Styles spécifiques aux widgets

### QTabWidget et QTabBar

```css
QTabWidget::pane {
    /* Le panneau contenant le contenu des onglets */
    border: 1px solid #bdc3c7;
    border-top: 0px;
}

QTabBar::tab {
    /* L'onglet lui-même */
    background-color: #ecf0f1;
    border: 1px solid #bdc3c7;
    border-bottom: 0px;
    border-top-left-radius: 4px;
    border-top-right-radius: 4px;
    padding: 5px 10px;
}

QTabBar::tab:selected {
    /* L'onglet sélectionné */
    background-color: white;
    border-bottom: 0px;
    margin-bottom: -1px; /* Chevauche la bordure du panneau */
}

QTabBar::tab:!selected {
    /* Les onglets non sélectionnés */
    margin-top: 2px; /* Rend les onglets non sélectionnés un peu plus petits */
}
```

### QSlider

```css
QSlider::groove:horizontal {
    /* Le rail du curseur */
    height: 8px;
    background: #bdc3c7;
    border-radius: 4px;
}

QSlider::handle:horizontal {
    /* La poignée du curseur */
    background: #3498db;
    border: none;
    width: 16px;
    height: 16px;
    margin: -4px 0;
    border-radius: 8px;
}

QSlider::add-page:horizontal {
    /* La partie à droite de la poignée */
    background: #bdc3c7;
    border-radius: 4px;
}

QSlider::sub-page:horizontal {
    /* La partie à gauche de la poignée (remplie) */
    background: #3498db;
    border-radius: 4px;
}
```

### QProgressBar

```css
QProgressBar {
    border: 1px solid #bdc3c7;
    border-radius: 5px;
    text-align: center;
    background-color: #ecf0f1;
}

QProgressBar::chunk {
    background-color: #3498db;
    width: 20px; /* pour créer un effet de bloc */
}
```

### QTreeView et QTableView

```css
QTreeView, QTableView {
    alternate-background-color: #f0f0f0; /* Couleur des lignes alternées */
    background-color: white;
    selection-background-color: #3498db;
    selection-color: white;
}

QTreeView::item, QTableView::item {
    padding: 5px;
}

QTreeView::item:selected, QTableView::item:selected {
    background-color: #3498db;
}

QHeaderView::section {
    background-color: #ecf0f1;
    padding: 5px;
    border: 1px solid #bdc3c7;
    border-top: 0px;
    border-left: 0px;
}
```

## Mode sombre (Dark Theme)

Voici un exemple de thème sombre pour votre application :

```css
/* Thème sombre */
QWidget {
    background-color: #2c3e50;
    color: #ecf0f1;
}

QLineEdit, QTextEdit, QPlainTextEdit, QSpinBox, QDoubleSpinBox, QComboBox {
    border: 1px solid #34495e;
    border-radius: 4px;
    padding: 3px;
    background-color: #34495e;
    color: #ecf0f1;
    selection-background-color: #3498db;
}

QPushButton {
    background-color: #3498db;
    color: #ecf0f1;
    border-radius: 4px;
    padding: 5px 15px;
    min-height: 20px;
}

QPushButton:hover {
    background-color: #2980b9;
}

QPushButton:pressed {
    background-color: #1a5276;
}

QTabWidget::pane {
    border: 1px solid #34495e;
}

QTabBar::tab {
    background-color: #2c3e50;
    border: 1px solid #34495e;
    padding: 5px 10px;
}

QTabBar::tab:selected {
    background-color: #34495e;
}

QMenu {
    background-color: #34495e;
    border: 1px solid #2c3e50;
}

QMenu::item {
    padding: 5px 30px 5px 20px;
}

QMenu::item:selected {
    background-color: #3498db;
}

QScrollBar:vertical {
    background-color: #2c3e50;
    width: 14px;
}

QScrollBar::handle:vertical {
    background-color: #34495e;
    min-height: 20px;
    border-radius: 7px;
}

QScrollBar::add-line:vertical, QScrollBar::sub-line:vertical {
    height: 0px;
}
```

## Exemples pratiques

### Exemple 1 : Formulaire stylisé

Voici un exemple de formulaire avec un style personnalisé :

```cpp
// main.cpp
#include <QApplication>
#include <QWidget>
#include <QVBoxLayout>
#include <QFormLayout>
#include <QLineEdit>
#include <QLabel>
#include <QPushButton>
#include <QFile>

int main(int argc, char *argv[])
{
    QApplication app(argc, argv);

    // Création du formulaire
    QWidget fenetre;
    fenetre.setWindowTitle("Formulaire stylisé");
    fenetre.resize(400, 250);

    QVBoxLayout *layout = new QVBoxLayout(&fenetre);

    QLabel *titre = new QLabel("Formulaire d'inscription");
    titre->setObjectName("titre");
    layout->addWidget(titre);

    QFormLayout *form = new QFormLayout();
    layout->addLayout(form);

    QLineEdit *champNom = new QLineEdit();
    champNom->setPlaceholderText("Votre nom");
    form->addRow("Nom :", champNom);

    QLineEdit *champEmail = new QLineEdit();
    champEmail->setPlaceholderText("votre@email.com");
    form->addRow("Email :", champEmail);

    QLineEdit *champMdp = new QLineEdit();
    champMdp->setPlaceholderText("Mot de passe");
    champMdp->setEchoMode(QLineEdit::Password);
    form->addRow("Mot de passe :", champMdp);

    QPushButton *boutonValider = new QPushButton("S'inscrire");
    boutonValider->setObjectName("boutonValider");
    layout->addWidget(boutonValider);

    // Application du style
    QString styleSheet = R"(
        QWidget {
            font-family: Arial;
            background-color: white;
        }

        QLabel#titre {
            font-size: 18px;
            font-weight: bold;
            color: #3498db;
            margin: 10px 0px;
        }

        QLineEdit {
            border: 1px solid #bdc3c7;
            border-radius: 4px;
            padding: 8px;
            font-size: 14px;
        }

        QLineEdit:focus {
            border: 2px solid #3498db;
        }

        QPushButton#boutonValider {
            background-color: #2ecc71;
            color: white;
            border-radius: 4px;
            padding: 10px;
            font-size: 14px;
            margin-top: 10px;
        }

        QPushButton#boutonValider:hover {
            background-color: #27ae60;
        }

        QPushButton#boutonValider:pressed {
            background-color: #1e8449;
        }
    )";

    fenetre.setStyleSheet(styleSheet);
    fenetre.show();

    return app.exec();
}
```

### Exemple 2 : Dashboard avec statistiques

Un exemple de tableau de bord avec un style moderne :

```cpp
// Supposons que nous avons une interface avec différentes sections
// La mise en page est faite, nous nous concentrons sur les styles

QString styleDashboard = R"(
    /* Style global */
    QWidget#dashboard {
        background-color: #f5f5f5;
        font-family: "Segoe UI", sans-serif;
    }

    /* En-tête */
    QLabel#headerLabel {
        font-size: 24px;
        color: #2c3e50;
        padding: 10px;
        border-bottom: 1px solid #e0e0e0;
    }

    /* Cartes de statistiques */
    QFrame.stat-card {
        background-color: white;
        border-radius: 8px;
        padding: 15px;
    }

    QLabel.stat-title {
        font-size: 14px;
        color: #7f8c8d;
    }

    QLabel.stat-value {
        font-size: 28px;
        font-weight: bold;
        color: #2c3e50;
    }

    /* Différentes couleurs pour les cartes */
    QFrame#card1 {
        border-top: 4px solid #3498db;
    }

    QFrame#card2 {
        border-top: 4px solid #2ecc71;
    }

    QFrame#card3 {
        border-top: 4px solid #e74c3c;
    }

    QFrame#card4 {
        border-top: 4px solid #f39c12;
    }

    /* Graphiques */
    QFrame#chartContainer {
        background-color: white;
        border-radius: 8px;
        padding: 15px;
    }

    /* Tableau */
    QTableView {
        background-color: white;
        alternate-background-color: #f9f9f9;
        border-radius: 8px;
        border: none;
    }

    QTableView::item {
        padding: 8px;
    }

    QHeaderView::section {
        background-color: #ecf0f1;
        font-weight: bold;
        padding: 8px;
        border: none;
    }

    /* Menus latéraux */
    QFrame#sidebar {
        background-color: #2c3e50;
        min-width: 200px;
    }

    QPushButton.sidebar-button {
        background-color: transparent;
        color: #ecf0f1;
        text-align: left;
        padding: 12px 15px;
        border: none;
        border-radius: 0;
    }

    QPushButton.sidebar-button:hover {
        background-color: #34495e;
    }

    QPushButton.sidebar-button:checked {
        background-color: #3498db;
        font-weight: bold;
    }
)";

// Application du style
dashboardWidget->setStyleSheet(styleDashboard);
```

## Conseils et bonnes pratiques

1. **Organisation du code**
   - Gardez vos styles dans des fichiers séparés (.qss)
   - Commentez vos styles pour faciliter la maintenance
   - Structurez votre fichier par groupes logiques (contrôles, fenêtres, etc.)

2. **Performance**
   - Les styles complexes peuvent impacter les performances
   - Évitez d'appliquer trop de styles dynamiquement
   - Préférez des fichiers QSS précompilés pour les grandes applications

3. **Compatibilité**
   - Testez vos styles sur différentes plateformes
   - Certaines propriétés peuvent ne pas fonctionner sur tous les widgets
   - Les styles peuvent interagir différemment avec les thèmes systèmes

4. **Débogage**
   - Utilisez l'inspecteur de widgets de Qt Creator pour voir la hiérarchie
   - Appliquez temporairement des styles simples pour identifier les problèmes
   - Les erreurs de syntaxe QSS n'affichent pas toujours des avertissements

## Conclusion

Qt Style Sheets est un moyen puissant de personnaliser l'apparence de vos applications Qt. Avec une syntaxe familière inspirée des CSS, vous pouvez créer des interfaces attrayantes qui se démarquent tout en maintenant la cohérence visuelle.

Que vous souhaitiez simplement ajuster quelques couleurs ou créer un thème complet, QSS vous offre la flexibilité nécessaire pour répondre à vos besoins de design. En maîtrisant les sélecteurs, les propriétés et les pseudo-états, vous pourrez transformer radicalement l'apparence de vos applications Qt6.

Dans la prochaine section, nous explorerons l'internationalisation avec Qt Linguist pour rendre vos applications accessibles à un public international.

⏭️ [Internationalisation (i18n) avec Qt Linguist](/03-interface-utilisateur/04-internationalisation-i18n-avec-qt-linguist.md)
