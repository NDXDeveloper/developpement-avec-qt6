# 11. Bonnes Pratiques Qt6

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

## Introduction aux bonnes pratiques

Bienvenue dans ce chapitre consacré aux bonnes pratiques lors du développement avec Qt6. Que vous soyez débutant ou développeur plus expérimenté, l'adoption de bonnes pratiques vous permettra de créer des applications Qt6 plus robustes, maintenables et performantes.

## Pourquoi les bonnes pratiques sont importantes

Le développement avec Qt6 offre une grande flexibilité, mais cette liberté s'accompagne de responsabilités. Les bonnes pratiques vous aident à :

- Éviter les fuites de mémoire et les crashs
- Améliorer les performances de votre application
- Faciliter la maintenance et l'évolution de votre code
- Permettre une meilleure collaboration en équipe
- Créer des applications plus sécurisées

## Principes généraux

### Respecter le cycle de vie des objets Qt

Qt utilise un système parent-enfant pour gérer automatiquement la destruction des objets. Lorsqu'un objet parent est détruit, tous ses enfants le sont également.

```cpp
// Bonne pratique : Le bouton sera automatiquement détruit avec la fenêtre
QPushButton* button = new QPushButton("Cliquez-moi", this);

// À éviter : Objet sans parent qui devra être géré manuellement
QPushButton* buttonOrphelin = new QPushButton("Bouton orphelin");
// ...plus tard, il faut penser à le détruire
delete buttonOrphelin;
```

### Utiliser les mécanismes Qt plutôt que leurs équivalents C++

Qt offre des alternatives optimisées à de nombreuses fonctionnalités standard de C++ :

```cpp
// Préférer QString à std::string
QString texte = "Bonjour";

// Préférer QList à std::vector
QList<int> nombres;

// Préférer les conteneurs Qt qui sont optimisés pour l'écosystème Qt
QMap<QString, int> dictionnaire;
```

### Comprendre le système de propriété de Qt

Qt utilise un système de propriétés très puissant. Exploitez-le :

```cpp
// Définir une propriété
setProperty("etatActif", true);

// Lire une propriété
bool estActif = property("etatActif").toBool();

// Utiliser les propriétés pour le style CSS
setStyleSheet("QPushButton[etatActif='true'] { background-color: green; }");
```

## Éviter les pièges courants

### Ne pas mélanger les threads UI et de travail

L'interface utilisateur de Qt s'exécute dans le thread principal. Les opérations longues doivent être déplacées dans des threads séparés :

```cpp
// Mauvaise pratique : bloquer l'interface utilisateur
void MaFenetre::surBoutonClique() {
    // Cette opération longue bloquera l'interface
    operationLongue();
}

// Bonne pratique : utiliser un thread séparé
void MaFenetre::surBoutonClique() {
    QThread* thread = new QThread();
    MonTravailleur* travailleur = new MonTravailleur();
    travailleur->moveToThread(thread);

    connect(thread, &QThread::started, travailleur, &MonTravailleur::executer);
    connect(travailleur, &MonTravailleur::termine, thread, &QThread::quit);
    connect(thread, &QThread::finished, travailleur, &QObject::deleteLater);
    connect(thread, &QThread::finished, thread, &QObject::deleteLater);

    thread->start();
}
```

### Éviter les connexions de signal/slot dangereuses

```cpp
// À éviter : connexion qui peut entraîner des boucles infinies
connect(slider, &QSlider::valueChanged, spinBox, &QSpinBox::setValue);
connect(spinBox, &QSpinBox::valueChanged, slider, &QSlider::setValue);

// Solution : utiliser un blocage temporaire
connect(slider, &QSlider::valueChanged, this, [this](int value) {
    // Bloquer temporairement les signaux du spinBox
    QSignalBlocker blocker(spinBox);
    spinBox->setValue(value);
});

connect(spinBox, &QSpinBox::valueChanged, this, [this](int value) {
    QSignalBlocker blocker(slider);
    slider->setValue(value);
});
```

## Optimisation des ressources

### Charger les ressources efficacement

Qt offre un système de ressources compilées (.qrc) qui est plus efficace que le chargement de fichiers externes :

```cpp
// Dans le fichier ressources.qrc
<RCC>
    <qresource prefix="/images">
        <file>logo.png</file>
    </qresource>
</RCC>

// Dans le code - accès efficace
QPixmap pixmap(":/images/logo.png");
```

### Réutiliser les composants coûteux

Certains objets Qt sont coûteux à créer. Réutilisez-les lorsque possible :

```cpp
// Mauvaise pratique : créer un pinceau à chaque dessin
void MaWidget::paintEvent(QPaintEvent*) {
    QPainter painter(this);
    QBrush brush(Qt::red);  // Création coûteuse à chaque dessin
    painter.fillRect(rect(), brush);
}

// Bonne pratique : réutiliser le pinceau
class MaWidget : public QWidget {
    Q_OBJECT
private:
    QBrush m_brush;
public:
    MaWidget(QWidget* parent = nullptr) : QWidget(parent), m_brush(Qt::red) {}

    void paintEvent(QPaintEvent*) override {
        QPainter painter(this);
        painter.fillRect(rect(), m_brush);  // Réutilisation
    }
};
```

## Documentation et lisibilité

### Documenter votre code Qt

Une bonne documentation aide à comprendre rapidement l'intention du code :

```cpp
/**
 * @brief Classe représentant la fenêtre principale de l'application
 *
 * Cette fenêtre gère l'affichage des données utilisateur et
 * permet la navigation entre les différentes vues.
 */
class MainWindow : public QMainWindow {
    Q_OBJECT

public:
    /**
     * @brief Constructeur de la fenêtre principale
     * @param parent Le widget parent (nullptr par défaut)
     */
    explicit MainWindow(QWidget *parent = nullptr);

    // ...
};
```

### Nommage cohérent

Adoptez une convention de nommage cohérente pour votre code Qt :

```cpp
// Conventions communes en Qt
class MaClasse;               // Classes en PascalCase
void maFonction();            // Méthodes en camelCase
int m_variableMembre;         // Variables membres préfixées par m_
QObject* parent = nullptr;    // Paramètres en camelCase
const int MAX_TAILLE = 100;   // Constantes en MAJUSCULES_AVEC_UNDERSCORES
```

## Testabilité

### Concevoir pour les tests

Structurez votre code pour qu'il soit facilement testable :

```cpp
// Difficile à tester : logique métier dans l'interface utilisateur
void MaFenetre::surBoutonCalculer() {
    double resultat = valeur1->text().toDouble() + valeur2->text().toDouble();
    resultatLabel->setText(QString::number(resultat));
}

// Facile à tester : logique métier séparée
class Calculateur {
public:
    double additionner(double a, double b) { return a + b; }
};

void MaFenetre::surBoutonCalculer() {
    double a = valeur1->text().toDouble();
    double b = valeur2->text().toDouble();
    double resultat = m_calculateur.additionner(a, b);
    resultatLabel->setText(QString::number(resultat));
}
```

### Tester régulièrement

Intégrez des tests dans votre workflow de développement :

```cpp
// Test unitaire avec Qt Test
#include <QtTest>

class TestCalculateur : public QObject {
    Q_OBJECT

private slots:
    void testAddition() {
        Calculateur calc;
        QCOMPARE(calc.additionner(2.0, 3.0), 5.0);
    }
};

QTEST_MAIN(TestCalculateur)
#include "test_calculateur.moc"
```

## Conclusion

L'adoption de bonnes pratiques dans vos projets Qt6 est un investissement qui porte ses fruits sur le long terme. Ces principes vous aideront à éviter les pièges courants et à construire des applications robustes et maintenables.

Dans les sections suivantes, nous approfondirons des aspects spécifiques de ces bonnes pratiques, en commençant par la gestion mémoire et les pointeurs intelligents.

⏭️ [Gestion mémoire et pointeurs intelligents](/11-bonnes-pratiques-qt6/01-gestion-memoire-et-pointeurs-intelligents.md)
