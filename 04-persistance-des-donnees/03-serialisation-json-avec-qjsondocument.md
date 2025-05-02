# 4.3 Sérialisation JSON avec QJsonDocument

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

## Introduction au format JSON

JSON (JavaScript Object Notation) est un format de données textuelles léger, facile à lire pour les humains et simple à analyser pour les machines. Il est devenu l'un des formats les plus populaires pour l'échange de données, particulièrement dans les applications web et les API REST.

## Avantages du format JSON

Le JSON présente plusieurs avantages par rapport à d'autres formats de données :

- **Lisibilité** : Format texte facile à comprendre et à déboguer
- **Légèreté** : Moins verbeux que XML et donc plus compact
- **Compatibilité** : Supporté par presque tous les langages de programmation
- **Interopérabilité** : Idéal pour communiquer avec des systèmes externes
- **Indépendance** : Ne dépend pas de Qt, contrairement à QDataStream

## Structure du JSON

Un document JSON est structuré selon ces règles simples :

- Les données sont organisées en paires clé/valeur (comme un dictionnaire)
- Les données sont séparées par des virgules
- Les objets sont entourés par des accolades `{}`
- Les tableaux sont entourés par des crochets `[]`
- Les valeurs peuvent être de type :
  - Chaîne de caractères (entre guillemets doubles)
  - Nombre
  - Objet
  - Tableau
  - Booléen (`true` ou `false`)
  - `null`

Exemple de document JSON :

```json
{
    "nom": "Alice Dupont",
    "age": 28,
    "estEtudiant": false,
    "adresse": {
        "rue": "123 Avenue des Champs",
        "ville": "Paris",
        "codePostal": "75008"
    },
    "competences": ["C++", "Qt", "JavaScript", "SQL"]
}
```

## Support JSON dans Qt

Qt offre un ensemble de classes dédiées au traitement du JSON :

- **QJsonDocument** : Représente un document JSON complet
- **QJsonObject** : Représente un objet JSON (ensemble de paires clé/valeur)
- **QJsonArray** : Représente un tableau JSON
- **QJsonValue** : Représente une valeur JSON (peut être de n'importe quel type)
- **QJsonParseError** : Utilisé pour signaler les erreurs d'analyse du JSON

## Création d'un document JSON

Voici comment créer un document JSON simple en C++ avec Qt :

```cpp
#include <QJsonDocument>
#include <QJsonObject>
#include <QJsonArray>
#include <QDebug>

void creerDocumentJSON()
{
    // Création d'un objet JSON
    QJsonObject personne;

    // Ajout de propriétés simples
    personne["nom"] = "Alice Dupont";
    personne["age"] = 28;
    personne["estEtudiant"] = false;

    // Création et ajout d'un sous-objet
    QJsonObject adresse;
    adresse["rue"] = "123 Avenue des Champs";
    adresse["ville"] = "Paris";
    adresse["codePostal"] = "75008";

    personne["adresse"] = adresse;

    // Création et ajout d'un tableau
    QJsonArray competences;
    competences.append("C++");
    competences.append("Qt");
    competences.append("JavaScript");
    competences.append("SQL");

    personne["competences"] = competences;

    // Création du document JSON à partir de l'objet
    QJsonDocument document(personne);

    // Conversion en chaîne formatée (indentée)
    QString jsonFormate = document.toJson(QJsonDocument::Indented);

    qDebug().noquote() << "Document JSON créé :" << jsonFormate;
}
```

## Lecture d'un document JSON

Pour lire un document JSON :

```cpp
void lireDocumentJSON(const QString &jsonString)
{
    // Conversion de la chaîne en QJsonDocument
    QJsonParseError erreur;
    QJsonDocument document = QJsonDocument::fromJson(jsonString.toUtf8(), &erreur);

    if (erreur.error != QJsonParseError::NoError) {
        qDebug() << "Erreur d'analyse JSON :" << erreur.errorString();
        return;
    }

    // Vérification que le document contient un objet
    if (!document.isObject()) {
        qDebug() << "Le document JSON n'est pas un objet";
        return;
    }

    // Récupération de l'objet principal
    QJsonObject personne = document.object();

    // Lecture des propriétés simples
    QString nom = personne["nom"].toString();
    int age = personne["age"].toInt();
    bool estEtudiant = personne["estEtudiant"].toBool();

    qDebug() << "Nom :" << nom;
    qDebug() << "Âge :" << age;
    qDebug() << "Est étudiant :" << estEtudiant;

    // Lecture d'un sous-objet
    if (personne.contains("adresse") && personne["adresse"].isObject()) {
        QJsonObject adresse = personne["adresse"].toObject();

        QString rue = adresse["rue"].toString();
        QString ville = adresse["ville"].toString();
        QString codePostal = adresse["codePostal"].toString();

        qDebug() << "Adresse :";
        qDebug() << "  Rue :" << rue;
        qDebug() << "  Ville :" << ville;
        qDebug() << "  Code postal :" << codePostal;
    }

    // Lecture d'un tableau
    if (personne.contains("competences") && personne["competences"].isArray()) {
        QJsonArray competences = personne["competences"].toArray();

        qDebug() << "Compétences :";
        for (const QJsonValue &valeur : competences) {
            qDebug() << "  -" << valeur.toString();
        }
    }
}
```

## Sauvegarde et chargement depuis un fichier

Pour sauvegarder un document JSON dans un fichier et le charger :

```cpp
#include <QFile>
#include <QJsonDocument>
#include <QJsonObject>
#include <QDebug>

bool sauvegarderJSON(const QJsonDocument &document, const QString &nomFichier)
{
    QFile fichier(nomFichier);
    if (!fichier.open(QIODevice::WriteOnly)) {
        qDebug() << "Impossible d'ouvrir le fichier" << nomFichier << "en écriture :"
                 << fichier.errorString();
        return false;
    }

    // Écriture du document JSON formaté dans le fichier
    fichier.write(document.toJson(QJsonDocument::Indented));
    return true;
}

QJsonDocument chargerJSON(const QString &nomFichier)
{
    QFile fichier(nomFichier);
    if (!fichier.open(QIODevice::ReadOnly)) {
        qDebug() << "Impossible d'ouvrir le fichier" << nomFichier << "en lecture :"
                 << fichier.errorString();
        return QJsonDocument();
    }

    // Lecture du contenu du fichier
    QByteArray donnees = fichier.readAll();

    // Analyse du JSON
    QJsonParseError erreur;
    QJsonDocument document = QJsonDocument::fromJson(donnees, &erreur);

    if (erreur.error != QJsonParseError::NoError) {
        qDebug() << "Erreur d'analyse JSON :" << erreur.errorString()
                 << "à la position" << erreur.offset;
        return QJsonDocument();
    }

    return document;
}
```

## Sérialisation de classes personnalisées en JSON

Contrairement à QDataStream, Qt n'offre pas de mécanisme automatique pour sérialiser des classes en JSON. Vous devez implémenter vos propres méthodes de conversion :

```cpp
// Définition de la classe
class Personne
{
public:
    Personne() {}
    Personne(const QString &nom, int age, const QDate &dateNaissance)
        : m_nom(nom), m_age(age), m_dateNaissance(dateNaissance) {}

    QString nom() const { return m_nom; }
    int age() const { return m_age; }
    QDate dateNaissance() const { return m_dateNaissance; }

    void setNom(const QString &nom) { m_nom = nom; }
    void setAge(int age) { m_age = age; }
    void setDateNaissance(const QDate &date) { m_dateNaissance = date; }

    // Conversion en QJsonObject
    QJsonObject toJson() const
    {
        QJsonObject json;
        json["nom"] = m_nom;
        json["age"] = m_age;
        json["dateNaissance"] = m_dateNaissance.toString(Qt::ISODate);
        return json;
    }

    // Création depuis QJsonObject
    static Personne fromJson(const QJsonObject &json)
    {
        Personne personne;

        if (json.contains("nom") && json["nom"].isString())
            personne.m_nom = json["nom"].toString();

        if (json.contains("age") && json["age"].isDouble())
            personne.m_age = json["age"].toInt();

        if (json.contains("dateNaissance") && json["dateNaissance"].isString())
            personne.m_dateNaissance = QDate::fromString(json["dateNaissance"].toString(), Qt::ISODate);

        return personne;
    }

private:
    QString m_nom;
    int m_age;
    QDate m_dateNaissance;
};
```

Utilisation de cette classe avec le JSON :

```cpp
// Sauvegarde d'une liste de personnes en JSON
bool sauvegarderPersonnesJSON(const QList<Personne> &personnes, const QString &nomFichier)
{
    // Création d'un tableau JSON
    QJsonArray tableauPersonnes;

    // Conversion de chaque personne en objet JSON
    for (const Personne &personne : personnes) {
        tableauPersonnes.append(personne.toJson());
    }

    // Création du document JSON
    QJsonDocument document(tableauPersonnes);

    // Sauvegarde dans un fichier
    return sauvegarderJSON(document, nomFichier);
}

// Chargement d'une liste de personnes depuis un fichier JSON
QList<Personne> chargerPersonnesJSON(const QString &nomFichier)
{
    QList<Personne> personnes;

    // Chargement du document JSON
    QJsonDocument document = chargerJSON(nomFichier);

    // Vérification que le document contient un tableau
    if (!document.isArray()) {
        qDebug() << "Le document JSON n'est pas un tableau";
        return personnes;
    }

    // Récupération du tableau
    QJsonArray tableauPersonnes = document.array();

    // Conversion de chaque objet JSON en personne
    for (const QJsonValue &valeur : tableauPersonnes) {
        if (valeur.isObject()) {
            personnes.append(Personne::fromJson(valeur.toObject()));
        }
    }

    return personnes;
}
```

## Gestion des erreurs avec JSON

La gestion des erreurs est essentielle lors du traitement de données JSON :

```cpp
void gererErreursJSON(const QByteArray &donnees)
{
    QJsonParseError erreur;
    QJsonDocument document = QJsonDocument::fromJson(donnees, &erreur);

    if (erreur.error != QJsonParseError::NoError) {
        switch (erreur.error) {
            case QJsonParseError::UnterminatedObject:
                qDebug() << "Objet non terminé (accolade fermante manquante)";
                break;
            case QJsonParseError::MissingNameSeparator:
                qDebug() << "Séparateur de nom manquant (deux-points)";
                break;
            case QJsonParseError::UnterminatedArray:
                qDebug() << "Tableau non terminé (crochet fermant manquant)";
                break;
            case QJsonParseError::MissingValueSeparator:
                qDebug() << "Séparateur de valeur manquant (virgule)";
                break;
            case QJsonParseError::IllegalValue:
                qDebug() << "Valeur illégale";
                break;
            // Autres cas d'erreur
            default:
                qDebug() << "Erreur d'analyse JSON :" << erreur.errorString();
        }

        // Affichage du contexte de l'erreur
        qDebug() << "Position de l'erreur :" << erreur.offset;

        // Extraire une portion du texte autour de l'erreur pour le contexte
        int debut = qMax(0, erreur.offset - 10);
        int longueur = qMin(20, donnees.size() - debut);
        QByteArray contexte = donnees.mid(debut, longueur);

        qDebug() << "Contexte :" << contexte;
    }
}
```

## Exemple pratique : Configuration d'application en JSON

Le JSON est idéal pour stocker des configurations d'application. Voici un exemple complet :

```cpp
// gestionnaire_config.h
#ifndef GESTIONNAIRE_CONFIG_H
#define GESTIONNAIRE_CONFIG_H

#include <QObject>
#include <QString>
#include <QJsonObject>

class GestionnaireConfig : public QObject
{
    Q_OBJECT

public:
    explicit GestionnaireConfig(QObject *parent = nullptr);

    bool chargerConfiguration(const QString &nomFichier);
    bool sauvegarderConfiguration(const QString &nomFichier = QString());

    // Accesseurs pour les paramètres courants
    QString langue() const;
    bool modeNuit() const;
    int taillePolice() const;
    QString dossierDonnees() const;

    // Modificateurs pour les paramètres
    void setLangue(const QString &langue);
    void setModeNuit(bool actif);
    void setTaillePolice(int taille);
    void setDossierDonnees(const QString &dossier);

    // Réinitialisation aux valeurs par défaut
    void reinitialiser();

private:
    QString m_fichierConfig;  // Nom du fichier de configuration
    QJsonObject m_config;     // Configuration actuelle

    // Paramètres avec leurs valeurs par défaut
    void chargerValeursParDefaut();
};

#endif // GESTIONNAIRE_CONFIG_H
```

```cpp
// gestionnaire_config.cpp
#include "gestionnaire_config.h"
#include <QFile>
#include <QJsonDocument>
#include <QJsonParseError>
#include <QDebug>
#include <QDir>
#include <QStandardPaths>

GestionnaireConfig::GestionnaireConfig(QObject *parent)
    : QObject(parent)
{
    chargerValeursParDefaut();
}

void GestionnaireConfig::chargerValeursParDefaut()
{
    m_config = QJsonObject();

    // Définition des valeurs par défaut
    m_config["langue"] = "fr";  // Français par défaut
    m_config["modeNuit"] = false;
    m_config["taillePolice"] = 12;

    // Utilisation du dossier Documents comme emplacement par défaut
    QString dossierDefaut = QStandardPaths::writableLocation(QStandardPaths::DocumentsLocation);
    m_config["dossierDonnees"] = QDir(dossierDefaut).filePath("MonApplication");
}

bool GestionnaireConfig::chargerConfiguration(const QString &nomFichier)
{
    m_fichierConfig = nomFichier;

    QFile fichier(nomFichier);
    if (!fichier.exists()) {
        qDebug() << "Le fichier de configuration n'existe pas, utilisation des valeurs par défaut";
        return false;
    }

    if (!fichier.open(QIODevice::ReadOnly)) {
        qDebug() << "Impossible d'ouvrir le fichier de configuration :" << fichier.errorString();
        return false;
    }

    QByteArray donnees = fichier.readAll();
    fichier.close();

    QJsonParseError erreur;
    QJsonDocument document = QJsonDocument::fromJson(donnees, &erreur);

    if (erreur.error != QJsonParseError::NoError) {
        qDebug() << "Erreur d'analyse de la configuration :" << erreur.errorString();
        return false;
    }

    if (!document.isObject()) {
        qDebug() << "Le document de configuration n'est pas un objet JSON valide";
        return false;
    }

    QJsonObject nouvelleConfig = document.object();

    // Fusion avec les valeurs par défaut pour garantir que tous les paramètres existent
    for (auto it = nouvelleConfig.begin(); it != nouvelleConfig.end(); ++it) {
        m_config[it.key()] = it.value();
    }

    qDebug() << "Configuration chargée depuis" << nomFichier;
    return true;
}

bool GestionnaireConfig::sauvegarderConfiguration(const QString &nomFichier)
{
    QString fichierCible = nomFichier.isEmpty() ? m_fichierConfig : nomFichier;

    if (fichierCible.isEmpty()) {
        qDebug() << "Aucun nom de fichier spécifié pour la sauvegarde de la configuration";
        return false;
    }

    QFile fichier(fichierCible);
    if (!fichier.open(QIODevice::WriteOnly)) {
        qDebug() << "Impossible d'ouvrir le fichier de configuration en écriture :" << fichier.errorString();
        return false;
    }

    QJsonDocument document(m_config);
    fichier.write(document.toJson(QJsonDocument::Indented));
    fichier.close();

    qDebug() << "Configuration sauvegardée dans" << fichierCible;
    return true;
}

// Accesseurs
QString GestionnaireConfig::langue() const
{
    return m_config["langue"].toString();
}

bool GestionnaireConfig::modeNuit() const
{
    return m_config["modeNuit"].toBool();
}

int GestionnaireConfig::taillePolice() const
{
    return m_config["taillePolice"].toInt();
}

QString GestionnaireConfig::dossierDonnees() const
{
    return m_config["dossierDonnees"].toString();
}

// Modificateurs
void GestionnaireConfig::setLangue(const QString &langue)
{
    m_config["langue"] = langue;
}

void GestionnaireConfig::setModeNuit(bool actif)
{
    m_config["modeNuit"] = actif;
}

void GestionnaireConfig::setTaillePolice(int taille)
{
    m_config["taillePolice"] = taille;
}

void GestionnaireConfig::setDossierDonnees(const QString &dossier)
{
    m_config["dossierDonnees"] = dossier;
}

void GestionnaireConfig::reinitialiser()
{
    chargerValeursParDefaut();
}
```

## Communication avec des APIs REST

Le JSON est le format de prédilection pour communiquer avec des APIs REST. Voici un exemple simple :

```cpp
#include <QNetworkAccessManager>
#include <QNetworkRequest>
#include <QNetworkReply>
#include <QJsonDocument>
#include <QJsonObject>
#include <QJsonArray>
#include <QUrl>
#include <QDebug>

class ClientAPI : public QObject
{
    Q_OBJECT

public:
    ClientAPI(QObject *parent = nullptr) : QObject(parent)
    {
        m_manager = new QNetworkAccessManager(this);

        // Connexion du signal de fin de requête
        connect(m_manager, &QNetworkAccessManager::finished,
                this, &ClientAPI::onReplyFinished);
    }

    // Exemple de requête GET
    void obtenirDonnees(const QString &endpoint)
    {
        QUrl url(QString("https://api.exemple.com/%1").arg(endpoint));
        QNetworkRequest requete(url);

        // Définition des en-têtes
        requete.setHeader(QNetworkRequest::ContentTypeHeader, "application/json");

        // Envoi de la requête GET
        m_manager->get(requete);
    }

    // Exemple de requête POST avec données JSON
    void envoyerDonnees(const QString &endpoint, const QJsonObject &donnees)
    {
        QUrl url(QString("https://api.exemple.com/%1").arg(endpoint));
        QNetworkRequest requete(url);

        // Définition des en-têtes
        requete.setHeader(QNetworkRequest::ContentTypeHeader, "application/json");

        // Conversion des données en JSON
        QJsonDocument document(donnees);
        QByteArray donneesJson = document.toJson();

        // Envoi de la requête POST
        m_manager->post(requete, donneesJson);
    }

private slots:
    // Traitement de la réponse
    void onReplyFinished(QNetworkReply *reply)
    {
        // Vérification des erreurs
        if (reply->error() != QNetworkReply::NoError) {
            qDebug() << "Erreur réseau :" << reply->errorString();
            reply->deleteLater();
            return;
        }

        // Lecture des données reçues
        QByteArray donneesReponse = reply->readAll();

        // Analyse du JSON
        QJsonParseError erreur;
        QJsonDocument document = QJsonDocument::fromJson(donneesReponse, &erreur);

        if (erreur.error != QJsonParseError::NoError) {
            qDebug() << "Erreur d'analyse JSON :" << erreur.errorString();
            reply->deleteLater();
            return;
        }

        // Traitement du document JSON
        if (document.isObject()) {
            QJsonObject objet = document.object();
            qDebug() << "Réponse JSON (objet) :" << objet;

            // Émission d'un signal avec les données reçues
            emit donneesRecues(objet);
        }
        else if (document.isArray()) {
            QJsonArray tableau = document.array();
            qDebug() << "Réponse JSON (tableau) de" << tableau.size() << "éléments";

            // Émission d'un signal avec les données reçues
            emit listeRecue(tableau);
        }

        // Nettoyage
        reply->deleteLater();
    }

signals:
    void donneesRecues(const QJsonObject &donnees);
    void listeRecue(const QJsonArray &liste);

private:
    QNetworkAccessManager *m_manager;
};
```

## Comparaison entre JSON et autres formats

| Caractéristique | JSON | QDataStream | XML | Base de données |
|-----------------|------|------------|-----|----------------|
| Format | Texte | Binaire | Texte | Variable |
| Lisibilité | Excellente | Aucune | Bonne | Variable |
| Taille | Moyenne | Très compacte | Verbeux | Variable |
| Performance | Moyenne | Excellente | Faible | Excellente |
| Compatibilité externe | Excellente | Limitée à Qt | Bonne | Variable |
| Schéma de validation | Avec JSON Schema | Non | Avec XSD | Oui |
| Cas d'utilisation | APIs, config, données simples | Sauvegarde interne | Documents structurés | Données complexes |

## Bonnes pratiques pour le JSON dans Qt

1. **Validation** : Toujours vérifier que le JSON est valide avant de l'utiliser
2. **Sécurité** : Valider les données entrantes pour éviter les injections
3. **Gestion d'erreurs** : Utiliser QJsonParseError pour des messages d'erreur précis
4. **Documentation** : Documenter la structure de vos objets JSON
5. **Performances** : Pour de grands documents, éviter de charger tout en mémoire
6. **Compatibilité** : Prévoir la rétrocompatibilité en cas de modification du format

## Conclusion

QJsonDocument et les classes associées offrent une solution flexible et interopérable pour la persistance des données. Le JSON est particulièrement adapté pour :

- Les fichiers de configuration
- La communication avec des APIs web
- L'échange de données entre applications hétérogènes
- Le stockage de données dont la structure peut évoluer

Dans la prochaine section, nous explorerons la sérialisation XML avec QXmlStreamWriter/Reader, qui offre une alternative plus structurée pour les données hiérarchiques complexes.

⏭️ [Sérialisation XML avec QXmlStreamWriter/Reader](/04-persistance-des-donnees/04-serialisation-xml-avec-qxmlstreamwriter-reader.md)
