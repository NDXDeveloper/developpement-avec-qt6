# 7.4 Modèle d'acteur avec Qt

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

## Introduction au modèle d'acteur

Le modèle d'acteur est une approche élégante de la programmation concurrente qui diffère des techniques traditionnelles basées sur les threads et les verrous. Ce modèle considère les "acteurs" comme l'unité fondamentale de calcul : chaque acteur est une entité isolée qui peut :

1. Recevoir et envoyer des messages
2. Créer d'autres acteurs
3. Modifier son propre comportement interne

Le grand avantage de ce modèle est qu'il évite la complexité des mécanismes de synchronisation comme les mutex et les sémaphores, rendant le code plus simple et moins sujet aux erreurs.

## Pourquoi utiliser le modèle d'acteur ?

- **Simplicité** : Pas besoin de gérer manuellement les verrous et la synchronisation
- **Modularité** : Les acteurs forment des composants bien isolés et réutilisables
- **Évolutivité** : Facile à distribuer sur plusieurs threads, processus ou même machines
- **Résistance aux erreurs** : Un acteur qui plante n'affecte pas les autres acteurs

## Mise en œuvre du modèle d'acteur avec Qt

Qt n'offre pas directement un framework d'acteurs comme Akka (pour Java) ou Erlang, mais grâce au système de signaux et slots et aux classes threading, il est possible d'implémenter ce modèle de manière élégante.

### Principes de base d'une implémentation d'acteur dans Qt

1. Un acteur est un `QObject` qui vit dans son propre thread
2. Les messages sont transmis entre acteurs via les signaux et slots
3. Chaque acteur traite les messages de façon séquentielle
4. Les acteurs ne partagent pas d'état mais communiquent uniquement par messages

### Structure de base d'un acteur Qt

```cpp
class Actor : public QObject
{
    Q_OBJECT
public:
    explicit Actor(QObject *parent = nullptr);
    ~Actor();

    // Démarre l'acteur dans son propre thread
    void start();

public slots:
    // Slots pour recevoir différents types de messages
    void handleMessage1(const Message1 &msg);
    void handleMessage2(const Message2 &msg);

signals:
    // Signaux pour envoyer des messages à d'autres acteurs
    void sendResponse1(const Response1 &response);
    void sendResponse2(const Response2 &response);

private:
    QThread *m_thread;
};
```

### Implémentation simple d'un acteur

```cpp
Actor::Actor(QObject *parent) : QObject(parent), m_thread(new QThread)
{
    // Déplacer l'acteur dans son propre thread
    this->moveToThread(m_thread);

    // Connecter le signal destroyed au slot quit du thread
    connect(this, &Actor::destroyed, m_thread, &QThread::quit);
    connect(m_thread, &QThread::finished, m_thread, &QThread::deleteLater);
}

Actor::~Actor()
{
    // Assurez-vous que le thread se termine
    if (m_thread->isRunning()) {
        m_thread->quit();
        m_thread->wait();
    }
}

void Actor::start()
{
    // Démarrer le thread de l'acteur
    if (!m_thread->isRunning()) {
        m_thread->start();
    }
}
```

## Exemple concret : Système de traitement d'images avec acteurs

Imaginons un système simple de traitement d'images utilisant des acteurs :

1. **ImageLoader** : charge les images depuis le disque
2. **ImageProcessor** : applique des filtres aux images
3. **ImageSaver** : enregistre les images traitées sur le disque

### Définition des messages

D'abord, définissons les messages qui seront échangés entre nos acteurs :

```cpp
// Représente une image à traiter
struct ImageMessage {
    QString filePath;  // Chemin du fichier source
    QImage image;      // Données de l'image
    QString outputPath; // Chemin de sortie
    QList<QString> filters; // Filtres à appliquer
};

// Représente une notification de progression
struct ProgressMessage {
    QString filePath;  // Chemin du fichier concerné
    int progressPercent; // Pourcentage de progression
    QString stage;     // Étape actuelle (chargement, traitement, sauvegarde)
};
```

### Implémentation des acteurs

#### 1. ImageLoader

```cpp
class ImageLoader : public QObject
{
    Q_OBJECT
public:
    explicit ImageLoader(QObject *parent = nullptr);
    ~ImageLoader();

    void start();

public slots:
    void loadImage(const QString &filePath, const QString &outputPath,
                  const QList<QString> &filters);

signals:
    void imageLoaded(const ImageMessage &msg);
    void progressUpdated(const ProgressMessage &progress);
    void error(const QString &filePath, const QString &errorMessage);

private:
    QThread *m_thread;
};

// Implémentation
void ImageLoader::loadImage(const QString &filePath, const QString &outputPath,
                           const QList<QString> &filters)
{
    // Mise à jour de la progression
    ProgressMessage progress;
    progress.filePath = filePath;
    progress.stage = "Chargement";
    progress.progressPercent = 0;
    emit progressUpdated(progress);

    // Chargement de l'image
    QImage image(filePath);

    if (image.isNull()) {
        emit error(filePath, "Impossible de charger l'image");
        return;
    }

    // Création du message à envoyer au processeur
    ImageMessage msg;
    msg.filePath = filePath;
    msg.image = image;
    msg.outputPath = outputPath;
    msg.filters = filters;

    // Mise à jour de la progression
    progress.progressPercent = 100;
    emit progressUpdated(progress);

    // Envoi de l'image chargée
    emit imageLoaded(msg);
}
```

#### 2. ImageProcessor

```cpp
class ImageProcessor : public QObject
{
    Q_OBJECT
public:
    explicit ImageProcessor(QObject *parent = nullptr);
    ~ImageProcessor();

    void start();

public slots:
    void processImage(const ImageMessage &msg);

signals:
    void imageProcessed(const ImageMessage &msg);
    void progressUpdated(const ProgressMessage &progress);
    void error(const QString &filePath, const QString &errorMessage);

private:
    QThread *m_thread;

    // Méthodes pour appliquer différents filtres
    QImage applyGrayscale(const QImage &image);
    QImage applyBlur(const QImage &image);
    QImage applySharpen(const QImage &image);
};

// Implémentation
void ImageProcessor::processImage(const ImageMessage &msg)
{
    // Mise à jour de la progression
    ProgressMessage progress;
    progress.filePath = msg.filePath;
    progress.stage = "Traitement";
    progress.progressPercent = 0;
    emit progressUpdated(progress);

    // Copie du message pour ne pas modifier l'original
    ImageMessage processedMsg = msg;
    QImage processedImage = msg.image;

    // Application des filtres
    int totalFilters = msg.filters.size();
    for (int i = 0; i < totalFilters; ++i) {
        QString filter = msg.filters[i];

        if (filter == "grayscale") {
            processedImage = applyGrayscale(processedImage);
        } else if (filter == "blur") {
            processedImage = applyBlur(processedImage);
        } else if (filter == "sharpen") {
            processedImage = applySharpen(processedImage);
        }

        // Mise à jour de la progression
        progress.progressPercent = ((i + 1) * 100) / totalFilters;
        emit progressUpdated(progress);
    }

    // Mise à jour de l'image traitée
    processedMsg.image = processedImage;

    // Envoi de l'image traitée
    emit imageProcessed(processedMsg);
}
```

#### 3. ImageSaver

```cpp
class ImageSaver : public QObject
{
    Q_OBJECT
public:
    explicit ImageSaver(QObject *parent = nullptr);
    ~ImageSaver();

    void start();

public slots:
    void saveImage(const ImageMessage &msg);

signals:
    void imageSaved(const QString &filePath, const QString &outputPath);
    void progressUpdated(const ProgressMessage &progress);
    void error(const QString &filePath, const QString &errorMessage);

private:
    QThread *m_thread;
};

// Implémentation
void ImageSaver::saveImage(const ImageMessage &msg)
{
    // Mise à jour de la progression
    ProgressMessage progress;
    progress.filePath = msg.filePath;
    progress.stage = "Sauvegarde";
    progress.progressPercent = 0;
    emit progressUpdated(progress);

    // Sauvegarde de l'image
    bool success = msg.image.save(msg.outputPath);

    if (!success) {
        emit error(msg.filePath, "Impossible de sauvegarder l'image");
        return;
    }

    // Mise à jour de la progression
    progress.progressPercent = 100;
    emit progressUpdated(progress);

    // Notification de sauvegarde réussie
    emit imageSaved(msg.filePath, msg.outputPath);
}
```

### Coordination des acteurs

Pour coordonner nos acteurs, nous avons besoin d'un superviseur qui connecte les signaux et slots appropriés :

```cpp
class ImageProcessingSystem : public QObject
{
    Q_OBJECT
public:
    explicit ImageProcessingSystem(QObject *parent = nullptr);
    ~ImageProcessingSystem();

    // Démarre le système
    void start();

    // Soumet une image à traiter
    void submitImage(const QString &filePath, const QString &outputPath,
                    const QList<QString> &filters);

signals:
    // Signaux pour informer l'interface utilisateur
    void progressUpdated(const QString &filePath, const QString &stage, int percent);
    void processingCompleted(const QString &filePath, const QString &outputPath);
    void processingError(const QString &filePath, const QString &errorMessage);

private slots:
    // Slots pour gérer les notifications de progression
    void handleProgress(const ProgressMessage &progress);
    void handleError(const QString &filePath, const QString &errorMessage);
    void handleImageSaved(const QString &filePath, const QString &outputPath);

private:
    ImageLoader *m_loader;
    ImageProcessor *m_processor;
    ImageSaver *m_saver;
};

// Implémentation
ImageProcessingSystem::ImageProcessingSystem(QObject *parent) : QObject(parent)
{
    // Création des acteurs
    m_loader = new ImageLoader();
    m_processor = new ImageProcessor();
    m_saver = new ImageSaver();

    // Connexion des signaux entre acteurs
    connect(m_loader, &ImageLoader::imageLoaded,
            m_processor, &ImageProcessor::processImage);
    connect(m_processor, &ImageProcessor::imageProcessed,
            m_saver, &ImageSaver::saveImage);

    // Connexion des signaux de progression
    connect(m_loader, &ImageLoader::progressUpdated,
            this, &ImageProcessingSystem::handleProgress);
    connect(m_processor, &ImageProcessor::progressUpdated,
            this, &ImageProcessingSystem::handleProgress);
    connect(m_saver, &ImageSaver::progressUpdated,
            this, &ImageProcessingSystem::handleProgress);

    // Connexion des signaux d'erreur
    connect(m_loader, &ImageLoader::error,
            this, &ImageProcessingSystem::handleError);
    connect(m_processor, &ImageProcessor::error,
            this, &ImageProcessingSystem::handleError);
    connect(m_saver, &ImageSaver::error,
            this, &ImageProcessingSystem::handleError);

    // Connexion du signal de fin de traitement
    connect(m_saver, &ImageSaver::imageSaved,
            this, &ImageProcessingSystem::handleImageSaved);
}

void ImageProcessingSystem::start()
{
    // Démarrage des acteurs dans leurs threads respectifs
    m_loader->start();
    m_processor->start();
    m_saver->start();
}

void ImageProcessingSystem::submitImage(const QString &filePath,
                                      const QString &outputPath,
                                      const QList<QString> &filters)
{
    // Demande de chargement de l'image
    QMetaObject::invokeMethod(m_loader, "loadImage",
                            Q_ARG(QString, filePath),
                            Q_ARG(QString, outputPath),
                            Q_ARG(QList<QString>, filters));
}
```

### Utilisation dans une interface Qt

```cpp
// Dans la classe MainWindow
void MainWindow::on_processButton_clicked()
{
    QString filePath = ui->filePathEdit->text();
    QString outputPath = ui->outputPathEdit->text();

    // Récupération des filtres sélectionnés
    QList<QString> filters;
    if (ui->grayscaleCheckBox->isChecked())
        filters.append("grayscale");
    if (ui->blurCheckBox->isChecked())
        filters.append("blur");
    if (ui->sharpenCheckBox->isChecked())
        filters.append("sharpen");

    // Soumission de l'image au système
    m_processingSystem->submitImage(filePath, outputPath, filters);
}

// Connexion des signaux du système à l'interface
void MainWindow::setupConnections()
{
    connect(m_processingSystem, &ImageProcessingSystem::progressUpdated,
            this, &MainWindow::updateProgressBar);
    connect(m_processingSystem, &ImageProcessingSystem::processingCompleted,
            this, &MainWindow::handleProcessingCompleted);
    connect(m_processingSystem, &ImageProcessingSystem::processingError,
            this, &MainWindow::handleProcessingError);
}
```

## Avantages du modèle d'acteur dans Qt

1. **Isolation claire** : Chaque acteur effectue une tâche spécifique dans son propre thread.

2. **Communication par messages** : Les acteurs interagissent uniquement via des messages (signaux/slots), ce qui évite les problèmes de synchronisation.

3. **Scalabilité** : Facile d'ajouter de nouveaux acteurs ou de modifier le flux de traitement.

4. **Maintenabilité** : Code plus clair et modulaire, avec des responsabilités bien définies.

## Bonnes pratiques pour le modèle d'acteur dans Qt

1. **Gardez les acteurs légers** : Un acteur doit avoir une responsabilité unique et bien définie.

2. **Évitez le partage d'état** : Les acteurs ne doivent pas partager de données directement, uniquement par messages.

3. **Messages immuables** : Préférez des messages qui ne peuvent pas être modifiés après leur création pour éviter les problèmes de concurrence.

4. **Utilisez QueuedConnection** : Assurez-vous que les connexions entre acteurs utilisent `Qt::QueuedConnection` pour garantir la thread-safety.

5. **Gérez correctement les erreurs** : Chaque acteur doit signaler ses erreurs de manière appropriée.

## Limites du modèle d'acteur dans Qt

1. **Pas de supervision native** : Contrairement à certains frameworks d'acteurs, Qt n'offre pas de mécanisme intégré pour surveiller et redémarrer les acteurs défaillants.

2. **Surcoût de communication** : La communication par messages peut être moins efficace que l'accès direct à des données partagées pour certains cas d'utilisation.

3. **Débogage plus complexe** : Le flux d'exécution asynchrone peut rendre le débogage plus difficile.

## Conclusion

Le modèle d'acteur est une approche puissante de la programmation concurrente qui s'intègre naturellement avec le système de signaux et slots de Qt. Il permet de créer des applications concurrentes robustes sans avoir à gérer manuellement les verrous et autres mécanismes de synchronisation.

En utilisant ce modèle, vous pouvez concevoir des systèmes évolutifs et modulaires, où chaque composant (acteur) a une responsabilité claire et communique par messages, ce qui facilite la maintenance et l'extension du code.

Dans Qt, bien que le modèle d'acteur ne soit pas directement fourni comme framework, les outils existants (QObject, signaux/slots, QThread) permettent de l'implémenter de manière élégante et efficace.

⏭️ [Tests et débogage](/08-tests-et-debogage)
