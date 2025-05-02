# 3.4 Internationalisation (i18n) avec Qt Linguist

L'internationalisation, souvent abrégée "i18n" (car il y a 18 lettres entre le "i" et le "n"), est le processus qui consiste à préparer votre application pour qu'elle puisse être facilement traduite dans différentes langues et adaptée à différentes cultures. Qt offre un excellent support pour l'internationalisation grâce à divers outils, dont Qt Linguist.

## Pourquoi internationaliser votre application ?

Internationaliser votre application présente plusieurs avantages :

- **Toucher un public plus large** : vos utilisateurs viennent probablement de différents pays
- **Améliorer l'expérience utilisateur** : les gens préfèrent utiliser des logiciels dans leur langue maternelle
- **Répondre aux exigences locales** : certains marchés exigent que les logiciels soient disponibles dans la langue locale
- **Se préparer à l'expansion** : même si vous ne ciblez qu'un marché actuellement, il est plus facile d'internationaliser dès le début

## Le processus d'internationalisation avec Qt

L'internationalisation avec Qt implique plusieurs étapes :

1. **Marquer les textes à traduire** dans votre code source
2. **Extraire ces textes** dans des fichiers de traduction (.ts)
3. **Traduire les textes** à l'aide de Qt Linguist
4. **Compiler les traductions** en fichiers binaires (.qm)
5. **Charger les traductions** dans votre application

Examinons chacune de ces étapes en détail.

## Marquer les textes à traduire

### La fonction tr()

Pour marquer les chaînes à traduire dans votre code C++, utilisez la fonction `tr()` :

```cpp
// Au lieu de ceci :
QPushButton *bouton = new QPushButton("Cliquer ici");

// Écrivez ceci :
QPushButton *bouton = new QPushButton(tr("Cliquer ici"));
```

La fonction `tr()` est définie dans la classe `QObject` et est disponible dans toutes les classes qui héritent de `QObject` (et utilisent la macro `Q_OBJECT`).

### Contexte de traduction

La fonction `tr()` utilise le nom de la classe comme contexte de traduction, ce qui aide les traducteurs à comprendre où apparaît le texte :

```cpp
class MainWindow : public QMainWindow
{
    Q_OBJECT
public:
    MainWindow()
    {
        // "MainWindow" sera le contexte de traduction
        setWindowTitle(tr("Mon Application"));
    }
};
```

### Traduction en dehors d'une classe

Si vous devez traduire du texte en dehors d'une classe (par exemple, dans la fonction `main()`), utilisez `QCoreApplication::translate()` :

```cpp
int main(int argc, char *argv[])
{
    QApplication app(argc, argv);

    // Premier paramètre : contexte, second paramètre : texte à traduire
    QString message = QCoreApplication::translate("Main", "Bienvenue dans mon application !");

    QMessageBox::information(nullptr, QCoreApplication::translate("Main", "Bienvenue"),
                            message);

    return app.exec();
}
```

### Marquer le texte dans les fichiers UI

Si vous utilisez Qt Designer pour créer votre interface, les textes des widgets sont automatiquement marqués pour la traduction, à condition que vous cochiez l'option "Traduisible" dans l'éditeur de propriétés :

![Option Traduisible dans Qt Designer](illustration_qt_designer_translate.png)

### Pluriels et contexte

Pour gérer les pluriels, utilisez `QCoreApplication::translate()` avec la méthode `Translate::tr()` :

```cpp
// n est le nombre d'éléments
QString message = QCoreApplication::translate("MainWindow",
                                            "%n fichier(s) trouvé(s)", "", n);
```

Si vous avez besoin de fournir un contexte supplémentaire aux traducteurs, utilisez la fonction `QT_TRANSLATE_NOOP()` :

```cpp
// Le contexte "MainWindow|FileDialog" aide le traducteur à comprendre
// que ce "Open" apparaît dans une boîte de dialogue de fichier
QString commande = tr(QT_TRANSLATE_NOOP("MainWindow|FileDialog", "Ouvrir"));
```

## Extraire les textes pour la traduction

### Utiliser lupdate

Pour extraire les textes à traduire, Qt fournit l'outil `lupdate`. Vous pouvez l'exécuter depuis Qt Creator ou depuis la ligne de commande :

1. Dans Qt Creator : Cliquez sur le menu Outils > Externe > Linguist > Mettre à jour les traductions (lupdate)

2. En ligne de commande :
   ```
   lupdate monprojet.pro -ts fr.ts en.ts es.ts
   ```

Cela créera ou mettra à jour des fichiers `.ts` pour chaque langue (ici français, anglais et espagnol).

### Configuration dans le fichier .pro

Pour intégrer la traduction à votre projet, ajoutez ces lignes à votre fichier `.pro` :

```
TRANSLATIONS = fr.ts \
              en.ts \
              es.ts

# Pour Qt 6, ajoutez ceci :
QT += linguisttools
```

## Traduire les textes avec Qt Linguist

### Démarrer Qt Linguist

Qt Linguist est un outil graphique qui facilite la traduction :

1. Lancez Qt Linguist depuis Qt Creator : Outils > Externe > Linguist > Linguist
2. Ou lancez-le directement depuis votre installation Qt

### Interface de Qt Linguist

![Interface de Qt Linguist](illustration_qt_linguist.png)

L'interface de Qt Linguist se compose de plusieurs sections :

- **Volet de contexte** : liste les contextes de traduction
- **Volet des chaînes** : liste les chaînes à traduire pour le contexte sélectionné
- **Volet de traduction** : permet d'entrer les traductions
- **Volet de phrases** : affiche les suggestions de traduction

### Processus de traduction

Pour traduire votre application avec Qt Linguist :

1. Ouvrez un fichier `.ts` dans Qt Linguist
2. Sélectionnez un contexte dans le volet de contexte
3. Pour chaque chaîne à traduire :
   - Lisez la chaîne source
   - Entrez la traduction dans le champ de traduction
   - Cliquez sur le bouton "Marquer comme terminé" (ou appuyez sur Ctrl+Entrée)
4. Passez à la chaîne suivante (Ctrl+J)
5. Enregistrez le fichier régulièrement (Ctrl+S)

### Qualité des traductions

Qt Linguist propose plusieurs fonctionnalités pour assurer la qualité des traductions :

- **Validation** : vérifie que les marqueurs de format (%1, %2, etc.) sont présents dans la traduction
- **Phrases** : suggère des traductions basées sur des traductions précédentes
- **Commentaires** : permet d'ajouter des notes pour les traducteurs
- **Vérification orthographique** : si configurée, vérifie l'orthographe des traductions

## Compiler les traductions

### Utiliser lrelease

Une fois les traductions terminées, vous devez les compiler en fichiers binaires `.qm` :

1. Dans Qt Creator : Outils > Externe > Linguist > Publier les traductions (lrelease)

2. En ligne de commande :
   ```
   lrelease monprojet.pro
   ```

Cela générera un fichier `.qm` pour chaque fichier `.ts`.

### Automatiser avec qmake

Pour que vos traductions soient automatiquement compilées lors de la compilation du projet, ajoutez ceci à votre fichier `.pro` :

```
CONFIG += lrelease
```

## Charger les traductions dans votre application

### Charger une traduction spécifique

Pour charger une traduction dans votre application :

```cpp
int main(int argc, char *argv[])
{
    QApplication app(argc, argv);

    // Création d'un traducteur
    QTranslator translator;

    // Chargement du fichier de traduction
    if (translator.load("fr.qm", ":/translations/")) {
        // Installation du traducteur
        QApplication::installTranslator(&translator);
    }

    MainWindow w;
    w.show();

    return app.exec();
}
```

### Détecter la langue du système

Pour charger automatiquement la traduction correspondant à la langue du système :

```cpp
int main(int argc, char *argv[])
{
    QApplication app(argc, argv);

    // Obtention de la locale du système
    QString locale = QLocale::system().name(); // Par exemple "fr_FR" pour français

    // Création d'un traducteur
    QTranslator translator;

    // Tentative de chargement de la traduction
    if (translator.load("monapp_" + locale, ":/translations/")) {
        QApplication::installTranslator(&translator);
    } else {
        // Fallback sur la locale principale (fr_FR -> fr)
        QString language = locale.split('_').first(); // "fr" dans "fr_FR"
        if (translator.load("monapp_" + language, ":/translations/")) {
            QApplication::installTranslator(&translator);
        }
    }

    MainWindow w;
    w.show();

    return app.exec();
}
```

### Inclure les traductions dans les ressources

Pour que les traductions soient embarquées dans votre exécutable, ajoutez-les à votre fichier de ressources `.qrc` :

```xml
<RCC>
    <qresource prefix="/translations">
        <file>monapp_fr.qm</file>
        <file>monapp_en.qm</file>
        <file>monapp_es.qm</file>
    </qresource>
</RCC>
```

### Changer la langue à la volée

Pour permettre à l'utilisateur de changer de langue sans redémarrer l'application :

```cpp
void MainWindow::changerLangue(const QString &langue)
{
    // Supprimer le traducteur actuel
    if (m_traducteur) {
        QApplication::removeTranslator(m_traducteur);
        delete m_traducteur;
    }

    // Créer un nouveau traducteur
    m_traducteur = new QTranslator(this);

    // Charger la nouvelle traduction
    if (m_traducteur->load("monapp_" + langue, ":/translations/")) {
        QApplication::installTranslator(m_traducteur);
    }

    // Mettre à jour l'interface
    ui->retranslateUi(this);

    // Si vous avez d'autres textes traduits ailleurs
    updateTexts();
}
```

La méthode `retranslateUi()` est générée automatiquement par `uic` pour les fichiers UI créés avec Qt Designer.

## Internationalisation au-delà du texte

L'internationalisation ne se limite pas à la traduction du texte. Voici d'autres aspects à considérer :

### Formats de date et heure

Utilisez les classes `QDate`, `QTime` et `QDateTime` avec `QLocale` pour formater correctement les dates et heures :

```cpp
QLocale locale(QLocale::French);  // ou QLocale::system() pour la locale du système
QDateTime dateHeure = QDateTime::currentDateTime();

// Format de date selon la locale
QString dateFormatee = locale.toString(dateHeure, QLocale::ShortFormat);
```

### Formats numériques

Utilisez `QLocale` pour formater correctement les nombres :

```cpp
QLocale locale(QLocale::French);
double nombre = 12345.67;

// Format de nombre selon la locale (12 345,67 en français, 12,345.67 en anglais)
QString nombreFormate = locale.toString(nombre, 'f', 2);
```

### Direction du texte (LTR/RTL)

Pour les langues écrites de droite à gauche (arabe, hébreu) :

```cpp
// Dans un QWidget ou QApplication
setLayoutDirection(Qt::RightToLeft);

// Dans un QTextEdit ou QTextDocument
textEdit->document()->setDefaultTextOption(QTextOption(Qt::AlignRight));
```

### Images et icônes culturellement adaptées

Adaptez les images et icônes pour différentes cultures :

```cpp
QIcon getAdaptedIcon(const QString &nom)
{
    QLocale locale = QLocale::system();
    QString cheminIcon = ":/icons/";

    // Ajouter le code de langue si une version spécifique existe
    if (QFile::exists(cheminIcon + nom + "_" + locale.name() + ".png")) {
        return QIcon(cheminIcon + nom + "_" + locale.name() + ".png");
    }

    // Sinon utiliser l'icône par défaut
    return QIcon(cheminIcon + nom + ".png");
}
```

## Bonnes pratiques d'internationalisation

### Planifier dès le début

- **Pensez international** dès la conception de votre application
- **Évitez les textes codés en dur** dans votre code
- **Prévoyez de l'espace** pour les traductions (qui peuvent être plus longues que le texte original)

### Pour les développeurs

- **Utilisez toujours `tr()`** pour les chaînes visibles par l'utilisateur
- **Ajoutez des commentaires** pour les traducteurs sur les chaînes ambiguës
- **Utilisez des variables** pour les ordres de mots différents :
  ```cpp
  tr("%1 a trouvé %2").arg(nomUtilisateur).arg(nomFichier);
  ```
- **Testez** avec différentes langues, y compris celles qui se lisent de droite à gauche

### Pour les traducteurs

- **Comprenez le contexte** de chaque chaîne
- **Maintenez la cohérence** dans les termes utilisés
- **Respectez les marqueurs** (%1, %2, etc.)
- **Adaptez culturellement** plutôt que de traduire littéralement

## Exemple complet

Voici un exemple complet d'application internationalisée :

```cpp
// main.cpp
#include <QApplication>
#include <QTranslator>
#include <QLocale>
#include <QLibraryInfo>
#include "mainwindow.h"

int main(int argc, char *argv[])
{
    QApplication app(argc, argv);

    // Traducteur pour l'application
    QTranslator appTranslator;
    QString locale = QLocale::system().name();

    if (appTranslator.load("myapp_" + locale, ":/translations")) {
        app.installTranslator(&appTranslator);
    }

    // Traducteur pour les dialogues standard de Qt
    QTranslator qtTranslator;
    if (qtTranslator.load("qt_" + locale, QLibraryInfo::path(QLibraryInfo::TranslationsPath))) {
        app.installTranslator(&qtTranslator);
    }

    MainWindow w;
    w.show();

    return app.exec();
}

// mainwindow.h
class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    MainWindow(QWidget *parent = nullptr);

private slots:
    void changerLangue();
    void ajouterFichier();
    void actualiserInterface();

private:
    Ui::MainWindow *ui;
    QTranslator *m_traducteur;
    int m_nombreFichiers;
};

// mainwindow.cpp
MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent), ui(new Ui::MainWindow), m_traducteur(nullptr), m_nombreFichiers(0)
{
    ui->setupUi(this);

    // Menu Langue
    QMenu *menuLangue = new QMenu(tr("Langue"), this);
    menuLangue->addAction("Français", [this](){ changerLangue("fr"); });
    menuLangue->addAction("English", [this](){ changerLangue("en"); });
    menuLangue->addAction("Español", [this](){ changerLangue("es"); });

    ui->menuBar->addMenu(menuLangue);

    // Connecter les signaux
    connect(ui->actionAjouter, &QAction::triggered, this, &MainWindow::ajouterFichier);

    actualiserInterface();
}

void MainWindow::changerLangue(const QString &langue)
{
    // Supprimer le traducteur précédent
    if (m_traducteur) {
        QApplication::removeTranslator(m_traducteur);
        delete m_traducteur;
    }

    m_traducteur = new QTranslator(this);

    if (m_traducteur->load("myapp_" + langue, ":/translations")) {
        QApplication::installTranslator(m_traducteur);
    }

    // Mettre à jour l'interface
    ui->retranslateUi(this);
    actualiserInterface();
}

void MainWindow::ajouterFichier()
{
    m_nombreFichiers++;
    actualiserInterface();
}

void MainWindow::actualiserInterface()
{
    // Utilisation du pluriel
    ui->labelStatut->setText(tr("%n fichier(s) ajouté(s)", "", m_nombreFichiers));

    // Format de date selon la locale
    QLocale locale;
    QString date = locale.toString(QDate::currentDate(), locale.dateFormat());
    ui->labelDate->setText(tr("Date : %1").arg(date));
}
```

## Conclusion

L'internationalisation est un processus crucial qui permet de rendre votre application accessible à un public international. Qt offre un ensemble d'outils puissants pour faciliter ce processus, notamment Qt Linguist pour la traduction des textes.

En suivant les bonnes pratiques présentées dans ce tutoriel, vous pourrez créer des applications véritablement internationales qui s'adaptent à la langue et à la culture de vos utilisateurs. N'oubliez pas que l'internationalisation va au-delà de la simple traduction de texte ; elle implique également l'adaptation des formats de date, de nombre, et d'autres aspects culturels.

Avec Qt6, internationaliser votre application dès le début du développement est un investissement qui facilitera son évolution future et élargira considérablement son audience potentielle.
