# 8.3 Profilage avec Qt Performance Analyzer

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

## Qu'est-ce que le profilage et pourquoi est-il important ?

Le profilage est le processus qui consiste à analyser les performances de votre application pour identifier les parties qui sont lentes ou qui consomment trop de ressources. Contrairement au débogage qui vous aide à trouver des bugs, le profilage vous aide à rendre votre application plus rapide et plus efficace.

Qt fournit un outil puissant appelé **Qt Performance Analyzer** (anciennement Qt QML Profiler) qui vous permet de:

- Identifier les goulots d'étranglement de performance
- Analyser l'utilisation du CPU
- Mesurer le temps passé dans différentes fonctions
- Détecter les problèmes de mémoire
- Optimiser les performances de rendu

## Quand utiliser le profilage ?

Utilisez le profilage lorsque:
- Votre application semble lente ou ne répond pas rapidement
- L'interface utilisateur n'est pas fluide
- Vous avez des animations saccadées
- Votre application consomme trop de mémoire
- La batterie se décharge rapidement (applications mobiles)
- Vous voulez optimiser votre application avant sa publication

## Configuration du Qt Performance Analyzer

### Prérequis

Pour utiliser Qt Performance Analyzer, vous devez:
1. Utiliser Qt Creator 4.7 ou plus récent
2. Compiler votre application en mode **Debug** ou **Profile** (pas en mode Release)
3. S'assurer que les informations de débogage sont incluses

### Activer le profilage dans votre projet

1. Ouvrez votre projet dans Qt Creator
2. Dans le volet à gauche, basculez vers le mode **Projects**
3. Sous **Build & Run**, sélectionnez votre configuration **Debug** ou créez une configuration **Profile**
4. Dans les paramètres de compilation (Build Settings), assurez-vous que:
   - Le mode de compilation est **Debug** ou **Profile**
   - Les informations de débogage sont activées

## Lancer le Qt Performance Analyzer

Il existe deux façons principales de lancer le Qt Performance Analyzer:

### Méthode 1: Depuis l'IDE Qt Creator

1. Ouvrez votre projet dans Qt Creator
2. Allez dans le menu **Analyser**
3. Sélectionnez **Performance Analyzer**
4. Cliquez sur **Start** pour lancer votre application avec le profiler activé

### Méthode 2: Attacher à une application en cours d'exécution

1. Lancez votre application normalement
2. Dans Qt Creator, allez dans le menu **Analyser**
3. Sélectionnez **Performance Analyzer**
4. Cliquez sur **Attach to Running Application...**
5. Sélectionnez votre application dans la liste

## Interface du Qt Performance Analyzer

L'interface du Qt Performance Analyzer se compose de plusieurs sections:

![Interface du Qt Performance Analyzer](https://doc.qt.io/qtcreator/images/qtc-analyzer-overview.png)

### Vue chronologique (Timeline)

La vue chronologique montre l'activité de votre application au fil du temps. Elle est divisée en plusieurs pistes :

- **Événements QML/JS** : Montre les événements liés à QML et JavaScript
- **Rendu** : Affiche les opérations de rendu graphique
- **Compilation** : Indique les activités de compilation du moteur QML
- **Mémoire** : Montre l'utilisation de la mémoire
- **Événements système** : Affiche les événements système comme l'entrée utilisateur

### Vue statistique (Statistics)

Cette vue présente un résumé de toutes les fonctions et méthodes exécutées, classées par:
- Temps total passé dans la fonction
- Nombre d'appels
- Temps moyen par appel

### Vue Flame Graph (Graphe de flammes)

Cette visualisation permet de voir facilement quelles fonctions consomment le plus de temps de processeur. Les fonctions les plus larges sur le graphique sont celles qui prennent le plus de temps.

## Collecter des données de performance

### Enregistrement manuel

1. Lancez votre application avec le profiler
2. Cliquez sur **Record** pour commencer à collecter des données
3. Utilisez votre application, en particulier les parties que vous souhaitez optimiser
4. Cliquez sur **Stop** pour arrêter l'enregistrement

### Enregistrement limité

Pour réduire la quantité de données collectées:
1. Configurez une durée d'enregistrement limitée dans les options
2. Utilisez des points de départ et d'arrêt d'enregistrement dans votre code:

```cpp
#include <QQmlProfiler>

void startRecording() {
    QQmlProfiler::startProfiling();
}

void stopRecording() {
    QQmlProfiler::stopProfiling();
}
```

## Analyser les résultats

### Identifier les goulots d'étranglement

1. Dans la vue **Statistics**, triez les fonctions par "Total Time" (temps total)
2. Les fonctions en haut de la liste sont celles qui prennent le plus de temps
3. Concentrez-vous sur les fonctions où vous passez plus de 5-10% du temps total

### Utiliser le Flame Graph

Le Flame Graph est particulièrement utile pour comprendre l'imbrication des appels de fonction:

1. Les barres les plus larges représentent les fonctions qui consomment le plus de temps
2. La hauteur des barres représente la profondeur de la pile d'appels
3. Les barres au sommet sont les fonctions actuellement exécutées
4. Les barres en bas sont celles qui ont appelé d'autres fonctions

### Analyser les problèmes de rendu

Pour les applications graphiques, faites attention à:
1. Fréquence d'images trop basse (moins de 60 FPS)
2. Rendus redondants (éléments redessinés sans nécessité)
3. Synchronisations bloquantes avec le thread principal

## Problèmes courants et leurs solutions

### Problème: Fonctions JavaScript lentes dans QML

**Symptômes:**
- Fonctions JavaScript apparaissant en haut de la liste des temps d'exécution
- Interface utilisateur qui ne répond pas pendant le traitement

**Solutions:**
1. Déplacer le code JavaScript complexe dans des méthodes C++
2. Utiliser `Qt.callLater()` pour les opérations qui ne sont pas critiques
3. Éviter les calculs lourds dans les gestionnaires d'événements

### Problème: Trop de mises à jour de l'interface

**Symptômes:**
- Nombreux événements de rendu dans la timeline
- Utilisation élevée du CPU pendant l'affichage

**Solutions:**
1. Regrouper les mises à jour avec `Qt.callLater()`
2. Utiliser `Item.visible: false` pendant les modifications massives
3. Réduire les liaisons de propriétés (bindings) complexes

### Problème: Fréquence d'images basse

**Symptômes:**
- Animations saccadées
- Intervalles irréguliers entre les rendus

**Solutions:**
1. Simplifier les animations complexes
2. Réduire le nombre d'éléments animés simultanément
3. Utiliser des images précalculées au lieu d'animations générées
4. Activer l'accélération matérielle pour les éléments appropriés

## Techniques d'optimisation courantes

### 1. Mise en cache des résultats

Pour les opérations coûteuses qui sont appelées fréquemment avec les mêmes paramètres:

```cpp
QHash<QString, QPixmap> imageCache;

QPixmap getImage(const QString &path) {
    if (!imageCache.contains(path)) {
        imageCache[path] = QPixmap(path);
    }
    return imageCache[path];
}
```

### 2. Lazy Loading (Chargement paresseux)

Chargez les ressources uniquement lorsqu'elles sont nécessaires:

```qml
ListView {
    model: largeModel
    delegate: Item {
        Component {
            id: heavyComponent
            ComplexItem { /* ... */ }
        }

        Loader {
            sourceComponent: visible ? heavyComponent : null
        }
    }
}
```

### 3. Réduire les liaisons QML

Remplacez les liaisons complexes par des fonctions:

```qml
// Inefficace: réévalué à chaque changement de a, b ou c
property int result: a * b * c + Math.sqrt(a + b)

// Plus efficace:
function calculateResult() {
    return a * b * c + Math.sqrt(a + b);
}
property int result: calculateResult()
```

### 4. Utiliser les Worker Threads

Déplacez les calculs intensifs hors du thread principal:

```qml
WorkerScript {
    id: worker
    source: "worker.js"

    onMessage: {
        // Traiter le résultat ici
    }
}

function startHeavyCalculation() {
    worker.sendMessage({ input: value })
}
```

## Profilage de la mémoire

Qt Performance Analyzer peut également vous aider à identifier les problèmes d'utilisation de la mémoire:

### Observer la consommation mémoire

1. Dans la timeline, examinez la piste "Memory Usage"
2. Recherchez les tendances à la hausse continue (fuites potentielles)
3. Identifiez les pics de mémoire soudains

### Détecter les fuites de mémoire

Pour les applications qui fonctionnent longtemps:
1. Effectuez des actions répétitives (ouvrir/fermer des fenêtres, naviguer entre les écrans)
2. Observez si la mémoire continue d'augmenter sans jamais redescendre
3. Utilisez des outils spécialisés comme Valgrind pour des analyses plus détaillées

## Exercice pratique: Profiler une application simple

Voici un exemple de workflow complet pour profiler une application:

1. Créez une application simple avec un problème de performance évident:

```qml
// main.qml
import QtQuick 2.15
import QtQuick.Window 2.15
import QtQuick.Controls 2.15

Window {
    visible: true
    width: 640
    height: 480
    title: "Profiling Example"

    Button {
        text: "Calculate"
        onClicked: {
            // Opération inefficace
            var result = 0;
            for (var i = 0; i < 1000000; i++) {
                result += Math.sqrt(i);
            }
            resultText.text = "Result: " + result;
        }
    }

    Text {
        id: resultText
        anchors.centerIn: parent
        text: "Press the button"
    }
}
```

2. Lancez l'application avec Qt Performance Analyzer
3. Cliquez sur le bouton "Calculate" et observez le gel de l'interface
4. Analysez les résultats du profilage
5. Optimisez le code en déplaçant le calcul dans un Worker Thread
6. Profilez à nouveau pour vérifier l'amélioration

## Astuces avancées

### 1. Profilage ciblé

Pour profiler seulement certaines parties de votre code:

```cpp
#include <QElapsedTimer>
#include <QDebug>

void someFunction() {
    QElapsedTimer timer;
    timer.start();

    // Code à profiler ici

    qDebug() << "Function took" << timer.elapsed() << "ms";
}
```

### 2. Comparaison de profils

Qt Creator vous permet de sauvegarder et comparer plusieurs sessions de profilage:
1. Enregistrez un profil de référence
2. Apportez des modifications à votre code
3. Enregistrez un nouveau profil
4. Comparez les résultats pour vérifier les améliorations

### 3. Utiliser les annotations de profilage

Qt fournit des macros pour annoter votre code pour le profilage:

```cpp
#include <QLoggingCategory>

// Définir une catégorie de journalisation pour le profilage
Q_LOGGING_CATEGORY(timing, "app.timing")

void complexFunction() {
    // Loguer le début de la fonction
    qCDebug(timing) << "Starting complex calculation";

    // Code ici

    // Loguer la fin
    qCDebug(timing) << "Finished complex calculation";
}
```

## Résumé

Le profilage est une étape essentielle pour créer des applications Qt performantes. Avec Qt Performance Analyzer, vous pouvez:

1. Identifier les parties de votre code qui prennent le plus de temps
2. Détecter les problèmes de rendu et d'animation
3. Surveiller l'utilisation de la mémoire
4. Vérifier l'efficacité de vos optimisations

Rappelez-vous ces principes d'optimisation:
- Mesurez avant d'optimiser
- Concentrez-vous sur les 20% du code qui causent 80% des problèmes
- Vérifiez toujours l'impact de vos optimisations en profilant avant et après

En intégrant régulièrement le profilage dans votre processus de développement, vous créerez des applications Qt plus rapides, plus fluides et plus économes en ressources.

⏭️ [Gestion des erreurs et exceptions](/08-tests-et-debogage/04-gestion-des-erreurs-et-exceptions.md)
