# 5.1 API réseau de Qt (QNetworkAccessManager)

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

## Introduction à QNetworkAccessManager

QNetworkAccessManager est la classe principale de Qt pour gérer les communications réseau. Elle agit comme un centre de contrôle pour toutes vos requêtes HTTP, vous permettant d'accéder facilement à des ressources en ligne, d'interagir avec des API REST, de télécharger des fichiers, et bien plus encore.

Cette classe est conçue pour simplifier les opérations réseau tout en offrant une approche asynchrone, ce qui signifie que votre interface utilisateur reste réactive pendant que les opérations réseau s'exécutent en arrière-plan.

## Configuration de votre projet

Pour utiliser QNetworkAccessManager, vous devez d'abord ajouter le module network à votre projet Qt.

Dans votre fichier `.pro` :
```
QT += network
```

Et dans vos fichiers source, incluez l'en-tête nécessaire :
```cpp
#include <QNetworkAccessManager>
#include <QNetworkReply>
#include <QNetworkRequest>
```

## Création et utilisation de base

### Initialisation

Typiquement, vous créez une instance de QNetworkAccessManager dans votre classe et vous la conservez comme membre :

```cpp
class MaClasse : public QObject
{
    Q_OBJECT
public:
    MaClasse(QObject *parent = nullptr);

private:
    QNetworkAccessManager *manager;
};

MaClasse::MaClasse(QObject *parent) : QObject(parent)
{
    manager = new QNetworkAccessManager(this);
    // QObject prend en charge la suppression automatique
}
```

### Effectuer une requête GET simple

Voici comment effectuer une requête GET basique :

```cpp
// Créer une requête
QNetworkRequest requete(QUrl("https://api.exemple.com/donnees"));

// Envoyer la requête et obtenir un pointeur vers la réponse
QNetworkReply *reponse = manager->get(requete);

// Connecter le signal finished() pour traiter la réponse
connect(reponse, &QNetworkReply::finished, this, [=]() {
    if(reponse->error() == QNetworkReply::NoError) {
        // Lire les données
        QByteArray donnees = reponse->readAll();
        qDebug() << "Données reçues : " << donnees;
    } else {
        // Gérer l'erreur
        qDebug() << "Erreur : " << reponse->errorString();
    }

    // Important : libérer la mémoire
    reponse->deleteLater();
});
```

## Types de requêtes HTTP

QNetworkAccessManager prend en charge tous les types de requêtes HTTP standard :

### GET - Récupérer des données

```cpp
QNetworkReply *reponse = manager->get(requete);
```

### POST - Envoyer des données

```cpp
QByteArray donnees = "nom=Jean&age=30";
QNetworkReply *reponse = manager->post(requete, donnees);
```

### PUT - Mettre à jour des ressources

```cpp
QByteArray donnees = "{\"nom\": \"Jean\", \"age\": 31}";
QNetworkReply *reponse = manager->put(requete, donnees);
```

### DELETE - Supprimer des ressources

```cpp
QNetworkReply *reponse = manager->deleteResource(requete);
```

## Manipulation des en-têtes HTTP

Pour des requêtes plus avancées, vous devrez souvent personnaliser les en-têtes HTTP :

```cpp
QNetworkRequest requete(QUrl("https://api.exemple.com/donnees"));

// Définir le type de contenu pour JSON
requete.setHeader(QNetworkRequest::ContentTypeHeader, "application/json");

// Ajouter un en-tête d'autorisation
requete.setRawHeader("Authorization", "Bearer votre_token_ici");

// Envoyer la requête
QNetworkReply *reponse = manager->get(requete);
```

## Gestion des réponses

### Traitement asynchrone avec les signaux

La méthode recommandée pour traiter les réponses est d'utiliser les signaux et slots :

```cpp
// Méthode 1: Connecter le signal finished() du QNetworkReply
QNetworkReply *reponse = manager->get(requete);
connect(reponse, &QNetworkReply::finished, this, &MaClasse::traiterReponse);

// Méthode 2: Connecter le signal finished(QNetworkReply*) du QNetworkAccessManager
connect(manager, &QNetworkAccessManager::finished,
        this, &MaClasse::traiterToutesLesReponses);

// Fonction de traitement
void MaClasse::traiterReponse()
{
    // sender() renvoie l'objet qui a émis le signal
    QNetworkReply *reponse = qobject_cast<QNetworkReply*>(sender());
    if (reponse) {
        // Traiter la réponse

        // Libérer la mémoire
        reponse->deleteLater();
    }
}

void MaClasse::traiterToutesLesReponses(QNetworkReply *reponse)
{
    // Traiter la réponse

    // La libération de la mémoire est toujours nécessaire
    reponse->deleteLater();
}
```

### Lecture des données de réponse

```cpp
void MaClasse::traiterReponse()
{
    QNetworkReply *reponse = qobject_cast<QNetworkReply*>(sender());
    if (!reponse) return;

    // Vérifier s'il y a des erreurs
    if (reponse->error() == QNetworkReply::NoError) {
        // Lire toutes les données en une fois
        QByteArray donnees = reponse->readAll();

        // Si la réponse est du JSON, on peut la convertir
        QJsonDocument document = QJsonDocument::fromJson(donnees);

        if (!document.isNull()) {
            if (document.isObject()) {
                QJsonObject objet = document.object();
                qDebug() << "Nom:" << objet["nom"].toString();
            } else if (document.isArray()) {
                QJsonArray tableau = document.array();
                qDebug() << "Nombre d'éléments:" << tableau.size();
            }
        }
    } else {
        // Traiter l'erreur
        qDebug() << "Erreur réseau:" << reponse->errorString();

        // Vérifier le code d'état HTTP
        int statusCode = reponse->attribute(QNetworkRequest::HttpStatusCodeAttribute).toInt();
        qDebug() << "Code HTTP:" << statusCode;
    }

    reponse->deleteLater();
}
```

## Suivi du progrès des téléchargements

Pour les requêtes volumineuses, il est utile de suivre la progression :

```cpp
QNetworkReply *reponse = manager->get(requete);

// Connecter le signal pour suivre la progression
connect(reponse, &QNetworkReply::downloadProgress,
        this, &MaClasse::suivreProgression);

void MaClasse::suivreProgression(qint64 recu, qint64 total)
{
    if (total > 0) {
        // Calculer le pourcentage
        int pourcentage = (recu * 100) / total;
        qDebug() << "Téléchargement:" << pourcentage << "% terminé";

        // Mettre à jour une barre de progression si nécessaire
        // progressBar->setValue(pourcentage);
    } else {
        qDebug() << "Téléchargement en cours... (taille inconnue)";
    }
}
```

## Annulation des requêtes

Vous pouvez annuler une requête en cours :

```cpp
QNetworkReply *reponse = manager->get(requete);

// Plus tard, si vous devez annuler
reponse->abort();
```

## Gestion des redirections HTTP

Par défaut, QNetworkAccessManager gère automatiquement les redirections HTTP. Vous pouvez contrôler ce comportement :

```cpp
// Désactiver le suivi automatique des redirections
QNetworkRequest requete(QUrl("https://api.exemple.com"));
requete.setAttribute(QNetworkRequest::FollowRedirectsAttribute, false);

// Ou avec QNetworkAccessManager (pour toutes les requêtes)
manager->setRedirectPolicy(QNetworkRequest::NoLessSafeRedirectPolicy);
```

## Gestion de l'authentification

QNetworkAccessManager peut gérer l'authentification HTTP automatiquement :

```cpp
// Connecter le signal d'authentification
connect(manager, &QNetworkAccessManager::authenticationRequired,
        this, &MaClasse::fournirAuthentification);

void MaClasse::fournirAuthentification(QNetworkReply *reply, QAuthenticator *authenticator)
{
    // Fournir les identifiants
    authenticator->setUser("nom_utilisateur");
    authenticator->setPassword("mot_de_passe");
}
```

## Gestion de la Cache

QNetworkAccessManager peut utiliser un cache pour stocker les réponses HTTP :

```cpp
// Créer un cache disque
QNetworkDiskCache *cache = new QNetworkDiskCache(this);
cache->setCacheDirectory(QStandardPaths::writableLocation(QStandardPaths::CacheLocation));

// Appliquer le cache au manager
manager->setCache(cache);

// Configurer la politique de cache pour une requête
QNetworkRequest requete(QUrl("https://api.exemple.com/donnees"));
requete.setAttribute(QNetworkRequest::CacheLoadControlAttribute,
                    QNetworkRequest::PreferCache);
```

## Exemple complet

Voici un exemple complet qui montre comment effectuer une requête GET vers une API REST et traiter la réponse JSON :

```cpp
#include <QCoreApplication>
#include <QNetworkAccessManager>
#include <QNetworkReply>
#include <QNetworkRequest>
#include <QJsonDocument>
#include <QJsonObject>
#include <QJsonArray>
#include <QDebug>

class MeteoClient : public QObject
{
    Q_OBJECT
public:
    MeteoClient(QObject *parent = nullptr) : QObject(parent)
    {
        manager = new QNetworkAccessManager(this);

        // Connecter le signal finished
        connect(manager, &QNetworkAccessManager::finished,
                this, &MeteoClient::traiterReponse);
    }

    void obtenirMeteo(const QString &ville)
    {
        QString url = QString("https://api.exemple.com/meteo?ville=%1").arg(ville);
        QNetworkRequest requete(QUrl(url));

        // Ajouter des en-têtes si nécessaire
        requete.setHeader(QNetworkRequest::ContentTypeHeader, "application/json");

        // Envoyer la requête
        manager->get(requete);
    }

private slots:
    void traiterReponse(QNetworkReply *reponse)
    {
        // Vérifier les erreurs
        if (reponse->error() != QNetworkReply::NoError) {
            qDebug() << "Erreur:" << reponse->errorString();
            reponse->deleteLater();
            return;
        }

        // Lire la réponse
        QByteArray donnees = reponse->readAll();

        // Analyser le JSON
        QJsonDocument doc = QJsonDocument::fromJson(donnees);
        if (doc.isNull()) {
            qDebug() << "Erreur: Impossible d'analyser la réponse JSON";
            reponse->deleteLater();
            return;
        }

        // Traiter les données
        QJsonObject meteo = doc.object();

        qDebug() << "Météo pour" << meteo["ville"].toString();
        qDebug() << "Température:" << meteo["temperature"].toDouble() << "°C";
        qDebug() << "Conditions:" << meteo["conditions"].toString();

        // Libérer la mémoire
        reponse->deleteLater();
    }

private:
    QNetworkAccessManager *manager;
};

int main(int argc, char *argv[])
{
    QCoreApplication app(argc, argv);

    MeteoClient client;
    client.obtenirMeteo("Paris");

    return app.exec();
}

#include "main.moc"  // Nécessaire pour Q_OBJECT quand le code est dans le main
```

## Conseils pour les débutants

1. **Toujours libérer la mémoire** : N'oubliez jamais d'appeler `deleteLater()` sur les objets QNetworkReply
2. **Gestion des erreurs** : Vérifiez toujours les erreurs avant de traiter les données
3. **Asynchronisme** : Ne bloquez jamais en attendant une réponse, utilisez les signaux et slots
4. **Timeout** : Définissez des délais d'attente pour éviter que votre application ne reste bloquée
5. **SSL/TLS** : Pour les requêtes HTTPS, assurez-vous que votre application Qt a été compilée avec la prise en charge SSL

## Conclusion

QNetworkAccessManager est une classe puissante qui simplifie considérablement les communications réseau dans vos applications Qt. En suivant les principes asynchrones de Qt, vous pouvez créer des applications réseau réactives et robustes.

À mesure que vous vous familiariserez avec cette API, vous découvrirez qu'elle peut gérer des scénarios de plus en plus complexes, de l'authentification à la mise en cache, en passant par le téléchargement de fichiers volumineux.

⏭️ [WebSockets avec Qt](/05-communication-reseau/02-websockets-avec-qt.md)
