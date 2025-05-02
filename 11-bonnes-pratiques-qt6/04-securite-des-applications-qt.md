# 11.4 Sécurité des applications Qt

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

## Introduction à la sécurité des applications

La sécurité est un aspect fondamental du développement d'applications modernes, souvent négligé jusqu'à ce qu'un incident se produise. Dans ce chapitre, nous explorerons les bonnes pratiques et techniques pour sécuriser vos applications Qt6, en adoptant une approche préventive plutôt que réactive.

## Principes fondamentaux de sécurité

### Défense en profondeur

La défense en profondeur consiste à mettre en place plusieurs couches de sécurité, afin qu'une faille dans une couche ne compromette pas l'ensemble du système :

```cpp
// Exemple de défense en profondeur
// Couche 1: Validation des entrées
bool validerDonnees(const QString& entree) {
    // Vérifier que l'entrée respecte un format attendu
    QRegularExpression regex("^[a-zA-Z0-9_]{3,16}$");
    return regex.match(entree).hasMatch();
}

// Couche 2: Échappement des données avant utilisation
QString preparerPourSQL(const QString& entree) {
    // Échapper les caractères spéciaux pour éviter les injections SQL
    QSqlQuery query;
    return query.driver()->escapeIdentifier(entree, QSqlDriver::FieldName);
}

// Couche 3: Moindre privilège dans la base de données
void configurerConnexionBD() {
    QSqlDatabase db = QSqlDatabase::addDatabase("QMYSQL");
    db.setUserName("utilisateur_lecture_seule");  // Compte avec privilèges limités
    // ...
}
```

### Principe du moindre privilège

Accordez à chaque composant uniquement les privilèges dont il a besoin pour fonctionner :

```cpp
// Au lieu de demander tous les droits
// MAUVAISE PRATIQUE
void demanderTousLesDroits() {
    // Demander accès complet au système de fichiers
    QFile::setPermissions("donnees.db", QFile::ReadOwner | QFile::WriteOwner |
                                        QFile::ReadUser | QFile::WriteUser |
                                        QFile::ReadGroup | QFile::WriteGroup |
                                        QFile::ReadOther | QFile::WriteOther);
}

// BONNE PRATIQUE
void demanderDroitsMinimaux() {
    // Demander uniquement les droits nécessaires
    QFile::setPermissions("donnees.db", QFile::ReadOwner | QFile::WriteOwner);
}
```

## Sécurisation des entrées utilisateur

### Validation des entrées

Toujours valider les entrées utilisateur avant de les traiter :

```cpp
// Widget pour saisir un numéro de téléphone avec validation
class ChampTelephone : public QLineEdit {
    Q_OBJECT

public:
    ChampTelephone(QWidget* parent = nullptr) : QLineEdit(parent) {
        // Valider pendant la saisie
        QRegularExpression regex("^[0-9\\+\\-\\(\\)\\s]{0,20}$");
        QValidator* validator = new QRegularExpressionValidator(regex, this);
        setValidator(validator);

        // Connecter le signal de modification pour une validation supplémentaire
        connect(this, &QLineEdit::textChanged, this, &ChampTelephone::validerFormat);
    }

private slots:
    void validerFormat(const QString& texte) {
        // Vérification supplémentaire (nombre de chiffres, etc.)
        int chiffres = 0;
        for (QChar c : texte) {
            if (c.isDigit()) chiffres++;
        }

        // Appliquer un style visuel en fonction de la validité
        if (chiffres >= 8 && chiffres <= 15) {
            setStyleSheet("background-color: #e0ffe0;");  // Vert pâle = OK
        } else {
            setStyleSheet("background-color: #ffe0e0;");  // Rouge pâle = Erreur
        }
    }
};
```

### Protection contre l'injection SQL

Utilisez des requêtes préparées pour éviter les injections SQL :

```cpp
// DANGEREUX: Concaténation directe de chaînes dans une requête SQL
void requeteDangereuse(const QString& nomUtilisateur) {
    QSqlQuery query;
    // Vulnérable à l'injection SQL !
    query.exec("SELECT * FROM utilisateurs WHERE nom = '" + nomUtilisateur + "'");
}

// SÉCURISÉ: Utilisation de requêtes préparées
void requeteSecurisee(const QString& nomUtilisateur) {
    QSqlQuery query;
    query.prepare("SELECT * FROM utilisateurs WHERE nom = :nom");
    query.bindValue(":nom", nomUtilisateur);
    query.exec();
}
```

### Protection contre les attaques XSS (Cross-Site Scripting)

Si votre application affiche du contenu HTML, assurez-vous de l'échapper correctement :

```cpp
// DANGEREUX: Affichage direct de texte non échappé
void afficherDangereux(const QString& message) {
    // Vulnérable aux attaques XSS !
    ui->navigateurWeb->setHtml("<div>" + message + "</div>");
}

// SÉCURISÉ: Échappement du HTML
void afficherSecurise(const QString& message) {
    // Échapper les caractères spéciaux HTML
    QString messageSecurise = message.toHtmlEscaped();
    ui->navigateurWeb->setHtml("<div>" + messageSecurise + "</div>");
}
```

## Protection des données sensibles

### Stockage sécurisé des informations d'authentification

N'enregistrez jamais les mots de passe en clair :

```cpp
// MAUVAIS: Stockage en clair
void enregistrerUtilisateurNonSecurise(const QString& nom, const QString& motDePasse) {
    QSettings settings;
    settings.setValue("utilisateur/nom", nom);
    settings.setValue("utilisateur/motdepasse", motDePasse);  // Dangereux !
}

// BON: Hachage du mot de passe avec sel
void enregistrerUtilisateurSecurise(const QString& nom, const QString& motDePasse) {
    QSettings settings;

    // Générer un sel aléatoire
    QByteArray sel = QUuid::createUuid().toByteArray();

    // Combiner le mot de passe et le sel
    QByteArray combinaison = motDePasse.toUtf8() + sel;

    // Calculer le hachage
    QByteArray hash = QCryptographicHash::hash(combinaison, QCryptographicHash::Sha256);

    // Enregistrer le nom, le sel et le hachage
    settings.setValue("utilisateur/nom", nom);
    settings.setValue("utilisateur/sel", sel);
    settings.setValue("utilisateur/hash", hash);
}
```

### Protection des données en transit

Utilisez le chiffrement pour les communications réseau :

```cpp
// Configuration d'une connexion HTTPS sécurisée
QNetworkAccessManager* creerManagerSecurise() {
    QNetworkAccessManager* manager = new QNetworkAccessManager();

    // Configurer TLS/SSL
    QSslConfiguration config = QSslConfiguration::defaultConfiguration();
    config.setProtocol(QSsl::TlsV1_2OrLater);  // Utiliser au moins TLS 1.2

    // Appliquer la configuration à toutes les requêtes
    manager->connectToHostEncrypted("api.exemple.com", 443, config);

    return manager;
}

// Utilisation
void envoyerDonneesSecurisees(const QUrl& url, const QByteArray& donnees) {
    QNetworkAccessManager* manager = creerManagerSecurise();

    QNetworkRequest requete(url);
    requete.setSslConfiguration(QSslConfiguration::defaultConfiguration());
    requete.setHeader(QNetworkRequest::ContentTypeHeader, "application/json");

    // Envoi sécurisé via HTTPS
    manager->post(requete, donnees);
}
```

### Utilisation du stockage sécurisé pour les secrets

Utilisez un gestionnaire de mots de passe système lorsque disponible :

```cpp
#include <QKeychain>  // Nécessite le module QtKeychain

// Enregistrer un secret de façon sécurisée
void enregistrerSecret(const QString& cle, const QString& secret) {
    QKeychain::WritePasswordJob job(QStringLiteral("MonApplication"));
    job.setKey(cle);
    job.setTextData(secret);
    job.exec();

    if (job.error()) {
        qWarning() << "Erreur lors de l'enregistrement du secret:" << job.errorString();
    }
}

// Récupérer un secret
QString recupererSecret(const QString& cle) {
    QKeychain::ReadPasswordJob job(QStringLiteral("MonApplication"));
    job.setKey(cle);
    job.exec();

    if (!job.error()) {
        return job.textData();
    } else {
        qWarning() << "Erreur lors de la récupération du secret:" << job.errorString();
        return QString();
    }
}
```

## Sécurisation des ressources

### Protection des ressources compilées

Les ressources Qt (.qrc) sont intégrées dans l'exécutable mais ne sont pas chiffrées par défaut. Pour les données sensibles, implémentez votre propre couche de chiffrement :

```cpp
// Charger et déchiffrer une ressource protégée
QByteArray chargerRessourceProtegee(const QString& chemin, const QByteArray& cle) {
    // Charger la ressource chiffrée
    QFile file(chemin);
    if (!file.open(QIODevice::ReadOnly)) {
        return QByteArray();
    }

    QByteArray donneesCryptees = file.readAll();
    file.close();

    // Déchiffrer avec la clé
    QAESEncryption encryption(QAESEncryption::AES_256, QAESEncryption::CBC);
    QByteArray iv = donneesCryptees.left(16);  // Vecteur d'initialisation
    QByteArray donnees = encryption.decode(donneesCryptees.mid(16), cle, iv);

    return donnees;
}
```

### Vérification de l'intégrité des fichiers

Ajoutez des vérifications d'intégrité pour les fichiers importants :

```cpp
// Vérifier l'intégrité d'un fichier par hachage
bool verifierIntegriteFichier(const QString& chemin, const QByteArray& hashAttendu) {
    QFile file(chemin);
    if (!file.open(QIODevice::ReadOnly)) {
        return false;
    }

    QCryptographicHash hash(QCryptographicHash::Sha256);
    if (!hash.addData(&file)) {
        file.close();
        return false;
    }

    file.close();
    QByteArray hashCalcule = hash.result();

    // Comparer le hash calculé avec celui attendu
    return hashCalcule == hashAttendu;
}
```

## Sécurité du code et des plugins

### Chargement sécurisé de plugins

Vérifiez les plugins avant de les charger :

```cpp
// Fonction pour charger un plugin de façon sécurisée
QPluginLoader* chargerPluginSecurise(const QString& chemin) {
    // 1. Vérifier l'existence et la lisibilité du fichier
    QFileInfo info(chemin);
    if (!info.exists() || !info.isReadable() || !info.isFile()) {
        qWarning() << "Plugin inaccessible:" << chemin;
        return nullptr;
    }

    // 2. Vérifier la signature du plugin (exemple simplifié)
    if (!verifierSignaturePlugin(chemin)) {
        qWarning() << "Signature de plugin invalide:" << chemin;
        return nullptr;
    }

    // 3. Charger le plugin avec des restrictions
    QPluginLoader* loader = new QPluginLoader(chemin);

    // 4. Vérifier les métadonnées avant l'instanciation
    QJsonObject metadata = loader->metaData();
    if (!validerMetadonnees(metadata)) {
        delete loader;
        return nullptr;
    }

    return loader;
}

// Utilisation
void chargerPlugins() {
    QDir repertoirePlugins("./plugins");
    for (const QString& fichier : repertoirePlugins.entryList(QDir::Files)) {
        QString cheminComplet = repertoirePlugins.absoluteFilePath(fichier);
        QPluginLoader* loader = chargerPluginSecurise(cheminComplet);

        if (loader && loader->load()) {
            QObject* plugin = loader->instance();
            if (plugin) {
                integrerPlugin(plugin);
            } else {
                delete loader;
            }
        }
    }
}
```

### Protection contre les scripts malveillants dans QML

Si votre application utilise QML et permet de charger des fichiers QML dynamiquement, prenez des précautions :

```cpp
// Charger un fichier QML de manière sécurisée
QQmlComponent* chargerQMLSecurise(QQmlEngine* engine, const QString& chemin) {
    // 1. Vérifier le chemin
    QFileInfo info(chemin);
    if (!info.exists() || !info.isReadable() ||
        !info.absoluteFilePath().startsWith(QDir::currentPath())) {
        qWarning() << "Tentative d'accès non autorisé:" << chemin;
        return nullptr;
    }

    // 2. Créer un environnement restreint
    QQmlEngine* engineRestreint = new QQmlEngine();

    // 3. Désactiver les imports non sécurisés
    QStringList modulesAutorises = {"QtQuick", "QtQuick.Controls"};
    engineRestreint->setImportPathList(modulesAutorises);

    // 4. Charger le composant
    QQmlComponent* composant = new QQmlComponent(engineRestreint, QUrl::fromLocalFile(chemin));

    if (composant->isError()) {
        qWarning() << "Erreurs lors du chargement QML:" << composant->errors();
        delete composant;
        return nullptr;
    }

    return composant;
}
```

## Protections contre le reverse engineering

### Obscurcissement du code

L'obscurcissement ne garantit pas la sécurité mais peut compliquer la tâche des attaquants :

```cpp
// Exemple d'obscurcissement simple (à des fins éducatives seulement)
// Au lieu de:
bool verifierLicence(const QString& cle) {
    return cle == "ABC123";
}

// Utiliser des techniques d'obscurcissement:
bool verifierLicence(const QString& cle) {
    // Clé dispersée dans le code
    static const char fragments[] = {'A', '3', 'C', 'B', '2', '1'};
    static const int ordre[] = {0, 3, 2, 5, 4, 1};

    // Reconstitution dynamique de la clé
    QString cleAttendue;
    for (int i = 0; i < 6; ++i) {
        cleAttendue.append(fragments[ordre[i]]);
    }

    // Vérification indirecte
    int somme = 0;
    for (int i = 0; i < cle.length() && i < cleAttendue.length(); ++i) {
        somme += (cle[i].unicode() ^ cleAttendue[i].unicode());
    }

    return somme == 0 && cle.length() == cleAttendue.length();
}
```

### Protection contre le débogage

Détectez les débogueurs et réagissez en conséquence :

```cpp
#include <QCoreApplication>

#ifdef Q_OS_WINDOWS
#include <windows.h>
#endif

bool detecterDebogueur() {
#ifdef Q_OS_WINDOWS
    // Windows: Utiliser IsDebuggerPresent
    return IsDebuggerPresent();
#elif defined(Q_OS_LINUX)
    // Linux: Vérifier /proc/self/status
    QFile file("/proc/self/status");
    if (file.open(QIODevice::ReadOnly | QIODevice::Text)) {
        QTextStream in(&file);
        QString line;
        while (in.readLineInto(&line)) {
            if (line.startsWith("TracerPid:")) {
                bool ok;
                int pid = line.split(':').value(1).trimmed().toInt(&ok);
                return ok && pid != 0;
            }
        }
    }
#elif defined(Q_OS_MACOS)
    // macOS: Utiliser ptrace ou sysctl
    // (code spécifique requis)
#endif
    return false;
}

void reagirAuDebogage() {
    if (detecterDebogueur()) {
        // Réagir de manière subtile plutôt que d'arrêter l'application
        qsrand(QTime::currentTime().msec());
        QTimer::singleShot(qrand() % 10000, []() {
            // Introduire un comportement aléatoire difficile à déboguer
        });
    }
}
```

## Sécurité des communications réseau

### Validation des certificats SSL/TLS

Assurez-vous de valider correctement les certificats :

```cpp
void configurerValidationCertificats(QNetworkAccessManager* manager) {
    // Connecter le signal de vérification de certificat
    connect(manager, &QNetworkAccessManager::sslErrors,
            [](QNetworkReply* reply, const QList<QSslError>& errors) {
        // NE PAS utiliser ceci en production:
        // reply->ignoreSslErrors();

        // Au lieu de cela, validez chaque erreur spécifiquement
        QList<QSslError> erreursAcceptables;
        for (const QSslError& erreur : errors) {
            // Vérifier si l'erreur est acceptable dans votre contexte
            if (erreur.error() == QSslError::SelfSignedCertificate) {
                // Vous pourriez vérifier l'empreinte d'un certificat auto-signé connu
                if (erreur.certificate().digest() == EMPREINTE_CONNUE) {
                    erreursAcceptables.append(erreur);
                }
            }
        }

        // Ignorer uniquement les erreurs spécifiquement validées
        if (!erreursAcceptables.isEmpty()) {
            reply->ignoreSslErrors(erreursAcceptables);
        } else {
            // Informer l'utilisateur des problèmes de sécurité
            qWarning() << "Erreurs SSL détectées:" << errors;
        }
    });
}
```

### Durcissement de la configuration TLS

Renforcez la sécurité des connexions TLS :

```cpp
QSslConfiguration configurerTLSSecurise() {
    QSslConfiguration config = QSslConfiguration::defaultConfiguration();

    // Utiliser uniquement TLS 1.2 ou supérieur
    config.setProtocol(QSsl::TlsV1_2OrLater);

    // Spécifier les suites de chiffrement sécurisées
    QList<QSslCipher> ciphers = config.ciphers();
    QList<QSslCipher> secureCiphers;

    for (const QSslCipher& cipher : ciphers) {
        // Exclure les suites de chiffrement faibles
        if (cipher.keyExchangeMethod() != "DH" &&
            cipher.encryptionMethod().contains("AES") &&
            cipher.usedBits() >= 128) {
            secureCiphers.append(cipher);
        }
    }

    config.setCiphers(secureCiphers);

    return config;
}

// Utilisation
void appliquerConfigurationSecurisee() {
    QSslConfiguration config = configurerTLSSecurise();
    QSslConfiguration::setDefaultConfiguration(config);
}
```

## Mise à jour et correction des vulnérabilités

### Mécanisme de mise à jour sécurisé

Implémentez un système de mise à jour sécurisé :

```cpp
class ServiceMiseAJour : public QObject {
    Q_OBJECT

private:
    QNetworkAccessManager* m_manager;
    QUrl m_urlServeurMiseAJour;
    QString m_cheminApplication;
    QByteArray m_clePublique;  // Clé pour vérifier les signatures

public:
    ServiceMiseAJour(QObject* parent = nullptr) : QObject(parent) {
        m_manager = new QNetworkAccessManager(this);

        // Configurer la communication sécurisée
        QSslConfiguration config = configurerTLSSecurise();
        QNetworkRequest::setDefaultSslConfiguration(config);

        // Chemin de l'application actuelle
        m_cheminApplication = QCoreApplication::applicationFilePath();
    }

    void verifierMiseAJour() {
        // Préparer la requête avec la version actuelle
        QNetworkRequest requete(m_urlServeurMiseAJour);
        requete.setHeader(QNetworkRequest::ContentTypeHeader, "application/json");

        QJsonObject donnees;
        donnees["version"] = VERSION_APPLICATION;
        donnees["plateforme"] = QSysInfo::productType();
        donnees["architecture"] = QSysInfo::currentCpuArchitecture();

        QJsonDocument doc(donnees);

        // Envoyer la requête
        QNetworkReply* reply = m_manager->post(requete, doc.toJson());

        connect(reply, &QNetworkReply::finished, [this, reply]() {
            if (reply->error() == QNetworkReply::NoError) {
                traiterReponseMAJ(reply->readAll());
            } else {
                emit erreurMiseAJour(reply->errorString());
            }
            reply->deleteLater();
        });
    }

private:
    void traiterReponseMAJ(const QByteArray& reponse) {
        QJsonDocument doc = QJsonDocument::fromJson(reponse);
        if (doc.isNull()) {
            emit erreurMiseAJour("Réponse invalide");
            return;
        }

        QJsonObject obj = doc.object();

        // Vérifier si une mise à jour est disponible
        if (obj["disponible"].toBool()) {
            QString nouvelleVersion = obj["version"].toString();
            QUrl urlTelechargement = QUrl(obj["url"].toString());
            QByteArray signatureHex = QByteArray::fromHex(obj["signature"].toString().toLatin1());

            emit miseAJourDisponible(nouvelleVersion);

            // Demander confirmation à l'utilisateur
            if (confirmerMiseAJour(nouvelleVersion)) {
                telechargerMiseAJour(urlTelechargement, signatureHex);
            }
        } else {
            emit aucuneMiseAJourDisponible();
        }
    }

    void telechargerMiseAJour(const QUrl& url, const QByteArray& signatureAttendue) {
        QNetworkRequest requete(url);
        QNetworkReply* reply = m_manager->get(requete);

        connect(reply, &QNetworkReply::downloadProgress,
                this, &ServiceMiseAJour::progressionTelechargement);

        connect(reply, &QNetworkReply::finished, [this, reply, signatureAttendue]() {
            if (reply->error() == QNetworkReply::NoError) {
                QByteArray donnees = reply->readAll();

                // Vérifier la signature
                if (verifierSignature(donnees, signatureAttendue, m_clePublique)) {
                    installerMiseAJour(donnees);
                } else {
                    emit erreurMiseAJour("Signature invalide");
                }
            } else {
                emit erreurMiseAJour(reply->errorString());
            }
            reply->deleteLater();
        });
    }

    bool verifierSignature(const QByteArray& donnees, const QByteArray& signature, const QByteArray& clePublique) {
        // Implémenter la vérification de signature cryptographique
        // (Utiliser OpenSSL, Qt Cryptographic Architecture, etc.)
        return true;  // Exemple simplifié
    }

    void installerMiseAJour(const QByteArray& donnees) {
        // Enregistrer le fichier de mise à jour
        QString cheminMAJ = QDir::tempPath() + "/mise_a_jour.zip";
        QFile fichier(cheminMAJ);
        if (fichier.open(QIODevice::WriteOnly)) {
            fichier.write(donnees);
            fichier.close();

            // Extraire et installer la mise à jour
            // ...

            emit miseAJourTerminee();
        } else {
            emit erreurMiseAJour("Impossible d'écrire le fichier de mise à jour");
        }
    }

    bool confirmerMiseAJour(const QString& version) {
        // Demander confirmation à l'utilisateur
        return true;  // Exemple simplifié
    }

signals:
    void miseAJourDisponible(const QString& version);
    void aucuneMiseAJourDisponible();
    void progressionTelechargement(qint64 recu, qint64 total);
    void miseAJourTerminee();
    void erreurMiseAJour(const QString& message);
};
```

### Journalisation sécurisée des événements

Mettez en place un système de journalisation pour détecter les incidents de sécurité :

```cpp
class JournalSecurite : public QObject {
    Q_OBJECT

private:
    QFile m_fichierJournal;
    QTextStream m_flux;
    QMutex m_mutex;

public:
    enum NiveauSecurite {
        Information,
        Avertissement,
        Alerte
    };

    JournalSecurite(QObject* parent = nullptr) : QObject(parent) {
        // Créer ou ouvrir le fichier journal
        QString cheminJournal = QStandardPaths::writableLocation(QStandardPaths::AppDataLocation) + "/securite.log";
        m_fichierJournal.setFileName(cheminJournal);

        if (m_fichierJournal.open(QIODevice::WriteOnly | QIODevice::Append | QIODevice::Text)) {
            m_flux.setDevice(&m_fichierJournal);
        } else {
            qWarning() << "Impossible d'ouvrir le fichier journal de sécurité";
        }
    }

    ~JournalSecurite() {
        if (m_fichierJournal.isOpen()) {
            m_fichierJournal.close();
        }
    }

public slots:
    void journaliser(NiveauSecurite niveau, const QString& message) {
        QMutexLocker locker(&m_mutex);

        if (!m_fichierJournal.isOpen()) {
            return;
        }

        // Horodatage précis
        QString horodatage = QDateTime::currentDateTime().toString("yyyy-MM-dd hh:mm:ss.zzz");

        // Niveau de sécurité en texte
        QString niveauTexte;
        switch (niveau) {
            case Information: niveauTexte = "INFO"; break;
            case Avertissement: niveauTexte = "AVERT"; break;
            case Alerte: niveauTexte = "ALERTE"; break;
        }

        // Informations contextuelles
        QString utilisateur = QProcessEnvironment::systemEnvironment().value("USERNAME", "inconnu");
        QString adresseIP = obtenirAdresseIPLocale();

        // Écrire l'entrée du journal
        m_flux << QString("[%1] [%2] [%3@%4] %5\n")
                    .arg(horodatage)
                    .arg(niveauTexte)
                    .arg(utilisateur)
                    .arg(adresseIP)
                    .arg(message);
        m_flux.flush();

        // Pour les alertes, prendre des mesures supplémentaires
        if (niveau == Alerte) {
            envoyerNotificationAlerte(message);
        }
    }

private:
    QString obtenirAdresseIPLocale() {
        QString adresseIP = "inconnue";
        for (const QHostAddress& adresse : QNetworkInterface::allAddresses()) {
            if (adresse.protocol() == QAbstractSocket::IPv4Protocol && adresse != QHostAddress::LocalHost) {
                adresseIP = adresse.toString();
                break;
            }
        }
        return adresseIP;
    }

    void envoyerNotificationAlerte(const QString& message) {
        // Envoyer une notification à l'administrateur
        // ...
    }
};

// Utilisation
void exemple() {
    JournalSecurite* journal = new JournalSecurite();

    // Journaliser une tentative d'authentification
    journal->journaliser(JournalSecurite::Information, "Utilisateur connecté: admin");

    // Journaliser une tentative suspecte
    journal->journaliser(JournalSecurite::Alerte, "10 tentatives d'authentification échouées pour l'utilisateur: admin");
}
```

## Exemple complet : Application sécurisée

Voici un exemple d'intégration de plusieurs techniques de sécurité dans une application Qt6. Cet exemple montre comment implémenter une application de connexion sécurisée pour un système de gestion interne.

### Structure du projet

```
ApplicationSecurisee/
├── main.cpp
├── core/
│   ├── securitymanager.h
│   ├── securitymanager.cpp
│   ├── authservice.h
│   ├── authservice.cpp
│   └── encryptionservice.h
│   └── encryptionservice.cpp
├── ui/
│   ├── loginwindow.h
│   ├── loginwindow.cpp
│   ├── mainwindow.h
│   └── mainwindow.cpp
└── utils/
    ├── securitylog.h
    ├── securitylog.cpp
    ├── certificateutils.h
    └── certificateutils.cpp
```

### Implémentation

#### 1. Le point d'entrée (main.cpp)

```cpp
// main.cpp
#include <QApplication>
#include <QSslSocket>
#include <QMessageBox>
#include "core/securitymanager.h"
#include "ui/loginwindow.h"

int main(int argc, char *argv[]) {
    // Vérifier le support SSL avant tout
    if (!QSslSocket::supportsSsl()) {
        QMessageBox::critical(nullptr, "Erreur de sécurité",
            "Cette application nécessite le support SSL/TLS qui n'est pas disponible sur ce système.\n"
            "Veuillez installer les bibliothèques OpenSSL appropriées.");
        return 1;
    }

    QApplication app(argc, argv);
    app.setApplicationName("ApplicationSecurisee");
    app.setOrganizationName("MaCompagnie");

    // Initialiser le gestionnaire de sécurité
    SecurityManager securityManager;

    // Vérifier si l'application est en cours de débogage
    if (securityManager.isBeingDebugged()) {
        // En production, vous pourriez prendre des mesures contre le débogage
        // Pour ce tutoriel, nous affichons simplement un avertissement
        qWarning() << "Application en cours de débogage détectée !";
    }

    // Configurer la journalisation sécurisée
    SecurityLog::initialize();
    SecurityLog::instance()->log(SecurityLog::Information, "Application démarrée");

    // Vérifier l'intégrité des fichiers critiques
    if (!securityManager.verifyApplicationIntegrity()) {
        SecurityLog::instance()->log(SecurityLog::Alert, "Échec de vérification d'intégrité");
        QMessageBox::critical(nullptr, "Erreur de sécurité",
            "L'intégrité de l'application a été compromise.\n"
            "Veuillez réinstaller l'application à partir d'une source fiable.");
        return 1;
    }

    // Configurer TLS pour toutes les connexions réseau
    securityManager.configureSecureNetworking();

    // Afficher la fenêtre de connexion
    LoginWindow loginWindow;
    loginWindow.show();

    return app.exec();
}
```

#### 2. Gestionnaire de sécurité (securitymanager.h/cpp)

```cpp
// securitymanager.h
#pragma once

#include <QObject>
#include <QNetworkAccessManager>
#include <QSslConfiguration>

class SecurityManager : public QObject {
    Q_OBJECT

public:
    explicit SecurityManager(QObject* parent = nullptr);
    ~SecurityManager();

    // Vérification de débogueur
    bool isBeingDebugged() const;

    // Vérification d'intégrité
    bool verifyApplicationIntegrity() const;

    // Configuration réseau sécurisée
    void configureSecureNetworking();

    // Service de vérification des mises à jour
    void checkForSecurityUpdates();

private:
    QNetworkAccessManager* m_networkManager;
    QSslConfiguration m_secureConfig;

    // Liste des empreintes de fichiers critiques
    QMap<QString, QByteArray> m_fileHashes;

    // Charger les empreintes des fichiers
    void loadFileHashes();

    // Vérifier une mise à jour
    void verifyUpdate(const QByteArray& data, const QByteArray& signature);

private slots:
    void handleUpdateResponse();
};

// securitymanager.cpp
#include "securitymanager.h"
#include "../utils/securitylog.h"
#include <QCryptographicHash>
#include <QFile>
#include <QDir>
#include <QCoreApplication>
#include <QSslCipher>
#include <QNetworkRequest>
#include <QNetworkReply>
#include <QJsonDocument>
#include <QJsonObject>

#ifdef Q_OS_WINDOWS
#include <windows.h>
#endif

SecurityManager::SecurityManager(QObject* parent) : QObject(parent) {
    // Initialiser le gestionnaire réseau
    m_networkManager = new QNetworkAccessManager(this);

    // Charger les empreintes de fichiers connus
    loadFileHashes();

    // Configurer TLS
    m_secureConfig = QSslConfiguration::defaultConfiguration();
    m_secureConfig.setProtocol(QSsl::TlsV1_2OrLater);

    // Utiliser seulement des suites de chiffrement fortes
    QList<QSslCipher> secureCiphers;
    for (const QSslCipher& cipher : m_secureConfig.ciphers()) {
        if (cipher.keyExchangeMethod() != "DH" &&
            cipher.encryptionMethod().contains("AES") &&
            cipher.usedBits() >= 128) {
            secureCiphers.append(cipher);
        }
    }
    m_secureConfig.setCiphers(secureCiphers);
}

SecurityManager::~SecurityManager() {
    // Nettoyage sécurisé
}

bool SecurityManager::isBeingDebugged() const {
#ifdef Q_OS_WINDOWS
    return IsDebuggerPresent();
#elif defined(Q_OS_LINUX)
    // Code de détection pour Linux
    QFile file("/proc/self/status");
    if (file.open(QIODevice::ReadOnly | QIODevice::Text)) {
        QTextStream in(&file);
        QString line;
        while (in.readLineInto(&line)) {
            if (line.startsWith("TracerPid:")) {
                bool ok;
                int pid = line.split(':').value(1).trimmed().toInt(&ok);
                return ok && pid != 0;
            }
        }
    }
    return false;
#else
    return false;
#endif
}

bool SecurityManager::verifyApplicationIntegrity() const {
    for (auto it = m_fileHashes.begin(); it != m_fileHashes.end(); ++it) {
        QString filePath = it.key();
        QByteArray expectedHash = it.value();

        QFile file(filePath);
        if (!file.open(QIODevice::ReadOnly)) {
            SecurityLog::instance()->log(SecurityLog::Alert,
                QString("Fichier critique manquant: %1").arg(filePath));
            return false;
        }

        QByteArray fileData = file.readAll();
        QByteArray actualHash = QCryptographicHash::hash(fileData, QCryptographicHash::Sha256);

        if (actualHash != expectedHash) {
            SecurityLog::instance()->log(SecurityLog::Alert,
                QString("Intégrité compromise pour: %1").arg(filePath));
            return false;
        }
    }

    return true;
}

void SecurityManager::configureSecureNetworking() {
    // Appliquer la configuration TLS sécurisée
    QSslConfiguration::setDefaultConfiguration(m_secureConfig);

    // Configurer la validation de certificat
    connect(m_networkManager, &QNetworkAccessManager::sslErrors,
            [](QNetworkReply* reply, const QList<QSslError>& errors) {
        // Journaliser les erreurs SSL
        for (const QSslError& error : errors) {
            SecurityLog::instance()->log(SecurityLog::Avertissement,
                QString("Erreur SSL: %1").arg(error.errorString()));
        }

        // Ne pas ignorer les erreurs SSL automatiquement !
        // (Ne pas utiliser reply->ignoreSslErrors() ici)
    });
}

void SecurityManager::loadFileHashes() {
    // Dans une application réelle, ces empreintes seraient stockées
    // de manière sécurisée ou obtenues d'un serveur d'authentification
    // Pour ce tutoriel, nous utilisons des valeurs fictives

    QString appPath = QCoreApplication::applicationDirPath();
    m_fileHashes[appPath + "/ApplicationSecurisee.exe"] = QByteArray::fromHex("0123456789abcdef");
    m_fileHashes[appPath + "/ressources/config.dat"] = QByteArray::fromHex("fedcba9876543210");
}

void SecurityManager::checkForSecurityUpdates() {
    QUrl updateUrl("https://secure-updates.example.com/check");
    QNetworkRequest request(updateUrl);
    request.setSslConfiguration(m_secureConfig);

    // Ajouter des en-têtes anti-falsification
    request.setHeader(QNetworkRequest::UserAgentHeader, "ApplicationSecurisee/1.0");

    // Ajouter un nonce pour éviter les attaques par rejeu
    QString nonce = QString::number(QDateTime::currentMSecsSinceEpoch());
    request.setRawHeader("X-Nonce", nonce.toUtf8());

    // Créer la requête
    QJsonObject requestData;
    requestData["version"] = "1.0";
    requestData["timestamp"] = QDateTime::currentDateTime().toString(Qt::ISODate);

    QNetworkReply* reply = m_networkManager->post(request, QJsonDocument(requestData).toJson());

    connect(reply, &QNetworkReply::finished, this, &SecurityManager::handleUpdateResponse);
}

void SecurityManager::handleUpdateResponse() {
    QNetworkReply* reply = qobject_cast<QNetworkReply*>(sender());
    if (!reply) return;

    if (reply->error() == QNetworkReply::NoError) {
        QByteArray responseData = reply->readAll();

        // Analyser la réponse
        QJsonDocument doc = QJsonDocument::fromJson(responseData);
        QJsonObject response = doc.object();

        if (response["updateAvailable"].toBool()) {
            QByteArray updateData = QByteArray::fromBase64(response["updateData"].toString().toLatin1());
            QByteArray signature = QByteArray::fromBase64(response["signature"].toString().toLatin1());

            // Vérifier la mise à jour
            verifyUpdate(updateData, signature);
        }
    } else {
        SecurityLog::instance()->log(SecurityLog::Avertissement,
            QString("Échec de vérification des mises à jour: %1").arg(reply->errorString()));
    }

    reply->deleteLater();
}

void SecurityManager::verifyUpdate(const QByteArray& data, const QByteArray& signature) {
    // Vérifier la signature de la mise à jour
    // (Code de vérification cryptographique)

    // Si valide, installer la mise à jour
    // ...
}
```

#### 3. Service d'authentification (authservice.h/cpp)

```cpp
// authservice.h
#pragma once

#include <QObject>
#include <QString>
#include <QByteArray>

class AuthService : public QObject {
    Q_OBJECT

public:
    explicit AuthService(QObject* parent = nullptr);

    // Enregistrement d'un nouvel utilisateur (simplifiée pour l'exemple)
    bool registerUser(const QString& username, const QString& password);

    // Authentification
    bool authenticate(const QString& username, const QString& password);

    // Verrouillage de compte
    bool isAccountLocked(const QString& username) const;

private:
    // Stockage sécurisé des informations d'authentification
    struct UserAuth {
        QByteArray passwordHash;
        QByteArray salt;
        int failedAttempts;
        QDateTime lastFailedAttempt;
    };

    QMap<QString, UserAuth> m_users;

    // Générer un sel aléatoire
    QByteArray generateSalt() const;

    // Hacher un mot de passe avec sel
    QByteArray hashPassword(const QString& password, const QByteArray& salt) const;

    // Vérifier le verrouillage
    void checkAndUpdateLockStatus(const QString& username, bool authSuccess);
};

// authservice.cpp
#include "authservice.h"
#include "../utils/securitylog.h"
#include <QCryptographicHash>
#include <QDateTime>
#include <QSettings>
#include <QUuid>

AuthService::AuthService(QObject* parent) : QObject(parent) {
    // Charger les utilisateurs depuis le stockage sécurisé
    QSettings settings;
    int userCount = settings.beginReadArray("users");

    for (int i = 0; i < userCount; ++i) {
        settings.setArrayIndex(i);
        QString username = settings.value("username").toString();

        UserAuth auth;
        auth.passwordHash = settings.value("passwordHash").toByteArray();
        auth.salt = settings.value("salt").toByteArray();
        auth.failedAttempts = settings.value("failedAttempts", 0).toInt();
        auth.lastFailedAttempt = settings.value("lastFailedAttempt").toDateTime();

        m_users[username] = auth;
    }

    settings.endArray();
}

bool AuthService::registerUser(const QString& username, const QString& password) {
    // Vérifier que le nom d'utilisateur est valide
    if (username.isEmpty() || username.length() < 3) {
        return false;
    }

    // Vérifier que le mot de passe est suffisamment fort
    if (password.length() < 8 || !password.contains(QRegularExpression("[A-Z]")) ||
        !password.contains(QRegularExpression("[a-z]")) || !password.contains(QRegularExpression("[0-9]"))) {
        return false;
    }

    // Vérifier si l'utilisateur existe déjà
    if (m_users.contains(username)) {
        return false;
    }

    // Créer un nouvel utilisateur
    UserAuth auth;
    auth.salt = generateSalt();
    auth.passwordHash = hashPassword(password, auth.salt);
    auth.failedAttempts = 0;

    m_users[username] = auth;

    // Enregistrer dans le stockage sécurisé
    QSettings settings;
    int userCount = m_users.size();

    settings.beginWriteArray("users", userCount);
    int index = 0;
    for (auto it = m_users.begin(); it != m_users.end(); ++it, ++index) {
        settings.setArrayIndex(index);
        settings.setValue("username", it.key());
        settings.setValue("passwordHash", it.value().passwordHash);
        settings.setValue("salt", it.value().salt);
        settings.setValue("failedAttempts", it.value().failedAttempts);
        settings.setValue("lastFailedAttempt", it.value().lastFailedAttempt);
    }
    settings.endArray();

    SecurityLog::instance()->log(SecurityLog::Information,
        QString("Nouvel utilisateur enregistré: %1").arg(username));

    return true;
}

bool AuthService::authenticate(const QString& username, const QString& password) {
    // Vérifier si l'utilisateur existe
    if (!m_users.contains(username)) {
        SecurityLog::instance()->log(SecurityLog::Avertissement,
            QString("Tentative d'authentification avec un utilisateur inconnu: %1").arg(username));
        return false;
    }

    // Vérifier si le compte est verrouillé
    if (isAccountLocked(username)) {
        SecurityLog::instance()->log(SecurityLog::Avertissement,
            QString("Tentative d'authentification sur un compte verrouillé: %1").arg(username));
        return false;
    }

    const UserAuth& auth = m_users[username];

    // Vérifier le mot de passe
    QByteArray hash = hashPassword(password, auth.salt);
    bool success = (hash == auth.passwordHash);

    // Mettre à jour le statut de verrouillage
    checkAndUpdateLockStatus(username, success);

    if (success) {
        SecurityLog::instance()->log(SecurityLog::Information,
            QString("Authentification réussie: %1").arg(username));
    } else {
        SecurityLog::instance()->log(SecurityLog::Avertissement,
            QString("Échec d'authentification: %1").arg(username));
    }

    return success;
}

bool AuthService::isAccountLocked(const QString& username) const {
    if (!m_users.contains(username)) {
        return false;
    }

    const UserAuth& auth = m_users[username];

    // Vérifier si trop de tentatives échouées récentes
    if (auth.failedAttempts >= 5) {
        // Vérifier si le temps de verrouillage est écoulé (30 minutes)
        QDateTime lockoutEnd = auth.lastFailedAttempt.addSecs(30 * 60);
        if (QDateTime::currentDateTime() < lockoutEnd) {
            return true;
        }
    }

    return false;
}

QByteArray AuthService::generateSalt() const {
    // Générer un sel aléatoire
    return QUuid::createUuid().toByteArray();
}

QByteArray AuthService::hashPassword(const QString& password, const QByteArray& salt) const {
    // Combiner le mot de passe et le sel
    QByteArray passwordData = password.toUtf8();
    QByteArray combined = passwordData + salt;

    // Utiliser PBKDF2 ou une fonction similaire (simplifié pour l'exemple)
    QByteArray hash = combined;
    for (int i = 0; i < 10000; ++i) {
        hash = QCryptographicHash::hash(hash, QCryptographicHash::Sha256);
    }

    return hash;
}

void AuthService::checkAndUpdateLockStatus(const QString& username, bool authSuccess) {
    if (!m_users.contains(username)) {
        return;
    }

    UserAuth& auth = m_users[username];

    if (authSuccess) {
        // Réinitialiser les tentatives échouées
        auth.failedAttempts = 0;
    } else {
        // Incrémenter les tentatives échouées
        auth.failedAttempts++;
        auth.lastFailedAttempt = QDateTime::currentDateTime();

        // Enregistrer les modifications
        QSettings settings;
        settings.beginGroup("users");
        settings.beginGroup(username);
        settings.setValue("failedAttempts", auth.failedAttempts);
        settings.setValue("lastFailedAttempt", auth.lastFailedAttempt);
        settings.endGroup();
        settings.endGroup();

        // Vérifier si le compte doit être verrouillé
        if (auth.failedAttempts >= 5) {
            SecurityLog::instance()->log(SecurityLog::Alerte,
                QString("Compte verrouillé après %1 tentatives échouées: %2")
                .arg(auth.failedAttempts).arg(username));
        }
    }
}
```

#### 4. Service de chiffrement (encryptionservice.h/cpp)

```cpp
// encryptionservice.h
#pragma once

#include <QObject>
#include <QString>
#include <QByteArray>

class EncryptionService : public QObject {
    Q_OBJECT

public:
    explicit EncryptionService(QObject* parent = nullptr);

    // Générer une clé de chiffrement
    QByteArray generateKey() const;

    // Chiffrer des données
    QByteArray encrypt(const QByteArray& data, const QByteArray& key) const;

    // Déchiffrer des données
    QByteArray decrypt(const QByteArray& encryptedData, const QByteArray& key) const;

    // Chiffrer un fichier
    bool encryptFile(const QString& sourceFile, const QString& destFile, const QByteArray& key) const;

    // Déchiffrer un fichier
    bool decryptFile(const QString& sourceFile, const QString& destFile, const QByteArray& key) const;
};

// encryptionservice.cpp
#include "encryptionservice.h"
#include "../utils/securitylog.h"
#include <QFile>
#include <QCryptographicHash>
#include <QRandomGenerator>

EncryptionService::EncryptionService(QObject* parent) : QObject(parent) {
    // Initialiser le service de chiffrement
}

QByteArray EncryptionService::generateKey() const {
    // Générer une clé aléatoire forte
    QByteArray key;
    key.resize(32); // 256 bits

    // Remplir avec des données aléatoires
    for (int i = 0; i < key.size(); ++i) {
        key[i] = static_cast<char>(QRandomGenerator::global()->bounded(256));
    }

    return key;
}

QByteArray EncryptionService::encrypt(const QByteArray& data, const QByteArray& key) const {
    // Pour un tutoriel simple, nous utilisons une implémentation basique
    // Dans une application réelle, utilisez une bibliothèque cryptographique éprouvée

    // Créer un vecteur d'initialisation aléatoire (IV)
    QByteArray iv;
    iv.resize(16); // 128 bits
    for (int i = 0; i < iv.size(); ++i) {
        iv[i] = static_cast<char>(QRandomGenerator::global()->bounded(256));
    }

    // Dans une application réelle, utilisez un algorithme comme AES-GCM
    // avec une bibliothèque comme OpenSSL ou Qt Cryptographic Architecture

    // Simuler le chiffrement (ceci est une simulation simplifiée et NON SÉCURISÉE)
    QByteArray keyHash = QCryptographicHash::hash(key, QCryptographicHash::Sha256);
    QByteArray result = iv; // Ajouter l'IV au début des données chiffrées

    // XOR basic (NE PAS UTILISER EN PRODUCTION)
    for (int i = 0; i < data.size(); ++i) {
        result.append(data[i] ^ keyHash[i % keyHash.size()]);
    }

    return result;
}

QByteArray EncryptionService::decrypt(const QByteArray& encryptedData, const QByteArray& key) const {
    // Vérifier la taille minimale (doit contenir au moins l'IV)
    if (encryptedData.size() <= 16) {
        return QByteArray();
    }

    // Extraire le vecteur d'initialisation
    QByteArray iv = encryptedData.left(16);
    QByteArray encData = encryptedData.mid(16);

    // Dans une application réelle, utilisez le même algorithme que pour le chiffrement

    // Simuler le déchiffrement (ceci est une simulation simplifiée et NON SÉCURISÉE)
    QByteArray keyHash = QCryptographicHash::hash(key, QCryptographicHash::Sha256);
    QByteArray result;

    // XOR basic (NE PAS UTILISER EN PRODUCTION)
    for (int i = 0; i < encData.size(); ++i) {
        result.append(encData[i] ^ keyHash[i % keyHash.size()]);
    }

    return result;
}

bool EncryptionService::encryptFile(const QString& sourceFile, const QString& destFile, const QByteArray& key) const {
    QFile source(sourceFile);
    if (!source.open(QIODevice::ReadOnly)) {
        SecurityLog::instance()->log(SecurityLog::Avertissement,
            QString("Échec d'ouverture du fichier source pour chiffrement: %1").arg(sourceFile));
        return false;
    }

    QByteArray data = source.readAll();
    source.close();

    QByteArray encryptedData = encrypt(data, key);

    QFile dest(destFile);
    if (!dest.open(QIODevice::WriteOnly)) {
        SecurityLog::instance()->log(SecurityLog::Avertissement,
            QString("Échec d'ouverture du fichier destination pour chiffrement: %1").arg(destFile));
        return false;
    }

    dest.write(encryptedData);
    dest.close();

    SecurityLog::instance()->log(SecurityLog::Information,
        QString("Fichier chiffré: %1 -> %2").arg(sourceFile).arg(destFile));

    return true;
}

bool EncryptionService::decryptFile(const QString& sourceFile, const QString& destFile, const QByteArray& key) const {
    QFile source(sourceFile);
    if (!source.open(QIODevice::ReadOnly)) {
        SecurityLog::instance()->log(SecurityLog::Avertissement,
            QString("Échec d'ouverture du fichier source pour déchiffrement: %1").arg(sourceFile));
        return false;
    }

    QByteArray encryptedData = source.readAll();
    source.close();

    QByteArray data = decrypt(encryptedData, key);

    QFile dest(destFile);
    if (!dest.open(QIODevice::WriteOnly)) {
        SecurityLog::instance()->log(SecurityLog::Avertissement,
            QString("Échec d'ouverture du fichier destination pour déchiffrement: %1").arg(destFile));
        return false;
    }

    dest.write(data);
    dest.close();

    SecurityLog::instance()->log(SecurityLog::Information,
        QString("Fichier déchiffré: %1 -> %2").arg(sourceFile).arg(destFile));

    return true;
}
```

#### 5. Interface utilisateur de connexion (loginwindow.h/cpp)

```cpp
// loginwindow.h
#pragma once

#include <QMainWindow>
#include <QLineEdit>
#include <QPushButton>
#include <QLabel>
#include "../core/authservice.h"

class LoginWindow : public QMainWindow {
    Q_OBJECT

public:
    explicit LoginWindow(QWidget* parent = nullptr);

private slots:
    void onLoginButtonClicked();
    void onRegisterButtonClicked();

private:
    QLineEdit* m_usernameEdit;
    QLineEdit* m_passwordEdit;
    QPushButton* m_loginButton;
    QPushButton* m_registerButton;
    QLabel* m_statusLabel;

    AuthService* m_authService;

    // Délai pour ralentir les tentatives d'authentification
    void secureDelay();
};

// loginwindow.cpp
#include "loginwindow.h"
#include "mainwindow.h"
#include "../utils/securitylog.h"
#include <QVBoxLayout>
#include <QHBoxLayout>
#include <QFormLayout>
#include <QMessageBox>
#include <QTimer>
#include <QRegularExpressionValidator>

LoginWindow::LoginWindow(QWidget* parent) : QMainWindow(parent) {
    setWindowTitle("Connexion sécurisée");
    setMinimumWidth(400);

    // Créer le service d'authentification
    m_authService = new AuthService(this);

    // Créer les widgets
    QWidget* centralWidget = new QWidget(this);
    setCentralWidget(centralWidget);

    QVBoxLayout* mainLayout = new QVBoxLayout(centralWidget);

    QFormLayout* formLayout = new QFormLayout();

    m_usernameEdit = new QLineEdit();
    m_passwordEdit = new QLineEdit();
    m_passwordEdit->setEchoMode(QLineEdit::Password);

    // Ajouter des validateurs pour les entrées
    QRegularExpression usernameRegex("^[a-zA-Z0-9_]{3,16}$");
    m_usernameEdit->setValidator(new QRegularExpressionValidator(usernameRegex, this));

    formLayout->addRow("Nom d'utilisateur:", m_usernameEdit);
    formLayout->addRow("Mot de passe:", m_passwordEdit);

    mainLayout->addLayout(formLayout);

    QHBoxLayout* buttonLayout = new QHBoxLayout();

    m_loginButton = new QPushButton("Connexion");
    m_registerButton = new QPushButton("S'inscrire");

    buttonLayout->addWidget(m_loginButton);
    buttonLayout->addWidget(m_registerButton);

    mainLayout->addLayout(buttonLayout);

    m_statusLabel = new QLabel();
    m_statusLabel->setAlignment(Qt::AlignCenter);
    mainLayout->addWidget(m_statusLabel);

    // Connecter les signaux
    connect(m_loginButton, &QPushButton::clicked, this, &LoginWindow::onLoginButtonClicked);
    connect(m_registerButton, &QPushButton::clicked, this, &LoginWindow::onRegisterButtonClicked);
    connect(m_passwordEdit, &QLineEdit::returnPressed, this, &LoginWindow::onLoginButtonClicked);
}

void LoginWindow::onLoginButtonClicked() {
    QString username = m_usernameEdit->text();
    QString password = m_passwordEdit->text();

    // Vérifier si les champs sont vides
    if (username.isEmpty() || password.isEmpty()) {
        m_statusLabel->setText("Veuillez remplir tous les champs");
        m_statusLabel->setStyleSheet("color: red;");
        return;
    }

    // Désactiver les boutons pendant la vérification
    m_loginButton->setEnabled(false);
    m_registerButton->setEnabled(false);
    m_statusLabel->setText("Vérification...");

    // Délai de sécurité pour ralentir les attaques par force brute
    secureDelay();

    // Vérifier si le compte est verrouillé
    if (m_authService->isAccountLocked(username)) {
        m_statusLabel->setText("Ce compte est temporairement verrouillé. Veuillez réessayer plus tard.");
        m_statusLabel->setStyleSheet("color: red;");

        // Réactiver les boutons
        m_loginButton->setEnabled(true);
        m_registerButton->setEnabled(true);

        return;
    }

    // Authentifier l'utilisateur
    bool success = m_authService->authenticate(username, password);

    if (success) {
        // Connexion réussie
        m_statusLabel->setText("Connexion réussie!");
        m_statusLabel->setStyleSheet("color: green;");

        // Ouvrir la fenêtre principale
        MainWindow* mainWindow = new MainWindow(username);
        mainWindow->show();

        // Fermer la fenêtre de connexion
        QTimer::singleShot(500, this, &LoginWindow::close);
    } else {
        // Échec de connexion
        m_statusLabel->setText("Nom d'utilisateur ou mot de passe incorrect.");
        m_statusLabel->setStyleSheet("color: red;");

        // Effacer le mot de passe
        m_passwordEdit->clear();

        // Réactiver les boutons
        m_loginButton->setEnabled(true);
        m_registerButton->setEnabled(true);
    }
}

void LoginWindow::onRegisterButtonClicked() {
    QString username = m_usernameEdit->text();
    QString password = m_passwordEdit->text();

    // Vérifier si les champs sont vides
    if (username.isEmpty() || password.isEmpty()) {
        m_statusLabel->setText("Veuillez remplir tous les champs");
        m_statusLabel->setStyleSheet("color: red;");
        return;
    }

    // Vérifier que le mot de passe est suffisamment fort
    if (password.length() < 8) {
        m_statusLabel->setText("Le mot de passe doit contenir au moins 8 caractères");
        m_statusLabel->setStyleSheet("color: red;");
        return;
    }

    if (!password.contains(QRegularExpression("[A-Z]"))) {
        m_statusLabel->setText("Le mot de passe doit contenir au moins une majuscule");
        m_statusLabel->setStyleSheet("color: red;");
        return;
    }

    if (!password.contains(QRegularExpression("[a-z]"))) {
        m_statusLabel->setText("Le mot de passe doit contenir au moins une minuscule");
        m_statusLabel->setStyleSheet("color: red;");
        return;
    }

    if (!password.contains(QRegularExpression("[0-9]"))) {
        m_statusLabel->setText("Le mot de passe doit contenir au moins un chiffre");
        m_statusLabel->setStyleSheet("color: red;");
        return;
    }

    // Désactiver les boutons pendant l'enregistrement
    m_loginButton->setEnabled(false);
    m_registerButton->setEnabled(false);
    m_statusLabel->setText("Création du compte...");

    // Délai de sécurité pour ralentir les attaques par force brute
    secureDelay();

    // Enregistrer l'utilisateur
    bool success = m_authService->registerUser(username, password);

    if (success) {
        // Enregistrement réussi
        m_statusLabel->setText("Compte créé avec succès! Vous pouvez vous connecter.");
        m_statusLabel->setStyleSheet("color: green;");

        // Effacer le mot de passe
        m_passwordEdit->clear();
    } else {
        // Échec d'enregistrement
        m_statusLabel->setText("Ce nom d'utilisateur est déjà utilisé ou invalide.");
        m_statusLabel->setStyleSheet("color: red;");
    }

    // Réactiver les boutons
    m_loginButton->setEnabled(true);
    m_registerButton->setEnabled(true);
}

void LoginWindow::secureDelay() {
    // Introduire un délai constant pour éviter les attaques temporelles
    // et ralentir les tentatives de force brute
    QElapsedTimer timer;
    timer.start();

    // Attendre au moins 1 seconde, indépendamment de la réussite ou l'échec
    while (timer.elapsed() < 1000) {
        QCoreApplication::processEvents(QEventLoop::AllEvents, 100);
    }
}
```

#### 6. Journal de sécurité (securitylog.h/cpp)

```cpp
// securitylog.h
#pragma once

#include <QObject>
#include <QString>
#include <QFile>
#include <QTextStream>
#include <QMutex>
#include <QDateTime>

class SecurityLog : public QObject {
    Q_OBJECT

public:
    enum LogLevel {
        Information,
        Avertissement,
        Alerte
    };

    // Singleton
    static SecurityLog* instance();
    static void initialize();

    // Journalisation
    void log(LogLevel level, const QString& message);

private:
    explicit SecurityLog(QObject* parent = nullptr);
    ~SecurityLog();

    // Instance unique
    static SecurityLog* s_instance;

    // Fichier de journal
    QFile m_logFile;
    QTextStream m_stream;

    // Mutex pour l'accès concurrent
    QMutex m_mutex;

    // Convertir le niveau en texte
    QString levelToString(LogLevel level) const;
};

// securitylog.cpp
#include "securitylog.h"
#include <QStandardPaths>
#include <QDir>
#include <QCoreApplication>
#include <QFileInfo>

// Initialisation de l'instance statique
SecurityLog* SecurityLog::s_instance = nullptr;

SecurityLog* SecurityLog::instance() {
    if (!s_instance) {
        initialize();
    }
    return s_instance;
}

void SecurityLog::initialize() {
    if (!s_instance) {
        s_instance = new SecurityLog();
    }
}

SecurityLog::SecurityLog(QObject* parent) : QObject(parent) {
    // Créer le répertoire des journaux si nécessaire
    QString logDir = QStandardPaths::writableLocation(QStandardPaths::AppDataLocation)
                    + "/logs";
    QDir().mkpath(logDir);

    // Nom du fichier avec la date
    QString logFilename = logDir + "/security_"
                         + QDateTime::currentDateTime().toString("yyyy-MM-dd")
                         + ".log";

    // Ouvrir le fichier journal
    m_logFile.setFileName(logFilename);
    bool fileExists = m_logFile.exists();

    if (m_logFile.open(QIODevice::WriteOnly | QIODevice::Append | QIODevice::Text)) {
        m_stream.setDevice(&m_logFile);

        // Ajouter un en-tête au début d'un nouveau fichier
        if (!fileExists) {
            m_stream << "========== JOURNAL DE SÉCURITÉ ==========" << Qt::endl;
            m_stream << "Application: " << QCoreApplication::applicationName() << Qt::endl;
            m_stream << "Date: " << QDateTime::currentDateTime().toString(Qt::ISODate) << Qt::endl;
            m_stream << "=========================================" << Qt::endl;
        }

        // Journaliser le démarrage
        m_stream << Qt::endl << "--- Session démarrée: "
                << QDateTime::currentDateTime().toString(Qt::ISODate) << " ---" << Qt::endl;
        m_stream.flush();
    }
}

SecurityLog::~SecurityLog() {
    // Journaliser la fin de session
    QMutexLocker locker(&m_mutex);

    if (m_logFile.isOpen()) {
        m_stream << "--- Session terminée: "
                << QDateTime::currentDateTime().toString(Qt::ISODate) << " ---" << Qt::endl;
        m_stream.flush();
        m_logFile.close();
    }
}

void SecurityLog::log(LogLevel level, const QString& message) {
    QMutexLocker locker(&m_mutex);

    if (!m_logFile.isOpen()) {
        return;
    }

    // Horodatage
    QString timestamp = QDateTime::currentDateTime().toString("yyyy-MM-dd hh:mm:ss.zzz");

    // Niveau en texte
    QString levelStr = levelToString(level);

    // Écrire l'entrée
    m_stream << QString("[%1] [%2] %3")
                .arg(timestamp)
                .arg(levelStr.leftJustified(5))
                .arg(message)
             << Qt::endl;
    m_stream.flush();

    // En mode débug, afficher aussi dans la console
#ifdef QT_DEBUG
    qDebug() << QString("[SECURITY] [%1] %2").arg(levelStr).arg(message);
#endif
}

QString SecurityLog::levelToString(LogLevel level) const {
    switch (level) {
        case Information: return "INFO";
        case Avertissement: return "AVERT";
        case Alerte: return "ALERT";
        default: return "UNKN";
    }
}
```

#### 7. Utilisation de l'application sécurisée

Voici comment l'application pourrait être utilisée dans un cas concret :

```cpp
// Utilisation sécurisée de l'application pour stocker des données sensibles
void MainWindow::sauvegarderDonneesConfidentielles() {
    // Récupérer les données à protéger
    QString donneesConfidentielles = ui->noteConfidentielleEdit->toPlainText();

    if (donneesConfidentielles.isEmpty()) {
        QMessageBox::warning(this, "Avertissement", "Aucune donnée à sauvegarder.");
        return;
    }

    // Service de chiffrement
    EncryptionService* encryptionService = new EncryptionService(this);

    // Demander un mot de passe pour le chiffrement
    bool ok;
    QString motDePasse = QInputDialog::getText(this, "Protection",
        "Entrez un mot de passe pour protéger ces données:",
        QLineEdit::Password, QString(), &ok);

    if (!ok || motDePasse.isEmpty()) {
        return;
    }

    // Dériver une clé de chiffrement du mot de passe
    QByteArray sel = QUuid::createUuid().toByteArray();
    QByteArray cle = QCryptographicHash::hash(motDePasse.toUtf8() + sel,
                                             QCryptographicHash::Sha256);

    // Chiffrer les données
    QByteArray donneesBrutes = donneesConfidentielles.toUtf8();
    QByteArray donneesChiffrees = encryptionService->encrypt(donneesBrutes, cle);

    // Choisir l'emplacement de sauvegarde
    QString cheminFichier = QFileDialog::getSaveFileName(this, "Sauvegarder le fichier sécurisé",
        QStandardPaths::writableLocation(QStandardPaths::DocumentsLocation) + "/donnees_securisees.enc",
        "Fichiers chiffrés (*.enc)");

    if (cheminFichier.isEmpty()) {
        return;
    }

    // Sauvegarder les données chiffrées
    QFile fichier(cheminFichier);
    if (fichier.open(QIODevice::WriteOnly)) {
        // Sauvegarder le sel au début du fichier
        fichier.write(sel);
        fichier.write(donneesChiffrees);
        fichier.close();

        SecurityLog::instance()->log(SecurityLog::Information,
            "Données confidentielles sauvegardées de manière sécurisée");

        QMessageBox::information(this, "Succès",
            "Les données ont été chiffrées et sauvegardées avec succès.");
    } else {
        SecurityLog::instance()->log(SecurityLog::Avertissement,
            "Échec de sauvegarde des données confidentielles: " + fichier.errorString());

        QMessageBox::critical(this, "Erreur",
            "Impossible de sauvegarder le fichier: " + fichier.errorString());
    }
}

void MainWindow::ouvrirDonneesConfidentielles() {
    // Choisir le fichier à ouvrir
    QString cheminFichier = QFileDialog::getOpenFileName(this, "Ouvrir un fichier sécurisé",
        QStandardPaths::writableLocation(QStandardPaths::DocumentsLocation),
        "Fichiers chiffrés (*.enc)");

    if (cheminFichier.isEmpty()) {
        return;
    }

    // Lire le fichier chiffré
    QFile fichier(cheminFichier);
    if (!fichier.open(QIODevice::ReadOnly)) {
        SecurityLog::instance()->log(SecurityLog::Avertissement,
            "Échec d'ouverture du fichier chiffré: " + fichier.errorString());

        QMessageBox::critical(this, "Erreur",
            "Impossible d'ouvrir le fichier: " + fichier.errorString());
        return;
    }

    // Lire le contenu
    QByteArray contenu = fichier.readAll();
    fichier.close();

    // Vérifier la taille minimale
    if (contenu.size() <= 16) {
        QMessageBox::critical(this, "Erreur", "Le fichier est corrompu ou vide.");
        return;
    }

    // Extraire le sel
    QByteArray sel = contenu.left(16);
    QByteArray donneesChiffrees = contenu.mid(16);

    // Demander le mot de passe
    bool ok;
    QString motDePasse = QInputDialog::getText(this, "Déchiffrement",
        "Entrez le mot de passe pour déchiffrer ces données:",
        QLineEdit::Password, QString(), &ok);

    if (!ok || motDePasse.isEmpty()) {
        return;
    }

    // Dériver la clé
    QByteArray cle = QCryptographicHash::hash(motDePasse.toUtf8() + sel,
                                             QCryptographicHash::Sha256);

    // Service de chiffrement
    EncryptionService* encryptionService = new EncryptionService(this);

    // Déchiffrer les données
    QByteArray donneesBrutes = encryptionService->decrypt(donneesChiffrees, cle);

    // Vérifier si le déchiffrement a réussi
    if (donneesBrutes.isEmpty()) {
        SecurityLog::instance()->log(SecurityLog::Avertissement,
            "Tentative de déchiffrement échouée, mot de passe probablement incorrect");

        QMessageBox::critical(this, "Erreur",
            "Impossible de déchiffrer les données. Le mot de passe est peut-être incorrect.");
        return;
    }

    // Afficher les données déchiffrées
    QString donneesTexte = QString::fromUtf8(donneesBrutes);
    ui->noteConfidentielleEdit->setPlainText(donneesTexte);

    SecurityLog::instance()->log(SecurityLog::Information,
        "Données confidentielles déchiffrées avec succès");

    QMessageBox::information(this, "Succès",
        "Les données ont été déchiffrées avec succès.");
}
```

## Conclusion et résumé des bonnes pratiques

Notre exemple d'application sécurisée intègre de nombreuses bonnes pratiques :

1. **Validation des entrées** : Les champs de saisie sont validés pour éviter les injections et les formats invalides.

2. **Hachage sécurisé des mots de passe** : Utilisation de sels aléatoires et de fonctions de hachage itératives.

3. **Protection contre les attaques par force brute** : Verrouillage de compte après plusieurs échecs et délais constants.

4. **Chiffrement des données sensibles** : Utilisation de techniques de chiffrement modernes (dans une vraie application, utilisez des bibliothèques éprouvées comme OpenSSL).

5. **Gestion sécurisée des sessions** : Les tentatives d'authentification sont journalisées et surveillées.

6. **Vérification d'intégrité** : L'application vérifie son intégrité au démarrage.

7. **Communication réseau sécurisée** : Configuration de TLS robuste pour les communications.

8. **Journalisation de sécurité** : Toutes les actions sensibles sont enregistrées avec horodatage.

9. **Protection contre le débogage** : L'application détecte les tentatives de débogage.

10. **Gestion sécurisée des mises à jour** : Vérification cryptographique des mises à jour.

Ces principes peuvent être adaptés à n'importe quelle application Qt6, qu'il s'agisse d'une application de bureau, mobile ou embarquée.

Il est important de noter que la sécurité est un processus continu, pas un état final. Restez informé des nouvelles vulnérabilités et mettez régulièrement à jour vos pratiques de sécurité et vos bibliothèques.

Dans un environnement de production réel, n'oubliez pas de faire auditer votre code par des experts en sécurité et de réaliser des tests de pénétration pour identifier les éventuelles failles.


## Bonnes pratiques pour débutants

Pour les débutants qui souhaitent intégrer la sécurité dans leurs applications Qt, voici quelques conseils simples pour commencer :

1. **Commencez petit** : N'essayez pas d'implémenter toutes les mesures de sécurité en même temps. Commencez par les plus fondamentales comme la validation des entrées et le stockage sécurisé des mots de passe.

2. **Utilisez des bibliothèques éprouvées** : Pour le chiffrement et les fonctions cryptographiques, utilisez des bibliothèques bien établies plutôt que d'implémenter vos propres algorithmes.

3. **Pensez à la sécurité dès le début** : Il est plus facile d'intégrer la sécurité pendant la conception qu'après coup.

4. **Journalisez les événements** : Une bonne journalisation facilite le débogage et l'identification des problèmes de sécurité.

5. **Mettez à jour vos dépendances** : Les vieilles versions des bibliothèques peuvent contenir des vulnérabilités connues.

6. **Testez, testez, testez** : Essayez activement de "casser" votre propre application pour identifier les faiblesses.

Rappelez-vous que sécuriser une application est un équilibre entre la protection des données et l'expérience utilisateur. L'objectif est de créer une application qui soit à la fois sécurisée et conviviale.

🔝 Retour à la [Table des matières](/SOMMAIRE.md)
