# 3.1 Développement avec Qt Widgets

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

Qt Widgets est l'approche traditionnelle pour créer des interfaces utilisateur dans Qt. Cette technologie éprouvée vous permet de construire des applications de bureau robustes avec un aspect natif sur différentes plateformes. Dans cette section, nous explorerons les fondamentaux du développement avec Qt Widgets.

## Les bases des widgets

### Qu'est-ce qu'un widget ?

Un widget est un élément d'interface graphique : bouton, étiquette, champ de texte, case à cocher, etc. Dans Qt, la classe de base pour tous les widgets est `QWidget`. Chaque widget peut contenir d'autres widgets, créant ainsi une hiérarchie.

### Widgets courants

Voici quelques-uns des widgets les plus utilisés :

| Widget | Classe Qt | Description |
|--------|-----------|-------------|
| Bouton | QPushButton | Bouton standard sur lequel l'utilisateur peut cliquer |
| Étiquette | QLabel | Affiche du texte ou une image |
| Champ de texte | QLineEdit | Permet la saisie d'une ligne de texte |
| Zone de texte | QTextEdit | Permet la saisie de texte sur plusieurs lignes |
| Case à cocher | QCheckBox | Option qui peut être activée ou désactivée |
| Bouton radio | QRadioButton | Option exclusive dans un groupe |
| Liste déroulante | QComboBox | Menu déroulant pour sélectionner une option |
| Curseur | QSlider | Permet de sélectionner une valeur en déplaçant un curseur |
| Barre de progression | QProgressBar | Affiche la progression d'une opération |

## Création d'une interface simple

### Premier exemple : une fenêtre avec un bouton

```cpp
#include <QApplication>
#include <QWidget>
#include <QPushButton>

int main(int argc, char *argv[])
{
    // Création de l'application
    QApplication app(argc, argv);

    // Création d'une fenêtre
    QWidget fenetre;
    fenetre.setWindowTitle("Ma première fenêtre");
    fenetre.resize(300, 200);

    // Création d'un bouton à l'intérieur de la fenêtre
    QPushButton *bouton = new QPushButton("Cliquez-moi", &fenetre);
    bouton->setGeometry(100, 80, 100, 30);  // Position et taille

    // Connexion du signal clicked() du bouton à une lambda
    QObject::connect(bouton, &QPushButton::clicked, []() {
        qDebug() << "Le bouton a été cliqué !";
    });

    // Affichage de la fenêtre
    fenetre.show();

    // Démarrage de la boucle d'événements
    return app.exec();
}
```

Dans cet exemple, nous avons :
1. Créé une application QApplication
2. Créé une fenêtre QWidget
3. Ajouté un bouton QPushButton à la fenêtre
4. Connecté le signal clicked() du bouton à une fonction
5. Affiché la fenêtre
6. Démarré la boucle d'événements

## Layouts : organisation des widgets

Le positionnement manuel des widgets (comme avec `setGeometry()`) est fastidieux et ne s'adapte pas bien au redimensionnement de la fenêtre. Les layouts résolvent ce problème en organisant automatiquement vos widgets.

### Types de layouts principaux

Qt propose plusieurs types de layouts :

![Types de layouts](illustration_layouts.png)

#### QVBoxLayout : disposition verticale

```cpp
QWidget *fenetre = new QWidget();
QVBoxLayout *layout = new QVBoxLayout(fenetre);

QPushButton *bouton1 = new QPushButton("Bouton 1");
QPushButton *bouton2 = new QPushButton("Bouton 2");
QPushButton *bouton3 = new QPushButton("Bouton 3");

layout->addWidget(bouton1);
layout->addWidget(bouton2);
layout->addWidget(bouton3);

fenetre->show();
```

#### QHBoxLayout : disposition horizontale

```cpp
QWidget *fenetre = new QWidget();
QHBoxLayout *layout = new QHBoxLayout(fenetre);

QPushButton *bouton1 = new QPushButton("Bouton 1");
QPushButton *bouton2 = new QPushButton("Bouton 2");
QPushButton *bouton3 = new QPushButton("Bouton 3");

layout->addWidget(bouton1);
layout->addWidget(bouton2);
layout->addWidget(bouton3);

fenetre->show();
```

#### QGridLayout : disposition en grille

```cpp
QWidget *fenetre = new QWidget();
QGridLayout *layout = new QGridLayout(fenetre);

QPushButton *bouton1 = new QPushButton("Bouton 1");
QPushButton *bouton2 = new QPushButton("Bouton 2");
QPushButton *bouton3 = new QPushButton("Bouton 3");
QPushButton *bouton4 = new QPushButton("Bouton 4");

layout->addWidget(bouton1, 0, 0);  // Ligne 0, Colonne 0
layout->addWidget(bouton2, 0, 1);  // Ligne 0, Colonne 1
layout->addWidget(bouton3, 1, 0);  // Ligne 1, Colonne 0
layout->addWidget(bouton4, 1, 1);  // Ligne 1, Colonne 1

fenetre->show();
```

#### QFormLayout : disposition en formulaire

```cpp
QWidget *fenetre = new QWidget();
QFormLayout *layout = new QFormLayout(fenetre);

QLineEdit *champNom = new QLineEdit();
QLineEdit *champEmail = new QLineEdit();
QSpinBox *champAge = new QSpinBox();

layout->addRow("Nom :", champNom);
layout->addRow("Email :", champEmail);
layout->addRow("Âge :", champAge);

fenetre->show();
```

### Layouts imbriqués

Vous pouvez imbriquer les layouts pour créer des interfaces complexes :

```cpp
QWidget *fenetre = new QWidget();
QVBoxLayout *layoutPrincipal = new QVBoxLayout(fenetre);

// Section supérieure avec deux boutons côte à côte
QHBoxLayout *layoutSuperieur = new QHBoxLayout();
layoutSuperieur->addWidget(new QPushButton("Bouton 1"));
layoutSuperieur->addWidget(new QPushButton("Bouton 2"));

// Section inférieure avec un formulaire
QFormLayout *layoutInferieur = new QFormLayout();
layoutInferieur->addRow("Nom :", new QLineEdit());
layoutInferieur->addRow("Email :", new QLineEdit());

// Assemblage des sections
layoutPrincipal->addLayout(layoutSuperieur);
layoutPrincipal->addLayout(layoutInferieur);

fenetre->show();
```

## Dialogues et fenêtres

### QMainWindow : la base des applications complexes

Pour les applications plus complexes, Qt propose `QMainWindow`, qui facilite l'ajout de menus, barres d'outils, barres d'état, etc.

```cpp
#include <QApplication>
#include <QMainWindow>
#include <QMenuBar>
#include <QToolBar>
#include <QStatusBar>
#include <QLabel>

int main(int argc, char *argv[])
{
    QApplication app(argc, argv);

    // Création de la fenêtre principale
    QMainWindow mainWindow;
    mainWindow.setWindowTitle("Mon Application");
    mainWindow.resize(800, 600);

    // Création d'un widget central
    QLabel *label = new QLabel("Contenu principal");
    label->setAlignment(Qt::AlignCenter);
    mainWindow.setCentralWidget(label);

    // Création d'un menu
    QMenu *menuFichier = mainWindow.menuBar()->addMenu("&Fichier");
    QAction *actionNouveau = menuFichier->addAction("&Nouveau");
    QAction *actionQuitter = menuFichier->addAction("&Quitter");

    // Connexion de l'action Quitter
    QObject::connect(actionQuitter, &QAction::triggered, &app, &QApplication::quit);

    // Création d'une barre d'outils
    QToolBar *toolBar = mainWindow.addToolBar("Barre d'outils");
    toolBar->addAction(actionNouveau);
    toolBar->addAction(actionQuitter);

    // Création d'une barre d'état
    mainWindow.statusBar()->showMessage("Prêt");

    mainWindow.show();
    return app.exec();
}
```

### Dialogues standard

Qt propose de nombreux dialogues standard prêts à l'emploi :

```cpp
// Exemple de dialogues standard
QPushButton *boutonMessage = new QPushButton("Message");
QObject::connect(boutonMessage, &QPushButton::clicked, []() {
    QMessageBox::information(nullptr, "Information", "Ceci est un message d'information.");
});

QPushButton *boutonQuestion = new QPushButton("Question");
QObject::connect(boutonQuestion, &QPushButton::clicked, []() {
    int reponse = QMessageBox::question(nullptr, "Question",
                                       "Voulez-vous continuer ?",
                                       QMessageBox::Yes | QMessageBox::No);
    if (reponse == QMessageBox::Yes)
        qDebug() << "L'utilisateur a choisi Oui";
    else
        qDebug() << "L'utilisateur a choisi Non";
});

QPushButton *boutonFichier = new QPushButton("Ouvrir un fichier");
QObject::connect(boutonFichier, &QPushButton::clicked, []() {
    QString fichier = QFileDialog::getOpenFileName(nullptr, "Ouvrir un fichier",
                                                 QDir::homePath(),
                                                 "Documents (*.txt *.pdf)");
    if (!fichier.isEmpty())
        qDebug() << "Fichier sélectionné :" << fichier;
});
```

### Création de dialogues personnalisés

Vous pouvez créer vos propres dialogues en héritant de `QDialog` :

```cpp
// En-tête : formulaireutilisateur.h
class FormulaireUtilisateur : public QDialog
{
    Q_OBJECT
public:
    FormulaireUtilisateur(QWidget *parent = nullptr);

    QString nom() const;
    QString email() const;
    int age() const;

private:
    QLineEdit *m_champNom;
    QLineEdit *m_champEmail;
    QSpinBox *m_champAge;
};

// Implémentation : formulaireutilisateur.cpp
FormulaireUtilisateur::FormulaireUtilisateur(QWidget *parent)
    : QDialog(parent)
{
    setWindowTitle("Nouveau utilisateur");

    // Création des champs
    m_champNom = new QLineEdit(this);
    m_champEmail = new QLineEdit(this);
    m_champAge = new QSpinBox(this);
    m_champAge->setRange(0, 120);

    // Création des boutons standard
    QDialogButtonBox *boutons = new QDialogButtonBox(
        QDialogButtonBox::Ok | QDialogButtonBox::Cancel, this);

    connect(boutons, &QDialogButtonBox::accepted, this, &QDialog::accept);
    connect(boutons, &QDialogButtonBox::rejected, this, &QDialog::reject);

    // Mise en page
    QFormLayout *layout = new QFormLayout(this);
    layout->addRow("Nom :", m_champNom);
    layout->addRow("Email :", m_champEmail);
    layout->addRow("Âge :", m_champAge);
    layout->addWidget(boutons);
}

QString FormulaireUtilisateur::nom() const
{
    return m_champNom->text();
}

QString FormulaireUtilisateur::email() const
{
    return m_champEmail->text();
}

int FormulaireUtilisateur::age() const
{
    return m_champAge->value();
}

// Utilisation du dialogue
void maFonction()
{
    FormulaireUtilisateur formulaire;
    if (formulaire.exec() == QDialog::Accepted) {
        qDebug() << "Nom :" << formulaire.nom();
        qDebug() << "Email :" << formulaire.email();
        qDebug() << "Âge :" << formulaire.age();
    } else {
        qDebug() << "Opération annulée";
    }
}
```

## Widgets personnalisés

Pour des besoins spécifiques, vous pouvez créer vos propres widgets en héritant de `QWidget` ou d'une de ses sous-classes.

```cpp
// En-tête : compteurcircle.h
class CompteurCirculaire : public QWidget
{
    Q_OBJECT
public:
    CompteurCirculaire(QWidget *parent = nullptr);

    int valeur() const;
    void setValeur(int valeur);

signals:
    void valeurChangee(int nouvelle_valeur);

protected:
    void paintEvent(QPaintEvent *event) override;
    void mousePressEvent(QMouseEvent *event) override;

private:
    int m_valeur;
};

// Implémentation : compteurcircle.cpp
CompteurCirculaire::CompteurCirculaire(QWidget *parent)
    : QWidget(parent), m_valeur(0)
{
    setMinimumSize(100, 100);
}

int CompteurCirculaire::valeur() const
{
    return m_valeur;
}

void CompteurCirculaire::setValeur(int valeur)
{
    if (m_valeur != valeur) {
        m_valeur = valeur;
        emit valeurChangee(m_valeur);
        update();  // Déclenche un repaint
    }
}

void CompteurCirculaire::paintEvent(QPaintEvent *event)
{
    QPainter painter(this);
    painter.setRenderHint(QPainter::Antialiasing);

    // Dessine un cercle
    QRect rect = this->rect().adjusted(10, 10, -10, -10);
    painter.setPen(QPen(Qt::black, 2));
    painter.drawEllipse(rect);

    // Dessine la valeur
    painter.drawText(rect, Qt::AlignCenter, QString::number(m_valeur));
}

void CompteurCirculaire::mousePressEvent(QMouseEvent *event)
{
    // Incrémente la valeur quand on clique
    setValeur(m_valeur + 1);
}
```

## Utilisation de Qt Designer

Qt Designer est un outil visuel pour créer des interfaces utilisateur sans écrire de code. Il génère des fichiers `.ui` qui peuvent être chargés dans votre application.

### Création d'une interface avec Qt Designer

1. Dans Qt Creator, créez un nouveau fichier `.ui`
2. Glissez-déposez des widgets depuis la palette sur votre formulaire
3. Organisez-les avec des layouts
4. Définissez les propriétés dans l'éditeur de propriétés
5. Sauvegardez le fichier

### Chargement d'une interface .ui dans le code

Il existe deux façons principales d'utiliser les fichiers `.ui` :

#### 1. Compilation directe avec la méthode `uic`

```cpp
// mainwindow.h
class MainWindow : public QMainWindow
{
    Q_OBJECT
public:
    MainWindow(QWidget *parent = nullptr);

private:
    Ui::MainWindow *ui;
};

// mainwindow.cpp
#include "ui_mainwindow.h"  // Généré automatiquement à partir de mainwindow.ui

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent), ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    // Accès aux widgets par leur nom
    connect(ui->pushButton, &QPushButton::clicked, [this]() {
        ui->label->setText("Bonjour " + ui->lineEdit->text());
    });
}
```

#### 2. Chargement dynamique avec QUiLoader

```cpp
#include <QUiLoader>
#include <QFile>

QWidget *chargerInterface(const QString &fichierUI)
{
    QFile file(fichierUI);
    file.open(QFile::ReadOnly);

    QUiLoader loader;
    QWidget *widget = loader.load(&file);
    file.close();

    // Recherche des widgets par leur nom
    QPushButton *bouton = widget->findChild<QPushButton*>("pushButton");
    QLineEdit *champTexte = widget->findChild<QLineEdit*>("lineEdit");
    QLabel *etiquette = widget->findChild<QLabel*>("label");

    if (bouton && champTexte && etiquette) {
        QObject::connect(bouton, &QPushButton::clicked, [champTexte, etiquette]() {
            etiquette->setText("Bonjour " + champTexte->text());
        });
    }

    return widget;
}
```

## Signaux et slots avec les widgets

### Connexions de base

```cpp
QPushButton *bouton = new QPushButton("Cliquer");
QLabel *etiquette = new QLabel("Texte initial");

// Connexion du signal clicked() du bouton au slot setText() de l'étiquette
QObject::connect(bouton, &QPushButton::clicked, [etiquette]() {
    etiquette->setText("Bouton cliqué !");
});

// Connexion avec un slot personnalisé dans une classe
class MaClasse : public QObject
{
    Q_OBJECT
public:
    MaClasse() {
        QPushButton *bouton = new QPushButton("Cliquer");
        connect(bouton, &QPushButton::clicked, this, &MaClasse::surClic);
    }

public slots:
    void surClic() {
        qDebug() << "Le bouton a été cliqué !";
    }
};
```

### Transmission de données via les signaux

```cpp
QSlider *curseur = new QSlider(Qt::Horizontal);
curseur->setRange(0, 100);
QSpinBox *spinbox = new QSpinBox();
spinbox->setRange(0, 100);

// Connexion bidirectionnelle
QObject::connect(curseur, &QSlider::valueChanged, spinbox, &QSpinBox::setValue);
QObject::connect(spinbox, QOverload<int>::of(&QSpinBox::valueChanged), curseur, &QSlider::setValue);
```

## Exemple complet : une petite application

Mettons tout ensemble pour créer une petite application de liste de tâches :

```cpp
#include <QApplication>
#include <QMainWindow>
#include <QVBoxLayout>
#include <QHBoxLayout>
#include <QListWidget>
#include <QLineEdit>
#include <QPushButton>
#include <QLabel>

class GestionnaireTaches : public QMainWindow
{
    Q_OBJECT
public:
    GestionnaireTaches(QWidget *parent = nullptr) : QMainWindow(parent)
    {
        setWindowTitle("Gestionnaire de tâches");
        resize(400, 300);

        // Création du widget central
        QWidget *centralWidget = new QWidget(this);
        setCentralWidget(centralWidget);

        // Création des widgets
        QLabel *labelTitre = new QLabel("Liste de tâches", this);
        labelTitre->setStyleSheet("font-size: 18px; font-weight: bold;");

        m_listeTaches = new QListWidget(this);

        m_champNouvelleTache = new QLineEdit(this);
        m_champNouvelleTache->setPlaceholderText("Nouvelle tâche...");

        QPushButton *boutonAjouter = new QPushButton("Ajouter", this);
        QPushButton *boutonSupprimer = new QPushButton("Supprimer", this);

        // Connexion des signaux/slots
        connect(boutonAjouter, &QPushButton::clicked, this, &GestionnaireTaches::ajouterTache);
        connect(m_champNouvelleTache, &QLineEdit::returnPressed, this, &GestionnaireTaches::ajouterTache);
        connect(boutonSupprimer, &QPushButton::clicked, this, &GestionnaireTaches::supprimerTachesSelectionnees);

        // Création des layouts
        QVBoxLayout *layoutPrincipal = new QVBoxLayout(centralWidget);
        layoutPrincipal->addWidget(labelTitre);
        layoutPrincipal->addWidget(m_listeTaches);

        QHBoxLayout *layoutSaisie = new QHBoxLayout();
        layoutSaisie->addWidget(m_champNouvelleTache);
        layoutSaisie->addWidget(boutonAjouter);

        layoutPrincipal->addLayout(layoutSaisie);
        layoutPrincipal->addWidget(boutonSupprimer);

        // Ajout de quelques tâches de démonstration
        m_listeTaches->addItem("Apprendre Qt Widgets");
        m_listeTaches->addItem("Créer une application");
        m_listeTaches->addItem("Maîtriser les layouts");
    }

private slots:
    void ajouterTache()
    {
        QString texte = m_champNouvelleTache->text().trimmed();
        if (!texte.isEmpty()) {
            m_listeTaches->addItem(texte);
            m_champNouvelleTache->clear();
            m_champNouvelleTache->setFocus();
        }
    }

    void supprimerTachesSelectionnees()
    {
        QList<QListWidgetItem*> items = m_listeTaches->selectedItems();
        for (QListWidgetItem *item : items) {
            delete m_listeTaches->takeItem(m_listeTaches->row(item));
        }
    }

private:
    QListWidget *m_listeTaches;
    QLineEdit *m_champNouvelleTache;
};

int main(int argc, char *argv[])
{
    QApplication app(argc, argv);

    GestionnaireTaches fenetre;
    fenetre.show();

    return app.exec();
}

#include "main.moc"  // Nécessaire pour les classes avec Q_OBJECT dans le main
```

## Bonnes pratiques pour le développement avec Qt Widgets

1. **Utilisez les layouts** plutôt que de positionner manuellement les widgets
2. **Séparez l'interface de la logique** en gardant la logique métier dans des classes séparées
3. **Utilisez Qt Designer** pour les interfaces complexes
4. **Créez des widgets personnalisés** pour les éléments d'interface réutilisables
5. **Nommez vos widgets** de manière cohérente pour faciliter la maintenance
6. **Utilisez les feuilles de style (CSS)** pour personnaliser l'apparence
7. **Testez votre interface** sur différentes plateformes et résolutions

## Conclusion

Le développement avec Qt Widgets offre une approche puissante et éprouvée pour créer des applications de bureau. En maîtrisant les concepts de base comme les widgets, les layouts, les signaux et slots, vous pouvez créer des interfaces utilisateur riches et fonctionnelles qui s'adaptent aux différentes plateformes.

Dans la prochaine section, nous explorerons Qt Quick et QML, une approche plus moderne pour créer des interfaces utilisateur avec Qt6.

⏭️ [Développement avec Qt Quick/QML](/03-interface-utilisateur/02-developpement-avec-qt-quick-qml.md)
