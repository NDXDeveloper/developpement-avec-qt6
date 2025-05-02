# 11.1 Gestion mémoire et pointeurs intelligents

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

## Introduction à la gestion mémoire en Qt

La gestion de la mémoire est l'un des aspects les plus importants du développement d'applications robustes avec Qt6. Une mauvaise gestion de la mémoire peut entraîner des fuites mémoire, des crashs inattendus et des performances dégradées. Heureusement, Qt6 fournit plusieurs mécanismes pour faciliter cette tâche, même pour les débutants.

## Le système parent-enfant de Qt

Avant de parler des pointeurs intelligents, rappelons que Qt possède un mécanisme fondamental de gestion mémoire : le système parent-enfant.

```cpp
// Exemple: Création de widgets avec un parent
QWidget* fenetre = new QWidget();       // Widget parent
QPushButton* bouton = new QPushButton("Cliquer", fenetre); // Widget enfant

// Lors de la destruction de la fenêtre, le bouton sera automatiquement détruit
delete fenetre; // Le bouton est aussi détruit ici, pas besoin de delete bouton
```

Quand un objet QObject est créé avec un parent, ce dernier prend la responsabilité de le détruire. C'est la base de la gestion mémoire en Qt, mais elle présente quelques limitations :

- Elle ne fonctionne qu'avec les classes dérivées de QObject
- Elle n'est pas adaptée à toutes les situations (objets partagés, transfert de propriété, etc.)

C'est là que les pointeurs intelligents entrent en jeu.

## Qu'est-ce qu'un pointeur intelligent ?

Un pointeur intelligent est un objet qui se comporte comme un pointeur normal mais qui gère automatiquement la désallocation de la mémoire. En Qt6, plusieurs types de pointeurs intelligents sont disponibles :

### 1. QPointer - Le pointeur faible

`QPointer` est un pointeur qui devient automatiquement `nullptr` lorsque l'objet pointé est détruit.

```cpp
#include <QPointer>

void fonctionExemple() {
    QWidget* widget = new QWidget();

    // Création d'un QPointer pointant vers le widget
    QPointer<QWidget> pointeurFaible = widget;

    // Vérification que le widget existe toujours
    if (pointeurFaible) {
        pointeurFaible->show();
    }

    // Si quelqu'un d'autre détruit le widget...
    delete widget;

    // Maintenant pointeurFaible vaut nullptr automatiquement
    if (!pointeurFaible) {
        qDebug() << "Le widget a été détruit";
    }
}
```

**Cas d'utilisation** : `QPointer` est idéal quand vous avez besoin de référencer un objet Qt sans en être propriétaire.

### 2. QScopedPointer - Propriété exclusive

`QScopedPointer` détruit automatiquement l'objet qu'il contient lorsqu'il sort de sa portée (scope).

```cpp
#include <QScopedPointer>

void fonctionExemple() {
    // La ressource sera automatiquement libérée à la fin de la fonction
    QScopedPointer<QFile> fichier(new QFile("data.txt"));

    if (fichier->open(QIODevice::ReadOnly)) {
        // Utiliser le fichier...
    }

    // Pas besoin de delete - la destruction est automatique à la fin de la fonction
}
```

**Cas d'utilisation** : `QScopedPointer` est parfait pour des ressources qui ne doivent pas survivre au-delà de leur scope et qui ne sont pas partagées.

### 3. QSharedPointer - Propriété partagée

`QSharedPointer` maintient un compteur des références pointant vers un objet et ne le détruit que lorsque ce compteur atteint zéro.

```cpp
#include <QSharedPointer>

// Fonction qui crée et retourne un pointeur partagé
QSharedPointer<QNetworkReply> demarrerRequete() {
    QNetworkAccessManager* manager = new QNetworkAccessManager();
    QNetworkReply* reply = manager->get(QNetworkRequest(QUrl("https://qt.io")));

    // Création d'un QSharedPointer avec deleter personnalisé pour nettoyer le manager
    return QSharedPointer<QNetworkReply>(reply, [manager](QNetworkReply* r) {
        r->deleteLater();
        manager->deleteLater();
    });
}

void fonctionExemple() {
    // Obtention d'un pointeur partagé
    QSharedPointer<QNetworkReply> reponse = demarrerRequete();

    // Autre fonction qui utilise aussi ce pointeur partagé
    traiterReponse(reponse);

    // Lorsque reponse et toutes les copies sortiront de portée,
    // l'objet QNetworkReply sera nettoyé automatiquement
}
```

**Cas d'utilisation** : `QSharedPointer` est idéal pour les ressources partagées entre plusieurs parties du code.

### 4. QWeakPointer - Référence faible à un QSharedPointer

`QWeakPointer` permet de référencer un objet géré par un `QSharedPointer` sans augmenter son compteur de références.

```cpp
#include <QSharedPointer>
#include <QWeakPointer>

class GestionnaireCache {
private:
    QHash<QString, QWeakPointer<QImage>> m_cache;
public:
    QSharedPointer<QImage> obtenirImage(const QString& cle) {
        // Essayer de récupérer du cache
        QWeakPointer<QImage> reference = m_cache.value(cle);
        QSharedPointer<QImage> imagePartagee = reference.toStrongRef();

        if (imagePartagee) {
            // L'image est toujours en vie dans le cache
            return imagePartagee;
        }

        // Créer une nouvelle image
        imagePartagee = QSharedPointer<QImage>(new QImage(cle));

        // Stocker une référence faible dans le cache
        m_cache[cle] = imagePartagee;

        return imagePartagee;
    }
};
```

**Cas d'utilisation** : `QWeakPointer` est parfait pour implémenter des caches ou des structures de données qui ne doivent pas empêcher la libération des objets.

## Pointeurs intelligents C++11/14/17 avec Qt

Qt6 est totalement compatible avec les pointeurs intelligents standard de C++ :

### 1. std::unique_ptr

Équivalent à `QScopedPointer` mais du standard C++.

```cpp
#include <memory>

void fonctionExemple() {
    std::unique_ptr<QFile> fichier = std::make_unique<QFile>("data.txt");

    if (fichier->open(QIODevice::ReadOnly)) {
        // Utiliser le fichier...
    }

    // Destruction automatique à la fin de la fonction
}
```

### 2. std::shared_ptr

Équivalent à `QSharedPointer` mais du standard C++.

```cpp
#include <memory>

void fonctionExemple() {
    std::shared_ptr<QPixmap> image = std::make_shared<QPixmap>("image.png");

    // Partager avec d'autres fonctions
    afficherImage(image);
    modifierImage(image);

    // L'image sera détruite automatiquement quand toutes
    // les références shared_ptr disparaîtront
}
```

> **Note importante** : Pour les objets Qt qui héritent de QObject, évitez d'utiliser `std::shared_ptr` avec `deleteLater()`. Dans ce cas, préférez `QSharedPointer` qui a une meilleure intégration avec le système d'événements de Qt.

## Bonnes pratiques pour la gestion mémoire

### 1. Choisir le bon outil pour chaque situation

```
- Objet appartenant à un parent QObject → Système parent-enfant
- Objet à propriété exclusive → QScopedPointer ou std::unique_ptr
- Objet partagé → QSharedPointer ou std::shared_ptr
- Référence sans propriété à un QObject → QPointer
- Référence faible à un objet partagé → QWeakPointer
```

### 2. Éviter les mélanges inappropriés

```cpp
// À ÉVITER: Mélanger le système parent-enfant avec les pointeurs intelligents
QWidget* parent = new QWidget();
QSharedPointer<QPushButton> bouton = QSharedPointer<QPushButton>(new QPushButton(parent));

// Le parent pourrait détruire le bouton, mais QSharedPointer essaiera aussi de le détruire !
```

### 3. Préférer la création sur la pile quand c'est possible

```cpp
// Préférer ceci (allocation sur la pile)
QFile fichier("data.txt");
if (fichier.open(QIODevice::ReadOnly)) {
    // ...
}
// Destruction automatique à la fin de la portée

// À l'allocation dynamique avec pointeur intelligent
QScopedPointer<QFile> fichier(new QFile("data.txt"));
if (fichier->open(QIODevice::ReadOnly)) {
    // ...
}
```

### 4. Comprendre le cycle de vie des objets QObject

Quand on utilise `deleteLater()`, l'objet n'est pas immédiatement détruit :

```cpp
void fonctionExemple() {
    QTimer* timer = new QTimer();
    timer->setSingleShot(true);
    timer->start(1000);

    // Programmer la destruction
    timer->deleteLater();

    // L'objet timer existe toujours ici !
    // Il sera détruit lors du prochain traitement de la boucle d'événements
}
```

## Détection des fuites mémoire

Qt offre des outils pour détecter les fuites mémoire dans vos applications :

```cpp
// Activer le débogage mémoire de Qt (en début de programme, avant QApplication)
qputenv("QT_ENABLE_LEAK_CHECK", "1");
```

Vous pouvez également utiliser des outils externes comme Valgrind (Linux), Dr. Memory (Windows) ou le sanitizer d'LLVM.

## Exemple complet : Gestionnaire de ressources

Voici un exemple de classe qui gère des ressources avec des pointeurs intelligents :

```cpp
#include <QSharedPointer>
#include <QMap>
#include <QPixmap>

class GestionnaireRessources {
private:
    QMap<QString, QSharedPointer<QPixmap>> m_images;

public:
    QSharedPointer<QPixmap> obtenirImage(const QString& chemin) {
        // Vérifier si l'image est déjà chargée
        if (m_images.contains(chemin)) {
            return m_images[chemin];
        }

        // Charger une nouvelle image
        QSharedPointer<QPixmap> image = QSharedPointer<QPixmap>(new QPixmap(chemin));

        // Stocker dans le cache
        m_images[chemin] = image;

        return image;
    }

    void libererRessourcesInutilisees() {
        // Parcourir toutes les images et supprimer celles qui ne sont
        // plus référencées ailleurs que dans notre cache
        QMutableMapIterator<QString, QSharedPointer<QPixmap>> it(m_images);
        while (it.hasNext()) {
            it.next();
            if (it.value().use_count() == 1) {
                // Seule notre map possède une référence, on peut la supprimer
                it.remove();
            }
        }
    }
};
```

## Conclusion

La gestion mémoire est cruciale pour développer des applications Qt robustes. En utilisant les pointeurs intelligents appropriés, vous pouvez éviter les fuites mémoire et simplifier considérablement votre code.

Résumé des points clés :
- Utilisez le système parent-enfant pour les widgets et objets Qt
- Choisissez le pointeur intelligent adapté à chaque cas d'utilisation
- Préférez l'allocation sur la pile quand c'est possible
- Comprenez le fonctionnement de `deleteLater()` et de la boucle d'événements
- Utilisez des outils de détection de fuites mémoire pendant le développement

Dans la prochaine section, nous explorerons les techniques d'optimisation des performances pour vos applications Qt6.

⏭️ [Optimisation des performances](/11-bonnes-pratiques-qt6/02-optimisation-des-performances.md)
