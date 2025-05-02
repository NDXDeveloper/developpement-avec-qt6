# 1.2 Installation et configuration de l'environnement

Dans cette section, nous allons vous guider pas à pas pour installer Qt6 et configurer votre environnement de développement. Ne vous inquiétez pas si vous débutez, nous avons simplifié les instructions pour vous aider à démarrer rapidement.

## Téléchargement de Qt6

### Option 1 : Installateur en ligne (recommandé pour les débutants)

1. Rendez-vous sur le site officiel de Qt : [https://www.qt.io/download](https://www.qt.io/download)
2. Cliquez sur "Download Qt"
3. Choisissez l'option "Go open source"
4. Téléchargez l'"Online Installer" correspondant à votre système d'exploitation (Windows, macOS ou Linux)

### Option 2 : Installateur hors ligne

Si vous disposez d'une connexion Internet limitée, vous pouvez télécharger l'installateur hors ligne qui est plus volumineux mais ne nécessite pas de connexion pendant l'installation :

1. Sur la même page de téléchargement, recherchez l'option "Offline Installer"
2. Sélectionnez la version correspondant à votre système d'exploitation

## Processus d'installation

### Création d'un compte Qt

Lors de l'installation, vous serez invité à créer un compte Qt gratuit. Ce compte est nécessaire même pour l'utilisation de la version open source.

### Sélection des composants

L'installateur vous proposera de choisir les composants à installer :

![Exemple de l'écran de sélection des composants](illustration_selection_composants.png)

Pour débuter avec Qt6, nous vous recommandons les composants suivants :

1. **Qt 6.x (dernière version stable)** - Le framework de base
   - Cochez la case pour votre système d'exploitation principal
   - Si vous souhaitez développer pour plusieurs plateformes, cochez-les également

2. **Qt Creator** - L'environnement de développement intégré (IDE)
   - Il sera automatiquement sélectionné

3. **Outils de développement** - Compilateurs et outils nécessaires
   - Sur Windows : MinGW (plus simple pour les débutants) ou MSVC (Visual Studio)
   - Sur macOS : Xcode Command Line Tools (installées automatiquement si nécessaires)
   - Sur Linux : GCC (généralement déjà installé)

#### Conseils pour les débutants

- Évitez de tout sélectionner pour ne pas surcharger votre disque
- Vous pourrez toujours ajouter des composants plus tard si nécessaire
- Pour un premier projet, le kit standard pour votre système d'exploitation est suffisant

## Premier lancement de Qt Creator

Après l'installation, lancez Qt Creator :

1. Sur Windows : Depuis le menu Démarrer ou le raccourci du bureau
2. Sur macOS : Depuis le dossier Applications ou le Launchpad
3. Sur Linux : Depuis le menu des applications ou en tapant `qtcreator` dans le terminal

### Configuration initiale

Lors du premier démarrage, Qt Creator va détecter les kits de développement installés. Un "kit" est un ensemble d'outils comprenant :
- Un compilateur (comme MinGW ou MSVC)
- Une version de Qt (6.x)
- D'autres paramètres spécifiques à la plateforme

Qt Creator devrait configurer automatiquement vos kits. Vous verrez une liste comme celle-ci :

![Exemple de l'écran de configuration des kits](illustration_kits.png)

### Vérification de l'installation

Pour vérifier que tout fonctionne correctement :

1. Dans Qt Creator, cliquez sur **Fichier > Nouveau projet**
2. Choisissez **Application > Application Qt Widgets**
3. Suivez l'assistant pour créer un projet de base
4. Dans la page de sélection des kits, assurez-vous qu'au moins un kit Qt6 est sélectionné
5. Terminez la création du projet
6. Appuyez sur Ctrl+R (ou Cmd+R sur macOS) pour compiler et exécuter le projet

Si une fenêtre vide apparaît, félicitations ! Votre environnement Qt6 est correctement configuré.

## Configuration supplémentaire (facultatif)

### Personnalisation de Qt Creator

Qt Creator est hautement personnalisable :

1. Allez dans **Outils > Options** (ou **QtCreator > Préférences** sur macOS)
2. Explorez les différentes sections pour ajuster l'éditeur, les raccourcis clavier, etc.

### Installation d'extensions utiles

Qt Creator peut être étendu avec des plugins :

1. Dans **Aide > À propos des plugins**
2. Parcourez les plugins disponibles et activez ceux qui vous intéressent
3. Redémarrez Qt Creator pour appliquer les changements

## Résolution des problèmes courants

### Le compilateur n'est pas détecté

Si Qt Creator ne détecte pas votre compilateur :

1. Vérifiez que vous avez bien installé les outils de développement pendant l'installation
2. Sur Windows, assurez-vous que le chemin vers le compilateur est dans votre variable d'environnement PATH
3. Dans Qt Creator, allez dans **Outils > Options > Kits** et vérifiez la configuration

### Erreur "QT_QPA_PLATFORM not set"

Sur Linux, si vous rencontrez cette erreur :

1. Assurez-vous que les bibliothèques graphiques appropriées sont installées
   - Sur Ubuntu : `sudo apt-get install libgl1-mesa-dev`
   - Sur Fedora : `sudo dnf install mesa-libGL-devel`

## Prochaines étapes

Maintenant que votre environnement Qt6 est configuré, vous êtes prêt à commencer à développer ! Dans la section suivante, nous explorerons Qt Creator plus en détail et découvrirons ses fonctionnalités.
