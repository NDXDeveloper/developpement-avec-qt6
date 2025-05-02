# 7.2 Programmation asynchrone avec QFuture et QtConcurrent

## Introduction

La programmation asynchrone est une approche qui vous permet de lancer des opérations sans bloquer le thread principal de votre application. Dans Qt6, `QFuture` et le module `QtConcurrent` offrent une solution élégante et simple pour gérer ces opérations asynchrones.

Si vous avez trouvé que `QThread` demandait trop de code et de configuration, vous allez adorer la simplicité de `QtConcurrent` !

## Qu'est-ce que QtConcurrent ?

`QtConcurrent` est un module de haut niveau dans Qt qui simplifie énormément la programmation parallèle. Il vous permet d'exécuter des fonctions dans des threads séparés avec très peu de code, sans avoir à gérer manuellement la création des threads, la synchronisation ou la communication.

## Qu'est-ce que QFuture ?

`QFuture` représente le résultat d'une opération asynchrone. Il vous permet de :
- Récupérer le résultat d'une opération asynchrone
- Vérifier si l'opération est terminée
- Attendre la fin de l'opération
- Annuler l'opération (si c'est pris en charge)

## Comment ça fonctionne ensemble ?

1. Vous utilisez `QtConcurrent` pour lancer une opération dans un thread séparé
2. `QtConcurrent` vous renvoie un objet `QFuture` qui représente cette opération
3. Vous pouvez utiliser ce `QFuture` pour interagir avec l'opération en cours

## Les avantages par rapport à QThread

- **Beaucoup moins de code** à écrire
- **Pas besoin de sous-classer** ou de créer des objets worker
- **Pas besoin de gérer manuellement** les connexions de signaux et slots
- **API simple et intuitive**

## Fonctions principales de QtConcurrent

### 1. QtConcurrent::run

Cette fonction exécute une fonction ou une méthode lambda dans un thread séparé.

```cpp
// Exécuter une fonction simple dans un thread séparé
QFuture<int> future = QtConcurrent::run([]() {
    // Opération longue...
    QThread::sleep(2); // Simulation d'un travail de 2 secondes
    return 42; // Valeur de retour
});

// Récupérer le résultat (bloque jusqu'à ce que le résultat soit disponible)
int result = future.result();
```

### 2. QtConcurrent::map

Cette fonction applique une fonction à chaque élément d'une collection, en parallèle.

```cpp
// Une liste d'images à traiter
QList<QImage> images = loadImages();

// Traiter chaque image en parallèle
QFuture<void> future = QtConcurrent::map(images, [](QImage &image) {
    // Appliquer un filtre à l'image
    image = image.convertToFormat(QImage::Format_Grayscale8);
});

// Attendre que toutes les images soient traitées
future.waitForFinished();
```

### 3. QtConcurrent::mapped

Similaire à `map`, mais renvoie une nouvelle collection avec les résultats.

```cpp
// Une liste de noms de fichiers
QStringList filenames = {"image1.jpg", "image2.jpg", "image3.jpg"};

// Charger toutes les images en parallèle
QFuture<QImage> future = QtConcurrent::mapped(filenames, [](const QString &filename) {
    return QImage(filename);
});

// Récupérer toutes les images chargées
QList<QImage> images = future.results();
```

### 4. QtConcurrent::filter

Filtre une collection en parallèle selon un critère.

```cpp
// Une liste d'images
QList<QImage> images = loadImages();

// Filtrer les images trop petites
QFuture<void> future = QtConcurrent::filter(images, [](const QImage &image) {
    return image.width() >= 1024 && image.height() >= 768;
});

// Attendre la fin du filtrage
future.waitForFinished();
// Maintenant 'images' ne contient que les grandes images
```

## Utiliser QFutureWatcher pour suivre le progrès

`QFutureWatcher` vous permet de surveiller l'avancement d'une opération `QFuture` et d'être notifié lorsqu'elle est terminée.

```cpp
// Lancer une opération longue
QFuture<void> future = QtConcurrent::run([]() {
    for (int i = 0; i < 100; ++i) {
        // Simule un travail
        QThread::msleep(50);
        // Signale la progression
        QtConcurrent::reportProgress(i);
    }
});

// Créer un watcher pour surveiller cette opération
QFutureWatcher<void> watcher;
watcher.setFuture(future);

// Connecter les signaux pour être notifié des progrès et de la fin
connect(&watcher, &QFutureWatcher<void>::progressValueChanged,
        this, &MainWindow::updateProgressBar);
connect(&watcher, &QFutureWatcher<void>::finished,
        this, &MainWindow::operationFinished);
```

## Exemple pratique : Traitement d'image en parallèle

Imaginons que vous voulez appliquer un filtre à une collection d'images :

```cpp
// Classe principale de l'application
class ImageProcessor : public QObject
{
    Q_OBJECT
public:
    explicit ImageProcessor(QObject *parent = nullptr);

    // Méthode pour lancer le traitement
    void processImages(const QStringList &filenames);

signals:
    void progressUpdated(int value);
    void processingFinished(const QList<QImage> &processedImages);

private:
    QImage applyFilter(const QString &filename);
    QFutureWatcher<QImage> m_watcher;
};

// Implémentation
void ImageProcessor::processImages(const QStringList &filenames)
{
    // Lancer le traitement parallèle
    QFuture<QImage> future = QtConcurrent::mapped(filenames,
        [this](const QString &filename) {
            return applyFilter(filename);
        }
    );

    // Configurer le watcher
    m_watcher.setFuture(future);

    // Connecter les signaux (une seule fois, généralement dans le constructeur)
    connect(&m_watcher, &QFutureWatcher<QImage>::progressValueChanged,
            this, &ImageProcessor::progressUpdated);
    connect(&m_watcher, &QFutureWatcher<QImage>::finished,
            [this]() {
                emit processingFinished(m_watcher.future().results());
            });
}

QImage ImageProcessor::applyFilter(const QString &filename)
{
    // Charger l'image
    QImage image(filename);

    // Appliquer un filtre (par exemple, conversion en niveaux de gris)
    image = image.convertToFormat(QImage::Format_Grayscale8);

    // Simuler un traitement plus long
    QThread::msleep(500);

    return image;
}
```

Utilisation dans l'interface utilisateur :

```cpp
// Dans la classe MainWindow
void MainWindow::on_processButton_clicked()
{
    // Liste des images à traiter
    QStringList filenames = QFileDialog::getOpenFileNames(
        this, "Sélectionner des images", QString(), "Images (*.png *.jpg)");

    if (filenames.isEmpty())
        return;

    // Désactiver le bouton pendant le traitement
    ui->processButton->setEnabled(false);

    // Lancer le traitement
    m_imageProcessor->processImages(filenames);
}

// Slot pour mettre à jour la barre de progression
void MainWindow::updateProgressBar(int value)
{
    ui->progressBar->setValue(value);
}

// Slot appelé lorsque le traitement est terminé
void MainWindow::processingFinished(const QList<QImage> &processedImages)
{
    // Afficher les images traitées
    displayImages(processedImages);

    // Réactiver le bouton
    ui->processButton->setEnabled(true);

    // Afficher un message
    QMessageBox::information(this, "Terminé",
        QString("%1 images ont été traitées avec succès.").arg(processedImages.size()));
}
```

## Gérer les exceptions dans QtConcurrent

Les exceptions lancées dans les fonctions exécutées par `QtConcurrent` sont capturées et propagées via `QFuture` :

```cpp
QFuture<int> future = QtConcurrent::run([]() {
    // Simuler une erreur
    throw std::runtime_error("Quelque chose s'est mal passé");
    return 42;
});

// Attendre la fin
future.waitForFinished();

try {
    // Cette ligne va relancer l'exception
    int result = future.result();
} catch (const std::exception &e) {
    qDebug() << "Exception capturée :" << e.what();
}
```

## Bonnes pratiques avec QtConcurrent

1. **Évitez les opérations trop courtes** - La création d'un thread a un coût, donc utilisez `QtConcurrent` pour des opérations qui prennent au moins quelques millisecondes.

2. **Attention aux données partagées** - Comme avec tout code multithreading, faites attention aux accès concurrents aux données partagées.

3. **Utilisez QFutureWatcher** pour les opérations longues - Cela permet à votre interface utilisateur de rester réactive et d'afficher la progression.

4. **Ne bloquez pas le thread principal** - Évitez d'appeler `future.result()` ou `future.waitForFinished()` directement dans le thread principal.

5. **Choisissez la bonne fonction** - Utilisez `run` pour les opérations uniques, `map`/`mapped` pour appliquer une fonction à chaque élément d'une collection.

## Limites de QtConcurrent

- **Annulation limitée** - L'annulation d'opérations en cours n'est pas toujours possible.
- **Pas de priorité entre tâches** - Toutes les tâches sont traitées avec la même priorité.
- **Pool de threads partagé** - Par défaut, toutes les opérations `QtConcurrent` utilisent le même pool de threads.

## Conclusion

`QtConcurrent` et `QFuture` offrent une approche simple et puissante pour intégrer la programmation asynchrone dans vos applications Qt6. Ces outils vous permettent de tirer parti des processeurs multi-cœurs modernes sans la complexité généralement associée à la programmation multithreading.

Pour les opérations simples sur des collections ou pour exécuter une fonction dans un thread séparé, `QtConcurrent` est souvent un meilleur choix que `QThread` directement, car il nécessite beaucoup moins de code et de configuration.

