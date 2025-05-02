# 8.2 Débogage avec Qt Creator

Le débogage est une compétence essentielle pour tout développeur. Qt Creator intègre un débogueur puissant qui vous aide à trouver et à corriger les problèmes dans votre code. Ce chapitre vous guidera à travers les fonctionnalités de débogage de Qt Creator et vous montrera comment les utiliser efficacement.

## Qu'est-ce que le débogage ?

Le débogage est le processus qui consiste à identifier et à éliminer les erreurs (ou "bugs") dans votre code. Plutôt que d'ajouter des instructions `qDebug()` partout et de recompiler votre programme à chaque fois, un débogueur vous permet d'examiner l'état de votre programme pendant son exécution.

## Configuration du débogueur

Avant de commencer, assurez-vous que Qt Creator est correctement configuré pour le débogage :

1. Ouvrez Qt Creator
2. Allez dans **Outils > Options > Kits**
3. Sélectionnez votre kit et vérifiez qu'un débogueur est configuré (généralement GDB pour Linux/macOS ou CDB/MSVC pour Windows)
4. Assurez-vous que votre projet est en mode **Debug** (et non Release) en bas à gauche de Qt Creator

## Compilation en mode debug

Pour déboguer efficacement, vous devez compiler votre application en mode debug :

1. Cliquez sur le sélecteur de configuration en bas à gauche de Qt Creator
2. Choisissez **Debug** au lieu de **Release**

Le mode debug ajoute des informations de débogage à votre programme, ce qui le ralentit mais permet au débogueur de suivre précisément ce qui se passe.

## Lancer le débogueur

Pour lancer votre programme avec le débogueur :

1. Appuyez sur **F5** ou cliquez sur l'icône **Déboguer** (insecte vert)
2. Votre programme démarrera et le débogueur s'y attachera

## Les points d'arrêt - l'outil fondamental

Les points d'arrêt (breakpoints) sont l'outil le plus important du débogage. Ils indiquent au débogueur où suspendre l'exécution de votre programme.

### Ajouter un point d'arrêt

Pour ajouter un point d'arrêt :

1. Cliquez dans la marge gauche à côté du numéro de ligne où vous souhaitez interrompre l'exécution
2. Un point rouge apparaît, indiquant que le point d'arrêt est défini
3. Vous pouvez également appuyer sur **F9** lorsque le curseur est sur la ligne où vous souhaitez ajouter un point d'arrêt

![Exemple de point d'arrêt dans Qt Creator](https://doc.qt.io/qtcreator/images/qtc-debug-breakpoint.png)

### Types de points d'arrêt

Qt Creator prend en charge plusieurs types de points d'arrêt :

1. **Points d'arrêt de ligne** : S'arrête à une ligne spécifique
2. **Points d'arrêt conditionnels** : S'arrête uniquement lorsqu'une condition est vraie
3. **Points d'arrêt de fonction** : S'arrête lorsqu'une fonction spécifique est appelée

### Points d'arrêt conditionnels

Pour créer un point d'arrêt conditionnel :

1. Faites un clic droit sur un point d'arrêt existant
2. Sélectionnez **Modifier le point d'arrêt**
3. Cochez **Condition** et entrez une expression (comme `i > 10`)
4. Le programme s'arrêtera à ce point uniquement lorsque la condition sera vraie

### Gestion des points d'arrêt

Pour gérer vos points d'arrêt :

1. Ouvrez la vue **Points d'arrêt** (**Alt+8** ou menu **Déboguer > Fenêtres > Points d'arrêt**)
2. Vous pouvez activer/désactiver, modifier ou supprimer des points d'arrêt depuis cette vue

## Navigation dans le code pendant le débogage

Une fois que le débogueur est arrêté à un point d'arrêt, vous pouvez contrôler l'exécution du programme :

- **Pas à pas détaillé (F11)** : Exécute la ligne actuelle et entre dans les fonctions appelées
- **Pas à pas principal (F10)** : Exécute la ligne actuelle sans entrer dans les fonctions
- **Pas à pas sortant (Maj+F11)** : Continue l'exécution jusqu'à la fin de la fonction actuelle
- **Continuer (F5)** : Reprend l'exécution jusqu'au prochain point d'arrêt
- **Exécuter jusqu'au curseur (Ctrl+F10)** : Continue l'exécution jusqu'à la position du curseur

![Boutons de contrôle du débogueur](https://doc.qt.io/qtcreator/images/qtc-debug-continue.png)

## Examiner les variables

La partie la plus utile du débogage est d'examiner l'état de vos variables :

### Visualiser les variables locales

1. Lorsque le programme est en pause, regardez la fenêtre **Variables locales** (généralement à droite)
2. Elle affiche toutes les variables dans la portée actuelle
3. Cliquez sur le triangle à côté d'une variable pour développer sa structure

![Fenêtre des variables locales](https://doc.qt.io/qtcreator/images/qtc-debugging-locals.png)

### Ajouter des expressions à surveiller

Pour surveiller des expressions spécifiques :

1. Dans la fenêtre **Espions**, cliquez sur **Ajouter un espion**
2. Entrez le nom d'une variable ou une expression (comme `this->count()` ou `a + b`)
3. L'expression sera évaluée chaque fois que le programme s'arrêtera

### Modifier les valeurs des variables

Vous pouvez modifier la valeur d'une variable pendant le débogage :

1. Double-cliquez sur la valeur d'une variable dans la fenêtre **Variables locales** ou **Espions**
2. Entrez la nouvelle valeur
3. Appuyez sur Entrée pour confirmer

C'est très utile pour tester différents scénarios sans avoir à redémarrer le programme.

## Pile d'appels

La pile d'appels vous montre comment votre programme est arrivé à la position actuelle :

1. Examinez la fenêtre **Pile** (généralement en bas à droite)
2. Elle affiche la séquence d'appels de fonction qui a conduit à la position actuelle
3. Cliquez sur n'importe quelle entrée pour voir le code à ce niveau de la pile

![Vue de la pile d'appels](https://doc.qt.io/qtcreator/images/qtc-debug-stack.png)

## Débogage des interfaces graphiques Qt

Le débogage des interfaces Qt présente des défis particuliers :

### Mode visuel du débogueur

Qt Creator inclut des fonctionnalités pour inspecter visuellement les objets Qt :

1. Pendant le débogage, allez dans **Déboguer > Débogueurs QML/C++**
2. Utilisez **Déboguer > Afficher la hiérarchie des objets Qt** pour voir la structure de vos objets

### Débogage des signaux et slots

Pour déboguer les connexions de signaux et slots :

1. Placez un point d'arrêt dans un slot pour voir quand il est appelé
2. Utilisez **QSignalSpy** dans vos tests pour vérifier si un signal a été émis
3. Activez la journalisation des signaux et slots en ajoutant ceci au début de votre `main()` :

```cpp
QLoggingCategory::setFilterRules("qt.core.qobject.debug=true");
```

## Astuces de débogage avancées

### Exécuter des commandes du débogueur

Vous pouvez exécuter des commandes directement dans le débogueur :

1. Ouvrez la console du débogueur (**Alt+5**)
2. Tapez des commandes natives du débogueur (GDB/LLDB/CDB)

Par exemple, avec GDB, vous pouvez utiliser `p variableName` pour afficher une variable ou `bt` pour voir la pile d'appels complète.

### Points d'arrêt temporaires

Pour définir un point d'arrêt temporaire (qui disparaît après avoir été atteint une fois) :

1. Placez le curseur sur la ligne souhaitée
2. Appuyez sur **Maj+F9** au lieu de F9

### Déboguer un programme déjà en cours d'exécution

Vous pouvez attacher le débogueur à un programme Qt déjà en cours d'exécution :

1. Allez dans **Déboguer > Démarrer le débogage > Attacher au processus en cours d'exécution**
2. Sélectionnez votre programme dans la liste

## Débogage des problèmes courants

### Débogage des fuites mémoire

Pour déboguer les fuites mémoire, vous pouvez utiliser :

1. **Valgrind** (sur Linux/macOS) : **Analyser > Valgrind Memory Analyzer**
2. **Application Verifier** ou **Visual Leak Detector** (sur Windows)
3. Activez l'option **Check for memory leaks** dans les options de débogage de Qt Creator

### Débogage des crashs

Pour déboguer les crashs :

1. Exécutez votre programme en mode debug
2. Lorsqu'il plante, Qt Creator s'arrêtera à l'endroit du crash
3. Examinez la pile d'appels pour voir le chemin qui a conduit au crash
4. Vérifiez les variables pour identifier les valeurs problématiques

### Débogage des performances

Pour identifier les goulots d'étranglement :

1. Utilisez **Analyser > Performance Analyzer** dans Qt Creator
2. Il vous montrera quelles fonctions prennent le plus de temps

## Erreurs courantes et solutions

### Le débogueur ne s'arrête pas aux points d'arrêt

Causes possibles :
- Vous exécutez en mode Release au lieu de Debug
- Le code a été modifié mais pas recompilé
- Le fichier déboggé n'est pas le même que celui édité

Solution :
- Vérifiez que vous êtes en mode Debug
- Recompilez le projet entièrement (Rebuild)
- Vérifiez les chemins des fichiers sources

### Variables affichées comme "out of scope"

Cause :
- Les optimisations du compilateur peuvent modifier l'utilisation des variables

Solution :
- Ajoutez l'attribut `volatile` aux variables importantes
- Réduisez le niveau d'optimisation avec `-O0` dans les options du compilateur

## Bonnes pratiques de débogage

1. **Débogage incrémental** : Commencez par localiser approximativement le bug, puis affinez avec des points d'arrêt plus précis.

2. **Journal de bord** : Notez ce que vous découvrez pendant le débogage pour trouver des motifs.

3. **Tests de régression** : Après avoir corrigé un bug, écrivez un test qui vérifie que le bug ne réapparaît pas.

4. **Reproductibilité** : Essayez de trouver les étapes minimales pour reproduire le bug de manière fiable.

5. **Isolement** : Isolez le code problématique dans un projet de test plus petit si possible.

## Raccourcis clavier essentiels pour le débogage

| Raccourci | Action |
|-----------|--------|
| F5 | Démarrer/Continuer le débogage |
| Maj+F5 | Arrêter le débogage |
| F9 | Ajouter/Supprimer un point d'arrêt |
| F10 | Pas à pas principal |
| F11 | Pas à pas détaillé |
| Maj+F11 | Pas à pas sortant |
| Ctrl+F10 | Exécuter jusqu'au curseur |

## Conclusion

Le débogage est autant un art qu'une science. Plus vous pratiquerez, plus vous deviendrez efficace pour identifier et résoudre les problèmes dans votre code. Qt Creator offre un ensemble complet d'outils de débogage qui, une fois maîtrisés, vous feront gagner de nombreuses heures de développement.

N'oubliez pas que le meilleur bug est celui qui n'est jamais introduit. Combinez le débogage avec de bonnes pratiques de test et une conception soignée pour créer des applications Qt plus robustes et plus fiables.

Dans la section suivante, nous explorerons le profilage avec Qt Performance Analyzer, qui vous aidera à optimiser les performances de vos applications.
