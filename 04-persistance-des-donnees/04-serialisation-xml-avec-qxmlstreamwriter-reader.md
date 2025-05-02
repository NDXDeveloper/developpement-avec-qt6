# 4.4 Sérialisation XML avec QXmlStreamWriter/Reader

## Introduction au format XML

XML (eXtensible Markup Language) est un format de données textuelles structuré par des balises qui permet de représenter des informations hiérarchiques complexes. C'est un format largement utilisé pour l'échange de données, les fichiers de configuration et le stockage de documents structurés.

## Avantages du XML

Le XML présente plusieurs avantages spécifiques :

- **Structure hiérarchique** : Parfait pour représenter des données imbriquées
- **Validation stricte** : Possibilité de définir des schémas (DTD, XSD) pour valider la structure
- **Extensibilité** : Facile à étendre sans casser la compatibilité
- **Écosystème riche** : Nombreux outils et technologies associés (XPath, XSLT, etc.)
- **Lisibilité** : Format texte lisible par les humains
- **Standard établi** : Largement adopté dans de nombreux domaines

## Structure du XML

Un document XML est structuré selon ces règles de base :

- Le document commence généralement par une déclaration XML (`<?xml version="1.0" encoding="UTF-8"?>`)
- Les données sont organisées en éléments délimités par des balises (`<element>valeur</element>`)
- Les éléments peuvent avoir des attributs (`<element attribut="valeur">`)
- Les éléments peuvent être imbriqués pour créer une hiérarchie
- Tout document XML bien formé doit avoir un seul élément racine

Exemple de document XML :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<personne id="1">
    <nom>Alice Dupont</nom>
    <age>28</age>
    <estEtudiant>false</estEtudiant>
    <adresse>
        <rue>123 Avenue des Champs</rue>
        <ville>Paris</ville>
        <codePostal>75008</codePostal>
    </adresse>
    <competences>
        <competence>C++</competence>
        <competence>Qt</competence>
        <competence>JavaScript</competence>
        <competence>SQL</competence>
    </competences>
</personne>
```

## Les classes XML dans Qt

Qt propose plusieurs approches pour manipuler le XML :

1. **QXmlStreamWriter/Reader** : API moderne pour lire et écrire du XML efficacement (recommandée)
2. **QDomDocument** : API pour manipuler le DOM (Document Object Model) d'un document XML
3. **QXmlSimpleReader** : API SAX (Simple API for XML) pour traiter le XML de manière événementielle

Dans ce tutoriel, nous nous concentrerons sur QXmlStreamWriter et QXmlStreamReader, qui offrent le meilleur équilibre entre facilité d'utilisation et performances.

## Écriture de XML avec QXmlStreamWriter

QXmlStreamWriter permet de générer des documents XML de manière simple et efficace :

```cpp
#include <QFile>
#include <QXmlStreamWriter>
#include <QDebug>

bool ecrireDocumentXML()
{
    // Ouverture du fichier en écriture
    QFile fichier("personne.xml");
    if (!fichier.open(QIODevice::WriteOnly | QIODevice::Text)) {
        qDebug() << "Impossible d'ouvrir le fichier en écriture :"
                 << fichier.errorString();
        return false;
    }

    // Création du writer XML
    QXmlStreamWriter writer(&fichier);

    // Configuration du writer pour une mise en forme lisible
    writer.setAutoFormatting(true);
    writer.setAutoFormattingIndent(4);

    // Début du document
    writer.writeStartDocument();

    // Élément racine
    writer.writeStartElement("personne");
    writer.writeAttribute("id", "1");

    // Éléments simples
    writer.writeTextElement("nom", "Alice Dupont");
    writer.writeTextElement("age", "28");
    writer.writeTextElement("estEtudiant", "false");

    // Élément complexe : adresse
    writer.writeStartElement("adresse");
    writer.writeTextElement("rue", "123 Avenue des Champs");
    writer.writeTextElement("ville", "Paris");
    writer.writeTextElement("codePostal", "75008");
    writer.writeEndElement(); // Fin de l'adresse

    // Élément avec répétition : compétences
    writer.writeStartElement("competences");

    QStringList competences = {"C++", "Qt", "JavaScript", "SQL"};
    for (const QString &comp : competences) {
        writer.writeTextElement("competence", comp);
    }

    writer.writeEndElement(); // Fin des compétences

    // Fin de l'élément racine
    writer.writeEndElement();

    // Fin du document
    writer.writeEndDocument();

    return true;
}
```

## Lecture de XML avec QXmlStreamReader

QXmlStreamReader permet de lire et d'analyser du XML efficacement :

```cpp
#include <QFile>
#include <QXmlStreamReader>
#include <QDebug>

void lireDocumentXML()
{
    // Ouverture du fichier en lecture
    QFile fichier("personne.xml");
    if (!fichier.open(QIODevice::ReadOnly | QIODevice::Text)) {
        qDebug() << "Impossible d'ouvrir le fichier en lecture :"
                 << fichier.errorString();
        return;
    }

    // Création du reader XML
    QXmlStreamReader reader(&fichier);

    // Variables pour stocker les données lues
    QString idPersonne;
    QString nom;
    int age = 0;
    bool estEtudiant = false;
    QString rue, ville, codePostal;
    QStringList competences;

    // Élément courant pour le parsing
    QString elementCourant;

    // Analyse du document XML
    while (!reader.atEnd() && !reader.hasError()) {
        QXmlStreamReader::TokenType token = reader.readNext();

        // Au début d'un élément
        if (token == QXmlStreamReader::StartElement) {
            elementCourant = reader.name().toString();

            // Traitement selon l'élément
            if (elementCourant == "personne") {
                // Lecture de l'attribut id
                idPersonne = reader.attributes().value("id").toString();
            }
            else if (elementCourant == "nom" ||
                     elementCourant == "age" ||
                     elementCourant == "estEtudiant" ||
                     elementCourant == "rue" ||
                     elementCourant == "ville" ||
                     elementCourant == "codePostal" ||
                     elementCourant == "competence") {
                // Ces éléments sont traités lors de la lecture du texte
            }
            // Pas besoin de traiter spécifiquement les éléments adresse ou competences
            // car nous traiterons leurs enfants directement
        }
        // Au niveau du texte d'un élément
        else if (token == QXmlStreamReader::Characters && !reader.isWhitespace()) {
            // Stockage des données selon l'élément courant
            if (elementCourant == "nom") {
                nom = reader.text().toString();
            }
            else if (elementCourant == "age") {
                age = reader.text().toInt();
            }
            else if (elementCourant == "estEtudiant") {
                estEtudiant = (reader.text().toString().toLower() == "true");
            }
            else if (elementCourant == "rue") {
                rue = reader.text().toString();
            }
            else if (elementCourant == "ville") {
                ville = reader.text().toString();
            }
            else if (elementCourant == "codePostal") {
                codePostal = reader.text().toString();
            }
            else if (elementCourant == "competence") {
                competences.append(reader.text().toString());
            }
        }
    }

    // Vérification des erreurs
    if (reader.hasError()) {
        qDebug() << "Erreur lors de la lecture du XML :"
                 << reader.errorString();
        return;
    }

    // Affichage des données lues
    qDebug() << "Personne ID :" << idPersonne;
    qDebug() << "Nom :" << nom;
    qDebug() << "Âge :" << age;
    qDebug() << "Est étudiant :" << estEtudiant;
    qDebug() << "Adresse :";
    qDebug() << "  Rue :" << rue;
    qDebug() << "  Ville :" << ville;
    qDebug() << "  Code postal :" << codePostal;
    qDebug() << "Compétences :" << competences.join(", ");
}
```

## Approche alternative : Lecture basée sur les événements

Une approche plus adaptée pour les documents XML complexes consiste à utiliser un traitement basé sur les événements :

```cpp
void lireDocumentXMLParEvenement()
{
    QFile fichier("personne.xml");
    if (!fichier.open(QIODevice::ReadOnly | QIODevice::Text)) {
        qDebug() << "Impossible d'ouvrir le fichier";
        return;
    }

    QXmlStreamReader reader(&fichier);

    // Variables pour suivre l'état du parsing
    QString elementCourant;
    bool dansAdresse = false;
    bool dansCompetences = false;

    // Variables pour stocker les données
    QString idPersonne;
    QString nom;
    int age = 0;
    bool estEtudiant = false;
    QString rue, ville, codePostal;
    QStringList competences;

    while (!reader.atEnd() && !reader.hasError()) {
        QXmlStreamReader::TokenType token = reader.readNext();

        switch (token) {
            case QXmlStreamReader::StartElement:
                elementCourant = reader.name().toString();

                if (elementCourant == "personne") {
                    idPersonne = reader.attributes().value("id").toString();
                }
                else if (elementCourant == "adresse") {
                    dansAdresse = true;
                }
                else if (elementCourant == "competences") {
                    dansCompetences = true;
                }
                break;

            case QXmlStreamReader::Characters:
                if (!reader.isWhitespace()) {
                    if (elementCourant == "nom") {
                        nom = reader.text().toString();
                    }
                    else if (elementCourant == "age") {
                        age = reader.text().toInt();
                    }
                    else if (elementCourant == "estEtudiant") {
                        estEtudiant = (reader.text().toString().toLower() == "true");
                    }
                    else if (dansAdresse) {
                        if (elementCourant == "rue") {
                            rue = reader.text().toString();
                        }
                        else if (elementCourant == "ville") {
                            ville = reader.text().toString();
                        }
                        else if (elementCourant == "codePostal") {
                            codePostal = reader.text().toString();
                        }
                    }
                    else if (dansCompetences && elementCourant == "competence") {
                        competences.append(reader.text().toString());
                    }
                }
                break;

            case QXmlStreamReader::EndElement:
                elementCourant = reader.name().toString();

                if (elementCourant == "adresse") {
                    dansAdresse = false;
                }
                else if (elementCourant == "competences") {
                    dansCompetences = false;
                }
                break;

            default:
                break;
        }
    }

    // Affichage des résultats (comme précédemment)
}
```

## Sérialisation de classes personnalisées en XML

Comme pour JSON, Qt n'offre pas de mécanisme automatique pour sérialiser des classes en XML. Vous devez implémenter vos propres méthodes :

```cpp
// Définition de la classe
class Personne
{
public:
    Personne() : m_age(0), m_estEtudiant(false) {}
    Personne(const QString &nom, int age, bool estEtudiant)
        : m_nom(nom), m_age(age), m_estEtudiant(estEtudiant) {}

    // Getters
    QString nom() const { return m_nom; }
    int age() const { return m_age; }
    bool estEtudiant() const { return m_estEtudiant; }
    QString rue() const { return m_rue; }
    QString ville() const { return m_ville; }
    QString codePostal() const { return m_codePostal; }
    QStringList competences() const { return m_competences; }

    // Setters
    void setNom(const QString &nom) { m_nom = nom; }
    void setAge(int age) { m_age = age; }
    void setEstEtudiant(bool estEtudiant) { m_estEtudiant = estEtudiant; }
    void setRue(const QString &rue) { m_rue = rue; }
    void setVille(const QString &ville) { m_ville = ville; }
    void setCodePostal(const QString &codePostal) { m_codePostal = codePostal; }
    void setCompetences(const QStringList &competences) { m_competences = competences; }
    void ajouterCompetence(const QString &competence) { m_competences.append(competence); }

    // Méthode pour écrire l'objet en XML
    bool enregistrerXML(const QString &nomFichier) const
    {
        QFile fichier(nomFichier);
        if (!fichier.open(QIODevice::WriteOnly | QIODevice::Text)) {
            return false;
        }

        QXmlStreamWriter writer(&fichier);
        writer.setAutoFormatting(true);
        writer.setAutoFormattingIndent(4);

        writer.writeStartDocument();
        ecrireXML(writer);
        writer.writeEndDocument();

        return true;
    }

    // Méthode pour charger l'objet depuis XML
    bool chargerXML(const QString &nomFichier)
    {
        QFile fichier(nomFichier);
        if (!fichier.open(QIODevice::ReadOnly | QIODevice::Text)) {
            return false;
        }

        QXmlStreamReader reader(&fichier);

        while (!reader.atEnd() && !reader.hasError()) {
            if (reader.readNext() == QXmlStreamReader::StartElement &&
                reader.name() == "personne") {
                lireXML(reader);
                break;
            }
        }

        return !reader.hasError();
    }

private:
    // Méthode auxiliaire pour écrire en XML
    void ecrireXML(QXmlStreamWriter &writer) const
    {
        writer.writeStartElement("personne");

        writer.writeTextElement("nom", m_nom);
        writer.writeTextElement("age", QString::number(m_age));
        writer.writeTextElement("estEtudiant", m_estEtudiant ? "true" : "false");

        writer.writeStartElement("adresse");
        writer.writeTextElement("rue", m_rue);
        writer.writeTextElement("ville", m_ville);
        writer.writeTextElement("codePostal", m_codePostal);
        writer.writeEndElement(); // adresse

        writer.writeStartElement("competences");
        for (const QString &comp : m_competences) {
            writer.writeTextElement("competence", comp);
        }
        writer.writeEndElement(); // competences

        writer.writeEndElement(); // personne
    }

    // Méthode auxiliaire pour lire depuis XML
    void lireXML(QXmlStreamReader &reader)
    {
        QString elementCourant;
        bool dansAdresse = false;
        bool dansCompetences = false;

        m_competences.clear();

        while (!reader.atEnd() && !reader.hasError()) {
            QXmlStreamReader::TokenType token = reader.readNext();

            if (token == QXmlStreamReader::StartElement) {
                elementCourant = reader.name().toString();

                if (elementCourant == "adresse") {
                    dansAdresse = true;
                }
                else if (elementCourant == "competences") {
                    dansCompetences = true;
                }
            }
            else if (token == QXmlStreamReader::Characters && !reader.isWhitespace()) {
                if (elementCourant == "nom") {
                    m_nom = reader.text().toString();
                }
                else if (elementCourant == "age") {
                    m_age = reader.text().toInt();
                }
                else if (elementCourant == "estEtudiant") {
                    m_estEtudiant = (reader.text().toString().toLower() == "true");
                }
                else if (dansAdresse) {
                    if (elementCourant == "rue") {
                        m_rue = reader.text().toString();
                    }
                    else if (elementCourant == "ville") {
                        m_ville = reader.text().toString();
                    }
                    else if (elementCourant == "codePostal") {
                        m_codePostal = reader.text().toString();
                    }
                }
                else if (dansCompetences && elementCourant == "competence") {
                    m_competences.append(reader.text().toString());
                }
            }
            else if (token == QXmlStreamReader::EndElement) {
                elementCourant = reader.name().toString();

                if (elementCourant == "personne") {
                    // Fin de l'élément personne, on quitte la boucle
                    break;
                }
                else if (elementCourant == "adresse") {
                    dansAdresse = false;
                }
                else if (elementCourant == "competences") {
                    dansCompetences = false;
                }
            }
        }
    }

    QString m_nom;
    int m_age;
    bool m_estEtudiant;
    QString m_rue;
    QString m_ville;
    QString m_codePostal;
    QStringList m_competences;
};
```

## Gestion des erreurs avec QXmlStreamReader

La gestion des erreurs est importante lors de l'analyse de XML :

```cpp
void verifierErreursXML(QXmlStreamReader &reader)
{
    if (reader.hasError()) {
        qDebug() << "Erreur de parsing XML :";

        switch (reader.error()) {
            case QXmlStreamReader::NotWellFormedError:
                qDebug() << "Le document XML n'est pas bien formé";
                break;
            case QXmlStreamReader::PrematureEndOfDocumentError:
                qDebug() << "Fin prématurée du document";
                break;
            case QXmlStreamReader::UnexpectedElementError:
                qDebug() << "Élément inattendu";
                break;
            default:
                qDebug() << "Erreur" << reader.errorString();
        }

        qDebug() << "Position :" << reader.lineNumber() << ":" << reader.columnNumber();
    }
}
```

## Exemple pratique : Configuration d'application en XML

Le XML est souvent utilisé pour les fichiers de configuration d'application. Voici un exemple :

```cpp
class ConfigurationXML : public QObject
{
    Q_OBJECT

public:
    explicit ConfigurationXML(QObject *parent = nullptr)
        : QObject(parent), m_modeNuit(false), m_taillePolice(12) {}

    bool charger(const QString &fichier)
    {
        QFile file(fichier);
        if (!file.open(QIODevice::ReadOnly | QIODevice::Text)) {
            qDebug() << "Impossible d'ouvrir le fichier de configuration";
            return false;
        }

        QXmlStreamReader reader(&file);

        while (!reader.atEnd() && !reader.hasError()) {
            QXmlStreamReader::TokenType token = reader.readNext();

            if (token == QXmlStreamReader::StartElement) {
                if (reader.name() == "configuration") {
                    lireConfiguration(reader);
                }
            }
        }

        if (reader.hasError()) {
            qDebug() << "Erreur lors de la lecture du XML :" << reader.errorString();
            return false;
        }

        return true;
    }

    bool sauvegarder(const QString &fichier)
    {
        QFile file(fichier);
        if (!file.open(QIODevice::WriteOnly | QIODevice::Text)) {
            qDebug() << "Impossible d'ouvrir le fichier en écriture";
            return false;
        }

        QXmlStreamWriter writer(&file);
        writer.setAutoFormatting(true);

        writer.writeStartDocument();
        writer.writeStartElement("configuration");

        writer.writeStartElement("apparence");
        writer.writeTextElement("modeNuit", m_modeNuit ? "true" : "false");
        writer.writeTextElement("taillePolice", QString::number(m_taillePolice));
        writer.writeEndElement(); // apparence

        writer.writeStartElement("langue");
        writer.writeTextElement("code", m_codeLangue);
        writer.writeEndElement(); // langue

        writer.writeStartElement("chemins");
        writer.writeTextElement("dossierProjets", m_dossierProjets);
        writer.writeTextElement("dossierExport", m_dossierExport);
        writer.writeEndElement(); // chemins

        writer.writeStartElement("recents");
        for (const QString &fichier : m_fichiersRecents) {
            writer.writeTextElement("fichier", fichier);
        }
        writer.writeEndElement(); // recents

        writer.writeEndElement(); // configuration
        writer.writeEndDocument();

        return true;
    }

    // Accesseurs et mutateurs pour les propriétés
    bool modeNuit() const { return m_modeNuit; }
    void setModeNuit(bool mode) { m_modeNuit = mode; }

    int taillePolice() const { return m_taillePolice; }
    void setTaillePolice(int taille) { m_taillePolice = taille; }

    QString codeLangue() const { return m_codeLangue; }
    void setCodeLangue(const QString &code) { m_codeLangue = code; }

    QString dossierProjets() const { return m_dossierProjets; }
    void setDossierProjets(const QString &dossier) { m_dossierProjets = dossier; }

    QString dossierExport() const { return m_dossierExport; }
    void setDossierExport(const QString &dossier) { m_dossierExport = dossier; }

    QStringList fichiersRecents() const { return m_fichiersRecents; }
    void setFichiersRecents(const QStringList &fichiers) { m_fichiersRecents = fichiers; }

private:
    void lireConfiguration(QXmlStreamReader &reader)
    {
        while (!reader.atEnd()) {
            if (reader.readNext() == QXmlStreamReader::EndElement &&
                reader.name() == "configuration") {
                break;
            }

            if (reader.tokenType() == QXmlStreamReader::StartElement) {
                if (reader.name() == "apparence") {
                    lireApparence(reader);
                }
                else if (reader.name() == "langue") {
                    lireLangue(reader);
                }
                else if (reader.name() == "chemins") {
                    lireChemins(reader);
                }
                else if (reader.name() == "recents") {
                    lireRecents(reader);
                }
            }
        }
    }

    void lireApparence(QXmlStreamReader &reader)
    {
        while (!reader.atEnd()) {
            if (reader.readNext() == QXmlStreamReader::EndElement &&
                reader.name() == "apparence") {
                break;
            }

            if (reader.tokenType() == QXmlStreamReader::StartElement) {
                if (reader.name() == "modeNuit") {
                    reader.readNext();
                    if (reader.tokenType() == QXmlStreamReader::Characters) {
                        m_modeNuit = (reader.text().toString().toLower() == "true");
                    }
                }
                else if (reader.name() == "taillePolice") {
                    reader.readNext();
                    if (reader.tokenType() == QXmlStreamReader::Characters) {
                        m_taillePolice = reader.text().toInt();
                    }
                }
            }
        }
    }

    void lireLangue(QXmlStreamReader &reader)
    {
        while (!reader.atEnd()) {
            if (reader.readNext() == QXmlStreamReader::EndElement &&
                reader.name() == "langue") {
                break;
            }

            if (reader.tokenType() == QXmlStreamReader::StartElement) {
                if (reader.name() == "code") {
                    reader.readNext();
                    if (reader.tokenType() == QXmlStreamReader::Characters) {
                        m_codeLangue = reader.text().toString();
                    }
                }
            }
        }
    }

    void lireChemins(QXmlStreamReader &reader)
    {
        while (!reader.atEnd()) {
            if (reader.readNext() == QXmlStreamReader::EndElement &&
                reader.name() == "chemins") {
                break;
            }

            if (reader.tokenType() == QXmlStreamReader::StartElement) {
                if (reader.name() == "dossierProjets") {
                    reader.readNext();
                    if (reader.tokenType() == QXmlStreamReader::Characters) {
                        m_dossierProjets = reader.text().toString();
                    }
                }
                else if (reader.name() == "dossierExport") {
                    reader.readNext();
                    if (reader.tokenType() == QXmlStreamReader::Characters) {
                        m_dossierExport = reader.text().toString();
                    }
                }
            }
        }
    }

    void lireRecents(QXmlStreamReader &reader)
    {
        m_fichiersRecents.clear();

        while (!reader.atEnd()) {
            if (reader.readNext() == QXmlStreamReader::EndElement &&
                reader.name() == "recents") {
                break;
            }

            if (reader.tokenType() == QXmlStreamReader::StartElement) {
                if (reader.name() == "fichier") {
                    reader.readNext();
                    if (reader.tokenType() == QXmlStreamReader::Characters) {
                        m_fichiersRecents.append(reader.text().toString());
                    }
                }
            }
        }
    }

    // Propriétés de configuration
    bool m_modeNuit;
    int m_taillePolice;
    QString m_codeLangue;
    QString m_dossierProjets;
    QString m_dossierExport;
    QStringList m_fichiersRecents;
};
```

## Comparaison XML vs JSON

| Caractéristique | XML | JSON |
|-----------------|-----|------|
| Lisibilité | Bonne mais plus verbeuse | Excellente, plus concise |
| Structure | Hiérarchique, avec attributs | Hiérarchique, sans attributs |
| Validation | Schémas XSD puissants | Schémas JSON plus simples |
| Taille | Plus volumineux | Plus compact |
| Complexité | Plus complexe à manipuler | Plus simple à manipuler |
| Cas d'utilisation | Documents complexes, données fortement structurées | Configuration, API, données simples |

## Quand choisir XML plutôt que JSON ?

XML est préférable dans ces situations :

1. **Documents complexes** avec une structure hiérarchique profonde
2. Besoin de **validation stricte** avec des schémas XSD
3. Utilisation d'**attributs** pour qualifier les données
4. **Compatibilité** avec des systèmes existants qui utilisent XML
5. Besoin d'utiliser des technologies comme **XSLT** pour la transformation

## Bonnes pratiques pour le XML dans Qt

1. **Utiliser QXmlStreamWriter/Reader** plutôt que QDomDocument pour les performances
2. **Activer l'auto-formatage** pour une meilleure lisibilité du XML généré
3. **Vérifier systématiquement les erreurs** lors de la lecture de XML
4. **Structurer le code** en méthodes spécifiques pour chaque section du XML
5. **Gérer les espaces de noms** correctement si utilisés
6. **Valider le XML** si possible, surtout pour les formats critiques

## Conclusion

QXmlStreamWriter et QXmlStreamReader offrent une solution efficace et flexible pour la sérialisation de données complexes au format XML. Ce format est particulièrement adapté pour :

- Les fichiers de configuration d'application
- Les formats de documents structurés
- L'échange de données avec des systèmes hérités
- La représentation de données hiérarchiques complexes

Dans la prochaine section, nous explorerons une méthode plus simple pour stocker les paramètres d'application : QSettings.
