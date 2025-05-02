# 11.2 Optimisation des performances

## Introduction à l'optimisation des performances

Créer une application Qt6 fonctionnelle est une première étape importante, mais pour offrir une expérience utilisateur fluide et réactive, l'optimisation des performances devient essentielle. Ce chapitre vous guidera à travers les concepts fondamentaux et les techniques pratiques pour améliorer les performances de vos applications Qt6.

## Principes fondamentaux d'optimisation

### Optimiser intelligemment

Avant de commencer à optimiser votre code, gardez à l'esprit ces principes :

1. **Mesurer avant d'optimiser** : Ne devinez pas où se trouvent les problèmes de performance.
2. **Se concentrer sur les 20% de code qui causent 80% des problèmes** : C'est la loi de Pareto appliquée à l'optimisation.
3. **Préserver la lisibilité du code** : Une optimisation qui rend le code incompréhensible est rarement judicieuse.

## Outils de profilage

Qt6 offre plusieurs outils pour identifier les goulots d'étranglement dans votre application :

### Qt Creator Performance Analyzer

Qt Creator intègre des outils de profilage puissants :

```cpp
// Pour activer le profilage dans votre projet :
// 1. Ouvrez le fichier .pro de votre projet
// 2. Ajoutez la ligne suivante :
CONFIG += qmltestcase

// Ensuite lancez votre application avec le profileur activé
// (Depuis Qt Creator: Déboguer > Start Performance Analysis)
```

### Chronomètrer des sections de code

La classe `QElapsedTimer` permet de mesurer précisément le temps d'exécution :

```cpp
#include <QElapsedTimer>
#include <QDebug>

void fonctionAAnalyser() {
    QElapsedTimer timer;
    timer.start();

    // Code à mesurer
    for (int i = 0; i < 1000; ++i) {
        operationCouteuse();
    }

    qDebug() << "Temps écoulé:" << timer.elapsed() << "ms";
}
```

## Optimisation des interfaces graphiques

### Réduire les retraçages (repaints)

Les opérations de dessin sont coûteuses. Minimisez-les en :

```cpp
// Au lieu de mettre à jour tout le widget
void MauvaisWidget::mettreAJour() {
    // Force le redessin complet
    update();
}

// Mettre à jour seulement la partie nécessaire
void BonWidget::mettreAJour() {
    // Redessine uniquement la zone modifiée
    update(zoneModifiee);
}
```

### Éviter les redessinages inutiles

Utilisez `setUpdatesEnabled()` pour désactiver temporairement les redessinages :

```cpp
// Lors de mises à jour multiples
void mettreAJourMultiplesElements() {
    // Désactiver les mises à jour
    setUpdatesEnabled(false);

    // Effectuer plusieurs changements
    element1->setText("Nouveau texte");
    element2->setValue(50);
    element3->setVisible(true);
    // ... etc.

    // Réactiver les mises à jour (un seul redessinage se produira)
    setUpdatesEnabled(true);
}
```

### Utiliser le double buffering

Pour les widgets complexes, le double buffering permet d'éviter le scintillement :

```cpp
void MonWidget::paintEvent(QPaintEvent* event) {
    // Création d'un tampon hors écran
    QPixmap buffer(size());
    buffer.fill(Qt::transparent);

    // Dessiner sur le tampon
    QPainter painter(&buffer);
    dessinerElementsComplexes(&painter);

    // Dessiner le tampon sur le widget
    QPainter widgetPainter(this);
    widgetPainter.drawPixmap(0, 0, buffer);
}
```

### Optimiser les animations

Pour des animations fluides :

```cpp
// Préférer QPropertyAnimation pour les animations
QPropertyAnimation* animation = new QPropertyAnimation(objet, "geometry");
animation->setDuration(300);
animation->setStartValue(QRect(0, 0, 100, 30));
animation->setEndValue(QRect(100, 100, 200, 60));

// Utiliser une courbe d'accélération pour un mouvement naturel
animation->setEasingCurve(QEasingCurve::OutCubic);

animation->start();
```

## Optimisation du code

### Éviter les allocations inutiles

Les allocations dynamiques sont coûteuses :

```cpp
// MAUVAIS: Allocation répétée dans une boucle
void fonctionInoptimale() {
    for (int i = 0; i < 1000; ++i) {
        QString* texte = new QString("texte");
        // ...
        delete texte;
    }
}

// MIEUX: Réutiliser le même objet
void fonctionOptimisee() {
    QString texte;
    for (int i = 0; i < 1000; ++i) {
        texte = "texte";
        // ...
    }
}
```

### Utiliser la réservation de mémoire

Pour les conteneurs, réservez la mémoire à l'avance :

```cpp
// Sans réservation
QVector<int> vecteurInoptimal;
for (int i = 0; i < 10000; ++i) {
    vecteurInoptimal.append(i);  // Peut causer plusieurs réallocations
}

// Avec réservation
QVector<int> vecteurOptimal;
vecteurOptimal.reserve(10000);  // Alloue la mémoire une seule fois
for (int i = 0; i < 10000; ++i) {
    vecteurOptimal.append(i);
}
```

### Utiliser les bonnes structures de données

Choisir la structure adaptée à l'utilisation :

```cpp
// Si vous avez besoin d'accès rapide par index
QVector<QString> listeNoms;  // Accès O(1) par index

// Si vous avez besoin d'insertions/suppressions rapides
QList<QString> listeModifiable;  // Insertion/suppression rapide

// Si vous avez besoin de recherches par clé
QHash<QString, Utilisateur> tableUtilisateurs;  // Recherche O(1)
```

### Utiliser les références et le passage par constante

```cpp
// INEFFICACE: Copie inutile des gros objets
void fonctionInoptimale(QString texte, QImage image) {
    // ...
}

// EFFICACE: Utilisation de références constantes
void fonctionOptimisee(const QString& texte, const QImage& image) {
    // ...
}
```

## Multithreading pour les performances

### Déplacer les opérations lourdes hors du thread UI

```cpp
void MaFenetre::chargerDonnees() {
    // Créer un thread de travail
    QThread* thread = new QThread();
    Travailleur* travailleur = new Travailleur();
    travailleur->moveToThread(thread);

    // Connecter les signaux et slots
    connect(thread, &QThread::started, travailleur, &Travailleur::effectuerTacheLourde);
    connect(travailleur, &Travailleur::termine, this, &MaFenetre::afficherResultats);
    connect(travailleur, &Travailleur::termine, thread, &QThread::quit);
    connect(thread, &QThread::finished, travailleur, &QObject::deleteLater);
    connect(thread, &QThread::finished, thread, &QObject::deleteLater);

    // Démarrer le thread
    thread->start();
}
```

### Utiliser QtConcurrent pour les opérations parallèles

```cpp
#include <QtConcurrent>

void MaFenetre::traiterImages() {
    QList<QImage> images = chargerImages();

    // Traiter les images en parallèle
    QFuture<QList<QImage>> futur = QtConcurrent::mapped(images,
        [](const QImage& image) {
            return appliquerFiltre(image);
        }
    );

    // Configurer un watcher pour être notifié de la fin
    QFutureWatcher<QList<QImage>>* watcher = new QFutureWatcher<QList<QImage>>(this);
    connect(watcher, &QFutureWatcher<QList<QImage>>::finished,
            this, &MaFenetre::afficherImagesTraitees);
    watcher->setFuture(futur);
}
```

## Optimisation des modèles (Model/View)

### Utiliser les rôles de données efficacement

```cpp
QVariant MonModele::data(const QModelIndex& index, int role) const {
    // Optimisation: ne calculer que ce qui est nécessaire
    switch (role) {
        case Qt::DisplayRole:
            return m_elements[index.row()].nom;

        case Qt::DecorationRole:
            // Ne charger l'icône que si elle est visible
            return m_elements[index.row()].icone;

        // Ne pas calculer les données non demandées
        // ...
    }

    return QVariant();
}
```

### Charger les données progressivement

```cpp
class ModeleOptimise : public QAbstractItemModel {
private:
    QList<DonneesSommaire> m_sommaires;  // Données légères pour tous les éléments
    QHash<int, DonneesCompletes> m_donneesChargees;  // Cache de données complètes

public:
    QVariant data(const QModelIndex& index, int role) const override {
        int ligne = index.row();

        // Si on a besoin des données complètes et qu'elles ne sont pas chargées
        if (roleNecessiteDonneesCompletes(role) && !m_donneesChargees.contains(ligne)) {
            // Charger à la demande
            m_donneesChargees[ligne] = chargerDonneesCompletes(ligne);
        }

        // Renvoyer les données appropriées
        // ...
    }
};
```

## Optimisation du chargement de l'application

### Utiliser le chargement différé (lazy loading)

```cpp
class MonApplication : public QApplication {
private:
    // Pointeur, pas d'instance directe
    QScopedPointer<ModuleLourd> m_moduleOptionnel;

public:
    void initialiserModulesEssentiels() {
        // Charger uniquement ce qui est nécessaire au démarrage
    }

    void chargerModuleLourdSiNecessaire() {
        if (!m_moduleOptionnel) {
            // Charger uniquement quand nécessaire
            m_moduleOptionnel.reset(new ModuleLourd());
        }
        return m_moduleOptionnel.data();
    }
};
```

### Charger les ressources efficacement

```cpp
// Dans un fichier .qrc (moins d'accès disque)
<RCC>
    <qresource prefix="/images">
        <file>icones/fichier.png</file>
        <file>icones/dossier.png</file>
    </qresource>
</RCC>

// Charger depuis les ressources compilées (plus rapide)
QIcon iconeFichier(":/images/icones/fichier.png");
```

## Optimisations spécifiques à Qt Quick (QML)

### Utiliser les images de façon optimale

```qml
// Éviter de redimensionner les images à l'affichage
// MAUVAIS
Image {
    source: "grande_image.png"
    width: 64
    height: 64
}

// BON
Image {
    source: "icone_64x64.png"  // Image déjà à la bonne taille
}
```

### Réduire la complexité des items

```qml
// Au lieu d'utiliser de nombreux éléments individuels
// MAUVAIS
Rectangle {
    // 100 éléments individuels
    Repeater {
        model: 100
        Rectangle {
            // propriétés...
        }
    }
}

// BON - Utiliser des Canvas ou des composants optimisés
Rectangle {
    Canvas {
        onPaint: {
            var ctx = getContext("2d");
            // Dessiner les 100 éléments directement
        }
    }
}
```

## Exemple complet : Optimisation d'une application

Voici un exemple montrant plusieurs techniques d'optimisation :

```cpp
class ApplicationOptimisee : public QMainWindow {
    Q_OBJECT

private:
    QVector<QImage> m_imagesCache;
    QFutureWatcher<void>* m_watcher;

public:
    ApplicationOptimisee(QWidget* parent = nullptr) : QMainWindow(parent) {
        // Optimisation du démarrage
        QSettings settings;
        if (settings.value("premierLancement", true).toBool()) {
            // Initialisation complète au premier lancement
            initialiserCompletement();
            settings.setValue("premierLancement", false);
        } else {
            // Chargement minimal pour les lancements suivants
            initialiserMinimalement();
        }

        // Configurer les connexions pour le chargement différé
        m_watcher = new QFutureWatcher<void>(this);
        connect(m_watcher, &QFutureWatcher<void>::finished,
                this, &ApplicationOptimisee::finaliserInitialisation);
    }

    void initialiserMinimalement() {
        // Charger uniquement l'UI et les ressources essentielles
        centralWidget()->setUpdatesEnabled(false);
        setupUi();
        m_imagesCache.reserve(100);  // Réserver de l'espace pour le cache

        // Lancer le chargement du reste en arrière-plan
        QFuture<void> future = QtConcurrent::run([this]() {
            chargerRessourcesSecondaires();
        });
        m_watcher->setFuture(future);
    }

    void chargerRessourcesSecondaires() {
        // Charger les ressources non-critiques en arrière-plan
        for (const QString& chemin : listeImages()) {
            QImage img(chemin);
            m_imagesCache.append(img);
        }
    }

    void finaliserInitialisation() {
        // Mettre à jour l'interface avec les ressources chargées
        mettreAJourVignettes();
        centralWidget()->setUpdatesEnabled(true);
    }

    // Optimisation du dessin
    void mettreAJourVignettes() {
        // Mettre à jour toutes les vignettes d'un coup
        QSignalBlocker blocker(listView);
        modele->beginResetModel();
        // Mise à jour...
        modele->endResetModel();
    }
};
```

## Conseils d'optimisation rapides

1. **Analysez avant d'optimiser** : Utilisez les outils de profilage pour identifier les vrais problèmes.
2. **Minimisez les allocations dynamiques** : Réutilisez les objets quand c'est possible.
3. **Réduisez les retraçages** : Regroupez les mises à jour visuelles.
4. **Utilisez les threads** : Déplacez les tâches lourdes hors du thread principal.
5. **Optimisez les structures de données** : Choisissez la bonne structure pour chaque cas d'utilisation.
6. **Chargez progressivement** : Ne chargez que ce qui est nécessaire, quand c'est nécessaire.
7. **Utilisez le système de ressources** : Les ressources compilées (.qrc) sont plus rapides à charger.
8. **Mettez en cache les résultats** : Évitez de recalculer ce qui peut être mémorisé.

## Conclusion

L'optimisation des performances est un processus continu qui doit être intégré tout au long du cycle de développement. En appliquant les techniques présentées dans ce chapitre, vous pourrez créer des applications Qt6 rapides et réactives qui offriront une excellente expérience utilisateur.

Dans le prochain chapitre, nous aborderons comment créer des architectures d'applications robustes avec Qt6.
