# 4. Persistance des données

## Introduction à la persistance des données

La persistance des données est un concept fondamental dans le développement d'applications. Elle permet de sauvegarder des informations de manière durable, afin qu'elles soient disponibles même après la fermeture de l'application ou le redémarrage de l'ordinateur.

Dans cette section, nous allons explorer les différentes méthodes offertes par Qt6 pour gérer la persistance des données dans vos applications.

## Pourquoi la persistance des données est-elle importante ?

Imaginons une application de prise de notes. Si les notes créées par l'utilisateur disparaissaient à chaque fermeture de l'application, celle-ci serait pratiquement inutile ! La persistance des données permet de :

- Sauvegarder les préférences de l'utilisateur
- Conserver les données créées par l'utilisateur
- Mémoriser l'état de l'application entre chaque utilisation
- Partager des données entre différentes instances de l'application

## Vue d'ensemble des méthodes de persistance dans Qt6

Qt6 propose plusieurs approches pour la persistance des données, chacune adaptée à des besoins spécifiques :

| Méthode | Description | Cas d'utilisation |
|---------|-------------|-------------------|
| Qt SQL | Stockage structuré dans des bases de données relationnelles | Applications avec données complexes et relations |
| QDataStream | Sérialisation binaire native Qt | Sauvegarde rapide d'objets Qt |
| QJsonDocument | Sérialisation au format JSON | Échange de données avec des services web |
| QXmlStreamWriter/Reader | Sérialisation au format XML | Configuration, données structurées hiérarchiques |
| QSettings | Stockage simple de paramètres | Préférences utilisateur, état de l'application |

## Comprendre la sérialisation

Avant d'entrer dans les détails, il est important de comprendre ce qu'est la **sérialisation**. Il s'agit du processus de conversion d'un objet en mémoire (comme une liste, une structure ou une classe) en un format qui peut être stocké ou transmis, puis reconverti plus tard en un objet identique.

Dans Qt6, la sérialisation peut se faire en plusieurs formats :
- Format binaire (avec QDataStream)
- Format texte lisible (JSON, XML)

Chaque format présente ses avantages et inconvénients :

**Format binaire** :
- ✅ Rapide et compact
- ✅ Préserve les types Qt natifs
- ❌ Non lisible par l'humain
- ❌ Potentiellement incompatible entre différentes versions de Qt

**Format JSON** :
- ✅ Lisible par l'humain
- ✅ Compatible avec de nombreux systèmes et langages
- ✅ Idéal pour l'interopérabilité
- ❌ Moins efficace pour les grands volumes de données

**Format XML** :
- ✅ Très structuré et extensible
- ✅ Support de schémas de validation
- ✅ Standard bien établi
- ❌ Plus verbeux que JSON

## Préparation à la suite

Dans les sections suivantes, nous explorerons en détail chacune de ces méthodes de persistance des données, en commençant par Qt SQL pour le stockage en base de données relationelle.

Nous verrons comment :
- Connecter votre application à une base de données
- Sérialiser des objets Qt en différents formats
- Gérer les préférences utilisateur
- Choisir la méthode la plus adaptée à vos besoins spécifiques

Chaque méthode sera illustrée par des exemples de code simples et complets pour vous aider à comprendre et appliquer ces concepts dans vos propres applications.
