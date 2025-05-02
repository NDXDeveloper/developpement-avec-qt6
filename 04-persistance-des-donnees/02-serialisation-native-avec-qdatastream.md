# 4.2 Sérialisation native avec QDataStream

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

## Introduction à la sérialisation avec QDataStream

La sérialisation est le processus qui transforme des objets en mémoire en une séquence d'octets pouvant être stockée dans un fichier ou transmise sur un réseau. QDataStream est la solution native de Qt pour effectuer cette sérialisation au format binaire.

## Pourquoi utiliser QDataStream ?

QDataStream présente plusieurs avantages par rapport à d'autres méthodes de persistance :

- **Performance** : Format binaire compact et rapide à traiter
- **Support natif** des types Qt comme QString, QList, QMap, etc.
- **Préservation exacte** des données, y compris les types spécifiques à Qt
- **Contrôle de version** permettant de maintenir la compatibilité entre différentes versions de votre application

## QDataStream vs autres méthodes

| Méthode | Avantages | Inconvénients | Cas d'utilisation |
|---------|-----------|---------------|-------------------|
| QDataStream | Rapide, support natif des types Qt | Format binaire non lisible, spécifique à Qt | Sauvegarde locale de données complexes |
| JSON | Lisible, interopérable | Moins efficace en taille, conversion nécessaire | Communication avec services web |
| XML | Très structuré, validation de schéma | Verbeux, traitement plus lent | Documents complexes, configurations |
| Bases de données | Requêtes avancées, relations | Plus complexe à configurer | Grandes quantités de données structurées |

## Les bases de QDataStream

QDataStream fonctionne avec les flux d'entrée/sortie de Qt (QIODevice), notamment :

- **QFile** pour lire/écrire des fichiers
- **QBuffer** pour lire/écrire dans la mémoire
- **QTcpSocket** pour la communication réseau

Voici un exemple simple pour sauvegarder une liste de nombres dans un fichier :

```cpp
#include <QFile>
#include <QDataStream>
#include <QList>
#include <QDebug>

bool sauvegarderDonnees()
{
    // Données à sauvegarder
    QList<int> nombres = {10, 20, 30, 40, 50};
    QString titre = "Ma liste de nombres";

    // Ouverture du fichier en écriture
    QFile fichier("donnees.dat");
    if (!fichier.open(QIODevice::WriteOnly)) {
        qDebug() << "Impossible d'ouvrir le fichier en écriture";
        return false;
    }

    // Création du flux de données
    QDataStream flux(&fichier);

    // Définition de la version du format (important pour la compatibilité)
    flux.setVersion(QDataStream::Qt_6_0);

    // Écriture des données
    flux << titre << nombres;

    // Le fichier est fermé automatiquement à la fin de la portée
    return true;
}

bool chargerDonnees()
{
    QList<int> nombres;
    QString titre;

    // Ouverture du fichier en lecture
    QFile fichier("donnees.dat");
    if (!fichier.open(QIODevice::ReadOnly)) {
        qDebug() << "Impossible d'ouvrir le fichier en lecture";
        return false;
    }

    // Création du flux de données
    QDataStream flux(&fichier);

    // Définition de la version du format (doit correspondre à celle utilisée pour l'écriture)
    flux.setVersion(QDataStream::Qt_6_0);

    // Lecture des données
    flux >> titre >> nombres;

    // Affichage des données lues
    qDebug() << "Titre:" << titre;
    qDebug() << "Nombres:" << nombres;

    return true;
}
```

## Sérialisation de classes personnalisées

Pour sérialiser vos propres classes, vous devez surcharger les opérateurs `<<` et `>>` pour QDataStream. Voici un exemple avec une classe simple :

```cpp
// Définition de la classe
class Personne
{
public:
    Personne() {} // Constructeur par défaut nécessaire pour la désérialisation
    Personne(const QString &nom, int age, const QDate &dateNaissance)
        : m_nom(nom), m_age(age), m_dateNaissance(dateNaissance) {}

    QString nom() const { return m_nom; }
    int age() const { return m_age; }
    QDate dateNaissance() const { return m_dateNaissance; }

    void setNom(const QString &nom) { m_nom = nom; }
    void setAge(int age) { m_age = age; }
    void setDateNaissance(const QDate &date) { m_dateNaissance = date; }

    // Déclaration des opérateurs comme fonctions amies
    friend QDataStream &operator<<(QDataStream &flux, const Personne &personne);
    friend QDataStream &operator>>(QDataStream &flux, Personne &personne);

private:
    QString m_nom;
    int m_age;
    QDate m_dateNaissance;
};

// Implémentation de l'opérateur de sérialisation (<<)
QDataStream &operator<<(QDataStream &flux, const Personne &personne)
{
    flux << personne.m_nom << personne.m_age << personne.m_dateNaissance;
    return flux;
}

// Implémentation de l'opérateur de désérialisation (>>)
QDataStream &operator>>(QDataStream &flux, Personne &personne)
{
    flux >> personne.m_nom >> personne.m_age >> personne.m_dateNaissance;
    return flux;
}
```

Utilisation de cette classe avec QDataStream :

```cpp
bool sauvegarderPersonnes()
{
    // Création de quelques personnes
    QList<Personne> personnes;
    personnes.append(Personne("Alice Dupont", 28, QDate(1997, 5, 12)));
    personnes.append(Personne("Bob Martin", 35, QDate(1990, 11, 3)));
    personnes.append(Personne("Charlie Dubois", 22, QDate(2003, 2, 25)));

    // Ouverture du fichier en écriture
    QFile fichier("personnes.dat");
    if (!fichier.open(QIODevice::WriteOnly)) {
        return false;
    }

    QDataStream flux(&fichier);
    flux.setVersion(QDataStream::Qt_6_0);

    // Écriture de la version de notre format (pour notre propre gestion de version)
    const quint32 MAGIC_NUMBER = 0x50455253; // "PERS" en hexadécimal
    const quint16 VERSION = 1;

    flux << MAGIC_NUMBER << VERSION;

    // Écriture du nombre de personnes
    flux << (quint32)personnes.size();

    // Écriture de chaque personne
    for (const Personne &personne : personnes) {
        flux << personne;
    }

    return true;
}

bool chargerPersonnes(QList<Personne> &personnes)
{
    personnes.clear();

    QFile fichier("personnes.dat");
    if (!fichier.open(QIODevice::ReadOnly)) {
        return false;
    }

    QDataStream flux(&fichier);
    flux.setVersion(QDataStream::Qt_6_0);

    // Lecture et vérification du magic number et de la version
    quint32 magicNumber;
    quint16 version;

    flux >> magicNumber >> version;

    if (magicNumber != 0x50455253) {
        qDebug() << "Ce n'est pas un fichier de personnes valide";
        return false;
    }

    if (version > 1) {
        qDebug() << "Version de fichier non supportée";
        return false;
    }

    // Lecture du nombre de personnes
    quint32 nombrePersonnes;
    flux >> nombrePersonnes;

    // Lecture de chaque personne
    for (quint32 i = 0; i < nombrePersonnes; ++i) {
        Personne personne;
        flux >> personne;
        personnes.append(personne);
    }

    return true;
}
```

## Gestion des versions et compatibilité

La gestion de version est cruciale pour maintenir la compatibilité lorsque votre application évolue. QDataStream propose deux niveaux de gestion de version :

1. **Version de QDataStream** : Définit comment les types natifs de Qt sont sérialisés
2. **Version de votre format** : Que vous définissez vous-même (avec le MAGIC_NUMBER et VERSION dans l'exemple)

Pour gérer l'évolution de vos classes, voici quelques stratégies :

### Exemple d'évolution d'une classe

Imaginons que nous ajoutions un champ à notre classe Personne :

```cpp
// Version 2 de la classe Personne (ajout de l'adresse email)
class Personne
{
    // ... (même code qu'avant)

private:
    QString m_nom;
    int m_age;
    QDate m_dateNaissance;
    QString m_email; // Nouveau champ
};

// Opérateur de sérialisation version 2
QDataStream &operator<<(QDataStream &flux, const Personne &personne)
{
    flux << personne.m_nom << personne.m_age << personne.m_dateNaissance << personne.m_email;
    return flux;
}

// Opérateur de désérialisation avec gestion de version
QDataStream &operator>>(QDataStream &flux, Personne &personne)
{
    flux >> personne.m_nom >> personne.m_age >> personne.m_dateNaissance;

    // Vérification de la version (définie ailleurs dans le code)
    if (g_versionFichier >= 2) {
        flux >> personne.m_email;
    } else {
        // Valeur par défaut pour les anciennes versions
        personne.m_email = "inconnu@exemple.com";
    }

    return flux;
}
```

## Utilisation de QBuffer pour la sérialisation en mémoire

QDataStream peut aussi être utilisé avec QBuffer pour sérialiser des données en mémoire :

```cpp
// Sérialisation en mémoire
QByteArray serialiserEnMemoire(const Personne &personne)
{
    QByteArray donnees;
    QBuffer tampon(&donnees);
    tampon.open(QIODevice::WriteOnly);

    QDataStream flux(&tampon);
    flux.setVersion(QDataStream::Qt_6_0);
    flux << personne;

    tampon.close();
    return donnees;
}

// Désérialisation depuis la mémoire
Personne deserialiserDepuisMemoire(const QByteArray &donnees)
{
    Personne personne;
    QBuffer tampon(const_cast<QByteArray*>(&donnees));
    tampon.open(QIODevice::ReadOnly);

    QDataStream flux(&tampon);
    flux.setVersion(QDataStream::Qt_6_0);
    flux >> personne;

    tampon.close();
    return personne;
}
```

## Exemple pratique : Gestionnaire de sauvegarde

Voici un exemple complet d'une classe qui gère la sauvegarde et le chargement de données d'application :

```cpp
// gestionnaire_sauvegarde.h
#ifndef GESTIONNAIRE_SAUVEGARDE_H
#define GESTIONNAIRE_SAUVEGARDE_H

#include <QObject>
#include <QString>
#include <QList>
#include <QMap>
#include <QDate>

class Personne; // Déclaration anticipée

class GestionnaireSauvegarde : public QObject
{
    Q_OBJECT

public:
    explicit GestionnaireSauvegarde(QObject *parent = nullptr);

    bool sauvegarderDonnees(const QString &nomFichier, const QList<Personne> &personnes);
    bool chargerDonnees(const QString &nomFichier, QList<Personne> &personnes);

    QString dernierMessage() const { return m_dernierMessage; }

private:
    QString m_dernierMessage;
};

#endif // GESTIONNAIRE_SAUVEGARDE_H
```

```cpp
// gestionnaire_sauvegarde.cpp
#include "gestionnaire_sauvegarde.h"
#include "personne.h"
#include <QFile>
#include <QDataStream>
#include <QDebug>

// Magic number pour identifier nos fichiers de données
constexpr quint32 MAGIC_NUMBER = 0x50455253; // "PERS" en hexadécimal
constexpr quint16 VERSION_ACTUELLE = 1;

GestionnaireSauvegarde::GestionnaireSauvegarde(QObject *parent)
    : QObject(parent)
{
}

bool GestionnaireSauvegarde::sauvegarderDonnees(const QString &nomFichier, const QList<Personne> &personnes)
{
    QFile fichier(nomFichier);
    if (!fichier.open(QIODevice::WriteOnly)) {
        m_dernierMessage = tr("Impossible d'ouvrir le fichier %1 en écriture: %2")
                             .arg(nomFichier, fichier.errorString());
        return false;
    }

    QDataStream flux(&fichier);
    flux.setVersion(QDataStream::Qt_6_0);

    // Écriture de l'en-tête
    flux << MAGIC_NUMBER << VERSION_ACTUELLE;

    // Écriture du nombre de personnes
    flux << (quint32)personnes.size();

    // Écriture de chaque personne
    for (const Personne &personne : personnes) {
        flux << personne;
    }

    m_dernierMessage = tr("Sauvegarde réussie de %1 personnes dans %2")
                         .arg(personnes.size())
                         .arg(nomFichier);
    return true;
}

bool GestionnaireSauvegarde::chargerDonnees(const QString &nomFichier, QList<Personne> &personnes)
{
    personnes.clear();

    QFile fichier(nomFichier);
    if (!fichier.open(QIODevice::ReadOnly)) {
        m_dernierMessage = tr("Impossible d'ouvrir le fichier %1 en lecture: %2")
                             .arg(nomFichier, fichier.errorString());
        return false;
    }

    QDataStream flux(&fichier);
    flux.setVersion(QDataStream::Qt_6_0);

    // Lecture et vérification de l'en-tête
    quint32 magicNumber;
    quint16 version;

    flux >> magicNumber >> version;

    if (magicNumber != MAGIC_NUMBER) {
        m_dernierMessage = tr("Le fichier %1 n'est pas un fichier de données valide")
                             .arg(nomFichier);
        return false;
    }

    if (version > VERSION_ACTUELLE) {
        m_dernierMessage = tr("Le fichier %1 utilise une version plus récente (%2) non supportée")
                             .arg(nomFichier)
                             .arg(version);
        return false;
    }

    // Lecture du nombre de personnes
    quint32 nombrePersonnes;
    flux >> nombrePersonnes;

    // Lecture de chaque personne
    for (quint32 i = 0; i < nombrePersonnes; ++i) {
        Personne personne;
        flux >> personne;
        personnes.append(personne);
    }

    m_dernierMessage = tr("Chargement réussi de %1 personnes depuis %2")
                         .arg(nombrePersonnes)
                         .arg(nomFichier);
    return true;
}
```

## Conseils et bonnes pratiques

Pour une utilisation efficace de QDataStream :

1. **Toujours spécifier la version** avec `setVersion()` pour assurer la compatibilité.
2. **Vérifier les erreurs d'entrée/sortie** en testant le statut du flux avec `status()`.
3. **Utiliser des magic numbers** pour identifier vos fichiers.
4. **Inclure votre propre numéro de version** pour gérer l'évolution de vos formats.
5. **Prévoir la compatibilité ascendante** dans vos opérateurs de désérialisation.
6. **Tester la désérialisation** avec différentes versions de vos fichiers.

## Pièges courants à éviter

- **Oublier de définir la version** du flux.
- **Ne pas prévoir l'évolution** des classes sérialisées.
- **Utiliser des pointeurs** sans gérer correctement leur désérialisation.
- **Ignorer les erreurs d'entrée/sortie**.
- **Sérialiser des informations sensibles** sans chiffrement.

## Conclusion

QDataStream offre une solution puissante et native pour la sérialisation de données dans Qt. C'est particulièrement adapté pour :

- Les sauvegardes locales d'état d'application
- Le stockage efficace de données complexes
- La communication entre applications Qt

Dans la prochaine section, nous explorerons la sérialisation JSON avec QJsonDocument, un format plus interopérable et lisible par l'humain.

⏭️ [Sérialisation JSON avec QJsonDocument](/04-persistance-des-donnees/03-serialisation-json-avec-qjsondocument.md)
