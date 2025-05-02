# 7.1 QThread et gestion des threads

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

## Introduction à QThread

`QThread` est la classe fondamentale de Qt pour créer et gérer des threads. Elle vous permet d'exécuter des opérations en parallèle avec votre interface utilisateur, ce qui rend votre application plus réactive et performante.

Un thread peut être considéré comme un "mini-programme" qui s'exécute de manière indépendante à l'intérieur de votre application. Dans Qt, votre application démarre toujours avec un thread principal (aussi appelé thread GUI), qui est responsable de la gestion de l'interface utilisateur.

## Pourquoi utiliser QThread ?

Imaginons que vous développez une application qui doit télécharger un fichier volumineux depuis Internet. Si vous effectuez ce téléchargement dans le thread principal, votre interface utilisateur se figera jusqu'à la fin du téléchargement. Ce comportement est inacceptable pour une application moderne.

En utilisant `QThread`, vous pouvez déplacer cette opération longue dans un thread séparé, ce qui permet à votre interface utilisateur de rester réactive pendant le téléchargement.

## Deux approches pour utiliser QThread

Qt offre deux façons principales d'utiliser `QThread` :

1. **Sous-classer QThread** (approche traditionnelle)
2. **Déplacer un objet de travail dans un QThread** (approche recommandée)

Nous allons explorer ces deux approches, mais sachez que la seconde est généralement recommandée pour les nouvelles applications Qt6.

## Approche 1 : Sous-classer QThread (méthode traditionnelle)

Cette approche consiste à créer une classe dérivée de `QThread` et à redéfinir sa méthode `run()`.

```cpp
class DownloadThread : public QThread
{
    Q_OBJECT
public:
    DownloadThread(const QString &url, QObject *parent = nullptr)
        : QThread(parent), m_url(url) {}

signals:
    void progressUpdated(int percent);
    void downloadComplete(const QByteArray &data);

protected:
    void run() override
    {
        // Cette méthode s'exécute dans un nouveau thread
        QByteArray downloadedData;

        // Simulons un téléchargement
        for (int i = 0; i <= 100; i += 10) {
            // Code de téléchargement fictif
            QThread::msleep(500); // Simule un travail

            emit progressUpdated(i);
        }

        emit downloadComplete(downloadedData);
    }

private:
    QString m_url;
};
```

Utilisation :

```cpp
// Dans votre classe principale
DownloadThread *thread = new DownloadThread("https://example.com/bigfile.zip");

// Connexion des signaux
connect(thread, &DownloadThread::progressUpdated,
        this, &MainWindow::updateProgressBar);
connect(thread, &DownloadThread::downloadComplete,
        this, &MainWindow::handleDownloadedData);
connect(thread, &DownloadThread::finished,
        thread, &DownloadThread::deleteLater);

// Démarrage du thread
thread->start();
```

**Inconvénients de cette approche :**
- Elle mélange la logique de gestion des threads avec la logique métier
- Elle rend le code difficile à réutiliser

## Approche 2 : Déplacer un objet dans un QThread (recommandée)

L'approche moderne consiste à séparer votre code métier (le "travail") de la gestion du thread. Vous créez un objet "Worker" qui effectue le travail, puis vous le déplacez dans un thread.

```cpp
// L'objet worker
class Downloader : public QObject
{
    Q_OBJECT
public:
    Downloader(QObject *parent = nullptr) : QObject(parent) {}

public slots:
    void downloadFile(const QString &url)
    {
        QByteArray downloadedData;

        // Simulons un téléchargement
        for (int i = 0; i <= 100; i += 10) {
            // Code de téléchargement fictif
            QThread::msleep(500); // Simule un travail

            emit progressUpdated(i);
        }

        emit downloadComplete(downloadedData);
    }

signals:
    void progressUpdated(int percent);
    void downloadComplete(const QByteArray &data);
};
```

Utilisation :

```cpp
// Dans votre classe principale
QThread *thread = new QThread(this);
Downloader *downloader = new Downloader();

// Déplacer l'objet worker dans le thread
downloader->moveToThread(thread);

// Connexions
connect(thread, &QThread::started,
        downloader, [=]() { downloader->downloadFile("https://example.com/bigfile.zip"); });
connect(downloader, &Downloader::progressUpdated,
        this, &MainWindow::updateProgressBar);
connect(downloader, &Downloader::downloadComplete,
        this, &MainWindow::handleDownloadedData);
connect(downloader, &Downloader::downloadComplete,
        thread, &QThread::quit);
connect(thread, &QThread::finished,
        downloader, &Downloader::deleteLater);
connect(thread, &QThread::finished,
        thread, &QThread::deleteLater);

// Démarrage du thread
thread->start();
```

**Avantages de cette approche :**
- Séparation claire entre la logique métier et la gestion des threads
- Code plus facile à tester et à réutiliser
- Possibilité de déplacer le même objet worker dans différents threads

## Communication avec le thread principal

Lorsque vous travaillez avec des threads, rappelez-vous ces règles essentielles :

1. **Ne jamais manipuler l'interface utilisateur depuis un thread secondaire**
2. **Utiliser les signaux et slots pour communiquer entre les threads**

Qt gère automatiquement la communication entre les threads grâce aux connexions en file d'attente :

```cpp
// Connexion thread-safe (par défaut pour les objets dans des threads différents)
connect(worker, &Worker::resultReady,
        this, &MainWindow::handleResults,
        Qt::QueuedConnection);
```

## Synchronisation des threads

Pour éviter les problèmes d'accès concurrent aux données, Qt fournit plusieurs classes :

### QMutex

Utiliser un mutex pour protéger une section critique :

```cpp
QMutex mutex;
QList<int> sharedData;

// Dans un thread
void addToSharedData(int value)
{
    mutex.lock();
    sharedData.append(value);
    mutex.unlock();
}
```

Ou plus élégamment avec QMutexLocker :

```cpp
void addToSharedData(int value)
{
    QMutexLocker locker(&mutex);
    sharedData.append(value);
}
```

## Bonnes pratiques avec QThread

1. **Ne bloquez jamais le thread principal**
2. **Utilisez moveToThread plutôt que de sous-classer QThread**
3. **Nettoyez toujours vos threads** avec les connexions appropriées
4. **Évitez de partager des données entre threads** sans synchronisation
5. **Ne manipulez jamais des objets d'interface utilisateur depuis un thread secondaire**
6. **Préférez des connexions Qt::QueuedConnection entre les threads**

## Cycle de vie d'un QThread

Voici les étapes importantes du cycle de vie d'un thread Qt :

1. **Création** : `QThread *thread = new QThread();`
2. **Configuration** : Connexion des signaux et slots, déplacement des objets
3. **Démarrage** : `thread->start();`
4. **Exécution** : Le thread exécute sa tâche
5. **Terminaison** : `thread->quit();` ou lorsque la méthode `run()` se termine
6. **Nettoyage** : Appel à `deleteLater()` sur le thread et les objets associés

## Exemple complet : Traitement d'image en arrière-plan

Voici un exemple concret d'utilisation de QThread pour traiter une image en arrière-plan :

```cpp
// ImageProcessor.h
class ImageProcessor : public QObject
{
    Q_OBJECT
public:
    explicit ImageProcessor(QObject *parent = nullptr);

public slots:
    void processImage(const QImage &image);

signals:
    void processingStarted();
    void processingProgress(int percent);
    void processingFinished(const QImage &processedImage);

private:
    QImage applyBlurEffect(const QImage &image);
};

// ImageProcessor.cpp
void ImageProcessor::processImage(const QImage &image)
{
    emit processingStarted();

    // Simulons un traitement long
    emit processingProgress(0);
    QThread::msleep(500);

    // Première étape
    QImage result = image;
    emit processingProgress(33);
    QThread::msleep(500);

    // Deuxième étape
    result = applyBlurEffect(result);
    emit processingProgress(66);
    QThread::msleep(500);

    // Étape finale
    emit processingProgress(100);
    emit processingFinished(result);
}

// MainWindow.cpp
void MainWindow::setupImageProcessing()
{
    // Création du thread et du processor
    m_thread = new QThread(this);
    m_imageProcessor = new ImageProcessor();

    // Déplacement du processor dans le thread
    m_imageProcessor->moveToThread(m_thread);

    // Connexions
    connect(this, &MainWindow::requestImageProcessing,
            m_imageProcessor, &ImageProcessor::processImage);
    connect(m_imageProcessor, &ImageProcessor::processingStarted,
            this, &MainWindow::onProcessingStarted);
    connect(m_imageProcessor, &ImageProcessor::processingProgress,
            this, &MainWindow::updateProgressBar);
    connect(m_imageProcessor, &ImageProcessor::processingFinished,
            this, &MainWindow::displayProcessedImage);
    connect(m_imageProcessor, &ImageProcessor::processingFinished,
            m_thread, &QThread::quit);

    // Démarrage du thread seulement quand nécessaire
}

void MainWindow::on_processButton_clicked()
{
    if (!m_thread->isRunning()) {
        m_thread->start();
    }

    emit requestImageProcessing(m_originalImage);
}
```

## Conclusion

`QThread` est un outil puissant pour améliorer les performances et la réactivité de vos applications Qt. En suivant l'approche recommandée avec `moveToThread`, vous pouvez facilement intégrer le multithreading dans vos projets tout en gardant un code propre et maintenable.

N'oubliez pas que le multithreading introduit une complexité supplémentaire dans votre application. Utilisez-le lorsque c'est nécessaire, mais n'hésitez pas à explorer d'autres solutions comme `QtConcurrent` pour des cas d'utilisation plus simples, que nous verrons dans la section suivante.

⏭️ [Programmation asynchrone avec QFuture et QtConcurrent](/07-multithreading-et-concurrence/02-programmation-asynchrone-avec-qfuture-et-qtconcurrent.md)
