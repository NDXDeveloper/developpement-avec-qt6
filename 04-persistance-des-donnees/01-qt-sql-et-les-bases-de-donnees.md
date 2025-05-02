# 4.1 Qt SQL et les bases de données

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

## Introduction aux bases de données dans Qt

Les bases de données sont essentielles pour stocker et gérer des données structurées dans vos applications. Qt6 offre un module dédié, **Qt SQL**, qui fournit une interface unifiée pour travailler avec différents systèmes de bases de données relationnelles.

## Avantages des bases de données relationnelles

Avant de plonger dans le code, comprenons pourquoi utiliser une base de données relationnelle :

- **Organisation structurée** : Les données sont organisées en tables avec des relations entre elles
- **Requêtes puissantes** : Possibilité de filtrer, trier et combiner des données facilement
- **Intégrité des données** : Contraintes pour garantir la cohérence des données
- **Performance** : Optimisé pour gérer de grandes quantités de données
- **Accès concurrent** : Plusieurs utilisateurs peuvent accéder aux données simultanément

## Les systèmes de bases de données supportés par Qt SQL

Qt SQL prend en charge plusieurs moteurs de bases de données :

| Moteur de BD | Description | Driver Qt | Particularités |
|--------------|-------------|-----------|----------------|
| SQLite | BD légère, fichier unique | QSQLITE | Parfait pour les applications autonomes |
| MySQL/MariaDB | BD client-serveur populaire | QMYSQL | Nécessite l'installation du client MySQL |
| PostgreSQL | BD client-serveur avancée | QPSQL | Robuste et riche en fonctionnalités |
| Oracle | BD d'entreprise | QOCI | Pour applications professionnelles |
| Microsoft SQL Server | BD d'entreprise | QTDS | Environnements Windows |

Pour les débutants, **SQLite** est recommandé car il ne nécessite pas de serveur distinct et fonctionne avec un simple fichier.

## Concepts clés de Qt SQL

Qt SQL s'articule autour de plusieurs classes essentielles :

- **QSqlDatabase** : Représente une connexion à une base de données
- **QSqlQuery** : Permet d'exécuter des requêtes SQL
- **QSqlTableModel** : Modèle pour lier une table à une vue (approche MVC)
- **QSqlRelationalTableModel** : Extension de QSqlTableModel pour gérer les relations
- **QSqlRecord** : Représente un enregistrement (ligne) d'une table
- **QSqlError** : Encapsule les erreurs de base de données

## Connexion à une base de données

Voici comment établir une connexion à une base de données SQLite :

```cpp
#include <QSqlDatabase>
#include <QSqlError>
#include <QDebug>

bool connecterBaseDeDonnees()
{
    // Crée une connexion SQLite
    QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE");

    // Spécifie le chemin du fichier de base de données
    db.setDatabaseName("mon_application.db");

    // Ouvre la connexion
    if (!db.open()) {
        qDebug() << "Erreur de connexion :" << db.lastError().text();
        return false;
    }

    qDebug() << "Connexion réussie à la base de données";
    return true;
}
```

## Création d'une table

Pour créer une table dans votre base de données :

```cpp
bool creerTableUtilisateurs()
{
    QSqlQuery query;

    // Création d'une table utilisateurs
    bool success = query.exec(
        "CREATE TABLE IF NOT EXISTS utilisateurs ("
        "id INTEGER PRIMARY KEY AUTOINCREMENT, "
        "nom VARCHAR(50) NOT NULL, "
        "email VARCHAR(100) UNIQUE, "
        "date_inscription DATE"
        ")"
    );

    if (!success) {
        qDebug() << "Erreur lors de la création de la table :" << query.lastError().text();
        return false;
    }

    return true;
}
```

## Insertion de données

Pour ajouter des enregistrements dans votre table :

```cpp
bool ajouterUtilisateur(const QString &nom, const QString &email)
{
    QSqlQuery query;

    // Préparation de la requête avec des paramètres
    query.prepare(
        "INSERT INTO utilisateurs (nom, email, date_inscription) "
        "VALUES (:nom, :email, DATE('now'))"
    );

    // Liaison des valeurs aux paramètres
    query.bindValue(":nom", nom);
    query.bindValue(":email", email);

    // Exécution de la requête
    if (!query.exec()) {
        qDebug() << "Erreur lors de l'ajout d'un utilisateur :" << query.lastError().text();
        return false;
    }

    qDebug() << "Utilisateur ajouté avec l'ID :" << query.lastInsertId().toInt();
    return true;
}
```

## Récupération de données

Pour récupérer des données depuis votre base de données :

```cpp
void afficherUtilisateurs()
{
    QSqlQuery query("SELECT id, nom, email, date_inscription FROM utilisateurs");

    while (query.next()) {
        int id = query.value(0).toInt();
        QString nom = query.value(1).toString();
        QString email = query.value(2).toString();
        QDate date = query.value(3).toDate();

        qDebug() << "ID:" << id << "Nom:" << nom
                 << "Email:" << email << "Date:" << date.toString("dd/MM/yyyy");
    }
}
```

## Mise à jour et suppression

Pour modifier ou supprimer des données :

```cpp
// Mise à jour d'un utilisateur
bool mettreAJourEmail(int id, const QString &nouvelEmail)
{
    QSqlQuery query;
    query.prepare("UPDATE utilisateurs SET email = :email WHERE id = :id");
    query.bindValue(":email", nouvelEmail);
    query.bindValue(":id", id);

    return query.exec();
}

// Suppression d'un utilisateur
bool supprimerUtilisateur(int id)
{
    QSqlQuery query;
    query.prepare("DELETE FROM utilisateurs WHERE id = :id");
    query.bindValue(":id", id);

    return query.exec();
}
```

## Utilisation des modèles SQL

Qt offre une approche MVC (Modèle-Vue-Contrôleur) pour faciliter l'affichage des données de base de données dans les interfaces utilisateur. Voici un exemple simple avec QTableView :

```cpp
#include <QTableView>
#include <QSqlTableModel>

void afficherTableUtilisateurs(QWidget *parent)
{
    // Création du modèle
    QSqlTableModel *model = new QSqlTableModel(parent);
    model->setTable("utilisateurs");
    model->setEditStrategy(QSqlTableModel::OnManualSubmit); // Les modifications sont appliquées manuellement
    model->select(); // Charge les données

    // Définit les en-têtes de colonnes
    model->setHeaderData(0, Qt::Horizontal, tr("ID"));
    model->setHeaderData(1, Qt::Horizontal, tr("Nom"));
    model->setHeaderData(2, Qt::Horizontal, tr("Email"));
    model->setHeaderData(3, Qt::Horizontal, tr("Date d'inscription"));

    // Création de la vue
    QTableView *view = new QTableView(parent);
    view->setModel(model);
    view->hideColumn(0); // Cache la colonne ID
    view->show();
}
```

## Transactions

Les transactions sont essentielles pour garantir l'intégrité des données lors de modifications multiples :

```cpp
bool transfererFonds(int compteSource, int compteDestination, double montant)
{
    QSqlDatabase db = QSqlDatabase::database();

    // Démarre une transaction
    db.transaction();

    QSqlQuery query;

    // Retire l'argent du compte source
    query.prepare("UPDATE comptes SET solde = solde - :montant WHERE id = :id");
    query.bindValue(":montant", montant);
    query.bindValue(":id", compteSource);
    if (!query.exec()) {
        db.rollback(); // Annule la transaction en cas d'erreur
        return false;
    }

    // Ajoute l'argent au compte destination
    query.prepare("UPDATE comptes SET solde = solde + :montant WHERE id = :id");
    query.bindValue(":montant", montant);
    query.bindValue(":id", compteDestination);
    if (!query.exec()) {
        db.rollback(); // Annule la transaction en cas d'erreur
        return false;
    }

    // Valide la transaction
    return db.commit();
}
```

## Gestion des erreurs

La gestion appropriée des erreurs est cruciale lors de l'utilisation des bases de données :

```cpp
void gererErreur(const QSqlQuery &query)
{
    QSqlError erreur = query.lastError();

    qDebug() << "Type d'erreur :" << erreur.type();
    qDebug() << "Code d'erreur du pilote :" << erreur.nativeErrorCode();
    qDebug() << "Message d'erreur :" << erreur.text();

    // Vous pouvez ajouter un traitement spécifique selon le type d'erreur
    switch (erreur.type()) {
    case QSqlError::NoError:
        break;
    case QSqlError::ConnectionError:
        qDebug() << "Problème de connexion à la base de données";
        break;
    case QSqlError::StatementError:
        qDebug() << "Erreur dans l'instruction SQL";
        break;
    case QSqlError::TransactionError:
        qDebug() << "Erreur dans la transaction";
        break;
    default:
        qDebug() << "Erreur inconnue";
    }
}
```

## Bonnes pratiques

Pour une utilisation efficace et sécurisée de Qt SQL :

1. **Toujours utiliser les requêtes préparées** pour éviter les injections SQL
2. **Vérifier les retours de fonctions** pour détecter les erreurs
3. **Utiliser les transactions** pour les opérations multiples
4. **Fermer explicitement les connexions** quand elles ne sont plus nécessaires
5. **Éviter de coder en dur les informations de connexion** sensibles

## Exemple complet d'une classe GestionnaireDB

Voici un exemple de classe qui encapsule les opérations courantes de base de données :

```cpp
// gestionnaire_db.h
#ifndef GESTIONNAIRE_DB_H
#define GESTIONNAIRE_DB_H

#include <QObject>
#include <QString>
#include <QDate>

class GestionnaireDB : public QObject
{
    Q_OBJECT

public:
    explicit GestionnaireDB(QObject *parent = nullptr);
    ~GestionnaireDB();

    bool connecter(const QString &fichierDB);
    void deconnecter();

    bool creerTables();
    bool ajouterUtilisateur(const QString &nom, const QString &email);
    QList<QMap<QString, QVariant>> rechercherUtilisateurs(const QString &motCle);
    bool mettreAJourUtilisateur(int id, const QString &nom, const QString &email);
    bool supprimerUtilisateur(int id);

private:
    bool m_connecte;
};

#endif // GESTIONNAIRE_DB_H
```

```cpp
// gestionnaire_db.cpp
#include "gestionnaire_db.h"
#include <QSqlDatabase>
#include <QSqlQuery>
#include <QSqlError>
#include <QSqlRecord>
#include <QVariant>
#include <QDebug>

GestionnaireDB::GestionnaireDB(QObject *parent)
    : QObject(parent), m_connecte(false)
{
}

GestionnaireDB::~GestionnaireDB()
{
    if (m_connecte) {
        deconnecter();
    }
}

bool GestionnaireDB::connecter(const QString &fichierDB)
{
    QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE");
    db.setDatabaseName(fichierDB);

    if (!db.open()) {
        qDebug() << "Erreur de connexion :" << db.lastError().text();
        return false;
    }

    m_connecte = true;
    return true;
}

void GestionnaireDB::deconnecter()
{
    QSqlDatabase::database().close();
    QSqlDatabase::removeDatabase(QSqlDatabase::defaultConnection);
    m_connecte = false;
}

bool GestionnaireDB::creerTables()
{
    QSqlQuery query;
    bool success = query.exec(
        "CREATE TABLE IF NOT EXISTS utilisateurs ("
        "id INTEGER PRIMARY KEY AUTOINCREMENT, "
        "nom VARCHAR(50) NOT NULL, "
        "email VARCHAR(100) UNIQUE, "
        "date_inscription DATE"
        ")"
    );

    if (!success) {
        qDebug() << "Erreur lors de la création de la table :" << query.lastError().text();
    }

    return success;
}

bool GestionnaireDB::ajouterUtilisateur(const QString &nom, const QString &email)
{
    QSqlQuery query;
    query.prepare(
        "INSERT INTO utilisateurs (nom, email, date_inscription) "
        "VALUES (:nom, :email, DATE('now'))"
    );
    query.bindValue(":nom", nom);
    query.bindValue(":email", email);

    if (!query.exec()) {
        qDebug() << "Erreur lors de l'ajout d'un utilisateur :" << query.lastError().text();
        return false;
    }

    return true;
}

QList<QMap<QString, QVariant>> GestionnaireDB::rechercherUtilisateurs(const QString &motCle)
{
    QList<QMap<QString, QVariant>> resultats;
    QSqlQuery query;

    query.prepare(
        "SELECT id, nom, email, date_inscription FROM utilisateurs "
        "WHERE nom LIKE :motCle OR email LIKE :motCle"
    );
    query.bindValue(":motCle", "%" + motCle + "%");

    if (!query.exec()) {
        qDebug() << "Erreur lors de la recherche :" << query.lastError().text();
        return resultats;
    }

    while (query.next()) {
        QMap<QString, QVariant> utilisateur;
        utilisateur["id"] = query.value(0);
        utilisateur["nom"] = query.value(1);
        utilisateur["email"] = query.value(2);
        utilisateur["date_inscription"] = query.value(3);

        resultats.append(utilisateur);
    }

    return resultats;
}

bool GestionnaireDB::mettreAJourUtilisateur(int id, const QString &nom, const QString &email)
{
    QSqlQuery query;
    query.prepare(
        "UPDATE utilisateurs SET nom = :nom, email = :email "
        "WHERE id = :id"
    );
    query.bindValue(":nom", nom);
    query.bindValue(":email", email);
    query.bindValue(":id", id);

    if (!query.exec()) {
        qDebug() << "Erreur lors de la mise à jour :" << query.lastError().text();
        return false;
    }

    return query.numRowsAffected() > 0;
}

bool GestionnaireDB::supprimerUtilisateur(int id)
{
    QSqlQuery query;
    query.prepare("DELETE FROM utilisateurs WHERE id = :id");
    query.bindValue(":id", id);

    if (!query.exec()) {
        qDebug() << "Erreur lors de la suppression :" << query.lastError().text();
        return false;
    }

    return query.numRowsAffected() > 0;
}
```

## Conclusion

Qt SQL offre une solution robuste et flexible pour intégrer des bases de données dans vos applications. Pour les débutants, nous recommandons de commencer par SQLite qui est simple à mettre en place, puis d'explorer progressivement les fonctionnalités plus avancées comme les modèles relationnels et les transactions.

Dans la prochaine section, nous explorerons une autre méthode de persistance : la sérialisation avec QDataStream.

⏭️ [Sérialisation native avec QDataStream](/04-persistance-des-donnees/02-serialisation-native-avec-qdatastream.md)
