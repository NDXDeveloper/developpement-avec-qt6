# 7. Multithreading et concurrence

## Introduction

Le multithreading est une technique puissante qui permet à une application d'exécuter plusieurs tâches simultanément. Dans les applications modernes, cette capacité est essentielle pour créer des interfaces utilisateur réactives tout en effectuant des opérations longues ou complexes en arrière-plan.

Qt6 offre un ensemble complet d'outils pour gérer le multithreading et la concurrence, ce qui vous permet de développer des applications performantes sans vous soucier des problèmes complexes généralement associés à la programmation parallèle.

## Pourquoi utiliser le multithreading ?

Imaginez que vous développez une application qui doit charger un fichier volumineux. Si vous effectuez ce chargement dans le thread principal (celui qui gère l'interface utilisateur), votre application semblera "gelée" pendant toute la durée du chargement. Les boutons ne répondront plus, les animations s'arrêteront, et l'utilisateur pourrait penser que l'application a planté.

Voici les principaux avantages du multithreading :

- **Interface utilisateur réactive** : L'UI reste fluide pendant les opérations longues
- **Meilleures performances** : Utilisation optimale des processeurs multi-cœurs
- **Exécution parallèle** : Plusieurs tâches peuvent s'exécuter simultanément

## Les défis du multithreading

Le multithreading n'est pas sans difficultés. Voici les principaux défis :

- **Conditions de course (race conditions)** : Quand plusieurs threads accèdent aux mêmes données simultanément
- **Interblocages (deadlocks)** : Quand deux threads s'attendent mutuellement
- **Complexité accrue** : Le débogage d'applications multi-threads est plus difficile

Heureusement, Qt6 fournit des outils qui simplifient considérablement ces défis.

## Vue d'ensemble des outils de concurrence dans Qt6

Qt6 propose plusieurs approches pour gérer la concurrence :

### 1. Threads de bas niveau avec QThread

`QThread` est la classe fondamentale pour créer et gérer des threads dans Qt. Elle offre un contrôle précis mais nécessite une bonne compréhension des principes du multithreading.

### 2. API de haut niveau avec QtConcurrent

Pour simplifier l'utilisation du multithreading, Qt propose `QtConcurrent` qui permet d'exécuter des fonctions dans des threads séparés avec très peu de code.

### 3. Mécanismes de synchronisation

Pour coordonner les threads et éviter les problèmes de concurrence, Qt fournit :
- `QMutex` : Pour protéger l'accès aux données partagées
- `QSemaphore` : Pour contrôler l'accès à un nombre limité de ressources
- `QWaitCondition` : Pour la synchronisation basée sur des conditions

### 4. Tâches asynchrones avec QFuture

`QFuture` et `QFutureWatcher` permettent de suivre la progression des tâches exécutées dans d'autres threads et d'obtenir leurs résultats de manière asynchrone.

## Communication entre threads dans Qt

Un aspect fondamental du multithreading dans Qt est la communication entre les threads. En général, les widgets Qt ne sont pas thread-safe, ce qui signifie qu'ils ne peuvent être manipulés que depuis le thread principal.

Qt propose plusieurs méthodes sécurisées pour communiquer entre les threads :

### 1. Signaux et slots à travers les threads

Les signaux et slots de Qt fonctionnent naturellement entre les threads grâce à un système de file d'attente d'événements. Pour cela, utilisez `Qt::QueuedConnection` :

```cpp
// Connexion entre deux objets dans des threads différents
connect(workerObject, &Worker::resultReady,
        uiObject, &UI::updateUI,
        Qt::QueuedConnection);
```

### 2. Événements personnalisés

Vous pouvez créer et poster des événements personnalisés entre les threads :

```cpp
// Envoi d'un événement personnalisé au thread principal
QCoreApplication::postEvent(mainThreadObject, new MyCustomEvent(data));
```

## Utilisation sécurisée des threads dans Qt

Voici quelques règles importantes à suivre pour éviter les problèmes :

1. **Ne jamais manipuler l'interface utilisateur depuis un thread secondaire**
2. **Protéger l'accès aux données partagées** avec des mutex ou autres mécanismes de synchronisation
3. **Éviter de partager des objets Qt** entre les threads (sauf indication contraire dans la documentation)
4. **Utiliser les connexions en file d'attente** (`Qt::QueuedConnection`) pour la communication inter-threads

## Conclusion

Le multithreading dans Qt6 offre des possibilités puissantes pour améliorer les performances et la réactivité de vos applications. En comprenant les concepts de base et en utilisant les outils appropriés fournis par Qt, vous pouvez tirer parti de la programmation concurrentielle sans tomber dans ses pièges classiques.

Dans les sections suivantes, nous explorerons en détail chaque outil de concurrence de Qt6, avec des exemples pratiques et des conseils pour les utiliser efficacement.
