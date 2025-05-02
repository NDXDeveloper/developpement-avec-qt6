# 5.3 REST API avec Qt Network

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

## Introduction aux API REST

REST (Representational State Transfer) est un style d'architecture pour créer des services web. Une API REST est une interface qui permet à différentes applications de communiquer entre elles via le protocole HTTP. À l'heure actuelle, c'est l'une des méthodes les plus populaires pour développer des services web, car elle est simple, standardisée et facile à utiliser.

### Caractéristiques des API REST

- **Basées sur les ressources** : Les données sont représentées comme des ressources
- **Opérations standard** : Utilisation des méthodes HTTP (GET, POST, PUT, DELETE)
- **Sans état** : Chaque requête est indépendante
- **Format de données** : Généralement JSON ou XML

## Préparation de votre projet Qt

Pour travailler avec des API REST dans Qt, nous utiliserons principalement le module Qt Network.

### Configuration du projet

Ajoutez le module network à votre fichier `.pro` :

```
QT += network
```

Dans vos fichiers source, incluez les en-têtes nécessaires :

```cpp
#include <QNetworkAccessManager>
#include <QNetworkRequest>
#include <QNetworkReply>
#include <QJsonDocument>
#include <QJsonObject>
#include <QJsonArray>
```

## Requêtes REST basiques

### Lecture de données (GET)

La méthode GET est utilisée pour récupérer des données d'une API REST.

```cpp
void ApiClient::getDonnees()
{
    // Création d'une instance de QNetworkAccessManager
    QNetworkAccessManager *manager = new QNetworkAccessManager(this);

    // Connexion du signal finished pour récupérer la réponse
    connect(manager, &QNetworkAccessManager::finished,
            this, &ApiClient::onReponseRecue);

    // Construction de l'URL
    QUrl url("https://api.exemple.com/ressources");

    // Création et configuration de la requête
    QNetworkRequest requete(url);
    requete.setHeader(QNetworkRequest::ContentTypeHeader, "application/json");

    // Ajout d'un en-tête d'autorisation si nécessaire
    requete.setRawHeader("Authorization", "Bearer votre_token_ici");

    // Envoi de la requête GET
    manager->get(requete);
}

void ApiClient::onReponseRecue(QNetworkReply *reply)
{
    // Vérification des erreurs
    if (reply->error()) {
        qDebug() << "Erreur:" << reply->errorString();
        reply->deleteLater();
        return;
    }

    // Lecture des données de la réponse
    QByteArray responseData = reply->readAll();

    // Analyse du JSON
    QJsonDocument document = QJsonDocument::fromJson(responseData);

    // Vérification que le document est valide
    if (!document.isNull()) {
        if (document.isObject()) {
            // Traitement d'un objet JSON
            QJsonObject jsonObject = document.object();
            qDebug() << "Titre:" << jsonObject["titre"].toString();
            qDebug() << "Description:" << jsonObject["description"].toString();
        } else if (document.isArray()) {
            // Traitement d'un tableau JSON
            QJsonArray jsonArray = document.array();
            qDebug() << "Nombre d'éléments:" << jsonArray.size();

            // Parcourir les éléments du tableau
            for (int i = 0; i < jsonArray.size(); ++i) {
                QJsonObject obj = jsonArray[i].toObject();
                qDebug() << "Élément" << i << ":" << obj["nom"].toString();
            }
        }
    }

    // Important: libérer la mémoire du reply
    reply->deleteLater();
}
```

### Création de données (POST)

La méthode POST est utilisée pour créer de nouvelles ressources.

```cpp
void ApiClient::creerRessource(const QString &nom, const QString &description)
{
    QNetworkAccessManager *manager = new QNetworkAccessManager(this);

    connect(manager, &QNetworkAccessManager::finished,
            this, &ApiClient::onCreationTerminee);

    // Construction de l'URL
    QUrl url("https://api.exemple.com/ressources");

    // Création et configuration de la requête
    QNetworkRequest requete(url);
    requete.setHeader(QNetworkRequest::ContentTypeHeader, "application/json");

    // Création du contenu JSON
    QJsonObject json;
    json["nom"] = nom;
    json["description"] = description;

    QJsonDocument doc(json);
    QByteArray donnees = doc.toJson();

    // Envoi de la requête POST avec les données
    manager->post(requete, donnees);
}

void ApiClient::onCreationTerminee(QNetworkReply *reply)
{
    if (reply->error()) {
        qDebug() << "Erreur lors de la création:" << reply->errorString();
    } else {
        qDebug() << "Ressource créée avec succès!";

        // Traitement de la réponse si nécessaire...
    }

    reply->deleteLater();
}
```

### Mise à jour de données (PUT)

La méthode PUT est utilisée pour mettre à jour des ressources existantes.

```cpp
void ApiClient::mettreAJourRessource(int id, const QString &nom, const QString &description)
{
    QNetworkAccessManager *manager = new QNetworkAccessManager(this);

    connect(manager, &QNetworkAccessManager::finished,
            this, &ApiClient::onMiseAJourTerminee);

    // Construction de l'URL avec l'ID de la ressource
    QUrl url(QString("https://api.exemple.com/ressources/%1").arg(id));

    QNetworkRequest requete(url);
    requete.setHeader(QNetworkRequest::ContentTypeHeader, "application/json");

    QJsonObject json;
    json["nom"] = nom;
    json["description"] = description;

    QJsonDocument doc(json);
    QByteArray donnees = doc.toJson();

    // Envoi de la requête PUT avec les données
    manager->put(requete, donnees);
}
```

### Suppression de données (DELETE)

La méthode DELETE est utilisée pour supprimer des ressources.

```cpp
void ApiClient::supprimerRessource(int id)
{
    QNetworkAccessManager *manager = new QNetworkAccessManager(this);

    connect(manager, &QNetworkAccessManager::finished,
            this, &ApiClient::onSuppressionTerminee);

    // Construction de l'URL avec l'ID de la ressource
    QUrl url(QString("https://api.exemple.com/ressources/%1").arg(id));

    QNetworkRequest requete(url);

    // Envoi de la requête DELETE
    manager->deleteResource(requete);
}
```

## Gestion des erreurs

La gestion des erreurs est cruciale pour les applications qui interagissent avec des API REST.

```cpp
void ApiClient::traiterErreur(QNetworkReply *reply)
{
    // Récupérer le code d'état HTTP
    int statusCode = reply->attribute(QNetworkRequest::HttpStatusCodeAttribute).toInt();

    switch (statusCode) {
    case 400:
        qDebug() << "Erreur 400: Requête incorrecte";
        break;
    case 401:
        qDebug() << "Erreur 401: Non autorisé";
        // Peut-être déclencher une nouvelle authentification
        break;
    case 403:
        qDebug() << "Erreur 403: Accès interdit";
        break;
    case 404:
        qDebug() << "Erreur 404: Ressource non trouvée";
        break;
    case 500:
    case 501:
    case 502:
    case 503:
        qDebug() << "Erreur serveur:" << statusCode;
        break;
    default:
        qDebug() << "Erreur non gérée:" << statusCode;
    }

    // Lire le corps de la réponse qui peut contenir des détails sur l'erreur
    QByteArray responseData = reply->readAll();
    qDebug() << "Détails de l'erreur:" << responseData;
}
```

## Authentification

La plupart des API REST nécessitent une authentification. Voici comment gérer les méthodes d'authentification courantes.

### Authentification par jeton (Bearer Token)

```cpp
QNetworkRequest requete(url);
requete.setRawHeader("Authorization", "Bearer votre_jeton_ici");
```

### Authentification basique (nom d'utilisateur/mot de passe)

```cpp
// Combiner nom d'utilisateur et mot de passe
QString credentials = "nom_utilisateur:mot_de_passe";
QByteArray base64Credentials = credentials.toUtf8().toBase64();

QNetworkRequest requete(url);
requete.setRawHeader("Authorization", "Basic " + base64Credentials);
```

### Authentification OAuth

L'authentification OAuth est plus complexe, mais Qt fournit des classes pour vous aider, notamment `QOAuth2AuthorizationCodeFlow`.

```cpp
#include <QtNetworkAuth>

void configureOAuth()
{
    auto oauth2 = new QOAuth2AuthorizationCodeFlow(this);
    oauth2->setAuthorizationUrl(QUrl("https://exemple.com/oauth/authorize"));
    oauth2->setTokenUrl(QUrl("https://exemple.com/oauth/token"));
    oauth2->setClientIdentifier("votre_client_id");
    oauth2->setClientIdentifierSharedKey("votre_client_secret");

    // Connectez les signaux pour gérer l'authentification
    connect(oauth2, &QOAuth2AuthorizationCodeFlow::authorizeWithBrowser,
            &QDesktopServices::openUrl);

    connect(oauth2, &QOAuth2AuthorizationCodeFlow::granted, this, [=]() {
        qDebug() << "Authentification réussie";
        // Vous pouvez maintenant faire des requêtes authentifiées
    });

    // Lancer le processus d'authentification
    oauth2->grant();
}
```

## Création d'une classe d'API client réutilisable

Pour organiser votre code, il est préférable de créer une classe dédiée qui encapsule les interactions avec l'API.

```cpp
// api_client.h
#pragma once

#include <QObject>
#include <QNetworkAccessManager>
#include <QNetworkReply>
#include <QJsonDocument>

class ApiClient : public QObject
{
    Q_OBJECT

public:
    explicit ApiClient(QObject *parent = nullptr);

    // Méthodes pour accéder à l'API
    void getUtilisateurs();
    void getUtilisateur(int id);
    void creerUtilisateur(const QString &nom, const QString &email);
    void mettreAJourUtilisateur(int id, const QString &nom, const QString &email);
    void supprimerUtilisateur(int id);

    // Définir le jeton d'authentification
    void setToken(const QString &token) { m_token = token; }

signals:
    // Signaux émis lors de la réception de données
    void utilisateursRecus(const QJsonArray &utilisateurs);
    void utilisateurRecu(const QJsonObject &utilisateur);
    void utilisateurCree(const QJsonObject &utilisateur);
    void utilisateurMisAJour(const QJsonObject &utilisateur);
    void utilisateurSupprime(int id);
    void erreurSurvenue(const QString &message, int statusCode);

private slots:
    // Slots pour gérer les réponses du réseau
    void traiterReponseUtilisateurs(QNetworkReply *reply);
    void traiterReponseUtilisateur(QNetworkReply *reply);
    void traiterReponseCreation(QNetworkReply *reply);
    void traiterReponseMiseAJour(QNetworkReply *reply);
    void traiterReponseSuppression(QNetworkReply *reply);

private:
    QNetworkAccessManager *m_manager;
    QString m_baseUrl;
    QString m_token;

    // Méthode utilitaire pour créer des requêtes
    QNetworkRequest creerRequete(const QString &endpoint);
    void traiterErreur(QNetworkReply *reply);
};
```

```cpp
// api_client.cpp
#include "api_client.h"
#include <QJsonObject>
#include <QJsonArray>

ApiClient::ApiClient(QObject *parent) : QObject(parent)
{
    m_manager = new QNetworkAccessManager(this);
    m_baseUrl = "https://api.exemple.com/";
}

QNetworkRequest ApiClient::creerRequete(const QString &endpoint)
{
    QUrl url(m_baseUrl + endpoint);
    QNetworkRequest requete(url);

    requete.setHeader(QNetworkRequest::ContentTypeHeader, "application/json");

    if (!m_token.isEmpty()) {
        requete.setRawHeader("Authorization", "Bearer " + m_token.toUtf8());
    }

    return requete;
}

void ApiClient::getUtilisateurs()
{
    QNetworkRequest requete = creerRequete("utilisateurs");
    QNetworkReply *reply = m_manager->get(requete);

    connect(reply, &QNetworkReply::finished, this, [=]() {
        this->traiterReponseUtilisateurs(reply);
    });
}

void ApiClient::getUtilisateur(int id)
{
    QNetworkRequest requete = creerRequete(QString("utilisateurs/%1").arg(id));
    QNetworkReply *reply = m_manager->get(requete);

    connect(reply, &QNetworkReply::finished, this, [=]() {
        this->traiterReponseUtilisateur(reply);
    });
}

void ApiClient::creerUtilisateur(const QString &nom, const QString &email)
{
    QNetworkRequest requete = creerRequete("utilisateurs");

    QJsonObject donnees;
    donnees["nom"] = nom;
    donnees["email"] = email;

    QJsonDocument doc(donnees);
    QByteArray json = doc.toJson();

    QNetworkReply *reply = m_manager->post(requete, json);

    connect(reply, &QNetworkReply::finished, this, [=]() {
        this->traiterReponseCreation(reply);
    });
}

void ApiClient::traiterReponseUtilisateurs(QNetworkReply *reply)
{
    if (reply->error()) {
        traiterErreur(reply);
        reply->deleteLater();
        return;
    }

    QByteArray donnees = reply->readAll();
    QJsonDocument doc = QJsonDocument::fromJson(donnees);

    if (doc.isArray()) {
        emit utilisateursRecus(doc.array());
    }

    reply->deleteLater();
}

void ApiClient::traiterErreur(QNetworkReply *reply)
{
    int statusCode = reply->attribute(QNetworkRequest::HttpStatusCodeAttribute).toInt();
    QString message = reply->errorString();

    // Tenter de lire les détails de l'erreur dans le corps de la réponse
    QByteArray donnees = reply->readAll();
    QJsonDocument doc = QJsonDocument::fromJson(donnees);

    if (!doc.isNull() && doc.isObject()) {
        QJsonObject obj = doc.object();
        if (obj.contains("message")) {
            message = obj["message"].toString();
        } else if (obj.contains("error")) {
            message = obj["error"].toString();
        }
    }

    emit erreurSurvenue(message, statusCode);
}
```

## Utilisation de la classe ApiClient

```cpp
// Dans votre application
#include "api_client.h"

void MaFenetre::initialiserApi()
{
    m_apiClient = new ApiClient(this);

    // Définir le jeton d'authentification
    m_apiClient->setToken("votre_jeton_ici");

    // Connecter les signaux pour recevoir les données
    connect(m_apiClient, &ApiClient::utilisateursRecus,
            this, &MaFenetre::afficherUtilisateurs);

    connect(m_apiClient, &ApiClient::erreurSurvenue,
            this, &MaFenetre::afficherErreur);

    // Récupérer la liste des utilisateurs
    m_apiClient->getUtilisateurs();
}

void MaFenetre::afficherUtilisateurs(const QJsonArray &utilisateurs)
{
    // Effacer la liste existante
    ui->listeUtilisateurs->clear();

    // Remplir avec les nouvelles données
    for (const QJsonValue &valeur : utilisateurs) {
        QJsonObject utilisateur = valeur.toObject();
        QString nom = utilisateur["nom"].toString();
        QString email = utilisateur["email"].toString();

        QListWidgetItem *item = new QListWidgetItem(nom + " (" + email + ")");
        item->setData(Qt::UserRole, utilisateur["id"].toInt());
        ui->listeUtilisateurs->addItem(item);
    }
}

void MaFenetre::afficherErreur(const QString &message, int statusCode)
{
    QMessageBox::critical(this, "Erreur API",
                         QString("Erreur %1: %2").arg(statusCode).arg(message));
}

void MaFenetre::onBoutonAjouterClicked()
{
    QString nom = ui->nomLineEdit->text();
    QString email = ui->emailLineEdit->text();

    if (nom.isEmpty() || email.isEmpty()) {
        QMessageBox::warning(this, "Données manquantes",
                            "Veuillez remplir tous les champs.");
        return;
    }

    m_apiClient->creerUtilisateur(nom, email);

    // Effacer les champs
    ui->nomLineEdit->clear();
    ui->emailLineEdit->clear();
}
```

## Pagination des résultats

De nombreuses API REST utilisent la pagination pour limiter la quantité de données renvoyées dans une seule réponse.

```cpp
void ApiClient::getUtilisateurs(int page, int itemsParPage)
{
    QUrl url(m_baseUrl + "utilisateurs");

    // Ajouter les paramètres de pagination à l'URL
    QUrlQuery query;
    query.addQueryItem("page", QString::number(page));
    query.addQueryItem("limit", QString::number(itemsParPage));
    url.setQuery(query);

    QNetworkRequest requete(url);
    // ... le reste du code comme avant
}
```

## Téléchargement de fichiers

Les API REST peuvent également être utilisées pour télécharger des fichiers.

```cpp
void ApiClient::telechargerFichier(const QString &url, const QString &cheminLocal)
{
    QNetworkRequest requete(url);
    QNetworkReply *reply = m_manager->get(requete);

    // Connecter le signal pour suivre la progression
    connect(reply, &QNetworkReply::downloadProgress,
            this, &ApiClient::progressionTelechargement);

    // Fichier de destination
    m_fichierDestination = new QFile(cheminLocal);

    if (!m_fichierDestination->open(QIODevice::WriteOnly)) {
        qDebug() << "Erreur: Impossible d'ouvrir le fichier" << cheminLocal;
        reply->abort();
        reply->deleteLater();
        delete m_fichierDestination;
        m_fichierDestination = nullptr;
        return;
    }

    // Connecter le signal pour recevoir les données
    connect(reply, &QNetworkReply::readyRead, this, [=]() {
        if (m_fichierDestination) {
            m_fichierDestination->write(reply->readAll());
        }
    });

    // Connecter le signal pour la fin du téléchargement
    connect(reply, &QNetworkReply::finished, this, [=]() {
        if (m_fichierDestination) {
            m_fichierDestination->close();
            delete m_fichierDestination;
            m_fichierDestination = nullptr;
        }

        if (reply->error()) {
            traiterErreur(reply);
        } else {
            emit fichierTelecharge(cheminLocal);
        }

        reply->deleteLater();
    });
}
```

## Envoi de fichiers (Upload)

Pour envoyer un fichier à une API REST, vous devez utiliser un format multipart.

```cpp
void ApiClient::envoyerFichier(const QString &cheminFichier, const QString &nomFichier)
{
    QFile *fichier = new QFile(cheminFichier);

    if (!fichier->open(QIODevice::ReadOnly)) {
        qDebug() << "Erreur: Impossible d'ouvrir le fichier" << cheminFichier;
        delete fichier;
        return;
    }

    QNetworkRequest requete = creerRequete("fichiers/upload");

    // Créer les données multipart
    QHttpMultiPart *multiPart = new QHttpMultiPart(QHttpMultiPart::FormDataType);

    // Ajouter la partie du fichier
    QHttpPart filePart;
    filePart.setHeader(QNetworkRequest::ContentTypeHeader, QVariant("application/octet-stream"));
    filePart.setHeader(QNetworkRequest::ContentDispositionHeader,
                      QVariant("form-data; name=\"fichier\"; filename=\"" + nomFichier + "\""));
    filePart.setBodyDevice(fichier);
    fichier->setParent(multiPart); // Le fichier sera supprimé avec multiPart

    multiPart->append(filePart);

    // Envoyer la requête
    QNetworkReply *reply = m_manager->post(requete, multiPart);
    multiPart->setParent(reply); // Le multiPart sera supprimé avec reply

    // Connecter le signal pour la progression
    connect(reply, &QNetworkReply::uploadProgress,
            this, &ApiClient::progressionEnvoi);

    // Connecter le signal pour la fin de l'envoi
    connect(reply, &QNetworkReply::finished, this, [=]() {
        if (reply->error()) {
            traiterErreur(reply);
        } else {
            emit fichierEnvoye();
        }

        reply->deleteLater();
    });
}
```

## Bonnes pratiques pour l'utilisation des API REST

1. **Gestion des erreurs** : Toujours vérifier les erreurs et les codes de statut HTTP
2. **Libération des ressources** : Appeler `deleteLater()` sur les objets QNetworkReply
3. **Timeout** : Définir des délais d'attente pour éviter que votre application ne reste bloquée
4. **Gestion du jeton** : Mettre à jour le jeton d'authentification lorsqu'il expire
5. **Traitement asynchrone** : Éviter de bloquer l'interface utilisateur pendant les requêtes réseau
6. **Cache** : Mettre en cache les réponses qui ne changent pas souvent
7. **Rétrocompatibilité** : Être robuste face aux changements d'API
8. **Documentation** : Documenter les interactions avec l'API pour faciliter la maintenance

## Exemple d'application complète avec API REST

Voici un exemple d'application simple qui utilise une API REST pour gérer une liste de tâches.

```cpp
// task_manager.h
#pragma once

#include <QMainWindow>
#include "api_client.h"

namespace Ui {
class TaskManager;
}

class TaskManager : public QMainWindow
{
    Q_OBJECT

public:
    explicit TaskManager(QWidget *parent = nullptr);
    ~TaskManager();

private slots:
    void onConnexionClicked();
    void onAjouterClicked();
    void onModifierClicked();
    void onSupprimerClicked();

    void afficherTaches(const QJsonArray &taches);
    void afficherErreur(const QString &message, int statusCode);

private:
    Ui::TaskManager *ui;
    ApiClient *m_apiClient;
    int m_tacheSelectionneeId;

    void initialiserConnexions();
    void actualiserTaches();
};

// task_manager.cpp
#include "task_manager.h"
#include "ui_task_manager.h"
#include <QMessageBox>

TaskManager::TaskManager(QWidget *parent) :
    QMainWindow(parent),
    ui(new Ui::TaskManager),
    m_apiClient(new ApiClient(this)),
    m_tacheSelectionneeId(-1)
{
    ui->setupUi(this);

    // Désactiver les boutons tant que l'utilisateur n'est pas connecté
    ui->ajouterButton->setEnabled(false);
    ui->modifierButton->setEnabled(false);
    ui->supprimerButton->setEnabled(false);

    initialiserConnexions();
}

TaskManager::~TaskManager()
{
    delete ui;
}

void TaskManager::initialiserConnexions()
{
    // Connecter les boutons
    connect(ui->connexionButton, &QPushButton::clicked,
            this, &TaskManager::onConnexionClicked);
    connect(ui->ajouterButton, &QPushButton::clicked,
            this, &TaskManager::onAjouterClicked);
    connect(ui->modifierButton, &QPushButton::clicked,
            this, &TaskManager::onModifierClicked);
    connect(ui->supprimerButton, &QPushButton::clicked,
            this, &TaskManager::onSupprimerClicked);

    // Connecter les signaux de l'API
    connect(m_apiClient, &ApiClient::tachesRecues,
            this, &TaskManager::afficherTaches);
    connect(m_apiClient, &ApiClient::erreurSurvenue,
            this, &TaskManager::afficherErreur);

    // Connecter le signal de sélection de la liste
    connect(ui->tachesListWidget, &QListWidget::itemSelectionChanged,
            this, [=]() {
                QListWidgetItem *item = ui->tachesListWidget->currentItem();
                if (item) {
                    m_tacheSelectionneeId = item->data(Qt::UserRole).toInt();
                    ui->modifierButton->setEnabled(true);
                    ui->supprimerButton->setEnabled(true);
                } else {
                    m_tacheSelectionneeId = -1;
                    ui->modifierButton->setEnabled(false);
                    ui->supprimerButton->setEnabled(false);
                }
            });
}

void TaskManager::onConnexionClicked()
{
    QString username = ui->usernameLineEdit->text();
    QString password = ui->passwordLineEdit->text();

    if (username.isEmpty() || password.isEmpty()) {
        QMessageBox::warning(this, "Erreur", "Veuillez entrer le nom d'utilisateur et le mot de passe.");
        return;
    }

    // Appeler l'API pour obtenir un jeton
    m_apiClient->connecter(username, password);

    // Le signal tokenRecu sera émis si la connexion réussit
    connect(m_apiClient, &ApiClient::tokenRecu, this, [=](const QString &token) {
        ui->connexionButton->setEnabled(false);
        ui->usernameLineEdit->setEnabled(false);
        ui->passwordLineEdit->setEnabled(false);
        ui->ajouterButton->setEnabled(true);

        // Mettre à jour le jeton et récupérer les tâches
        m_apiClient->setToken(token);
        actualiserTaches();
    });
}

void TaskManager::actualiserTaches()
{
    m_apiClient->getTaches();
}

void TaskManager::afficherTaches(const QJsonArray &taches)
{
    ui->tachesListWidget->clear();

    for (const QJsonValue &valeur : taches) {
        QJsonObject tache = valeur.toObject();
        QString titre = tache["titre"].toString();
        bool terminee = tache["terminee"].toBool();

        QListWidgetItem *item = new QListWidgetItem;
        item->setText(titre);
        item->setData(Qt::UserRole, tache["id"].toInt());

        if (terminee) {
            item->setCheckState(Qt::Checked);
        } else {
            item->setCheckState(Qt::Unchecked);
        }

        ui->tachesListWidget->addItem(item);
    }
}

void TaskManager::afficherErreur(const QString &message, int statusCode)
{
    QMessageBox::critical(this, "Erreur API",
                         QString("Erreur %1: %2").arg(statusCode).arg(message));
}

void TaskManager::onAjouterClicked()
{
    QString titre = ui->titreLineEdit->text();

    if (titre.isEmpty()) {
        QMessageBox::warning(this, "Erreur", "Veuillez entrer un titre pour la tâche.");
        return;
    }

    m_apiClient->creerTache(titre, ui->descriptionTextEdit->toPlainText());

    // Effacer les champs
    ui->titreLineEdit->clear();
    ui->descriptionTextEdit->clear();

    // Actualiser la liste après l'ajout
    connect(m_apiClient, &ApiClient::tacheCreee, this, [=]() {
        actualiserTaches();
    });
}

void TaskManager::onModifierClicked()
{
    if (m_tacheSelectionneeId == -1) {
        return;
    }

    QString titre = ui->titreLineEdit->text();

    if (titre.isEmpty()) {
        QMessageBox::warning(this, "Erreur", "Veuillez entrer un titre pour la tâche.");
        return;
    }

    m_apiClient->mettreAJourTache(
        m_tacheSelectionneeId,
        titre,
        ui->descriptionTextEdit->toPlainText()
    );

    // Effacer les champs
    ui->titreLineEdit->clear();
    ui->descriptionTextEdit->clear();

    // Actualiser la liste après la modification
    connect(m_apiClient, &ApiClient::tacheMiseAJour, this, [=]() {
        actualiserTaches();
    });
}

void TaskManager::onSupprimerClicked()
{
    if (m_tacheSelectionneeId == -1) {
        return;
    }

    // Demander confirmation
    QMessageBox::StandardButton reponse = QMessageBox::question(
        this,
        "Confirmation",
        "Êtes-vous sûr de vouloir supprimer cette tâche ?",
        QMessageBox::Yes | QMessageBox::No
    );

    if (reponse == QMessageBox::Yes) {
        m_apiClient->supprimerTache(m_tacheSelectionneeId);

        // Actualiser la liste après la suppression
        connect(m_apiClient, &ApiClient::tacheSupprimee, this, [=]() {
            actualiserTaches();
        });
    }
}
```

## Gestion des timeouts

Il est important de définir des délais d'attente (timeouts) pour les requêtes réseau afin d'éviter que votre application ne reste bloquée en cas de problème de connectivité.

```cpp
QNetworkRequest requete(url);

// Définir un timeout de 30 secondes (30000 ms)
requete.setAttribute(QNetworkRequest::RequestAttributeTransferTimeout, 30000);
```

## Annulation des requêtes

Vous pouvez annuler une requête en cours si nécessaire :

```cpp
QNetworkReply *reply = m_manager->get(requete);

// Plus tard, si vous devez annuler
reply->abort();
```

## Gestion de la mise en cache

La mise en cache des réponses peut améliorer les performances de votre application.

```cpp
QNetworkDiskCache *cache = new QNetworkDiskCache(this);
cache->setCacheDirectory(QStandardPaths::writableLocation(QStandardPaths::CacheLocation));

m_manager->setCache(cache);

// Pour une requête spécifique
QNetworkRequest requete(url);
requete.setAttribute(QNetworkRequest::CacheLoadControlAttribute, QNetworkRequest::PreferCache);
```

## Compression des données

Qt Network gère automatiquement la compression/décompression des données:

```cpp
QNetworkRequest requete(url);
requete.setRawHeader("Accept-Encoding", "gzip, deflate");
```

## Utilisation des API asynchrones

Qt6 propose une nouvelle façon d'utiliser les API réseau avec `QNetworkInformation`:

```cpp
#include <QNetworkInformation>

void verifierConnectivite()
{
    if (QNetworkInformation::loadBackendByName("network")) {
        connect(QNetworkInformation::instance(), &QNetworkInformation::reachabilityChanged,
                this, &MaClasse::onConnectiviteChangee);

        // Vérifier l'état actuel
        if (QNetworkInformation::instance()->reachability()
            == QNetworkInformation::Reachability::Online) {
            qDebug() << "En ligne";
        } else {
            qDebug() << "Hors ligne";
        }
    } else {
        qDebug() << "Impossible de charger le backend réseau";
    }
}

void MaClasse::onConnectiviteChangee(QNetworkInformation::Reachability reachability)
{
    if (reachability == QNetworkInformation::Reachability::Online) {
        qDebug() << "La connexion Internet est rétablie";
        // Réessayer les requêtes en attente
    } else {
        qDebug() << "La connexion Internet est perdue";
        // Mettre en pause les requêtes ou passer en mode hors ligne
    }
}
```

## Implémentation d'une file d'attente de requêtes

Pour gérer efficacement plusieurs requêtes, vous pouvez implémenter une file d'attente :

```cpp
class RequestQueue : public QObject
{
    Q_OBJECT

public:
    explicit RequestQueue(QObject *parent = nullptr);

    void ajouterRequete(const QNetworkRequest &requete,
                        const QByteArray &donnees = QByteArray(),
                        const QString &methode = "GET");
    void demarrer();
    void arreter();

signals:
    void reponseRecue(QNetworkReply *reply);

private slots:
    void traiterReponse();
    void traiterProchaineDemande();

private:
    QNetworkAccessManager *m_manager;
    QQueue<QPair<QNetworkRequest, QPair<QByteArray, QString>>> m_file;
    bool m_enCours;
    QNetworkReply *m_replyActuel;
};

RequestQueue::RequestQueue(QObject *parent) : QObject(parent)
{
    m_manager = new QNetworkAccessManager(this);
    m_enCours = false;
    m_replyActuel = nullptr;
}

void RequestQueue::ajouterRequete(const QNetworkRequest &requete,
                                const QByteArray &donnees,
                                const QString &methode)
{
    m_file.enqueue(qMakePair(requete, qMakePair(donnees, methode)));

    if (!m_enCours) {
        traiterProchaineDemande();
    }
}

void RequestQueue::traiterProchaineDemande()
{
    if (m_file.isEmpty()) {
        m_enCours = false;
        return;
    }

    m_enCours = true;

    auto demande = m_file.dequeue();
    QNetworkRequest requete = demande.first;
    QByteArray donnees = demande.second.first;
    QString methode = demande.second.second;

    if (methode == "GET") {
        m_replyActuel = m_manager->get(requete);
    } else if (methode == "POST") {
        m_replyActuel = m_manager->post(requete, donnees);
    } else if (methode == "PUT") {
        m_replyActuel = m_manager->put(requete, donnees);
    } else if (methode == "DELETE") {
        m_replyActuel = m_manager->deleteResource(requete);
    }

    connect(m_replyActuel, &QNetworkReply::finished,
            this, &RequestQueue::traiterReponse);
}

void RequestQueue::traiterReponse()
{
    emit reponseRecue(m_replyActuel);

    m_replyActuel->deleteLater();
    m_replyActuel = nullptr;

    // Traiter la prochaine demande
    traiterProchaineDemande();
}

void RequestQueue::demarrer()
{
    if (!m_enCours && !m_file.isEmpty()) {
        traiterProchaineDemande();
    }
}

void RequestQueue::arreter()
{
    if (m_replyActuel) {
        m_replyActuel->abort();
    }

    m_enCours = false;
}
```

## Utilisation de la file d'attente de requêtes

```cpp
// Création de la file d'attente
RequestQueue *queue = new RequestQueue(this);

// Connexion du signal pour traiter les réponses
connect(queue, &RequestQueue::reponseRecue,
        this, &MaClasse::traiterReponse);

// Ajout de requêtes à la file d'attente
QNetworkRequest requete1(QUrl("https://api.exemple.com/ressources/1"));
QNetworkRequest requete2(QUrl("https://api.exemple.com/ressources/2"));

queue->ajouterRequete(requete1);
queue->ajouterRequete(requete2);

// Les requêtes seront traitées une par une
```

## Gestion efficace du JSON avec Qt

Qt fournit des classes pour manipuler facilement le JSON :

```cpp
// Conversion d'un objet C++ en JSON
class Utilisateur
{
public:
    int id;
    QString nom;
    QString email;

    // Convertir en QJsonObject
    QJsonObject toJson() const
    {
        QJsonObject json;
        json["id"] = id;
        json["nom"] = nom;
        json["email"] = email;
        return json;
    }

    // Créer à partir d'un QJsonObject
    static Utilisateur fromJson(const QJsonObject &json)
    {
        Utilisateur utilisateur;
        utilisateur.id = json["id"].toInt();
        utilisateur.nom = json["nom"].toString();
        utilisateur.email = json["email"].toString();
        return utilisateur;
    }
};

// Utilisation
Utilisateur utilisateur;
utilisateur.id = 1;
utilisateur.nom = "Jean Dupont";
utilisateur.email = "jean@exemple.com";

// Conversion en JSON
QJsonObject jsonObj = utilisateur.toJson();
QJsonDocument doc(jsonObj);
QByteArray json = doc.toJson();

// Envoi de la requête
QNetworkRequest requete(url);
requete.setHeader(QNetworkRequest::ContentTypeHeader, "application/json");
m_manager->post(requete, json);
```

## Synchronisation avec QtConcurrent

Pour les opérations qui nécessitent un traitement important des données reçues, vous pouvez utiliser `QtConcurrent` pour déplacer ce traitement dans un thread séparé :

```cpp
#include <QtConcurrent>

void ApiClient::traiterReponseUtilisateurs(QNetworkReply *reply)
{
    if (reply->error()) {
        // Gérer l'erreur...
        reply->deleteLater();
        return;
    }

    // Lire les données
    QByteArray donnees = reply->readAll();
    reply->deleteLater();

    // Traiter les données dans un thread séparé
    QFuture<QList<Utilisateur>> future = QtConcurrent::run([donnees]() {
        QList<Utilisateur> utilisateurs;

        QJsonDocument doc = QJsonDocument::fromJson(donnees);
        if (doc.isArray()) {
            QJsonArray array = doc.array();

            for (const QJsonValue &val : array) {
                if (val.isObject()) {
                    utilisateurs.append(Utilisateur::fromJson(val.toObject()));
                }
            }
        }

        return utilisateurs;
    });

    // Connecter un watcher pour être notifié lorsque le traitement est terminé
    QFutureWatcher<QList<Utilisateur>> *watcher = new QFutureWatcher<QList<Utilisateur>>(this);
    watcher->setFuture(future);

    connect(watcher, &QFutureWatcher<QList<Utilisateur>>::finished, this, [=]() {
        QList<Utilisateur> utilisateurs = watcher->result();

        // Émettre le signal avec les données traitées
        emit utilisateursRecus(utilisateurs);

        // Supprimer le watcher
        watcher->deleteLater();
    });
}
```

## Sérialisation et désérialisation automatique avec des macros

Pour simplifier davantage la conversion entre les objets C++ et JSON, vous pouvez utiliser des macros ou des techniques plus avancées :

```cpp
// Macro pour générer automatiquement les méthodes toJson et fromJson
#define MAKE_JSON_SERIALIZABLE(className, ...) \
    QJsonObject toJson() const { \
        QJsonObject json; \
        QStringList fields = QString(#__VA_ARGS__).split(','); \
        for (const QString &field : fields) { \
            QString trimmed = field.trimmed(); \
            json[trimmed] = QJsonValue::fromVariant(property(trimmed.toUtf8())); \
        } \
        return json; \
    } \
    static className fromJson(const QJsonObject &json) { \
        className obj; \
        QStringList fields = QString(#__VA_ARGS__).split(','); \
        for (const QString &field : fields) { \
            QString trimmed = field.trimmed(); \
            obj.setProperty(trimmed.toUtf8(), json[trimmed].toVariant()); \
        } \
        return obj; \
    }

// Utilisation
class Utilisateur : public QObject
{
    Q_OBJECT
    Q_PROPERTY(int id READ id WRITE setId)
    Q_PROPERTY(QString nom READ nom WRITE setNom)
    Q_PROPERTY(QString email READ email WRITE setEmail)

public:
    // Getters et setters...

    MAKE_JSON_SERIALIZABLE(Utilisateur, id, nom, email)
};
```

## Gestion des API REST avec des techniques plus modernes

Pour les projets plus importants, vous pourriez envisager d'utiliser des approches plus modernes pour gérer les interactions avec les API REST.

### Approche basée sur les classes générées

```cpp
// Définition d'une classe générique pour les API REST
template <typename T>
class RestApi : public QObject
{
    Q_OBJECT

public:
    explicit RestApi(QObject *parent = nullptr)
        : QObject(parent), m_manager(new QNetworkAccessManager(this)) {}

    void setBaseUrl(const QString &url) { m_baseUrl = url; }
    void setTokenAuthorization(const QString &token) {
        m_authHeader = "Bearer " + token;
    }

    void getAll();
    void getOne(int id);
    void create(const T &item);
    void update(int id, const T &item);
    void remove(int id);

signals:
    void itemsReceived(const QList<T> &items);
    void itemReceived(const T &item);
    void itemCreated(const T &item);
    void itemUpdated(const T &item);
    void itemRemoved(int id);
    void error(const QString &message, int statusCode);

private:
    QNetworkAccessManager *m_manager;
    QString m_baseUrl;
    QString m_authHeader;
    QString m_resourcePath;

    QNetworkRequest createRequest(const QString &endpoint);
    void handleError(QNetworkReply *reply);
};

// Implémentation
template <typename T>
void RestApi<T>::getAll()
{
    QNetworkRequest request = createRequest(m_resourcePath);
    QNetworkReply *reply = m_manager->get(request);

    connect(reply, &QNetworkReply::finished, this, [=]() {
        if (reply->error()) {
            handleError(reply);
        } else {
            QByteArray data = reply->readAll();
            QJsonDocument doc = QJsonDocument::fromJson(data);

            if (doc.isArray()) {
                QList<T> items;
                QJsonArray array = doc.array();

                for (const QJsonValue &val : array) {
                    if (val.isObject()) {
                        T item = T::fromJson(val.toObject());
                        items.append(item);
                    }
                }

                emit itemsReceived(items);
            }
        }

        reply->deleteLater();
    });
}

// Utilisation
class UserApi : public RestApi<User>
{
public:
    UserApi(QObject *parent = nullptr) : RestApi<User>(parent) {
        setBaseUrl("https://api.example.com");
        m_resourcePath = "users";
    }
};

// Dans votre application
UserApi *userApi = new UserApi(this);
userApi->setTokenAuthorization("votre_token");

connect(userApi, &UserApi::itemsReceived, this, [=](const QList<User> &users) {
    // Traiter la liste des utilisateurs
});

connect(userApi, &UserApi::error, this, [=](const QString &message, int statusCode) {
    // Gérer l'erreur
});

userApi->getAll();
```

## Dépannage et résolution des problèmes courants

### 1. Problème de certificat SSL

```cpp
// Ignorer les erreurs de certificat (à utiliser uniquement en développement !)
QSslConfiguration config = QSslConfiguration::defaultConfiguration();
config.setPeerVerifyMode(QSslSocket::VerifyNone);
requete.setSslConfiguration(config);
```

### 2. Débogage des requêtes et réponses

```cpp
// Afficher les détails de la requête
qDebug() << "Méthode:" << (reply->operation() == QNetworkAccessManager::GetOperation ? "GET" :
                          reply->operation() == QNetworkAccessManager::PostOperation ? "POST" :
                          reply->operation() == QNetworkAccessManager::PutOperation ? "PUT" :
                          reply->operation() == QNetworkAccessManager::DeleteOperation ? "DELETE" : "Autre");
qDebug() << "URL:" << reply->url().toString();
qDebug() << "En-têtes:";
const QList<QByteArray> requestHeaders = reply->request().rawHeaderList();
for (const QByteArray &header : requestHeaders) {
    qDebug() << " -" << header << ":" << reply->request().rawHeader(header);
}

// Afficher les détails de la réponse
qDebug() << "Code de statut:" << reply->attribute(QNetworkRequest::HttpStatusCodeAttribute).toInt();
qDebug() << "En-têtes de réponse:";
const QList<QByteArray> responseHeaders = reply->rawHeaderList();
for (const QByteArray &header : responseHeaders) {
    qDebug() << " -" << header << ":" << reply->rawHeader(header);
}
qDebug() << "Corps de la réponse:" << reply->readAll();
```

### 3. Gestion des redirections

```cpp
// Suivre les redirections
QNetworkRequest requete(url);
requete.setAttribute(QNetworkRequest::FollowRedirectsAttribute, true);
```

## Conclusion et meilleures pratiques

L'utilisation des API REST avec Qt Network offre une flexibilité et une puissance considérables pour développer des applications connectées. Voici quelques principes clés à retenir :

1. **Encapsuler votre code d'API** : Créez des classes dédiées pour gérer vos interactions avec les API
2. **Asynchronisme** : Utilisez toujours des approches asynchrones pour éviter de bloquer l'interface utilisateur
3. **Gestion des erreurs** : Mettez en place une gestion robuste des erreurs et des codes de statut
4. **Sécurité** : Utilisez HTTPS et gérez correctement les jetons d'authentification
5. **Timeout et annulation** : Définissez des délais d'attente et prévoyez l'annulation des requêtes
6. **Cache** : Mettez en cache les réponses lorsque cela est approprié pour améliorer les performances
7. **Documentation** : Documentez bien vos interfaces avec les API pour faciliter la maintenance

En suivant ces bonnes pratiques et en utilisant les techniques décrites dans ce tutoriel, vous serez en mesure de créer des applications Qt robustes qui interagissent efficacement avec des API REST.

⏭️ [Bluetooth et NFC avec Qt](/05-communication-reseau/04-bluetooth-et-nfc-avec-qt.md)
