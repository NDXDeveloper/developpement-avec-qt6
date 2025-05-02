# 7.3 Synchronisation avec QMutex et QSemaphore

## Introduction à la synchronisation

Lorsque vous travaillez avec plusieurs threads dans votre application Qt, vous rencontrerez inévitablement des situations où ces threads doivent accéder aux mêmes données. Sans mécanisme de protection, cela peut entraîner des **conditions de course** (race conditions) et des comportements imprévisibles.

Imaginez deux threads qui tentent de modifier le même compteur simultanément :
1. Le thread A lit la valeur (disons 5)
2. Le thread B lit aussi la valeur (5)
3. Le thread A ajoute 1 et écrit 6
4. Le thread B ajoute 1 et écrit 6 (au lieu de 7)

Résultat : un incrément est perdu ! C'est pour éviter ce genre de problèmes que Qt fournit des mécanismes de synchronisation comme `QMutex` et `QSemaphore`.

## QMutex : Exclusion mutuelle

### Qu'est-ce qu'un mutex ?

Un mutex (abréviation de "MUTual EXclusion" ou exclusion mutuelle) est comme un verrou qui protège une ressource partagée. Un seul thread à la fois peut posséder ce verrou.

### Fonctionnement de base

```cpp
QMutex mutex; // Créer un mutex
QList<int> sharedData; // Données partagées entre threads

void addToSharedData(int value)
{
    mutex.lock(); // Verrouiller le mutex

    // Section critique - un seul thread peut exécuter ce code à la fois
    sharedData.append(value);

    mutex.unlock(); // Déverrouiller le mutex
}
```

### Problèmes potentiels

Si vous oubliez d'appeler `unlock()`, les autres threads seront bloqués indéfiniment - c'est ce qu'on appelle un **deadlock** (interblocage).

### Solution plus propre : QMutexLocker

Pour éviter les oublis, Qt fournit `QMutexLocker` qui verrouille le mutex à sa création et le déverrouille automatiquement lorsqu'il est détruit (généralement à la fin du bloc).

```cpp
void addToSharedData(int value)
{
    QMutexLocker locker(&mutex); // Verrouille automatiquement

    sharedData.append(value);

    // locker est détruit à la fin de la fonction et déverrouille automatiquement
}
```

Même en cas d'exception, le mutex sera déverrouillé - c'est bien plus sûr !

### Types de mutex

Qt propose différents types de mutex :

1. **QMutex** - Le mutex de base
2. **QRecursiveMutex** - Peut être verrouillé plusieurs fois par le même thread

### Exemple pratique : Cache de données thread-safe

Voici un exemple simple d'un cache qui peut être utilisé par plusieurs threads :

```cpp
class ThreadSafeCache : public QObject
{
    Q_OBJECT
public:
    // Obtenir une valeur du cache
    QVariant getValue(const QString &key)
    {
        QMutexLocker locker(&m_mutex);
        return m_cache.value(key);
    }

    // Définir une valeur dans le cache
    void setValue(const QString &key, const QVariant &value)
    {
        QMutexLocker locker(&m_mutex);
        m_cache[key] = value;
    }

private:
    QMutex m_mutex;
    QHash<QString, QVariant> m_cache;
};
```

## QSemaphore : Compteur de ressources

### Qu'est-ce qu'un sémaphore ?

Un sémaphore est comme un compteur qui contrôle l'accès à un nombre limité de ressources identiques. Par exemple, pour limiter le nombre de threads qui peuvent accéder simultanément à un pool de connexions de base de données.

### Fonctionnement de base

```cpp
QSemaphore semaphore(5); // 5 ressources disponibles

void useResource()
{
    // Acquérir une ressource
    semaphore.acquire(); // Diminue le compteur de 1

    // Utiliser la ressource...

    // Libérer la ressource
    semaphore.release(); // Augmente le compteur de 1
}
```

### Méthodes principales

- `acquire(n = 1)` : Obtenir n ressources (bloque si pas assez disponibles)
- `tryAcquire(n = 1)` : Essayer d'obtenir n ressources sans bloquer (renvoie `true` si réussi)
- `release(n = 1)` : Libérer n ressources
- `available()` : Nombre de ressources disponibles

### QSemaphoreReleaser

Similaire à `QMutexLocker`, cette classe libère automatiquement les ressources acquises :

```cpp
void useResource()
{
    semaphore.acquire(); // Acquérir manuellement
    QSemaphoreReleaser releaser(&semaphore); // Libérera automatiquement

    // Utiliser la ressource...

    // Pas besoin d'appeler release() - fait automatiquement
}
```

### Exemple : Pool de connexions de base de données

```cpp
class DatabaseConnectionPool : public QObject
{
    Q_OBJECT
public:
    DatabaseConnectionPool(int maxConnections = 10)
        : m_connections(QVector<QSqlDatabase>(maxConnections)),
          m_semaphore(maxConnections)
    {
        // Initialiser les connexions
        for (int i = 0; i < maxConnections; ++i) {
            m_connections[i] = QSqlDatabase::addDatabase("QSQLITE",
                                QString("connection_%1").arg(i));
            m_connections[i].setDatabaseName("mydatabase.db");
            m_connections[i].open();
        }
    }

    // Obtenir une connexion du pool
    QSqlDatabase getConnection()
    {
        // Attendre qu'une connexion soit disponible
        m_semaphore.acquire();

        QMutexLocker locker(&m_mutex);
        for (int i = 0; i < m_connections.size(); ++i) {
            if (!m_usedConnections.contains(i)) {
                m_usedConnections.insert(i);
                return m_connections[i];
            }
        }

        // Ne devrait jamais arriver grâce au sémaphore
        return QSqlDatabase();
    }

    // Libérer une connexion
    void releaseConnection(const QSqlDatabase &connection)
    {
        QMutexLocker locker(&m_mutex);

        // Trouver l'index de cette connexion
        for (int i = 0; i < m_connections.size(); ++i) {
            if (m_connections[i].connectionName() == connection.connectionName()) {
                m_usedConnections.remove(i);
                m_semaphore.release();
                return;
            }
        }
    }

private:
    QVector<QSqlDatabase> m_connections;
    QSet<int> m_usedConnections;
    QMutex m_mutex;
    QSemaphore m_semaphore;
};
```

Utilisation :

```cpp
void workerFunction()
{
    // Obtenir une connexion
    QSqlDatabase db = connectionPool->getConnection();

    // Utiliser la connexion
    QSqlQuery query(db);
    query.exec("SELECT * FROM users");

    // Traiter les résultats...

    // Libérer la connexion
    connectionPool->releaseConnection(db);
}
```

## QReadWriteLock : Verrouillage lecture/écriture

Un autre outil utile est `QReadWriteLock`, qui permet à plusieurs threads de lire simultanément, mais garantit qu'un seul thread peut écrire à la fois.

```cpp
QReadWriteLock lock;
QVector<int> data;

// Lecture (plusieurs threads peuvent lire en même temps)
void readData(int index)
{
    QReadLocker locker(&lock);
    return data.value(index);
}

// Écriture (un seul thread peut écrire à la fois)
void writeData(int index, int value)
{
    QWriteLocker locker(&lock);
    if (index >= data.size())
        data.resize(index + 1);
    data[index] = value;
}
```

## QWaitCondition : Attente conditionnelle

`QWaitCondition` permet à un thread d'attendre qu'une condition soit remplie par un autre thread.

```cpp
QMutex mutex;
QWaitCondition dataReady;
QQueue<int> dataQueue;

// Producteur
void produceData()
{
    QMutexLocker locker(&mutex);

    // Produire des données
    dataQueue.enqueue(42);

    // Signaler que des données sont disponibles
    dataReady.wakeOne(); // Réveille un thread en attente
    // ou dataReady.wakeAll(); // Réveille tous les threads en attente
}

// Consommateur
void consumeData()
{
    QMutexLocker locker(&mutex);

    // Attendre que des données soient disponibles
    while (dataQueue.isEmpty())
        dataReady.wait(&mutex);

    // Traiter les données
    int value = dataQueue.dequeue();
    // Faire quelque chose avec value...
}
```

## Conseils pour éviter les deadlocks

Les deadlocks (interblocages) sont une des principales difficultés de la programmation multithreading. Voici quelques conseils pour les éviter :

1. **Toujours verrouiller les mutex dans le même ordre** - Si le thread A verrouille mutex1 puis mutex2, et que le thread B verrouille mutex2 puis mutex1, vous risquez un deadlock.

2. **Utiliser QMutexLocker, QReadLocker, QWriteLocker** - Ces classes garantissent que le mutex sera déverrouillé même en cas d'exception.

3. **Limiter la portée des verrous** - Ne verrouillez que le code qui a vraiment besoin de protection.

4. **Éviter les opérations bloquantes dans une section critique** - Si vous verrouillez un mutex puis appelez une fonction qui pourrait bloquer, vous risquez de causer des problèmes.

5. **Préférer QtConcurrent lorsque c'est possible** - Pour les opérations simples, `QtConcurrent` gère la synchronisation pour vous.

## Exemple complet : Producteur-Consommateur

Le problème classique du producteur-consommateur illustre bien l'utilisation des mécanismes de synchronisation :

```cpp
class SharedQueue : public QObject
{
    Q_OBJECT
public:
    SharedQueue(int maxSize = 10) : m_semFree(maxSize), m_semUsed(0) {}

    // Ajouter un élément (producteur)
    void enqueue(const QVariant &item)
    {
        // Attendre qu'il y ait de l'espace libre
        m_semFree.acquire();

        {
            QMutexLocker locker(&m_mutex);
            m_queue.enqueue(item);
        }

        // Signaler qu'un élément est disponible
        m_semUsed.release();
    }

    // Récupérer un élément (consommateur)
    QVariant dequeue()
    {
        // Attendre qu'un élément soit disponible
        m_semUsed.acquire();

        QVariant item;
        {
            QMutexLocker locker(&m_mutex);
            item = m_queue.dequeue();
        }

        // Signaler qu'un emplacement est libéré
        m_semFree.release();

        return item;
    }

    // Essayer de récupérer un élément sans bloquer
    bool tryDequeue(QVariant &item, int timeout = 0)
    {
        // Essayer d'obtenir un élément disponible
        if (!m_semUsed.tryAcquire(1, timeout))
            return false;

        {
            QMutexLocker locker(&m_mutex);
            item = m_queue.dequeue();
        }

        m_semFree.release();
        return true;
    }

private:
    QQueue<QVariant> m_queue;
    QMutex m_mutex;
    QSemaphore m_semFree;  // Compte les emplacements libres
    QSemaphore m_semUsed;  // Compte les éléments utilisés
};
```

Utilisation :

```cpp
// Dans la classe principale
SharedQueue *queue = new SharedQueue(100);

// Thread producteur
QThread *producerThread = new QThread();
Producer *producer = new Producer(queue);
producer->moveToThread(producerThread);
connect(producerThread, &QThread::started, producer, &Producer::produceItems);
producerThread->start();

// Thread consommateur
QThread *consumerThread = new QThread();
Consumer *consumer = new Consumer(queue);
consumer->moveToThread(consumerThread);
connect(consumerThread, &QThread::started, consumer, &Consumer::consumeItems);
consumerThread->start();
```

## Résumé

Les mécanismes de synchronisation de Qt vous permettent de coordonner efficacement l'accès aux ressources partagées entre plusieurs threads :

- **QMutex** pour protéger l'accès exclusif à une ressource
- **QSemaphore** pour gérer un pool limité de ressources
- **QReadWriteLock** pour permettre l'accès en lecture simultanée mais exclusif en écriture
- **QWaitCondition** pour synchroniser des threads basés sur des conditions

En utilisant ces outils correctement, vous pouvez créer des applications multi-threads robustes qui évitent les problèmes classiques comme les conditions de course et les deadlocks.

N'oubliez pas que la meilleure synchronisation est souvent celle que vous n'avez pas à gérer vous-même ! Lorsque c'est possible, utilisez `QtConcurrent` qui s'occupe de tous ces détails pour vous.
