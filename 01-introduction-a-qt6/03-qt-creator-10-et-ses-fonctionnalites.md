# 1.3 Qt Creator 10+ et ses fonctionnalités

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

Qt Creator est l'environnement de développement intégré (IDE) officiel pour Qt. La version 10+ offre de nombreuses fonctionnalités puissantes qui vous aideront à développer vos applications Qt6 efficacement. Découvrons ensemble ces outils qui rendront votre expérience de développement plus agréable.

## Interface utilisateur de Qt Creator

### Vue d'ensemble de l'interface

Lorsque vous ouvrez Qt Creator, vous découvrez une interface moderne divisée en plusieurs zones :

![Interface de Qt Creator](illustration_interface_qtcreator.png)

- **Barre de menu et barre d'outils** - En haut, accès à toutes les commandes et fonctions
- **Mode Bar** - Sur la gauche, permet de basculer entre différents modes
- **Éditeur** - Zone centrale, pour éditer votre code
- **Navigateur de projets** - Sur le côté, affiche les fichiers de votre projet
- **Panneau de sortie** - En bas, affiche les messages de compilation, débogage, etc.

### Les différents modes de travail

Qt Creator organise vos tâches en différents "modes", accessibles via la barre de gauche :

1. **Mode Bienvenue** - Page d'accueil avec accès rapide aux projets récents
2. **Mode Édition** - Pour écrire et modifier votre code
3. **Mode Design** - Interface visuelle pour concevoir vos interfaces utilisateur
4. **Mode Débogage** - Pour résoudre les problèmes dans votre code
5. **Mode Projets** - Pour configurer les paramètres de votre projet
6. **Mode Aide** - Accès à la documentation et aux exemples

## Éditeur de code avancé

### Coloration syntaxique et auto-complétion

L'éditeur de code de Qt Creator offre :

- **Coloration syntaxique** pour C++, QML et autres langages
- **Auto-complétion intelligente** qui comprend les classes et méthodes Qt
- **Indentation automatique** qui maintient votre code propre
- **Refactoring de code** pour restructurer facilement votre code

### Navigation intelligente

![Navigation dans le code](illustration_navigation_code.png)

- **Clic + F2** sur une fonction pour aller à sa définition
- **Ctrl + Clic** sur un symbole pour le suivre
- **Ctrl + Tab** pour naviguer entre les fichiers ouverts
- **Locator** (Ctrl + K) pour accéder rapidement à n'importe quelle partie de votre projet

## Concepteur d'interfaces visuelles

### Designer de widgets

Le Designer de widgets vous permet de créer des interfaces graphiques sans écrire de code :

![Qt Designer](illustration_qt_designer.png)

- **Glisser-déposer** des widgets depuis la palette sur votre formulaire
- **Éditeur de propriétés** pour modifier l'apparence et le comportement des widgets
- **Éditeur de signaux et slots** pour connecter visuellement les composants
- **Prévisualisation** pour tester votre interface avant la compilation

### Éditeur QML

Pour les interfaces modernes en QML, Qt Creator offre :

- **Éditeur visuel** pour QML avec prévisualisation en direct
- **Mode design/code** pour basculer entre l'éditeur visuel et le code
- **Auto-complétion spécifique à QML** pour vous aider à apprendre la syntaxe

## Outils de débogage intégrés

### Débogueur puissant

Le débogueur intégré vous aide à comprendre les problèmes dans votre application :

![Débogueur Qt Creator](illustration_debugger.png)

- **Points d'arrêt** pour suspendre l'exécution à des endroits précis
- **Exécution pas à pas** pour suivre le flux du programme
- **Surveillance des variables** pour suivre les valeurs pendant l'exécution
- **Pile d'appels** pour comprendre l'ordre d'exécution des fonctions

### Analyseur de performance

Pour optimiser votre application :

- **Profileur de mémoire** pour détecter les fuites mémoire
- **Analyseur de performances** pour identifier les goulots d'étranglement
- **Analyseur de code** pour trouver les problèmes potentiels

## Gestion de projets

### Support de CMake

Qt Creator 10+ offre un excellent support pour CMake, le système de build recommandé pour Qt6 :

- **Configuration automatique** des projets CMake
- **Intégration** des cibles et configurations CMake
- **Coloration syntaxique** pour les fichiers CMakeLists.txt

### Kits de développement

Les "kits" facilitent le développement multiplateforme :

- **Configuration de plusieurs compilateurs** pour différentes plateformes
- **Basculement facile** entre les plateformes cibles
- **Paramètres spécifiques à la plateforme** pour chaque kit

## Intégration Git

Qt Creator intègre directement les fonctionnalités Git :

![Intégration Git](illustration_git.png)

- **Clone, commit, push, pull** directement depuis l'IDE
- **Visualisation des différences** entre versions
- **Résolution de conflits** lors des fusions
- **Historique de fichiers** pour suivre les modifications

## Aide et documentation

### Documentation intégrée

Accédez à la documentation complète de Qt sans quitter l'IDE :

- **Recherche contextuelle** (F1) pour trouver la documentation d'une classe ou fonction
- **Exemples intégrés** pour apprendre par la pratique
- **Documentation hors ligne** disponible même sans connexion internet

### Assistant de complétion

Le nouvel assistant de complétion de Qt Creator 10+ utilise des technologies d'IA pour :

- **Suggestions de code** basées sur le contexte
- **Complétion de fonctions entières** pour accélérer le développement
- **Explications de code** pour vous aider à comprendre les concepts

## Personnalisation de l'environnement

### Thèmes et apparence

Qt Creator s'adapte à vos préférences :

- **Thèmes clair et sombre** pour l'interface
- **Thèmes d'éditeur** personnalisables pour la coloration syntaxique
- **Disposition des fenêtres** ajustable par glisser-déposer

### Raccourcis clavier

Configurez vos propres raccourcis pour travailler plus efficacement :

- **Schémas prédéfinis** (Visual Studio, Emacs, etc.)
- **Éditeur de raccourcis** pour créer vos propres combinaisons
- **Import/export** de configurations pour partager avec votre équipe

## Conseils pour les débutants

### Comment commencer avec Qt Creator

Pour vous familiariser rapidement avec Qt Creator :

1. **Explorez les exemples** - Allez dans Aide > Exemples et ouvrez quelques projets
2. **Utilisez l'aide contextuelle** - Appuyez sur F1 sur n'importe quel élément pour afficher sa documentation
3. **Personnalisez votre espace** - Adaptez l'IDE à votre style de travail dès le début

### Raccourcis clavier essentiels

Quelques raccourcis à connaître absolument :

- **Ctrl+R** (Cmd+R sur macOS) - Exécuter le projet
- **Ctrl+B** - Compiler le projet
- **F5** - Démarrer le débogage
- **Ctrl+K** - Ouvrir le localisateur pour naviguer rapidement
- **Ctrl+Espace** - Déclencher l'auto-complétion

## Fonctionnalités récentes de Qt Creator 10+

Les dernières versions de Qt Creator apportent des améliorations notables :

- **Support amélioré pour CMake**
- **Meilleures performances** pour les grands projets
- **Nouvel outil d'analyse de code C++** intégré
- **Thème sombre moderne** par défaut
- **Intégration de LSP** (Language Server Protocol) pour un meilleur support des langages

---

Dans la prochaine section, nous explorerons le principe fondamental de Qt : les signaux et slots, qui permettent la communication entre les objets de vos applications.

⏭️ [Principe des signaux et slots dans Qt6](/01-introduction-a-qt6/04-principe-des-signaux-et-slots-dans-qt6.md)
