# 5.4 Bluetooth et NFC avec Qt

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

## Introduction aux technologies sans fil dans Qt

Les technologies sans fil comme Bluetooth et NFC (Near Field Communication) sont essentielles pour de nombreuses applications modernes. Qt propose des modules dédiés qui simplifient grandement le développement d'applications utilisant ces technologies. Ce chapitre vous guidera à travers les bases de la programmation Bluetooth et NFC avec Qt.

## Première partie : Bluetooth avec Qt

### Qu'est-ce que Bluetooth ?

Bluetooth est une technologie de communication sans fil à courte portée qui permet l'échange de données entre appareils. Elle est utilisée dans de nombreux contextes, des écouteurs sans fil aux voitures connectées, en passant par les applications IoT (Internet des Objets).

Qt prend en charge deux versions principales de Bluetooth :
- **Bluetooth classique** : pour les transferts de données plus volumineux
- **Bluetooth Low Energy (BLE)** : optimisé pour la faible consommation d'énergie

### Configuration du projet pour Bluetooth

Pour utiliser Bluetooth dans votre projet Qt, vous devez d'abord ajouter le module Bluetooth à votre fichier `.pro` :

```
QT += bluetooth
```

Et dans vos fichiers source, incluez les en-têtes nécessaires :

```cpp
// Pour Bluetooth classique
#include <QBluetoothLocalDevice>
#include <QBluetoothDeviceDiscoveryAgent>
#include <QBluetoothSocket>

// Pour Bluetooth Low Energy
#include <QLowEnergyController>
#include <QLowEnergyService>
#include <QLowEnergyCharacteristic>
```

### Vérification de la disponibilité Bluetooth

Avant toute opération, il est essentiel de vérifier si Bluetooth est disponible sur l'appareil de l'utilisateur :

```cpp
bool verifierBluetoothDisponible()
{
    QBluetoothLocalDevice localDevice;
    if (!localDevice.isValid()) {
        qDebug() << "Bluetooth n'est pas disponible sur cet appareil";
        return false;
    }

    // Vérifier si Bluetooth est activé
    if (localDevice.hostMode() == QBluetoothLocalDevice::HostPoweredOff) {
        qDebug() << "Bluetooth est désactivé";
        return false;
    }

    qDebug() << "Bluetooth est disponible et activé";
    return true;
}
```

### Recherche d'appareils Bluetooth

La première étape pour utiliser Bluetooth est souvent de rechercher les appareils disponibles à proximité :

```cpp
class BluetoothScanner : public QObject
{
    Q_OBJECT

public:
    explicit BluetoothScanner(QObject *parent = nullptr) : QObject(parent)
    {
        // Créer l'agent de découverte
        m_discoveryAgent = new QBluetoothDeviceDiscoveryAgent(this);

        // Connecter les signaux
        connect(m_discoveryAgent, &QBluetoothDeviceDiscoveryAgent::deviceDiscovered,
                this, &BluetoothScanner::appareilDecouvert);
        connect(m_discoveryAgent, &QBluetoothDeviceDiscoveryAgent::finished,
                this, &BluetoothScanner::rechercheTerminee);
        connect(m_discoveryAgent, QOverload<QBluetoothDeviceDiscoveryAgent::Error>::of(&QBluetoothDeviceDiscoveryAgent::error),
                this, &BluetoothScanner::erreurRecherche);
    }

    void demarrerRecherche()
    {
        qDebug() << "Démarrage de la recherche d'appareils Bluetooth...";
        m_appareils.clear();

        // Démarrer la recherche
        m_discoveryAgent->start();
    }

    QList<QBluetoothDeviceInfo> appareilsDecouverts() const
    {
        return m_appareils;
    }

private slots:
    void appareilDecouvert(const QBluetoothDeviceInfo &info)
    {
        qDebug() << "Appareil découvert:" << info.name() << "(" << info.address().toString() << ")";
        m_appareils.append(info);

        emit appareilTrouve(info);
    }

    void rechercheTerminee()
    {
        qDebug() << "Recherche terminée," << m_appareils.size() << "appareils trouvés";
        emit rechercheFinie(m_appareils);
    }

    void erreurRecherche(QBluetoothDeviceDiscoveryAgent::Error error)
    {
        qDebug() << "Erreur lors de la recherche:" << m_discoveryAgent->errorString();
        emit erreur(m_discoveryAgent->errorString());
    }

signals:
    void appareilTrouve(const QBluetoothDeviceInfo &info);
    void rechercheFinie(const QList<QBluetoothDeviceInfo> &appareils);
    void erreur(const QString &message);

private:
    QBluetoothDeviceDiscoveryAgent *m_discoveryAgent;
    QList<QBluetoothDeviceInfo> m_appareils;
};
```

### Connexion à un appareil Bluetooth classique

Une fois que vous avez trouvé un appareil, vous pouvez vous y connecter en utilisant un socket Bluetooth :

```cpp
class BluetoothClient : public QObject
{
    Q_OBJECT

public:
    explicit BluetoothClient(QObject *parent = nullptr) : QObject(parent)
    {
        // Créer le socket
        m_socket = new QBluetoothSocket(QBluetoothServiceInfo::RfcommProtocol, this);

        // Connecter les signaux
        connect(m_socket, &QBluetoothSocket::connected,
                this, &BluetoothClient::connexionReussie);
        connect(m_socket, &QBluetoothSocket::disconnected,
                this, &BluetoothClient::deconnexion);
        connect(m_socket, &QBluetoothSocket::readyRead,
                this, &BluetoothClient::donneesRecues);
        connect(m_socket, QOverload<QBluetoothSocket::SocketError>::of(&QBluetoothSocket::error),
                this, &BluetoothClient::erreurSocket);
    }

    void connecterA(const QBluetoothDeviceInfo &appareil, const QBluetoothUuid &service)
    {
        qDebug() << "Tentative de connexion à" << appareil.name();

        // Se connecter au service spécifié sur l'appareil
        m_socket->connectToService(appareil.address(), service);
    }

    void envoyerDonnees(const QByteArray &donnees)
    {
        if (m_socket->state() == QBluetoothSocket::ConnectedState) {
            m_socket->write(donnees);
        } else {
            qDebug() << "Impossible d'envoyer des données : non connecté";
        }
    }

    void deconnecter()
    {
        m_socket->disconnectFromService();
    }

private slots:
    void connexionReussie()
    {
        qDebug() << "Connecté à l'appareil Bluetooth";
        emit connecte();
    }

    void deconnexion()
    {
        qDebug() << "Déconnecté de l'appareil Bluetooth";
        emit deconnecte();
    }

    void donneesRecues()
    {
        QByteArray donnees = m_socket->readAll();
        qDebug() << "Données reçues:" << donnees;

        emit reception(donnees);
    }

    void erreurSocket(QBluetoothSocket::SocketError error)
    {
        qDebug() << "Erreur socket:" << m_socket->errorString();
        emit erreur(m_socket->errorString());
    }

signals:
    void connecte();
    void deconnecte();
    void reception(const QByteArray &donnees);
    void erreur(const QString &message);

private:
    QBluetoothSocket *m_socket;
};
```

### Recherche de services Bluetooth

Avant de vous connecter à un appareil, vous devez souvent découvrir quels services il propose :

```cpp
class ServiceDiscovery : public QObject
{
    Q_OBJECT

public:
    explicit ServiceDiscovery(QObject *parent = nullptr) : QObject(parent)
    {
        m_discoveryAgent = nullptr;
    }

    void rechercherServices(const QBluetoothDeviceInfo &appareil)
    {
        // Créer un agent de découverte pour l'appareil spécifié
        if (m_discoveryAgent) {
            delete m_discoveryAgent;
        }

        m_discoveryAgent = new QBluetoothServiceDiscoveryAgent(appareil.address(), this);

        // Connecter les signaux
        connect(m_discoveryAgent, &QBluetoothServiceDiscoveryAgent::serviceDiscovered,
                this, &ServiceDiscovery::serviceDecouvert);
        connect(m_discoveryAgent, &QBluetoothServiceDiscoveryAgent::finished,
                this, &ServiceDiscovery::rechercheTerminee);
        connect(m_discoveryAgent, QOverload<QBluetoothServiceDiscoveryAgent::Error>::of(&QBluetoothServiceDiscoveryAgent::error),
                this, &ServiceDiscovery::erreurRecherche);

        // Démarrer la recherche
        m_discoveryAgent->start();
    }

private slots:
    void serviceDecouvert(const QBluetoothServiceInfo &info)
    {
        qDebug() << "Service découvert:" << info.serviceName();
        emit serviceTrouve(info);
    }

    void rechercheTerminee()
    {
        qDebug() << "Recherche de services terminée";
        emit rechercheFinie();
    }

    void erreurRecherche(QBluetoothServiceDiscoveryAgent::Error error)
    {
        qDebug() << "Erreur lors de la recherche de services:" << m_discoveryAgent->errorString();
        emit erreur(m_discoveryAgent->errorString());
    }

signals:
    void serviceTrouve(const QBluetoothServiceInfo &info);
    void rechercheFinie();
    void erreur(const QString &message);

private:
    QBluetoothServiceDiscoveryAgent *m_discoveryAgent;
};
```

### Création d'un serveur Bluetooth

Si vous souhaitez que votre application accepte des connexions Bluetooth entrantes, vous devez créer un serveur :

```cpp
class BluetoothServer : public QObject
{
    Q_OBJECT

public:
    explicit BluetoothServer(QObject *parent = nullptr) : QObject(parent)
    {
        // Créer le serveur
        m_server = new QBluetoothServer(QBluetoothServiceInfo::RfcommProtocol, this);

        // Connecter les signaux
        connect(m_server, &QBluetoothServer::newConnection,
                this, &BluetoothServer::nouvelleConnexion);
    }

    bool demarrer()
    {
        // Démarrer le serveur
        if (!m_server->listen()) {
            qDebug() << "Impossible de démarrer le serveur Bluetooth:" << m_server->error();
            return false;
        }

        // Créer un service qui sera visible pour les autres appareils
        QBluetoothServiceInfo serviceInfo;

        // Nom et description du service
        serviceInfo.setAttribute(QBluetoothServiceInfo::ServiceName, "Qt Bluetooth Server");
        serviceInfo.setAttribute(QBluetoothServiceInfo::ServiceDescription, "Exemple de serveur Bluetooth avec Qt");
        serviceInfo.setAttribute(QBluetoothServiceInfo::ServiceProvider, "QtProject");

        // UUID du service (généré aléatoirement ici)
        QBluetoothUuid uuid = QBluetoothUuid::createUuid();
        serviceInfo.setServiceUuid(uuid);

        // Définir le type de protocole
        QBluetoothServiceInfo::Sequence protocolDescriptorList;
        QBluetoothServiceInfo::Sequence protocol;
        protocol << QVariant::fromValue(QBluetoothUuid(QBluetoothUuid::Rfcomm))
                 << QVariant::fromValue(quint8(m_server->serverPort()));
        protocolDescriptorList.append(QVariant::fromValue(protocol));
        serviceInfo.setAttribute(QBluetoothServiceInfo::ProtocolDescriptorList, protocolDescriptorList);

        // Rendre le service disponible
        if (!serviceInfo.registerService()) {
            qDebug() << "Impossible d'enregistrer le service Bluetooth";
            return false;
        }

        m_serviceInfo = serviceInfo;
        qDebug() << "Serveur Bluetooth démarré, en attente de connexions...";
        return true;
    }

    void arreter()
    {
        // Désinscrire le service
        m_serviceInfo.unregisterService();

        // Fermer toutes les connexions
        for (QBluetoothSocket *socket : m_clients) {
            socket->disconnectFromService();
            socket->deleteLater();
        }
        m_clients.clear();

        // Arrêter le serveur
        m_server->close();
    }

    void diffuserMessage(const QByteArray &message)
    {
        // Envoyer le message à tous les clients connectés
        for (QBluetoothSocket *socket : m_clients) {
            socket->write(message);
        }
    }

private slots:
    void nouvelleConnexion()
    {
        // Accepter la connexion entrante
        QBluetoothSocket *socket = m_server->nextPendingConnection();
        if (!socket) {
            return;
        }

        // Connecter les signaux pour ce client
        connect(socket, &QBluetoothSocket::readyRead, this, [=]() {
            // Lire les données envoyées par le client
            QByteArray data = socket->readAll();
            qDebug() << "Données reçues de" << socket->peerName() << ":" << data;

            emit messageRecu(socket->peerAddress(), data);
        });

        connect(socket, &QBluetoothSocket::disconnected, this, [=]() {
            qDebug() << "Client déconnecté:" << socket->peerName();

            // Supprimer le client de la liste
            m_clients.removeAll(socket);
            socket->deleteLater();

            emit clientDeconnecte(socket->peerAddress());
        });

        // Ajouter le client à la liste
        m_clients.append(socket);

        qDebug() << "Nouveau client connecté:" << socket->peerName();
        emit clientConnecte(socket->peerAddress());
    }

signals:
    void clientConnecte(const QBluetoothAddress &adresse);
    void clientDeconnecte(const QBluetoothAddress &adresse);
    void messageRecu(const QBluetoothAddress &adresse, const QByteArray &message);

private:
    QBluetoothServer *m_server;
    QBluetoothServiceInfo m_serviceInfo;
    QList<QBluetoothSocket *> m_clients;
};
```

### Bluetooth Low Energy (BLE)

Le Bluetooth Low Energy est idéal pour les appareils à faible consommation d'énergie. Son fonctionnement est différent du Bluetooth classique :

```cpp
class BLEScanner : public QObject
{
    Q_OBJECT

public:
    explicit BLEScanner(QObject *parent = nullptr) : QObject(parent)
    {
        m_discoveryAgent = new QBluetoothDeviceDiscoveryAgent(this);
        m_discoveryAgent->setLowEnergyDiscoveryTimeout(5000); // 5 secondes

        // Configuration pour ne rechercher que les appareils BLE
        QBluetoothDeviceDiscoveryAgent::DiscoveryMethods methods;
        methods |= QBluetoothDeviceDiscoveryAgent::LowEnergyMethod;
        m_discoveryAgent->setDiscoveryMethods(methods);

        connect(m_discoveryAgent, &QBluetoothDeviceDiscoveryAgent::deviceDiscovered,
                this, &BLEScanner::appareilDecouvert);
        connect(m_discoveryAgent, &QBluetoothDeviceDiscoveryAgent::finished,
                this, &BLEScanner::rechercheTerminee);
    }

    void demarrerRecherche()
    {
        m_appareilsBLE.clear();
        m_discoveryAgent->start();
    }

private slots:
    void appareilDecouvert(const QBluetoothDeviceInfo &info)
    {
        // Vérifier si c'est un appareil BLE
        if (info.coreConfigurations() & QBluetoothDeviceInfo::LowEnergyCoreConfiguration) {
            qDebug() << "Appareil BLE découvert:" << info.name() << "(" << info.address().toString() << ")";
            m_appareilsBLE.append(info);

            emit appareilTrouve(info);
        }
    }

    void rechercheTerminee()
    {
        qDebug() << "Recherche BLE terminée," << m_appareilsBLE.size() << "appareils trouvés";
        emit rechercheFinie(m_appareilsBLE);
    }

signals:
    void appareilTrouve(const QBluetoothDeviceInfo &info);
    void rechercheFinie(const QList<QBluetoothDeviceInfo> &appareils);

private:
    QBluetoothDeviceDiscoveryAgent *m_discoveryAgent;
    QList<QBluetoothDeviceInfo> m_appareilsBLE;
};
```

### Connexion à un appareil BLE

Pour interagir avec un appareil BLE, vous utilisez un contrôleur BLE :

```cpp
class BLEController : public QObject
{
    Q_OBJECT

public:
    explicit BLEController(QObject *parent = nullptr) : QObject(parent),
        m_controller(nullptr), m_service(nullptr)
    {
    }

    void connecterA(const QBluetoothDeviceInfo &appareil)
    {
        // Créer un contrôleur pour l'appareil
        m_controller = QLowEnergyController::createCentral(appareil, this);

        // Connecter les signaux
        connect(m_controller, &QLowEnergyController::connected,
                this, &BLEController::appConnected);
        connect(m_controller, &QLowEnergyController::disconnected,
                this, &BLEController::appDisconnected);
        connect(m_controller, QOverload<QLowEnergyController::Error>::of(&QLowEnergyController::error),
                this, &BLEController::controllerError);
        connect(m_controller, &QLowEnergyController::serviceDiscovered,
                this, &BLEController::serviceDiscovered);
        connect(m_controller, &QLowEnergyController::discoveryFinished,
                this, &BLEController::serviceScanFinished);

        // Connecter à l'appareil
        m_controller->connectToDevice();
    }

    void deconnecter()
    {
        if (m_controller) {
            m_controller->disconnectFromDevice();
        }
    }

private slots:
    void appConnected()
    {
        qDebug() << "Connecté à l'appareil BLE";
        emit connecte();

        // Démarrer la découverte des services
        m_controller->discoverServices();
    }

    void appDisconnected()
    {
        qDebug() << "Déconnecté de l'appareil BLE";
        emit deconnecte();
    }

    void controllerError(QLowEnergyController::Error error)
    {
        qDebug() << "Erreur contrôleur BLE:" << m_controller->errorString();
        emit erreur(m_controller->errorString());
    }

    void serviceDiscovered(const QBluetoothUuid &uuid)
    {
        qDebug() << "Service BLE découvert:" << uuid.toString();
        emit serviceTrouve(uuid);
    }

    void serviceScanFinished()
    {
        qDebug() << "Scan des services BLE terminé";
        emit scanServicesFini();
    }

    void connecterAuService(const QBluetoothUuid &uuid)
    {
        m_service = m_controller->createServiceObject(uuid, this);
        if (!m_service) {
            qDebug() << "Impossible de créer l'objet service";
            return;
        }

        connect(m_service, &QLowEnergyService::stateChanged,
                this, &BLEController::serviceStateChanged);
        connect(m_service, &QLowEnergyService::characteristicChanged,
                this, &BLEController::characteristicChanged);
        connect(m_service, &QLowEnergyService::characteristicRead,
                this, &BLEController::characteristicRead);
        connect(m_service, &QLowEnergyService::characteristicWritten,
                this, &BLEController::characteristicWritten);

        m_service->discoverDetails();
    }

    void serviceStateChanged(QLowEnergyService::ServiceState newState)
    {
        if (newState == QLowEnergyService::ServiceDiscovered) {
            qDebug() << "Détails du service découverts";

            // Parcourir les caractéristiques du service
            for (const QLowEnergyCharacteristic &characteristic : m_service->characteristics()) {
                qDebug() << "Caractéristique découverte:" << characteristic.uuid().toString();
                qDebug() << "  Propriétés:" << characteristic.properties();

                emit caracteristiqueTrouvee(characteristic);
            }
        }
    }

    void characteristicChanged(const QLowEnergyCharacteristic &characteristic, const QByteArray &newValue)
    {
        qDebug() << "Caractéristique changée:" << characteristic.uuid().toString();
        qDebug() << "Nouvelle valeur:" << newValue;

        emit valeurChangee(characteristic, newValue);
    }

    void characteristicRead(const QLowEnergyCharacteristic &characteristic, const QByteArray &value)
    {
        qDebug() << "Lecture de caractéristique:" << characteristic.uuid().toString();
        qDebug() << "Valeur:" << value;

        emit lectureFinie(characteristic, value);
    }

    void characteristicWritten(const QLowEnergyCharacteristic &characteristic, const QByteArray &newValue)
    {
        qDebug() << "Écriture de caractéristique terminée:" << characteristic.uuid().toString();

        emit ecritureFinie(characteristic);
    }

public slots:
    void lireCaracteristique(const QLowEnergyCharacteristic &characteristic)
    {
        if (!m_service) return;

        m_service->readCharacteristic(characteristic);
    }

    void ecrireCaracteristique(const QLowEnergyCharacteristic &characteristic, const QByteArray &value)
    {
        if (!m_service) return;

        m_service->writeCharacteristic(characteristic, value,
                                     (characteristic.properties() & QLowEnergyCharacteristic::WriteWithoutResponse) ?
                                     QLowEnergyService::WriteWithoutResponse :
                                     QLowEnergyService::WriteWithResponse);
    }

    void activerNotifications(const QLowEnergyCharacteristic &characteristic)
    {
        if (!m_service) return;

        // Vérifier que la caractéristique supporte les notifications
        if (!(characteristic.properties() & QLowEnergyCharacteristic::Notify)) {
            qDebug() << "Cette caractéristique ne supporte pas les notifications";
            return;
        }

        // Trouver le descripteur de configuration client (CCC)
        QLowEnergyDescriptor cccDescriptor = characteristic.descriptor(QBluetoothUuid::ClientCharacteristicConfiguration);
        if (cccDescriptor.isValid()) {
            // Activer les notifications
            m_service->writeDescriptor(cccDescriptor, QByteArray::fromHex("0100"));
        }
    }

signals:
    void connecte();
    void deconnecte();
    void erreur(const QString &message);
    void serviceTrouve(const QBluetoothUuid &uuid);
    void scanServicesFini();
    void caracteristiqueTrouvee(const QLowEnergyCharacteristic &characteristic);
    void valeurChangee(const QLowEnergyCharacteristic &characteristic, const QByteArray &newValue);
    void lectureFinie(const QLowEnergyCharacteristic &characteristic, const QByteArray &value);
    void ecritureFinie(const QLowEnergyCharacteristic &characteristic);

private:
    QLowEnergyController *m_controller;
    QLowEnergyService *m_service;
};
```

### Exemple concret : Application de suivi de température BLE

Voici un exemple plus concret qui utilise BLE pour se connecter à un capteur de température et afficher les données :

```cpp
class TemperatureMonitor : public QObject
{
    Q_OBJECT

public:
    explicit TemperatureMonitor(QObject *parent = nullptr) : QObject(parent),
        m_scanner(new BLEScanner(this)),
        m_controller(new BLEController(this))
    {
        // Connecter les signaux du scanner
        connect(m_scanner, &BLEScanner::appareilTrouve,
                this, &TemperatureMonitor::appareilTrouve);
        connect(m_scanner, &BLEScanner::rechercheFinie,
                this, &TemperatureMonitor::rechercheFinie);

        // Connecter les signaux du contrôleur
        connect(m_controller, &BLEController::connecte,
                this, &TemperatureMonitor::appareilConnecte);
        connect(m_controller, &BLEController::deconnecte,
                this, &TemperatureMonitor::appareilDeconnecte);
        connect(m_controller, &BLEController::serviceTrouve,
                this, &TemperatureMonitor::serviceTrouve);
        connect(m_controller, &BLEController::caracteristiqueTrouvee,
                this, &TemperatureMonitor::caracteristiqueTrouvee);
        connect(m_controller, &BLEController::valeurChangee,
                this, &TemperatureMonitor::temperatureChangee);
    }

    void demarrerRecherche()
    {
        m_scanner->demarrerRecherche();
    }

public slots:
    void connecterACapteur(const QBluetoothDeviceInfo &info)
    {
        m_controller->connecterA(info);
    }

private slots:
    void appareilTrouve(const QBluetoothDeviceInfo &info)
    {
        // Vérifier si c'est notre capteur de température
        if (info.name().contains("TempSensor", Qt::CaseInsensitive)) {
            qDebug() << "Capteur de température trouvé:" << info.name();
            m_appareilCible = info;

            // Arrêter la recherche et se connecter
            m_scanner->demarrerRecherche();
            m_controller->connecterA(info);
        }
    }

    void rechercheFinie(const QList<QBluetoothDeviceInfo> &appareils)
    {
        bool capteurTrouve = false;

        // Parcourir tous les appareils trouvés
        for (const QBluetoothDeviceInfo &info : appareils) {
            if (info.name().contains("TempSensor", Qt::CaseInsensitive)) {
                capteurTrouve = true;
                break;
            }
        }

        if (!capteurTrouve) {
            qDebug() << "Aucun capteur de température trouvé";
            emit aucunCapteurTrouve();
        }
    }

    void appareilConnecte()
    {
        qDebug() << "Connecté au capteur de température";
        emit capteurConnecte(m_appareilCible.name());
    }

    void appareilDeconnecte()
    {
        qDebug() << "Déconnecté du capteur de température";
        emit capteurDeconnecte();
    }

    void serviceTrouve(const QBluetoothUuid &uuid)
    {
        // Vérifier si c'est le service de température
        if (uuid == QBluetoothUuid(QStringLiteral("0000180F-0000-1000-8000-00805F9B34FB"))) {
            qDebug() << "Service de température trouvé";

            // Connecter au service
            m_controller->connecterAuService(uuid);
        }
    }

    void caracteristiqueTrouvee(const QLowEnergyCharacteristic &characteristic)
    {
        // Vérifier si c'est la caractéristique de température
        if (characteristic.uuid() == QBluetoothUuid(QStringLiteral("00002A19-0000-1000-8000-00805F9B34FB"))) {
            qDebug() << "Caractéristique de température trouvée";

            // Activer les notifications pour recevoir les changements
            m_controller->activerNotifications(characteristic);

            // Lire la valeur actuelle
            m_controller->lireCaracteristique(characteristic);
        }
    }

void temperatureChangee(const QLowEnergyCharacteristic &characteristic, const QByteArray &value)
{
    // Vérifier que c'est la caractéristique de température
    if (characteristic.uuid() == QBluetoothUuid(QStringLiteral("00002A19-0000-1000-8000-00805F9B34FB"))) {
        // Convertir la valeur en température
        if (value.size() > 0) {
            // Le premier octet contient la valeur de la température
            int temp = static_cast<int>(static_cast<unsigned char>(value.at(0)));

            // Certains capteurs multiplient la température par 10 pour inclure une décimale
            float temperature = temp / 10.0f;

            qDebug() << "Température actuelle:" << temperature << "°C";

            // Émettre le signal avec la nouvelle température
            emit nouvelleTemperature(temperature);
        }
    }
}

signals:
    void aucunCapteurTrouve();
    void capteurConnecte(const QString &nom);
    void capteurDeconnecte();
    void nouvelleTemperature(float temperature);

private:
    BLEScanner *m_scanner;
    BLEController *m_controller;
    QBluetoothDeviceInfo m_appareilCible;
};
```

### Utilisation dans une interface utilisateur Qt

Pour intégrer le tout dans une application Qt, vous pouvez créer une interface utilisateur simple :

```cpp
// Dans un fichier header MainWindow.h
#include <QMainWindow>
#include <QPushButton>
#include <QLabel>
#include <QVBoxLayout>
#include <QListWidget>
#include "temperaturemonitor.h"

class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    MainWindow(QWidget *parent = nullptr);
    ~MainWindow();

private slots:
    void rechercherCapteurs();
    void connecterCapteur();
    void afficherTemperature(float temperature);
    void afficherErreurBluetooth(const QString &message);

private:
    QPushButton *m_rechercheButton;
    QPushButton *m_connectButton;
    QListWidget *m_appareilsListWidget;
    QLabel *m_statutLabel;
    QLabel *m_temperatureLabel;

    TemperatureMonitor *m_monitor;
    QList<QBluetoothDeviceInfo> m_appareils;
};

// Dans un fichier source MainWindow.cpp
#include "mainwindow.h"

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent), m_monitor(new TemperatureMonitor(this))
{
    // Créer les widgets
    m_rechercheButton = new QPushButton("Rechercher les capteurs", this);
    m_connectButton = new QPushButton("Connecter", this);
    m_appareilsListWidget = new QListWidget(this);
    m_statutLabel = new QLabel("Prêt", this);
    m_temperatureLabel = new QLabel("-- °C", this);

    // Configurer la mise en page
    QWidget *centralWidget = new QWidget(this);
    QVBoxLayout *layout = new QVBoxLayout(centralWidget);
    layout->addWidget(m_rechercheButton);
    layout->addWidget(m_appareilsListWidget);
    layout->addWidget(m_connectButton);
    layout->addWidget(m_statutLabel);
    layout->addWidget(m_temperatureLabel);

    setCentralWidget(centralWidget);

    // Configuration initiale
    m_connectButton->setEnabled(false);

    // Connecter les signaux des widgets
    connect(m_rechercheButton, &QPushButton::clicked,
            this, &MainWindow::rechercherCapteurs);
    connect(m_connectButton, &QPushButton::clicked,
            this, &MainWindow::connecterCapteur);
    connect(m_appareilsListWidget, &QListWidget::itemSelectionChanged, this, [=]() {
        m_connectButton->setEnabled(m_appareilsListWidget->currentRow() >= 0);
    });

    // Connecter les signaux du moniteur de température
    connect(m_monitor, &TemperatureMonitor::aucunCapteurTrouve, this, [=]() {
        m_statutLabel->setText("Aucun capteur trouvé");
    });
    connect(m_monitor, &TemperatureMonitor::capteurConnecte, this, [=](const QString &nom) {
        m_statutLabel->setText("Connecté à " + nom);
    });
    connect(m_monitor, &TemperatureMonitor::capteurDeconnecte, this, [=]() {
        m_statutLabel->setText("Déconnecté");
    });
    connect(m_monitor, &TemperatureMonitor::nouvelleTemperature,
            this, &MainWindow::afficherTemperature);

    // Définir la taille de la fenêtre
    resize(400, 300);
    setWindowTitle("Moniteur de température Bluetooth");
}

MainWindow::~MainWindow()
{
}

void MainWindow::rechercherCapteurs()
{
    m_appareilsListWidget->clear();
    m_statutLabel->setText("Recherche en cours...");
    m_rechercheButton->setEnabled(false);

    BLEScanner *scanner = new BLEScanner(this);

    connect(scanner, &BLEScanner::appareilTrouve, this, [=](const QBluetoothDeviceInfo &info) {
        // Ajouter l'appareil à la liste
        QString item = info.name() + " (" + info.address().toString() + ")";
        m_appareilsListWidget->addItem(item);

        // Stocker l'appareil
        m_appareils.append(info);
    });

    connect(scanner, &BLEScanner::rechercheFinie, this, [=](const QList<QBluetoothDeviceInfo> &) {
        m_statutLabel->setText("Recherche terminée");
        m_rechercheButton->setEnabled(true);

        if (m_appareilsListWidget->count() == 0) {
            m_statutLabel->setText("Aucun appareil trouvé");
        }
    });

    scanner->demarrerRecherche();
}

void MainWindow::connecterCapteur()
{
    int index = m_appareilsListWidget->currentRow();
    if (index >= 0 && index < m_appareils.size()) {
        QBluetoothDeviceInfo appareil = m_appareils.at(index);
        m_statutLabel->setText("Connexion à " + appareil.name() + "...");

        m_monitor->connecterACapteur(appareil);
    }
}

void MainWindow::afficherTemperature(float temperature)
{
    m_temperatureLabel->setText(QString("%1 °C").arg(temperature));
}

void MainWindow::afficherErreurBluetooth(const QString &message)
{
    m_statutLabel->setText("Erreur: " + message);
}
```

## Deuxième partie : NFC avec Qt

### Qu'est-ce que NFC ?

Near Field Communication (NFC) est une technologie de communication sans fil à très courte portée (quelques centimètres) qui permet l'échange de données entre périphériques. Elle est utilisée dans de nombreuses applications comme le paiement sans contact, les badges d'accès, ou la lecture d'étiquettes intelligentes.

### Configuration du projet pour NFC

Pour utiliser NFC dans votre projet Qt, vous devez d'abord ajouter le module NFC à votre fichier `.pro` :

```
QT += nfc
```

Et dans vos fichiers source, incluez les en-têtes nécessaires :

```cpp
#include <QNdefMessage>
#include <QNdefRecord>
#include <QNdefNfcTextRecord>
#include <QNearFieldManager>
#include <QNearFieldTarget>
```

### Vérification de la disponibilité NFC

Tout comme pour Bluetooth, il est important de vérifier si NFC est disponible sur l'appareil :

```cpp
bool verifierNfcDisponible()
{
    QNearFieldManager manager;

    if (!manager.isAvailable()) {
        qDebug() << "NFC n'est pas disponible sur cet appareil";
        return false;
    }

    qDebug() << "NFC est disponible";
    return true;
}
```

### Lecture de tags NFC

La lecture de tags NFC est l'une des fonctionnalités les plus utilisées :

```cpp
class NFCReader : public QObject
{
    Q_OBJECT

public:
    explicit NFCReader(QObject *parent = nullptr) : QObject(parent)
    {
        m_manager = new QNearFieldManager(this);

        // Connecter les signaux
        connect(m_manager, &QNearFieldManager::targetDetected,
                this, &NFCReader::cibleDetectee);
        connect(m_manager, &QNearFieldManager::targetLost,
                this, &NFCReader::ciblePerdue);
    }

    bool demarrer()
    {
        if (!m_manager->isAvailable()) {
            qDebug() << "NFC n'est pas disponible";
            return false;
        }

        // Démarrer la détection des tags NFC
        return m_manager->startTargetDetection();
    }

    void arreter()
    {
        m_manager->stopTargetDetection();
    }

private slots:
    void cibleDetectee(QNearFieldTarget *target)
    {
        qDebug() << "Tag NFC détecté";

        if (!target) {
            qDebug() << "Erreur: cible invalide";
            return;
        }

        // Connecter le signal pour recevoir les messages NDEF
        connect(target, &QNearFieldTarget::ndefMessageRead,
                this, &NFCReader::messageNdefLu);

        // Lire le message NDEF du tag
        target->readNdefMessages();

        emit tagDetecte();
    }

    void ciblePerdue(QNearFieldTarget *target)
    {
        qDebug() << "Tag NFC perdu";

        if (target) {
            target->deleteLater();
        }

        emit tagPerdu();
    }

    void messageNdefLu(QNdefMessage message)
    {
        qDebug() << "Message NDEF lu, contient" << message.recordCount() << "enregistrements";

        // Parcourir tous les enregistrements
        for (int i = 0; i < message.recordCount(); ++i) {
            QNdefRecord record = message.recordAt(i);

            // Vérifier le type d'enregistrement
            if (record.typeNameFormat() == QNdefRecord::NfcRtd &&
                record.type() == "T") {
                // C'est un enregistrement texte
                QNdefNfcTextRecord textRecord(record);
                qDebug() << "Texte:" << textRecord.text();

                emit texteDetecte(textRecord.text());
            } else if (record.typeNameFormat() == QNdefRecord::NfcRtd &&
                       record.type() == "U") {
                // C'est une URL
                QNdefNfcUriRecord uriRecord(record);
                qDebug() << "URL:" << uriRecord.uri().toString();

                emit urlDetectee(uriRecord.uri());
            } else {
                // Autre type d'enregistrement
                qDebug() << "Type d'enregistrement:" << record.type();

                emit enregistrementDetecte(record);
            }
        }
    }

signals:
    void tagDetecte();
    void tagPerdu();
    void texteDetecte(const QString &texte);
    void urlDetectee(const QUrl &url);
    void enregistrementDetecte(const QNdefRecord &record);

private:
    QNearFieldManager *m_manager;
};
```

### Écriture sur des tags NFC

L'écriture sur des tags NFC est également possible avec Qt :

```cpp
class NFCWriter : public QObject
{
    Q_OBJECT

public:
    explicit NFCWriter(QObject *parent = nullptr) : QObject(parent)
    {
        m_manager = new QNearFieldManager(this);

        // Connecter les signaux
        connect(m_manager, &QNearFieldManager::targetDetected,
                this, &NFCWriter::cibleDetectee);
        connect(m_manager, &QNearFieldManager::targetLost,
                this, &NFCWriter::ciblePerdue);
    }

    bool demarrer()
    {
        if (!m_manager->isAvailable()) {
            qDebug() << "NFC n'est pas disponible";
            return false;
        }

        // Démarrer la détection des tags NFC
        return m_manager->startTargetDetection();
    }

    void arreter()
    {
        m_manager->stopTargetDetection();
    }

    void ecrireTexte(const QString &texte)
    {
        m_messageAEcrire.clear();

        // Créer un enregistrement texte
        QNdefNfcTextRecord textRecord;
        textRecord.setText(texte);

        // Ajouter l'enregistrement au message
        m_messageAEcrire.append(textRecord);
    }

    void ecrireUrl(const QUrl &url)
    {
        m_messageAEcrire.clear();

        // Créer un enregistrement URL
        QNdefNfcUriRecord uriRecord;
        uriRecord.setUri(url);

        // Ajouter l'enregistrement au message
        m_messageAEcrire.append(uriRecord);
    }

private slots:
    void cibleDetectee(QNearFieldTarget *target)
    {
        qDebug() << "Tag NFC détecté";

        if (!target) {
            qDebug() << "Erreur: cible invalide";
            return;
        }

        // Vérifier si le tag accepte l'écriture
        if (!target->accessMethods().testFlag(QNearFieldTarget::NdefAccess)) {
            qDebug() << "Ce tag ne supporte pas l'écriture NDEF";
            emit ecritureEchec("Tag non compatible avec l'écriture NDEF");
            return;
        }

        // Écrire le message NDEF sur le tag
        if (m_messageAEcrire.isEmpty()) {
            qDebug() << "Aucun message à écrire";
            return;
        }

        // Connecter les signaux pour l'écriture
        connect(target, &QNearFieldTarget::ndefMessagesWritten,
                this, &NFCWriter::messagesEcrits);
        connect(target, &QNearFieldTarget::error,
                this, &NFCWriter::erreurEcriture);

        // Écrire le message
        target->writeNdefMessages(QList<QNdefMessage>() << m_messageAEcrire);

        emit tagDetecte();
    }

    void ciblePerdue(QNearFieldTarget *target)
    {
        qDebug() << "Tag NFC perdu";

        if (target) {
            target->deleteLater();
        }

        emit tagPerdu();
    }

    void messagesEcrits()
    {
        qDebug() << "Messages NDEF écrits avec succès";

        emit ecritureReussie();
    }

    void erreurEcriture(QNearFieldTarget::Error error, const QNearFieldTarget::RequestId &id)
    {
        QString messageErreur;

        switch (error) {
        case QNearFieldTarget::NoError:
            return; // Pas d'erreur
        case QNearFieldTarget::UnsupportedError:
            messageErreur = "Opération non supportée";
            break;
        case QNearFieldTarget::TargetOutOfRangeError:
            messageErreur = "Tag hors de portée";
            break;
        case QNearFieldTarget::NoResponseError:
            messageErreur = "Pas de réponse du tag";
            break;
        case QNearFieldTarget::ChecksumMismatchError:
            messageErreur = "Erreur de checksum";
            break;
        case QNearFieldTarget::InvalidParametersError:
            messageErreur = "Paramètres invalides";
            break;
        case QNearFieldTarget::NdefReadError:
            messageErreur = "Erreur de lecture NDEF";
            break;
        case QNearFieldTarget::NdefWriteError:
            messageErreur = "Erreur d'écriture NDEF";
            break;
        default:
            messageErreur = "Erreur inconnue";
        }

        qDebug() << "Erreur lors de l'écriture:" << messageErreur;

        emit ecritureEchec(messageErreur);
    }

signals:
    void tagDetecte();
    void tagPerdu();
    void ecritureReussie();
    void ecritureEchec(const QString &erreur);

private:
    QNearFieldManager *m_manager;
    QNdefMessage m_messageAEcrire;
};
```

### Exemple d'application NFC

Voici un exemple d'application simple qui permet de lire et d'écrire des tags NFC :

```cpp
// Dans un fichier header MainWindow.h
#include <QMainWindow>
#include <QPushButton>
#include <QLabel>
#include <QLineEdit>
#include <QVBoxLayout>
#include <QTabWidget>
#include "nfcreader.h"
#include "nfcwriter.h"

class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    MainWindow(QWidget *parent = nullptr);
    ~MainWindow();

private slots:
    void demarrerLecture();
    void demarrerEcriture();
    void afficherTexteDetecte(const QString &texte);
    void afficherUrlDetectee(const QUrl &url);
    void afficherEtatEcriture(bool succes, const QString &message = QString());

private:
    // Onglet de lecture
    QWidget *creerOngletLecture();
    QLabel *m_statutLectureLabel;
    QLabel *m_contenuLabel;
    QPushButton *m_demarrerLectureButton;
    QPushButton *m_arreterLectureButton;
    NFCReader *m_reader;

    // Onglet d'écriture
    QWidget *creerOngletEcriture();
    QLabel *m_statutEcritureLabel;
    QLineEdit *m_texteLineEdit;
    QLineEdit *m_urlLineEdit;
    QPushButton *m_ecrireTexteButton;
    QPushButton *m_ecrireUrlButton;
    QPushButton *m_arreterEcritureButton;
    NFCWriter *m_writer;
};

// Dans un fichier source MainWindow.cpp
#include "mainwindow.h"

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent), m_reader(new NFCReader(this)), m_writer(new NFCWriter(this))
{
    // Créer un widget d'onglets
    QTabWidget *tabWidget = new QTabWidget(this);
    tabWidget->addTab(creerOngletLecture(), "Lecture NFC");
    tabWidget->addTab(creerOngletEcriture(), "Écriture NFC");

    setCentralWidget(tabWidget);

    // Connecter les signaux du lecteur NFC
    connect(m_reader, &NFCReader::tagDetecte, this, [=]() {
        m_statutLectureLabel->setText("Tag NFC détecté");
    });
    connect(m_reader, &NFCReader::tagPerdu, this, [=]() {
        m_statutLectureLabel->setText("Tag NFC perdu");
    });
    connect(m_reader, &NFCReader::texteDetecte,
            this, &MainWindow::afficherTexteDetecte);
    connect(m_reader, &NFCReader::urlDetectee,
            this, &MainWindow::afficherUrlDetectee);

    // Connecter les signaux de l'écrivain NFC
    connect(m_writer, &NFCWriter::tagDetecte, this, [=]() {
        m_statutEcritureLabel->setText("Tag NFC détecté - écriture en cours...");
    });
    connect(m_writer, &NFCWriter::tagPerdu, this, [=]() {
        m_statutEcritureLabel->setText("Tag NFC perdu");
    });
    connect(m_writer, &NFCWriter::ecritureReussie, this, [=]() {
        afficherEtatEcriture(true, "Écriture réussie");
    });
    connect(m_writer, &NFCWriter::ecritureEchec, this, [=](const QString &erreur) {
        afficherEtatEcriture(false, "Erreur: " + erreur);
    });

    // Définir la taille de la fenêtre
    resize(400, 300);
    setWindowTitle("Application NFC Qt");

    // Vérifier si NFC est disponible
    if (!verifierNfcDisponible()) {
        m_statutLectureLabel->setText("NFC non disponible");
        m_statutEcritureLabel->setText("NFC non disponible");
        m_demarrerLectureButton->setEnabled(false);
        m_ecrireTexteButton->setEnabled(false);
        m_ecrireUrlButton->setEnabled(false);
    }
}

MainWindow::~MainWindow()
{
}

QWidget *MainWindow::creerOngletLecture()
{
    QWidget *widget = new QWidget();
    QVBoxLayout *layout = new QVBoxLayout(widget);

    m_statutLectureLabel = new QLabel("Prêt", widget);
    m_contenuLabel = new QLabel("", widget);
    m_demarrerLectureButton = new QPushButton("Démarrer la lecture", widget);
    m_arreterLectureButton = new QPushButton("Arrêter la lecture", widget);

    layout->addWidget(m_statutLectureLabel);
    layout->addWidget(m_contenuLabel);
    layout->addWidget(m_demarrerLectureButton);
    layout->addWidget(m_arreterLectureButton);

    m_arreterLectureButton->setEnabled(false);

    connect(m_demarrerLectureButton, &QPushButton::clicked,
            this, &MainWindow::demarrerLecture);
    connect(m_arreterLectureButton, &QPushButton::clicked, this, [=]() {
        m_reader->arreter();
        m_statutLectureLabel->setText("Lecture arrêtée");
        m_demarrerLectureButton->setEnabled(true);
        m_arreterLectureButton->setEnabled(false);
    });

    return widget;
}

QWidget *MainWindow::creerOngletEcriture()
{
    QWidget *widget = new QWidget();
    QVBoxLayout *layout = new QVBoxLayout(widget);

    m_statutEcritureLabel = new QLabel("Prêt", widget);

    QLabel *texteLabel = new QLabel("Texte à écrire:", widget);
    m_texteLineEdit = new QLineEdit(widget);
    m_ecrireTexteButton = new QPushButton("Écrire le texte", widget);

    QLabel *urlLabel = new QLabel("URL à écrire:", widget);
    m_urlLineEdit = new QLineEdit(widget);
    m_ecrireUrlButton = new QPushButton("Écrire l'URL", widget);

    m_arreterEcritureButton = new QPushButton("Arrêter l'écriture", widget);

    layout->addWidget(m_statutEcritureLabel);
    layout->addWidget(texteLabel);
    layout->addWidget(m_texteLineEdit);
    layout->addWidget(m_ecrireTexteButton);
    layout->addWidget(urlLabel);
    layout->addWidget(m_urlLineEdit);
    layout->addWidget(m_ecrireUrlButton);
    layout->addWidget(m_arreterEcritureButton);

    m_arreterEcritureButton->setEnabled(false);

    connect(m_ecrireTexteButton, &QPushButton::clicked, this, [=]() {
        QString texte = m_texteLineEdit->text();
        if (!texte.isEmpty()) {
            m_writer->ecrireTexte(texte);
            demarrerEcriture();
        }
    });

    connect(m_ecrireUrlButton, &QPushButton::clicked, this, [=]() {
        QString urlStr = m_urlLineEdit->text();
        if (!urlStr.isEmpty()) {
            QUrl url(urlStr);
            if (url.isValid()) {
                m_writer->ecrireUrl(url);
                demarrerEcriture();
            } else {
                m_statutEcritureLabel->setText("URL invalide");
            }
        }
    });

    connect(m_arreterEcritureButton, &QPushButton::clicked, this, [=]() {
        m_writer->arreter();
        m_statutEcritureLabel->setText("Écriture arrêtée");
        m_ecrireTexteButton->setEnabled(true);
        m_ecrireUrlButton->setEnabled(true);
        m_arreterEcritureButton->setEnabled(false);
    });

    return widget;
}

void MainWindow::demarrerLecture()
{
    if (m_reader->demarrer()) {
        m_statutLectureLabel->setText("En attente d'un tag NFC...");
        m_demarrerLectureButton->setEnabled(false);
        m_arreterLectureButton->setEnabled(true);
    } else {
        m_statutLectureLabel->setText("Impossible de démarrer la lecture NFC");
    }
}

void MainWindow::demarrerEcriture()
{
    if (m_writer->demarrer()) {
        m_statutEcritureLabel->setText("Approchez un tag NFC...");
        m_ecrireTexteButton->setEnabled(false);
        m_ecrireUrlButton->setEnabled(false);
        m_arreterEcritureButton->setEnabled(true);
    } else {
        m_statutEcritureLabel->setText("Impossible de démarrer l'écriture NFC");
    }
}

void MainWindow::afficherTexteDetecte(const QString &texte)
{
    m_contenuLabel->setText("Texte: " + texte);
}

void MainWindow::afficherUrlDetectee(const QUrl &url)
{
    m_contenuLabel->setText("URL: " + url.toString());
}

void MainWindow::afficherEtatEcriture(bool succes, const QString &message)
{
    if (succes) {
        m_statutEcritureLabel->setText(message);
    } else {
        m_statutEcritureLabel->setText(message);
    }

    m_ecrireTexteButton->setEnabled(true);
    m_ecrireUrlButton->setEnabled(true);
}
```

## Intégration de Bluetooth et NFC dans une même application

Dans certains cas, vous pourriez vouloir utiliser à la fois Bluetooth et NFC dans votre application. Par exemple, vous pourriez utiliser NFC pour l'appairage rapide d'un appareil Bluetooth :

```cpp
class BLENFCPairing : public QObject
{
    Q_OBJECT

public:
    explicit BLENFCPairing(QObject *parent = nullptr) : QObject(parent),
        m_nfcReader(new NFCReader(this)),
        m_bleController(new BLEController(this))
    {
        // Connecter les signaux du lecteur NFC
        connect(m_nfcReader, &NFCReader::urlDetectee,
                this, &BLENFCPairing::traiterUrl);

        // Connecter les signaux du contrôleur BLE
        connect(m_bleController, &BLEController::connecte,
                this, &BLENFCPairing::appareilConnecte);
        connect(m_bleController, &BLEController::deconnecte,
                this, &BLENFCPairing::appareilDeconnecte);
        connect(m_bleController, &BLEController::erreur,
                this, &BLENFCPairing::erreurBluetooth);
    }

    void demarrer()
    {
        // Démarrer la lecture NFC
        if (!m_nfcReader->demarrer()) {
            qDebug() << "Impossible de démarrer la lecture NFC";
            return;
        }

        qDebug() << "En attente d'un tag NFC pour l'appairage Bluetooth...";
    }

    void arreter()
    {
        m_nfcReader->arreter();
        m_bleController->deconnecter();
    }

private slots:
void traiterUrl(const QUrl &url)
{
    // Vérifier si l'URL contient les informations d'appairage Bluetooth
    if (url.scheme() == "ble") {
        QString adresseBle = url.host();
        qDebug() << "Adresse Bluetooth détectée:" << adresseBle;

        // Rechercher l'appareil Bluetooth
        BLEScanner scanner;
        connect(&scanner, &BLEScanner::appareilTrouve, this, [=](const QBluetoothDeviceInfo &info) {
            if (info.address().toString() == adresseBle) {
                qDebug() << "Appareil Bluetooth trouvé, tentative de connexion...";
                m_bleController->connecterA(info);
            }
        });

        scanner.demarrerRecherche();

        emit appairageInitie(adresseBle);
    } else {
        qDebug() << "URL non reconnue pour l'appairage Bluetooth:" << url.toString();
    }
}

void appareilConnecte()
{
    qDebug() << "Appareil Bluetooth connecté avec succès";

    emit appairageReussi();
}

void appareilDeconnecte()
{
    qDebug() << "Appareil Bluetooth déconnecté";

    emit appairagePerdu();
}

void erreurBluetooth(const QString &message)
{
    qDebug() << "Erreur Bluetooth:" << message;

    emit appairageEchec(message);
}

signals:
    void appairageInitie(const QString &adresse);
    void appairageReussi();
    void appairagePerdu();
    void appairageEchec(const QString &erreur);

private:
    NFCReader *m_nfcReader;
    BLEController *m_bleController;
};
```

### Création d'un tag NFC pour l'appairage Bluetooth

Pour créer un tag NFC qui initiera un appairage Bluetooth, vous pouvez utiliser le code suivant :

```cpp
void creerTagAppairageBluetooth(NFCWriter *writer, const QString &adresseBluetooth)
{
    // Créer une URL spéciale pour l'appairage
    QUrl url(QString("ble://%1").arg(adresseBluetooth));

    // Écrire l'URL sur le tag NFC
    writer->ecrireUrl(url);
    writer->demarrer();
}
```

## Bonnes pratiques pour Bluetooth et NFC

### Bonnes pratiques pour Bluetooth

1. **Vérifiez toujours la disponibilité** : Vérifiez que Bluetooth est disponible et activé sur l'appareil avant d'essayer de l'utiliser.

2. **Gérez la batterie** : Bluetooth Low Energy est conçu pour économiser la batterie, mais une utilisation incorrecte peut quand même épuiser rapidement la batterie. Désactivez le scan lorsqu'il n'est pas nécessaire.

3. **Gestion des erreurs** : Les communications sans fil peuvent être perturbées par divers facteurs. Votre application doit gérer correctement les erreurs et les déconnexions.

4. **Reconnexion automatique** : Implémentez une logique de reconnexion automatique pour améliorer l'expérience utilisateur.

```cpp
// Exemple de reconnexion automatique
void BLEController::appDisconnected()
{
    qDebug() << "Déconnecté de l'appareil BLE";
    emit deconnecte();

    // Tentative de reconnexion après 3 secondes
    QTimer::singleShot(3000, this, [=]() {
        if (m_autoReconnect && m_deviceInfo.isValid()) {
            qDebug() << "Tentative de reconnexion...";
            m_controller->connectToDevice();
        }
    });
}
```

5. **Permissions** : Sur les plateformes mobiles, assurez-vous d'avoir les permissions nécessaires dans votre manifeste d'application.

### Bonnes pratiques pour NFC

1. **Sessions courtes** : NFC est conçu pour des interactions rapides. N'attendez pas de l'utilisateur qu'il maintienne les appareils proches pendant une longue période.

2. **Feedback utilisateur** : Donnez toujours un retour visuel ou sonore lorsqu'un tag est détecté ou écrit.

3. **Vérification des capacités** : Tous les tags NFC n'ont pas les mêmes capacités. Vérifiez que le tag supporte les opérations que vous souhaitez effectuer.

4. **Protection contre l'écriture** : Pour les informations sensibles, envisagez d'utiliser des tags protégés en écriture après avoir écrit les données initiales.

## Débogage des applications Bluetooth et NFC

### Débogage Bluetooth

Le débogage des applications Bluetooth peut être difficile en raison de la nature invisible des communications sans fil. Voici quelques astuces :

```cpp
// Activer la journalisation détaillée pour Bluetooth
QLoggingCategory::setFilterRules("qt.bluetooth* = true");
```

Utilisez des outils comme "Bluetooth Scanner" sur Android ou "Bluetooth Explorer" sur macOS pour surveiller les appareils Bluetooth et leur état.

### Débogage NFC

Pour le débogage NFC, des outils comme "NFC Tools" peuvent être utiles pour inspecter le contenu des tags NFC.

```cpp
// Journalisation détaillée pour NFC
QLoggingCategory::setFilterRules("qt.nfc* = true");
```

## Limitations et considérations

### Limitations de Bluetooth

- **Compatibilité entre appareils** : Différents appareils peuvent implémenter les standards Bluetooth de manière légèrement différente.
- **Latence** : Bluetooth n'est pas conçu pour des transferts de données à très faible latence.
- **Portée** : La portée est généralement limitée à environ 10 mètres pour le Bluetooth classique et moins pour le BLE.

### Limitations de NFC

- **Distance très courte** : NFC ne fonctionne que sur quelques centimètres.
- **Capacité de stockage limitée** : Les tags NFC ont généralement une petite capacité de stockage.
- **Pas tous les appareils** : Tous les smartphones et tablettes ne disposent pas de NFC.

## Applications concrètes

### Beacon Bluetooth pour l'analyse de présence

Les beacons Bluetooth sont des petits appareils qui émettent périodiquement des signaux Bluetooth. Ils peuvent être utilisés pour détecter la présence d'utilisateurs à proximité.

```cpp
// Classe pour détecter les beacons
class BeaconScanner : public QObject
{
    Q_OBJECT

public:
    BeaconScanner(QObject *parent = nullptr);
    void startScan();
    void stopScan();

signals:
    void beaconDetected(const QString &uuid, int major, int minor, int rssi);

private slots:
    void deviceDiscovered(const QBluetoothDeviceInfo &info);

private:
    QBluetoothDeviceDiscoveryAgent *m_discoveryAgent;
    bool parseIBeaconData(const QByteArray &data, QString &uuid, int &major, int &minor);
};
```

### Système de paiement NFC

Une application de paiement simple basée sur NFC :

```cpp
class NFCPayment : public QObject
{
    Q_OBJECT

public:
    NFCPayment(QObject *parent = nullptr);

    void startPaymentMode(double amount);
    void cancelPayment();

signals:
    void paymentRequest(double amount);
    void paymentSuccess(double amount, const QString &transactionId);
    void paymentFailed(const QString &reason);

private slots:
    void tagDetected(QNearFieldTarget *target);
    void processPaymentResponse(const QNdefMessage &message);

private:
    QNearFieldManager *m_manager;
    double m_currentAmount;
    bool m_paymentInProgress;
};
```

## Projets avancés

### Réseau maillé Bluetooth (Mesh Network)

Bluetooth 5.0 et versions ultérieures prennent en charge les réseaux maillés, où plusieurs appareils peuvent communiquer entre eux sans avoir besoin d'un hub central.

La mise en œuvre d'un réseau maillé est complexe et dépasse le cadre de ce tutoriel, mais Qt fournit des API de bas niveau qui peuvent être utilisées pour créer de tels réseaux.

### Système d'authentification multi-facteurs avec NFC et Bluetooth

Un système d'authentification à plusieurs facteurs pourrait utiliser NFC pour l'identification initiale, puis Bluetooth pour une vérification continue de la présence de l'utilisateur.

## Conclusion

Bluetooth et NFC offrent des possibilités passionnantes pour créer des applications interactives et connectées. Grâce à Qt, l'implémentation de ces technologies devient accessible aux développeurs sur différentes plateformes.

En comprenant les concepts et les classes présentés dans ce tutoriel, vous avez maintenant les compétences nécessaires pour intégrer des communications sans fil dans vos applications Qt. Le potentiel est vaste, que ce soit pour des applications IoT, des systèmes de contrôle à distance, ou des interactions utilisateur innovantes.

N'hésitez pas à explorer davantage en consultant la documentation officielle de Qt pour Bluetooth et NFC, ainsi que les exemples fournis avec Qt.

## Ressources supplémentaires

- Documentation officielle de Qt pour Bluetooth : [Qt Bluetooth](https://doc.qt.io/qt-6/qtbluetooth-index.html)
- Documentation officielle de Qt pour NFC : [Qt NFC](https://doc.qt.io/qt-6/qtnfc-index.html)
- Spécifications Bluetooth : [Bluetooth SIG](https://www.bluetooth.com/specifications/)
- Standards NFC : [NFC Forum](https://nfc-forum.org/our-work/specifications-and-application-documents/)

## Exercices pratiques

Pour consolider vos connaissances, voici quelques exercices pratiques :

1. Créez une application de chat simple utilisant Bluetooth.
2. Développez un système de contrôle domotique où NFC est utilisé pour activer des profils, et Bluetooth pour communiquer avec les appareils.
3. Implémentez un système de partage de fichiers utilisant NFC pour l'initiation et Bluetooth pour le transfert des données.
4. Créez une application de présence qui utilise des beacons Bluetooth pour enregistrer quand les utilisateurs entrent ou quittent certaines zones.

⏭️ [Multimédia et graphiques](/06-multimedia-et-graphiques)
