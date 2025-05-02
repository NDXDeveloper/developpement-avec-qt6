# 8.4 Gestion des erreurs et exceptions

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

La gestion des erreurs est une partie essentielle du développement logiciel. Une application robuste doit être capable de détecter les problèmes, de les signaler clairement et, si possible, de les gérer élégamment. Dans ce chapitre, nous allons explorer les différentes approches de gestion des erreurs en Qt6, des plus simples aux plus sophistiquées.

## Comprendre les erreurs et les exceptions

Avant de plonger dans les détails techniques, clarifions quelques concepts fondamentaux :

- **Erreur** : Une condition anormale ou inattendue qui survient pendant l'exécution de votre programme.
- **Exception** : Un mécanisme de langage C++ pour signaler et gérer les erreurs, permettant de "lancer" (throw) une erreur et de la "capturer" (catch) ailleurs dans le code.
- **Gestion des erreurs** : L'ensemble des techniques utilisées pour détecter, signaler et répondre aux erreurs.

## Approches de gestion des erreurs en Qt

Qt propose plusieurs mécanismes pour gérer les erreurs. Chacun a ses avantages et ses cas d'utilisation appropriés :

### 1. Codes de retour et vérification de validité

L'approche la plus simple consiste à utiliser des valeurs de retour pour indiquer le succès ou l'échec :

```cpp
bool sauvegarderFichier(const QString &chemin, const QByteArray &donnees)
{
    QFile fichier(chemin);
    if (!fichier.open(QIODevice::WriteOnly)) {
        qWarning() << "Impossible d'ouvrir le fichier pour écriture:" << chemin;
        return false;
    }

    qint64 octetsEcrits = fichier.write(donnees);
    if (octetsEcrits != donnees.size()) {
        qWarning() << "Écriture incomplète:" << octetsEcrits << "sur" << donnees.size();
        return false;
    }

    return true;
}
```

De nombreuses classes Qt fournissent également des méthodes pour vérifier la validité d'une opération :

```cpp
QFile fichier("document.txt");
if (!fichier.exists()) {
    qDebug() << "Le fichier n'existe pas";
}

QNetworkReply *reponse = manager->get(requete);
if (reponse->error() != QNetworkReply::NoError) {
    qDebug() << "Erreur réseau:" << reponse->errorString();
}
```

#### Avantages
- Simple à comprendre et à utiliser
- Pas d'impact sur les performances
- Style cohérent avec beaucoup d'API Qt existantes

#### Inconvénients
- Facile d'oublier de vérifier les valeurs de retour
- L'information d'erreur peut être limitée
- Peut conduire à du code imbriqué profondément

### 2. Le système de journalisation de Qt

Qt fournit un système de journalisation puissant qui vous permet de classifier les messages en fonction de leur gravité :

```cpp
#include <QDebug>

void traiterDonnees(const QByteArray &donnees)
{
    if (donnees.isEmpty()) {
        qWarning() << "Les données fournies sont vides";
        return;
    }

    if (!formatValide(donnees)) {
        qCritical() << "Format de données invalide";
        return;
    }

    try {
        // Traitement des données...
        qDebug() << "Traitement terminé avec succès";
    } catch (const std::exception &e) {
        qCritical() << "Erreur lors du traitement:" << e.what();
    }
}
```

Les différents niveaux de journalisation sont :
- `qDebug()` : Informations de débogage (désactivées en mode release)
- `qInfo()` : Informations générales
- `qWarning()` : Avertissements (quelque chose ne va pas, mais le programme peut continuer)
- `qCritical()` : Erreurs critiques (problèmes graves affectant la fonctionnalité)
- `qFatal()` : Erreurs fatales (le programme va s'arrêter)

#### Personnalisation du gestionnaire de messages

Vous pouvez personnaliser la façon dont ces messages sont traités :

```cpp
void monGestionnaireMessages(QtMsgType type, const QMessageLogContext &context, const QString &msg)
{
    QByteArray localMsg = msg.toLocal8Bit();
    const char *file = context.file ? context.file : "";
    const char *function = context.function ? context.function : "";

    switch (type) {
    case QtDebugMsg:
        fprintf(stderr, "Debug: %s (%s:%u, %s)\n", localMsg.constData(), file, context.line, function);
        break;
    case QtInfoMsg:
        fprintf(stderr, "Info: %s (%s:%u, %s)\n", localMsg.constData(), file, context.line, function);
        break;
    case QtWarningMsg:
        fprintf(stderr, "Warning: %s (%s:%u, %s)\n", localMsg.constData(), file, context.line, function);
        break;
    case QtCriticalMsg:
        fprintf(stderr, "Critical: %s (%s:%u, %s)\n", localMsg.constData(), file, context.line, function);
        break;
    case QtFatalMsg:
        fprintf(stderr, "Fatal: %s (%s:%u, %s)\n", localMsg.constData(), file, context.line, function);
        abort();
    }
}

int main(int argc, char *argv[])
{
    qInstallMessageHandler(monGestionnaireMessages);
    QApplication app(argc, argv);
    // ...
    return app.exec();
}
```

### 3. Gestion des exceptions en Qt

Bien que Qt lui-même n'utilise pas beaucoup les exceptions, vous pouvez les utiliser dans votre code Qt. Voici un exemple simple :

```cpp
#include <QException>

// Définir une exception personnalisée
class MonException : public QException
{
public:
    MonException(const QString &message) : m_message(message) {}

    void raise() const override { throw *this; }
    MonException *clone() const override { return new MonException(*this); }
    QString message() const { return m_message; }

private:
    QString m_message;
};

// Utiliser l'exception
void fonctionRisquee()
{
    if (/* condition d'erreur */) {
        throw MonException("Quelque chose ne va pas");
    }
}

// Attraper l'exception
void fonctionAppelante()
{
    try {
        fonctionRisquee();
    } catch (const MonException &e) {
        qWarning() << "Exception attrapée:" << e.message();
    }
}
```

#### À savoir sur les exceptions en Qt

- Qt est compilé avec l'option `-no-exceptions` par défaut sur certaines plateformes
- Certaines classes Qt fournissent des exceptions (comme `QJsonParseError`)
- Les exceptions ne doivent pas traverser les limites des événements ou les slots Qt

### 4. Classe d'erreur dédiée

Pour des erreurs plus complexes, vous pouvez créer une classe dédiée :

```cpp
class Resultat
{
public:
    enum CodeErreur {
        Succes,
        ErreurFichier,
        ErreurReseau,
        ErreurPermission,
        ErreurFormat
    };

    Resultat() : m_code(Succes) {}
    Resultat(CodeErreur code, const QString &message = QString())
        : m_code(code), m_message(message) {}

    bool estSucces() const { return m_code == Succes; }
    CodeErreur code() const { return m_code; }
    QString message() const { return m_message; }

private:
    CodeErreur m_code;
    QString m_message;
};

Resultat chargerConfiguration(const QString &fichier)
{
    QFile f(fichier);
    if (!f.open(QIODevice::ReadOnly)) {
        return Resultat(Resultat::ErreurFichier,
                      "Impossible d'ouvrir " + fichier + ": " + f.errorString());
    }

    // Reste de la fonction...
    return Resultat(); // Succès
}

// Utilisation
void initialiser()
{
    Resultat res = chargerConfiguration("config.json");
    if (!res.estSucces()) {
        qWarning() << "Erreur de chargement:" << res.message();
        // Gérer l'erreur...
    }
}
```

### 5. Les assertions Qt

Les assertions sont utiles pour vérifier des conditions qui devraient toujours être vraies :

```cpp
#include <QDebug>

void diviser(int a, int b)
{
    Q_ASSERT(b != 0); // Vérifie que b n'est pas zéro

    int resultat = a / b;
    qDebug() << "Résultat:" << resultat;
}
```

Qt fournit plusieurs macros d'assertion :
- `Q_ASSERT(condition)` : Vérifie une condition, active uniquement en mode debug
- `Q_ASSERT_X(condition, where, message)` : Comme Q_ASSERT mais avec un message explicite
- `Q_ASSUME(condition)` : Indique au compilateur qu'une condition est toujours vraie, même en mode release

## Bonnes pratiques de gestion des erreurs

### 1. Choisir la bonne approche pour le contexte

- **Assertions** : Pour les erreurs de programmation (conditions qui ne devraient jamais se produire)
- **Valeurs de retour** : Pour les erreurs attendues et récupérables
- **Exceptions** : Pour les erreurs inattendues et difficiles à gérer localement
- **Journalisation** : Pour informer les développeurs et les utilisateurs

### 2. Messages d'erreur clairs

Un bon message d'erreur devrait:
- Décrire ce qui s'est passé
- Expliquer pourquoi c'est arrivé
- Suggérer comment résoudre le problème

```cpp
// Message peu utile
qWarning() << "Erreur de lecture";

// Message utile
qWarning() << "Impossible de lire le fichier de configuration"
           << chemin
           << ": "
           << fichier.errorString()
           << ". Vérifiez que le fichier existe et que vous avez les permissions nécessaires.";
```

### 3. Gestion des erreurs utilisateur

Pour les erreurs que l'utilisateur doit voir, utilisez des boîtes de dialogue :

```cpp
bool sauvegarderDocument(const QString &chemin)
{
    QFile fichier(chemin);
    if (!fichier.open(QIODevice::WriteOnly)) {
        QMessageBox::critical(this, "Erreur de sauvegarde",
                              "Impossible de sauvegarder le fichier " + chemin + ".\n\n" +
                              "Erreur: " + fichier.errorString());
        return false;
    }

    // Reste de la fonction...
    return true;
}
```

Pour des interactions plus sophistiquées :

```cpp
bool tenterOperation()
{
    bool succes = false;
    int tentatives = 0;

    while (!succes && tentatives < 3) {
        // Tenter l'opération...

        if (!succes) {
            tentatives++;
            QMessageBox msgBox;
            msgBox.setIcon(QMessageBox::Warning);
            msgBox.setText("L'opération a échoué.");
            msgBox.setInformativeText("Voulez-vous réessayer?");
            msgBox.setStandardButtons(QMessageBox::Retry | QMessageBox::Cancel);
            msgBox.setDefaultButton(QMessageBox::Retry);

            if (msgBox.exec() != QMessageBox::Retry) {
                break;
            }
        }
    }

    return succes;
}
```

### 4. Journalisation structurée avec les catégories de journalisation

Qt 5.2+ fournit un système de catégories de journalisation qui vous permet de filtrer les messages:

```cpp
// Définir des catégories
Q_LOGGING_CATEGORY(reseau, "app.reseau")
Q_LOGGING_CATEGORY(donnees, "app.donnees")
Q_LOGGING_CATEGORY(ui, "app.ui")

// Utiliser les catégories
void connexionServeur()
{
    qCDebug(reseau) << "Tentative de connexion...";

    if (/* échec de connexion */) {
        qCWarning(reseau) << "Échec de la connexion:" << erreur;
    }
}

// Configurer le filtrage
int main(int argc, char *argv[])
{
    // Activer seulement les avertissements et plus graves pour le réseau
    QLoggingCategory::setFilterRules("app.reseau.debug=false");
    // Désactiver complètement les messages UI
    QLoggingCategory::setFilterRules("app.ui.*=false");

    QApplication app(argc, argv);
    // ...
    return app.exec();
}
```

## Gestion avancée des erreurs

### 1. Erreurs asynchrones

Pour les opérations asynchrones, utilisez les signaux et slots pour reporter les erreurs :

```cpp
class Downloader : public QObject
{
    Q_OBJECT
public:
    void telechargerFichier(const QUrl &url);

signals:
    void telechargeTermine(const QString &cheminLocal);
    void erreurTelechargement(const QString &message);

private slots:
    void gererErreur(QNetworkReply::NetworkError erreur);

private:
    QNetworkAccessManager m_manager;
};

void Downloader::telechargerFichier(const QUrl &url)
{
    QNetworkRequest requete(url);
    QNetworkReply *reponse = m_manager.get(requete);

    connect(reponse, &QNetworkReply::finished, [this, reponse]() {
        if (reponse->error() == QNetworkReply::NoError) {
            // Traitement du succès...
            emit telechargeTermine(cheminLocal);
        } else {
            emit erreurTelechargement(reponse->errorString());
        }
        reponse->deleteLater();
    });

    connect(reponse, QOverload<QNetworkReply::NetworkError>::of(&QNetworkReply::error),
            this, &Downloader::gererErreur);
}
```

### 2. Traçage des erreurs

Pour les applications complexes, il peut être utile de tracer l'origine des erreurs :

```cpp
class TraceErreur
{
public:
    TraceErreur(const QString &fonction, const QString &fichier, int ligne)
        : m_fonction(fonction), m_fichier(fichier), m_ligne(ligne) {}

    QString toString() const
    {
        return QString("%1 (%2:%3)").arg(m_fonction, m_fichier).arg(m_ligne);
    }

private:
    QString m_fonction;
    QString m_fichier;
    int m_ligne;
};

class ResultatDetaille
{
public:
    // Constructeurs et autres méthodes...

    void ajouterTrace(const TraceErreur &trace)
    {
        m_traces.append(trace);
    }

    QString tracesFormatees() const
    {
        QString resultat = "Traces d'erreur:\n";
        for (const TraceErreur &trace : m_traces) {
            resultat += "  " + trace.toString() + "\n";
        }
        return resultat;
    }

private:
    QList<TraceErreur> m_traces;
    // Autres membres...
};

#define AJOUTER_TRACE(resultat) \
    resultat.ajouterTrace(TraceErreur(__FUNCTION__, __FILE__, __LINE__))

ResultatDetaille fonctionProfonde()
{
    ResultatDetaille res;
    // Si erreur...
    res.definirErreur("Quelque chose ne va pas");
    AJOUTER_TRACE(res);
    return res;
}

ResultatDetaille fonctionIntermediaire()
{
    ResultatDetaille res = fonctionProfonde();
    if (!res.estSucces()) {
        AJOUTER_TRACE(res);
    }
    return res;
}

void fonctionSuperieure()
{
    ResultatDetaille res = fonctionIntermediaire();
    if (!res.estSucces()) {
        AJOUTER_TRACE(res);
        qWarning() << "Erreur:" << res.message() << "\n" << res.tracesFormatees();
    }
}
```

## Résumé

La gestion des erreurs est un aspect fondamental du développement logiciel robuste. Qt fournit plusieurs outils pour vous aider :

- **Valeurs de retour et vérifications** : Simples et efficaces pour les erreurs locales
- **Système de journalisation** : Flexible et personnalisable pour informer les développeurs
- **Exceptions** : Puissantes pour les erreurs graves et inattendues
- **Classes d'erreur dédiées** : Complètes pour des informations d'erreur détaillées
- **Assertions** : Utiles pour détecter les erreurs de programmation

Choisissez l'approche qui convient à votre situation et n'oubliez pas : une bonne gestion des erreurs ne consiste pas seulement à éviter les crashs, mais aussi à créer une expérience utilisateur agréable même lorsque les choses ne se passent pas comme prévu.

⏭️ [Développement multiplateforme avec Qt6](/09-developpement-multiplateforme-avec-qt6)
