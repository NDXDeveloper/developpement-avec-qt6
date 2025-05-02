# 9.1 Configuration des projets multi-plateformes avec CMake

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

## Introduction à CMake dans Qt6

Depuis Qt6, CMake est devenu le système de build officiel, remplaçant qmake qui était utilisé dans les versions précédentes. CMake offre une grande flexibilité pour gérer des projets multi-plateformes et s'intègre parfaitement avec Qt Creator.

## Pourquoi utiliser CMake avec Qt6 ?

CMake présente plusieurs avantages pour les projets Qt multi-plateformes :

- **Standardisation** : CMake est devenu un standard de l'industrie pour les projets C++
- **Flexibilité** : Plus puissant que qmake pour gérer des configurations complexes
- **Écosystème** : Meilleure intégration avec d'autres outils et bibliothèques
- **Génération de projets** : Capable de générer des fichiers de projet pour différents IDE et systèmes de build

## Premiers pas avec CMake et Qt6

### Installation et prérequis

Pour utiliser CMake avec Qt6, vous devez :

1. Installer Qt6 avec Qt Creator
2. S'assurer que CMake (version 3.16 ou supérieure) est installé sur votre système
3. Vérifier que Qt Creator est configuré pour utiliser CMake

Dans Qt Creator, vous pouvez vérifier votre configuration dans :
`Édition > Préférences > Kits > [Votre kit] > CMake`

### Structure d'un projet CMake pour Qt6

Un projet Qt6 basique avec CMake comprend généralement ces fichiers :

- `CMakeLists.txt` : Fichier principal de configuration du projet
- `src/` : Dossier contenant les fichiers source
- `include/` : Dossier contenant les fichiers d'en-tête
- `resources/` : Dossier contenant les ressources (images, fichiers QML, etc.)

### Créer un projet Qt6 avec CMake

#### Méthode 1 : Utiliser Qt Creator

La façon la plus simple est d'utiliser Qt Creator :

1. Lancer Qt Creator
2. Sélectionner `Fichier > Nouveau projet/fichier`
3. Choisir un modèle de projet Qt (Application Qt Widgets ou Qt Quick)
4. Suivre l'assistant en sélectionnant CMake comme système de build

#### Méthode 2 : Créer manuellement les fichiers

Vous pouvez aussi créer un projet à partir de zéro :

1. Créer un dossier pour votre projet
2. Créer un fichier `CMakeLists.txt` à la racine
3. Ajouter vos fichiers source dans un sous-dossier `src/`

## Anatomie d'un fichier CMakeLists.txt pour Qt6

Voici un exemple de `CMakeLists.txt` basique pour une application Qt6 :

```cmake
# Version minimale de CMake requise
cmake_minimum_required(VERSION 3.16)

# Nom et version du projet
project(MonAppQt VERSION 1.0 LANGUAGES CXX)

# Configuration Qt
set(CMAKE_AUTOMOC ON)  # Activer le Meta-Object Compiler
set(CMAKE_AUTORCC ON)  # Activer le Resource Compiler
set(CMAKE_AUTOUIC ON)  # Activer l'UI Compiler

# Définir le standard C++
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Trouver les packages Qt nécessaires
find_package(Qt6 REQUIRED COMPONENTS Core Gui Widgets)

# Ajouter les fichiers sources
set(SOURCES
    src/main.cpp
    src/mainwindow.cpp
)

# Ajouter les fichiers d'en-tête
set(HEADERS
    include/mainwindow.h
)

# Ajouter les fichiers UI
set(UIS
    ui/mainwindow.ui
)

# Ajouter les fichiers de ressources
set(RESOURCES
    resources/resources.qrc
)

# Créer l'exécutable
add_executable(${PROJECT_NAME}
    ${SOURCES}
    ${HEADERS}
    ${UIS}
    ${RESOURCES}
)

# Lier les bibliothèques Qt
target_link_libraries(${PROJECT_NAME} PRIVATE
    Qt6::Core
    Qt6::Gui
    Qt6::Widgets
)

# Inclure les dossiers d'en-têtes
target_include_directories(${PROJECT_NAME} PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/include
)

# Installation (optionnel)
install(TARGETS ${PROJECT_NAME}
    BUNDLE DESTINATION .
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)
```

Expliquons les parties essentielles :

### Configuration de base

```cmake
cmake_minimum_required(VERSION 3.16)
project(MonAppQt VERSION 1.0 LANGUAGES CXX)
```

- `cmake_minimum_required` : Définit la version minimale de CMake nécessaire
- `project` : Définit le nom du projet, sa version et les langages utilisés

### Configuration Qt

```cmake
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)
```

Ces options activent les outils spécifiques à Qt :
- `AUTOMOC` : Active le Meta-Object Compiler pour les signaux et slots
- `AUTORCC` : Active la compilation des fichiers de ressources (.qrc)
- `AUTOUIC` : Active la compilation des fichiers d'interface utilisateur (.ui)

### Trouver Qt

```cmake
find_package(Qt6 REQUIRED COMPONENTS Core Gui Widgets)
```

Cette commande recherche les composants Qt6 installés sur votre système. Vous devez spécifier tous les modules Qt dont votre application a besoin.

### Compilation et édition de liens

```cmake
add_executable(${PROJECT_NAME} ...)
target_link_libraries(${PROJECT_NAME} PRIVATE Qt6::Core Qt6::Gui Qt6::Widgets)
```

- `add_executable` : Crée l'exécutable avec les fichiers source spécifiés
- `target_link_libraries` : Lie l'exécutable aux bibliothèques Qt nécessaires

## Configurations spécifiques aux plateformes

L'un des principaux avantages de CMake est sa capacité à gérer des configurations spécifiques à chaque plateforme.

### Détection de la plateforme

```cmake
if(WIN32)
    # Configuration Windows
elseif(APPLE)
    # Configuration macOS
elseif(ANDROID)
    # Configuration Android
elseif(UNIX AND NOT APPLE)
    # Configuration Linux
endif()
```

### Exemple : Configuration spécifique à Windows

```cmake
if(WIN32)
    # Ajouter l'icône Windows
    set(APP_ICON_RESOURCE_WINDOWS "${CMAKE_CURRENT_SOURCE_DIR}/resources/icon.rc")

    # Ajouter le manifeste Visual Studio
    set(CMAKE_EXE_LINKER_FLAGS
        "${CMAKE_EXE_LINKER_FLAGS} /MANIFEST:NO")

    # Inclure l'icône dans l'exécutable
    target_sources(${PROJECT_NAME} PRIVATE ${APP_ICON_RESOURCE_WINDOWS})
endif()
```

### Exemple : Configuration spécifique à macOS

```cmake
if(APPLE)
    # Définir les informations du bundle macOS
    set(MACOSX_BUNDLE_ICON_FILE appicon.icns)
    set(MACOSX_BUNDLE_SHORT_VERSION_STRING ${PROJECT_VERSION})
    set(MACOSX_BUNDLE_BUNDLE_VERSION ${PROJECT_VERSION})
    set(MACOSX_BUNDLE_BUNDLE_NAME ${PROJECT_NAME})

    # Ajouter l'icône au bundle
    set(APP_ICON_MACOSX ${CMAKE_CURRENT_SOURCE_DIR}/resources/appicon.icns)
    set_source_files_properties(${APP_ICON_MACOSX} PROPERTIES
        MACOSX_PACKAGE_LOCATION "Resources")
    target_sources(${PROJECT_NAME} PRIVATE ${APP_ICON_MACOSX})
endif()
```

## Gestion des ressources multi-plateformes

CMake facilite la gestion des ressources pour différentes plateformes :

```cmake
# Ressources communes
set(COMMON_RESOURCES resources/common/resources.qrc)

# Ressources spécifiques aux plateformes
if(ANDROID)
    set(PLATFORM_RESOURCES resources/android/android_resources.qrc)
elseif(WIN32)
    set(PLATFORM_RESOURCES resources/windows/windows_resources.qrc)
elseif(APPLE)
    set(PLATFORM_RESOURCES resources/macos/macos_resources.qrc)
else()
    set(PLATFORM_RESOURCES resources/linux/linux_resources.qrc)
endif()

# Ajouter toutes les ressources à l'exécutable
target_sources(${PROJECT_NAME} PRIVATE
    ${COMMON_RESOURCES}
    ${PLATFORM_RESOURCES}
)
```

## Création de projets pour différentes plateformes

CMake génère les fichiers de projet appropriés pour chaque plateforme cible. Voici comment configurer votre projet pour différentes plateformes :

### Pour Windows (Visual Studio)

```bash
mkdir build-windows
cd build-windows
cmake -G "Visual Studio 16 2019" -A x64 ..
```

### Pour macOS (Xcode)

```bash
mkdir build-macos
cd build-macos
cmake -G "Xcode" ..
```

### Pour Linux

```bash
mkdir build-linux
cd build-linux
cmake ..
make
```

### Pour Android

```bash
mkdir build-android
cd build-android
cmake .. -DCMAKE_TOOLCHAIN_FILE=$ANDROID_NDK/build/cmake/android.toolchain.cmake -DANDROID_ABI=arm64-v8a -DANDROID_PLATFORM=android-23
```

## Les variables conditionnelles

Les variables conditionnelles permettent de personnaliser la configuration selon la plateforme ou d'autres conditions :

```cmake
# Définir une option de compilation
option(USE_OPENGL "Utiliser OpenGL" ON)

# Utiliser l'option conditionnellement
if(USE_OPENGL)
    find_package(Qt6 REQUIRED COMPONENTS OpenGL)
    target_link_libraries(${PROJECT_NAME} PRIVATE Qt6::OpenGL)
    target_compile_definitions(${PROJECT_NAME} PRIVATE USE_OPENGL)
endif()
```

## Débogage de CMake

Si vous rencontrez des problèmes avec votre configuration CMake :

1. Utilisez `message()` pour afficher des informations :
   ```cmake
   message(STATUS "Système d'exploitation: ${CMAKE_SYSTEM_NAME}")
   message(STATUS "Compilateur: ${CMAKE_CXX_COMPILER_ID}")
   ```

2. Activez le mode verbeux lors de la génération :
   ```bash
   cmake -DCMAKE_VERBOSE_MAKEFILE=ON ..
   ```

3. Utilisez l'outil CMake GUI pour explorer et modifier les variables.

## Astuces pour les débutants

1. **Commencez petit** : Créez d'abord un projet simple qui fonctionne, puis ajoutez progressivement des fonctionnalités.

2. **Utilisez les templates** : Qt Creator fournit des templates de projet qui créent des fichiers CMake corrects.

3. **Consultez la documentation** : La documentation de CMake et Qt contient de nombreux exemples utiles.

4. **Organisez votre code** : Adoptez une structure de dossiers cohérente dès le début.

5. **Vérifiez régulièrement** : Testez votre projet sur toutes les plateformes cibles aussi tôt que possible.

## Conclusion

La configuration de projets multi-plateformes avec CMake peut sembler complexe au début, mais elle offre une grande flexibilité et puissance. En maîtrisant ces concepts de base, vous serez en mesure de créer des projets Qt6 qui fonctionnent de manière cohérente sur toutes les plateformes cibles.

Dans les sections suivantes du tutoriel, nous explorerons les spécificités de chaque plateforme et comment optimiser votre application pour chacune d'elles.

⏭️ [Spécificités Windows, Linux, macOS](/09-developpement-multiplateforme-avec-qt6/02-specificites-windows-linux-macos.md)
