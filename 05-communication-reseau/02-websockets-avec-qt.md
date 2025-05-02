# 5.2 WebSockets avec Qt

## Introduction aux WebSockets

Les WebSockets représentent une technologie de communication bidirectionnelle qui permet d'établir une connexion persistante entre un client et un serveur. Contrairement au HTTP traditionnel, où chaque requête nécessite l'établissement d'une nouvelle connexion, les WebSockets maintiennent une connexion ouverte, permettant ainsi l'échange de données en temps réel dans les deux sens.

### Avantages des WebSockets

- **Communication bidirectionnelle** : Les données peuvent circuler dans les deux sens à tout moment
- **Temps réel** : Transfert de données avec une latence minimale
- **Efficacité** : Moins de surcharge réseau comparé aux requêtes HTTP répétées
- **Persistance** : La connexion reste active jusqu'à ce qu'elle soit explicitement fermée

## Le module Qt WebSockets

Qt fournit un module dédié aux WebSockets qui simplifie considérablement le développement d'applications utilisant cette technologie. Ce module est disponible à partir de Qt 5.3 et a été amélioré dans Qt6.

### Configuration du projet

Pour utiliser les WebSockets dans votre projet Qt, vous devez d'abord ajouter le module WebSockets à votre fichier `.pro` :

```
QT += websockets
```

Et dans vos fichiers source, incluez les en-têtes nécessaires :

```cpp
#include <QWebSocket>               // Pour les clients
#include <QWebSocketServer>         // Pour les serveurs
```

## Client WebSocket

Voyons comment créer un client WebSocket simple.

### Création d'un client

```cpp
#include <QWebSocket>
#include <QDebug>

class SimpleClient : public QObject
{
    Q_OBJECT
public:
    SimpleClient(QObject *parent = nullptr);
    void connecterAuServeur(const QUrl &url);

private slots:
    void onConnected();
    void onTextMessageReceived(const QString &message);
    void onBinaryMessageReceived(const QByteArray &message);
    void onDisconnected();
    void onError(QAbstractSocket::SocketError error);

private:
    QWebSocket m_webSocket;
};

SimpleClient::SimpleClient(QObject *parent) : QObject(parent)
{
    // Connexion des signaux et slots
    connect(&m_webSocket, &QWebSocket::connected, this, &SimpleClient::onConnected);
    connect(&m_webSocket, &QWebSocket::disconnected, this, &SimpleClient::onDisconnected);
    connect(&m_webSocket, &QWebSocket::textMessageReceived,
            this, &SimpleClient::onTextMessageReceived);
    connect(&m_webSocket, &QWebSocket::binaryMessageReceived,
            this, &SimpleClient::onBinaryMessageReceived);
    connect(&m_webSocket,
            QOverload<QAbstractSocket::SocketError>::of(&QWebSocket::error),
            this, &SimpleClient::onError);
}

void SimpleClient::connecterAuServeur(const QUrl &url)
{
    qDebug() << "Connexion au serveur:" << url;
    m_webSocket.open(url);
}

void SimpleClient::onConnected()
{
    qDebug() << "WebSocket connecté";

    // Envoyer un message au serveur
    m_webSocket.sendTextMessage("Bonjour depuis le client Qt!");
}

void SimpleClient::onTextMessageReceived(const QString &message)
{
    qDebug() << "Message reçu:" << message;
}

void SimpleClient::onBinaryMessageReceived(const QByteArray &message)
{
    qDebug() << "Message binaire reçu, taille:" << message.size() << "octets";
}

void SimpleClient::onDisconnected()
{
    qDebug() << "WebSocket déconnecté";
}

void SimpleClient::onError(QAbstractSocket::SocketError error)
{
    qDebug() << "Erreur:" << m_webSocket.errorString();
}
```

### Utilisation du client

```cpp
int main(int argc, char *argv[])
{
    QCoreApplication app(argc, argv);

    SimpleClient client;
    client.connecterAuServeur(QUrl("ws://exemple.com:8080"));

    return app.exec();
}
```

## Serveur WebSocket

Maintenant, voyons comment créer un serveur WebSocket.

### Création d'un serveur

```cpp
#include <QWebSocketServer>
#include <QWebSocket>
#include <QDebug>

class SimpleServeur : public QObject
{
    Q_OBJECT
public:
    SimpleServeur(quint16 port, QObject *parent = nullptr);
    ~SimpleServeur();

private slots:
    void onNewConnection();
    void onTextMessageReceived(const QString &message);
    void onDisconnected();

private:
    QWebSocketServer *m_serveur;
    QList<QWebSocket *> m_clients;
};

SimpleServeur::SimpleServeur(quint16 port, QObject *parent) : QObject(parent)
{
    // Créer le serveur en mode non-sécurisé (ws://)
    m_serveur = new QWebSocketServer(
        "Serveur Simple",
        QWebSocketServer::NonSecureMode,
        this
    );

    // Démarrer le serveur sur le port spécifié
    if (m_serveur->listen(QHostAddress::Any, port)) {
        qDebug() << "Serveur démarré sur le port" << port;

        // Connecter le signal de nouvelle connexion
        connect(m_serveur, &QWebSocketServer::newConnection,
                this, &SimpleServeur::onNewConnection);
    } else {
        qDebug() << "Erreur: Impossible de démarrer le serveur";
    }
}

SimpleServeur::~SimpleServeur()
{
    // Fermer le serveur et libérer les ressources
    m_serveur->close();

    // Nettoyer les clients
    qDeleteAll(m_clients.begin(), m_clients.end());
}

void SimpleServeur::onNewConnection()
{
    // Récupérer le socket du nouveau client
    QWebSocket *socket = m_serveur->nextPendingConnection();

    qDebug() << "Nouveau client connecté:" << socket->peerAddress().toString();

    // Connecter les signaux du client
    connect(socket, &QWebSocket::textMessageReceived,
            this, &SimpleServeur::onTextMessageReceived);
    connect(socket, &QWebSocket::disconnected,
            this, &SimpleServeur::onDisconnected);

    // Ajouter le client à la liste
    m_clients.append(socket);

    // Envoyer un message de bienvenue
    socket->sendTextMessage("Bienvenue sur le serveur WebSocket Qt!");
}

void SimpleServeur::onTextMessageReceived(const QString &message)
{
    // Identifier le client qui a envoyé le message
    QWebSocket *client = qobject_cast<QWebSocket *>(sender());
    if (!client) return;

    qDebug() << "Message reçu de" << client->peerAddress().toString() << ":" << message;

    // Exemple: répondre au client
    client->sendTextMessage("Vous avez dit: " + message);

    // Exemple: diffuser le message à tous les clients (chat)
    for (QWebSocket *clientSocket : m_clients) {
        if (clientSocket != client) { // Ne pas renvoyer au client d'origine
            clientSocket->sendTextMessage(
                client->peerAddress().toString() + " dit: " + message
            );
        }
    }
}

void SimpleServeur::onDisconnected()
{
    // Identifier le client qui s'est déconnecté
    QWebSocket *client = qobject_cast<QWebSocket *>(sender());
    if (!client) return;

    qDebug() << "Client déconnecté:" << client->peerAddress().toString();

    // Supprimer le client de la liste
    m_clients.removeAll(client);

    // Libérer la mémoire
    client->deleteLater();
}
```

### Utilisation du serveur

```cpp
int main(int argc, char *argv[])
{
    QCoreApplication app(argc, argv);

    // Démarrer le serveur sur le port 8080
    SimpleServeur serveur(8080);

    return app.exec();
}
```

## WebSockets sécurisés (WSS)

Pour des communications sécurisées, vous pouvez utiliser WebSockets avec SSL/TLS (wss:// au lieu de ws://).

### Configuration d'un serveur sécurisé

```cpp
// Créer un serveur en mode sécurisé
m_serveur = new QWebSocketServer(
    "Serveur Sécurisé",
    QWebSocketServer::SecureMode,
    this
);

// Configurer SSL
QSslConfiguration sslConfiguration;
QFile certFile(":/certificat.pem");
QFile keyFile(":/cle.pem");
certFile.open(QIODevice::ReadOnly);
keyFile.open(QIODevice::ReadOnly);

QSslCertificate certificate(&certFile, QSsl::Pem);
QSslKey sslKey(&keyFile, QSsl::Rsa, QSsl::Pem);
certFile.close();
keyFile.close();

sslConfiguration.setPeerVerifyMode(QSslSocket::VerifyNone);
sslConfiguration.setLocalCertificate(certificate);
sslConfiguration.setPrivateKey(sslKey);

m_serveur->setSslConfiguration(sslConfiguration);
```

## Cas d'utilisation courants

Les WebSockets sont particulièrement utiles pour plusieurs types d'applications :

### 1. Applications de chat

Le code serveur ci-dessus illustre déjà un mécanisme de chat simple. Chaque message reçu est diffusé à tous les autres clients.

### 2. Tableaux de bord en temps réel

```cpp
// Côté serveur - Envoi périodique de mises à jour
QTimer *timer = new QTimer(this);
connect(timer, &QTimer::timeout, this, [this]() {
    // Créer des données de mise à jour (par exemple, des statistiques)
    QJsonObject stats;
    stats["utilisateurs"] = 1250;
    stats["messages"] = 8720;
    stats["serveurs"] = 5;

    QJsonDocument doc(stats);
    QString message = doc.toJson();

    // Envoyer à tous les clients
    for (QWebSocket *client : m_clients) {
        client->sendTextMessage(message);
    }
});
timer->start(5000); // Mise à jour toutes les 5 secondes
```

### 3. Jeux multijoueurs

```cpp
// Exemple simplifié - envoi de position de joueur
void envoyerPosition(QWebSocket *client, int x, int y)
{
    QJsonObject position;
    position["type"] = "position";
    position["x"] = x;
    position["y"] = y;
    position["joueur"] = client->property("identifiant").toString();

    QJsonDocument doc(position);
    QString message = doc.toJson();

    // Envoyer à tous les joueurs
    for (QWebSocket *autreClient : m_clients) {
        autreClient->sendTextMessage(message);
    }
}
```

## Gestion des erreurs et des déconnexions

La gestion robuste des erreurs est essentielle pour les applications WebSocket :

```cpp
// Dans la classe client
connect(&m_webSocket, &QWebSocket::disconnected, this, [this]() {
    qDebug() << "Déconnecté du serveur";

    // Tentative de reconnexion automatique
    QTimer::singleShot(5000, this, [this]() {
        qDebug() << "Tentative de reconnexion...";
        m_webSocket.open(m_url);
    });
});
```

## Bonnes pratiques

1. **Reconnexion automatique** : Implémentez une logique de reconnexion pour gérer les déconnexions inattendues
2. **Ping/Pong** : Utilisez les messages de ping pour vérifier que la connexion est toujours active
   ```cpp
   // Envoi périodique de ping
   QTimer *pingTimer = new QTimer(this);
   connect(pingTimer, &QTimer::timeout, this, [this]() {
       m_webSocket.ping();
   });
   pingTimer->start(30000); // Ping toutes les 30 secondes
   ```
3. **Format des messages** : Utilisez JSON pour structurer vos messages et faciliter leur traitement
4. **Gestion des états** : Maintenez un état clair de la connexion (connecté, déconnecté, en reconnexion)
5. **Sécurité** : Utilisez WSS (WebSockets Sécurisés) pour les données sensibles

## Débogage des WebSockets

Qt Creator facilite le débogage des WebSockets avec ses outils de journalisation. Vous pouvez activer la journalisation détaillée des WebSockets :

```cpp
// Activer la journalisation détaillée
QLoggingCategory::setFilterRules("qt.network.ssl.warning=false");
QLoggingCategory::setFilterRules("qt.websockets.debug=true");
```

## Exemple d'application complète

Voici un exemple plus complet d'une application de chat très simple :

```cpp
// chatclient.h
#pragma once
#include <QObject>
#include <QWebSocket>

class ChatClient : public QObject
{
    Q_OBJECT
public:
    explicit ChatClient(QObject *parent = nullptr);
    void connecter(const QString &url, const QString &pseudo);
    void envoyerMessage(const QString &message);
    void deconnecter();

signals:
    void messageRecu(const QString &expediteur, const QString &message);
    void connecte();
    void deconnecte();
    void erreur(const QString &message);

private slots:
    void onConnected();
    void onTextMessageReceived(const QString &message);
    void onDisconnected();
    void onError(QAbstractSocket::SocketError error);

private:
    QWebSocket m_webSocket;
    QString m_pseudo;
};

// chatclient.cpp
#include "chatclient.h"
#include <QJsonDocument>
#include <QJsonObject>

ChatClient::ChatClient(QObject *parent) : QObject(parent)
{
    connect(&m_webSocket, &QWebSocket::connected, this, &ChatClient::onConnected);
    connect(&m_webSocket, &QWebSocket::disconnected, this, &ChatClient::onDisconnected);
    connect(&m_webSocket, &QWebSocket::textMessageReceived,
            this, &ChatClient::onTextMessageReceived);
    connect(&m_webSocket,
            QOverload<QAbstractSocket::SocketError>::of(&QWebSocket::error),
            this, &ChatClient::onError);
}

void ChatClient::connecter(const QString &url, const QString &pseudo)
{
    m_pseudo = pseudo;
    m_webSocket.open(QUrl(url));
}

void ChatClient::envoyerMessage(const QString &message)
{
    if (m_webSocket.state() != QAbstractSocket::ConnectedState)
        return;

    QJsonObject json;
    json["type"] = "message";
    json["pseudo"] = m_pseudo;
    json["contenu"] = message;

    QJsonDocument doc(json);
    m_webSocket.sendTextMessage(doc.toJson());
}

void ChatClient::deconnecter()
{
    m_webSocket.close();
}

void ChatClient::onConnected()
{
    QJsonObject json;
    json["type"] = "connexion";
    json["pseudo"] = m_pseudo;

    QJsonDocument doc(json);
    m_webSocket.sendTextMessage(doc.toJson());

    emit connecte();
}

void ChatClient::onTextMessageReceived(const QString &message)
{
    QJsonDocument doc = QJsonDocument::fromJson(message.toUtf8());
    if (!doc.isObject())
        return;

    QJsonObject json = doc.object();

    if (json["type"].toString() == "message") {
        QString expediteur = json["pseudo"].toString();
        QString contenu = json["contenu"].toString();

        emit messageRecu(expediteur, contenu);
    }
}

void ChatClient::onDisconnected()
{
    emit deconnecte();
}

void ChatClient::onError(QAbstractSocket::SocketError error)
{
    emit erreur(m_webSocket.errorString());
}
```

Et voici comment utiliser ce client de chat dans une interface Qt Widgets :

```cpp
// mainwindow.h
#pragma once
#include <QMainWindow>
#include "chatclient.h"

namespace Ui {
class MainWindow;
}

class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    explicit MainWindow(QWidget *parent = nullptr);
    ~MainWindow();

private slots:
    void onConnecterClicked();
    void onEnvoyerClicked();
    void onMessageRecu(const QString &expediteur, const QString &message);
    void onConnecte();
    void onDeconnecte();
    void onErreur(const QString &message);

private:
    Ui::MainWindow *ui;
    ChatClient *m_client;
};

// mainwindow.cpp
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include <QMessageBox>

MainWindow::MainWindow(QWidget *parent) :
    QMainWindow(parent),
    ui(new Ui::MainWindow),
    m_client(new ChatClient(this))
{
    ui->setupUi(this);

    connect(ui->connecterButton, &QPushButton::clicked,
            this, &MainWindow::onConnecterClicked);
    connect(ui->envoyerButton, &QPushButton::clicked,
            this, &MainWindow::onEnvoyerClicked);

    connect(m_client, &ChatClient::messageRecu,
            this, &MainWindow::onMessageRecu);
    connect(m_client, &ChatClient::connecte,
            this, &MainWindow::onConnecte);
    connect(m_client, &ChatClient::deconnecte,
            this, &MainWindow::onDeconnecte);
    connect(m_client, &ChatClient::erreur,
            this, &MainWindow::onErreur);

    // Désactiver les contrôles jusqu'à la connexion
    ui->chatTextEdit->setEnabled(false);
    ui->envoyerButton->setEnabled(false);
}

MainWindow::~MainWindow()
{
    delete ui;
}

void MainWindow::onConnecterClicked()
{
    if (ui->connecterButton->text() == "Connecter") {
        QString url = ui->urlLineEdit->text();
        QString pseudo = ui->pseudoLineEdit->text();

        if (url.isEmpty() || pseudo.isEmpty()) {
            QMessageBox::warning(this, "Erreur", "L'URL et le pseudo sont requis");
            return;
        }

        m_client->connecter(url, pseudo);
        ui->connecterButton->setEnabled(false);
    } else {
        m_client->deconnecter();
    }
}

void MainWindow::onEnvoyerClicked()
{
    QString message = ui->messageLineEdit->text();
    if (!message.isEmpty()) {
        m_client->envoyerMessage(message);
        ui->messageLineEdit->clear();
    }
}

void MainWindow::onMessageRecu(const QString &expediteur, const QString &message)
{
    ui->chatTextEdit->append("<b>" + expediteur + ":</b> " + message);
}

void MainWindow::onConnecte()
{
    ui->statutLabel->setText("Connecté");
    ui->connecterButton->setText("Déconnecter");
    ui->connecterButton->setEnabled(true);
    ui->chatTextEdit->setEnabled(true);
    ui->envoyerButton->setEnabled(true);
    ui->urlLineEdit->setEnabled(false);
    ui->pseudoLineEdit->setEnabled(false);
}

void MainWindow::onDeconnecte()
{
    ui->statutLabel->setText("Déconnecté");
    ui->connecterButton->setText("Connecter");
    ui->connecterButton->setEnabled(true);
    ui->chatTextEdit->setEnabled(false);
    ui->envoyerButton->setEnabled(false);
    ui->urlLineEdit->setEnabled(true);
    ui->pseudoLineEdit->setEnabled(true);
}

void MainWindow::onErreur(const QString &message)
{
    QMessageBox::critical(this, "Erreur", message);
    onDeconnecte();
}
```

## Conclusion

Les WebSockets offrent un moyen puissant et efficace de créer des applications interactives en temps réel. Grâce au module Qt WebSockets, l'implémentation de clients et de serveurs WebSocket devient accessible même aux débutants.

En comprenant les concepts de base présentés dans ce tutoriel, vous êtes maintenant prêt à intégrer des communications WebSocket dans vos applications Qt pour créer des expériences utilisateur dynamiques et réactives.

Pour aller plus loin, explorez la documentation officielle de Qt sur les WebSockets et expérimentez avec différents types d'applications comme les tableaux de bord en temps réel, les jeux multijoueurs ou les systèmes de messagerie instantanée.
